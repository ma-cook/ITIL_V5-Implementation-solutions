<div align="center">

# Jira Service Management & Microsoft Intune Integration

### Connecting Custom Monitoring Tools to a Modern ITSM Platform

**SUPPLEMENTARY CHAPTER**
*Companion to: ITIL Version 5 and the Active Service Desk Research Paper*

**May 2026** · IT Service Management Practice Series

</div>

---

## Introduction and Purpose

The preceding chapters established the theoretical framework of an **Integrated Active Service Desk (IASD)** and provided custom software implementations for the three highest-volume recurring issue categories: printer faults, POS hardware, and Microsoft 365 issues. Those tools (PLA, PHMS, and MIT) all depend on a central ITSM platform to receive and act on the events they generate.

This chapter focuses on **Jira Service Management (JSM)** as the ITSM platform, providing integration patterns, API examples, automation rules, and Microsoft Intune co-management patterns that are not covered by the open-source platform guidance in Part 4 of the custom software chapter. It also addresses the **desktop office environment** monitoring gap - the broad category of endpoint health issues that affect standard office workers on managed Windows or macOS devices outside of POS contexts.

> 💡 Jira Service Management is chosen here because it is one of the most widely adopted ITSM platforms in mid-market and enterprise organisations, particularly those already using the Atlassian ecosystem (Jira Software, Confluence). Its REST API, built-in automation engine, and native integration with tools like Opsgenie, Slack, Microsoft Teams, and Azure DevOps make it a practical target for the custom integration patterns described in this series.

---

## Part 1 · Jira Service Management as an ITSM Platform

### 1.1 JSM Capabilities Relevant to ITIL v5 MSF

Jira Service Management maps onto ITIL v5's **Monitor, Support and Fulfil (MSF)** practice cluster as follows:

| ITIL v5 MSF Practice | JSM Feature Set |
| --- | --- |
| **Service Desk** | Request portal, SLA timers, agent queues, customer notifications |
| **Incident Management** | Issue type: *Incident*; priority matrix; escalation rules; on-call alerting (Opsgenie) |
| **Problem Management** | Issue type: *Problem*; linked incidents; root-cause fields; known error status |
| **Monitoring & Event Management** | JSM Alerts (via Opsgenie integration); webhook-triggered automation |
| **Service Request Management** | Request catalogue; approval workflows; SLA per request type |
| **Change Management** | Change requests with CAB approval; risk assessment; change calendar |

> ⚠️ JSM's **Change Management** module is part of the *ITSM* project template and requires configuration to enforce ITIL v5-aligned workflows. Out of the box it provides a basic workflow; the automation patterns in this chapter assume a configured project with custom statuses and approval transitions.

### 1.2 Project Structure for IASD Implementation

For a full IASD implementation, the following JSM project structure is recommended:

| Project Key | Project Name | Issue Types Used |
| --- | --- | --- |
| `ITSM` | IT Service Management | Incident, Problem, Change Request, Service Request |
| `HW` | Hardware & Peripherals | Incident (hardware), Problem (hardware pattern), Service Request (procurement) |
| `M365` | Microsoft 365 Support | Incident (M365), Problem (M365 pattern), Service Request (licence/access) |
| `POS` | POS Operations | Incident (POS terminal), Problem (POS pattern), Change Request (firmware) |

Keeping hardware and M365 issues in separate projects allows different SLA schemes, queue configurations, and automation rules without conflicting logic, while still linking issues via JSM's native issue-linking.

---

## Part 2 · JSM REST API Integration Patterns

### 2.1 Authentication and Base Client

All three custom tools (PLA, PHMS, MIT) should use a shared API client that handles JSM authentication. JSM uses **API tokens** (not session cookies) for server-to-server integration:

```python
# jsm_client.py — reusable Jira Service Management REST API client
import requests
from requests.auth import HTTPBasicAuth
import os

class JSMClient:
    """
    Thin wrapper around the Jira Service Management REST API v3.
    Credentials are read from environment variables — never hardcode.
    
    Required environment variables:
        JSM_BASE_URL   — e.g. https://yourorg.atlassian.net
        JSM_USER_EMAIL — service account email
        JSM_API_TOKEN  — API token from id.atlassian.com/manage-profile/security
    """

    def __init__(self):
        self.base = os.environ['JSM_BASE_URL'].rstrip('/')
        self.auth = HTTPBasicAuth(
            os.environ['JSM_USER_EMAIL'],
            os.environ['JSM_API_TOKEN']
        )
        self.headers = {'Accept': 'application/json',
                        'Content-Type': 'application/json'}

    def _post(self, path: str, payload: dict) -> dict:
        r = requests.post(
            f'{self.base}/rest/api/3{path}',
            json=payload,
            auth=self.auth,
            headers=self.headers,
            timeout=15
        )
        r.raise_for_status()
        return r.json()

    def _get(self, path: str, params: dict = None) -> dict:
        r = requests.get(
            f'{self.base}/rest/api/3{path}',
            params=params,
            auth=self.auth,
            headers=self.headers,
            timeout=15
        )
        r.raise_for_status()
        return r.json()

    def create_issue(self, project_key: str, issue_type: str,
                     summary: str, description_adf: dict,
                     priority: str = 'Medium',
                     labels: list = None,
                     custom_fields: dict = None) -> dict:
        """Create a Jira issue. description_adf must be Atlassian Document Format."""
        fields = {
            'project':   {'key': project_key},
            'issuetype': {'name': issue_type},
            'summary':   summary,
            'description': description_adf,
            'priority':  {'name': priority},
            'labels':    labels or []
        }
        if custom_fields:
            fields.update(custom_fields)
        return self._post('/issue', {'fields': fields})

    def add_comment(self, issue_key: str, body_adf: dict) -> dict:
        return self._post(f'/issue/{issue_key}/comment', {'body': body_adf})

    def link_issues(self, inward_key: str, outward_key: str,
                    link_type: str = 'is caused by') -> dict:
        payload = {
            'type': {'name': link_type},
            'inwardIssue':  {'key': inward_key},
            'outwardIssue': {'key': outward_key}
        }
        return self._post('/issueLink', payload)

    def search_issues(self, jql: str, fields: list = None) -> list:
        params = {'jql': jql, 'maxResults': 50,
                  'fields': ','.join(fields or ['summary', 'status', 'priority'])}
        return self._get('/search', params).get('issues', [])
```

### 2.2 Atlassian Document Format (ADF) Helper

JSM's REST API v3 requires issue descriptions and comments in **Atlassian Document Format (ADF)** - a JSON-based rich text format. The following helper converts plain text and structured data into ADF:

```python
# adf_builder.py — build Atlassian Document Format bodies

def adf_paragraph(text: str) -> dict:
    return {
        'type': 'paragraph',
        'content': [{'type': 'text', 'text': text}]
    }

def adf_heading(text: str, level: int = 2) -> dict:
    return {
        'type': 'heading',
        'attrs': {'level': level},
        'content': [{'type': 'text', 'text': text}]
    }

def adf_table(headers: list, rows: list) -> dict:
    """
    headers: list of column header strings
    rows: list of lists (cell values as strings)
    """
    def cell(text, is_header=False):
        return {
            'type': 'tableHeader' if is_header else 'tableCell',
            'content': [adf_paragraph(str(text))]
        }

    header_row = {
        'type': 'tableRow',
        'content': [cell(h, True) for h in headers]
    }
    body_rows = [
        {'type': 'tableRow', 'content': [cell(v) for v in row]}
        for row in rows
    ]

    return {
        'type': 'table',
        'attrs': {'isNumberColumnEnabled': False, 'layout': 'default'},
        'content': [header_row] + body_rows
    }

def adf_code_block(code: str, language: str = 'text') -> dict:
    return {
        'type': 'codeBlock',
        'attrs': {'language': language},
        'content': [{'type': 'text', 'text': code}]
    }

def adf_doc(*content_nodes: dict) -> dict:
    """Wrap content nodes into a top-level ADF document."""
    return {'version': 1, 'type': 'doc', 'content': list(content_nodes)}
```

### 2.3 Creating Incidents from the Printer Log Aggregator

The PLA's recurrence engine (from the Custom Software chapter) should create JSM incidents enriched with event history using ADF formatting:

```python
# pla_jsm_reporter.py — PLA integration with JSM
from jsm_client import JSMClient
from adf_builder import adf_doc, adf_heading, adf_paragraph, adf_table, adf_code_block

jsm = JSMClient()

def raise_printer_incident(printer_name: str, error_class: str,
                            count: int, recent_events: list):
    """
    Raise a JSM incident for a recurring printer fault.
    recent_events: list of dicts with keys: time, event_id, user, document
    """
    summary = f'Recurring {error_class} on {printer_name} ({count}× in 7 days)'

    event_rows = [
        [e['time'], e['event_id'], e['user'], e.get('document', '-')]
        for e in recent_events[-10:]  # last 10 events
    ]

    description = adf_doc(
        adf_heading('Automated Detection — Printer Log Aggregator', 2),
        adf_paragraph(
            f'The Printer Log Aggregator detected {count} occurrences of '
            f'{error_class} on {printer_name} within the last 7 days. '
            f'This has exceeded the recurrence threshold and a ticket has been '
            f'raised automatically.'
        ),
        adf_heading('Recent Event History', 3),
        adf_table(
            ['Timestamp', 'Event ID', 'User', 'Document'],
            event_rows
        ),
        adf_heading('Recommended Actions', 3),
        adf_paragraph(
            '1. Verify printer connectivity and driver version. '
            '2. Check print spooler service status on print server. '
            '3. Review toner and paper levels via SNMP dashboard. '
            '4. If fault persists after remediation, escalate to vendor support.'
        )
    )

    issue = jsm.create_issue(
        project_key='HW',
        issue_type='Incident',
        summary=summary,
        description_adf=description,
        priority='Medium',
        labels=['printer', 'auto-detected', 'recurrence'],
        custom_fields={
            'customfield_10200': printer_name,   # CI Name (custom field)
            'customfield_10201': 'Printer'        # Category (custom field)
        }
    )
    return issue['key']
```

### 2.4 Auto-Raising Problem Records

When the recurrence threshold for a problem-level pattern is met, a Problem record should be raised in JSM and linked to the triggering incident(s):

```python
# jsm_problem_raiser.py — auto-raise Problem records from pattern detection

def raise_problem_record(jsm: JSMClient, project_key: str,
                          summary: str, description_adf: dict,
                          related_incident_keys: list,
                          category_label: str) -> str:
    """
    Creates a Problem record and links all related incidents to it.
    Returns the Problem issue key.
    """
    problem = jsm.create_issue(
        project_key=project_key,
        issue_type='Problem',
        summary=summary,
        description_adf=description_adf,
        priority='High',
        labels=[category_label, 'auto-raised', 'problem-management']
    )
    problem_key = problem['key']

    for incident_key in related_incident_keys:
        jsm.link_issues(
            inward_key=incident_key,
            outward_key=problem_key,
            link_type='is caused by'
        )

    return problem_key
```

---

## Part 3 · JSM Automation Rules for IASD Workflows

### 3.1 Overview of JSM Automation

JSM's built-in **Automation** engine allows no-code/low-code rules to trigger on events, conditions, and actions within the platform. For the IASD model, automation rules handle the workflow transitions that would otherwise require manual technician action.

Automation rules are configured in *Project Settings → Automation* (project-scoped) or *Settings → Automation* (global, enterprise plan). Each rule has:

- **Trigger** - what starts the rule (issue created, comment added, scheduled, webhook received)
- **Conditions** - filters that must pass for the rule to run
- **Actions** - what the rule does (transition issue, add comment, send notification, call webhook)

### 3.2 Key Automation Rules for the IASD Model

#### Rule 1: Auto-Assign Printer Incidents to Hardware Queue

**Trigger:** Issue Created  
**Conditions:** `Project = HW` AND `Labels contains printer`  
**Actions:**
1. Assign to component owner: *Print Services*
2. Add comment: *"This incident was automatically raised by the Printer Log Aggregator. Event history is attached in the description."*
3. Send email notification to hardware queue

#### Rule 2: Auto-Transition When Microsoft Incident Active

**Trigger:** Issue Created  
**Conditions:** `Project = M365` AND `Labels contains ms-service-incident`  
**Actions:**
1. Transition to status: *Waiting for Vendor*
2. Add comment (from custom field `cf_ms_incident_id`): *"This issue correlates with active Microsoft service incident [incident ID]. Resolution is pending on Microsoft's side. The ticket will be updated automatically when the incident is resolved."*
3. Set SLA pause: *Waiting for Vendor*

#### Rule 3: Escalate Auto-Raised Incidents Not Acknowledged Within SLA

**Trigger:** Scheduled (every 15 minutes)  
**Conditions:** `Project in (HW, M365, POS)` AND `Labels contains auto-detected` AND `Status = Open` AND `Created > -30m`  
**Actions:**
1. Add comment: *"Automated reminder: this incident was raised automatically and has not been acknowledged. Escalating to queue manager."*
2. Set priority to: *High*
3. Notify: *Queue Manager*

#### Rule 4: Link Duplicate Printer Incidents to Open Problem

**Trigger:** Issue Created  
**Conditions:** `Project = HW` AND `Labels contains printer`  
**Actions:**
1. Search for open Problems: JQL - `project = HW AND issuetype = Problem AND status != Done AND labels = printer`
2. If found: link new incident to open problem (*"is caused by"*)
3. Add comment: *"This incident has been automatically linked to open Problem [KEY]. Technician should review the problem record for context before investigating."*

#### Rule 5: Auto-Close M365 Incidents When MS Incident Resolves

**Trigger:** Webhook received (from MIT's service health monitor)  
**Conditions:** Webhook payload contains `event_type = ms_incident_resolved` AND `service_name` matches  
**Actions:**
1. JQL search: `project = M365 AND labels = ms-service-incident AND cf_ms_incident_id = {{webhookData.incident_id}}`
2. For each found issue:
   - Add comment: *"Microsoft has resolved service incident [ID]. Closing this ticket. If your issue persists, please reopen."*
   - Transition to: *Resolved*
   - Set resolution: *External Vendor Resolution*

### 3.3 Webhook Integration for External Tool Triggers

The three custom tools (PLA, PHMS, MIT) can trigger JSM automation rules directly via **JSM's incoming webhooks** without going through the REST API for each event. This is useful for high-volume events where creating a full issue would generate noise.

Configure an incoming webhook in *JSM Automation → New Rule → Trigger: Incoming Webhook*. The webhook URL is provided by JSM and does not require authentication (but should be kept secret and rotated periodically).

```python
# jsm_webhook_notifier.py — send events to JSM automation webhook
import requests, os, json

JSM_WEBHOOK_URL = os.environ['JSM_WEBHOOK_URL']  # store in env, not code

def notify_jsm_webhook(event_type: str, payload: dict) -> bool:
    """
    Send an event to a JSM Automation webhook trigger.
    JSM automation rules filter on event_type to route appropriately.
    """
    body = {'event_type': event_type, **payload}
    r = requests.post(JSM_WEBHOOK_URL, json=body, timeout=10)
    return r.status_code == 200
```

---

## Part 4 · Microsoft Intune Integration

### 4.1 Intune's Role in the IASD Model

Microsoft Intune is the device management backbone for the IASD model in organisations using Microsoft 365. In the context of ITIL v5's **Monitor, Support and Fulfil** cluster, Intune functions as:

- **A monitoring data source** - device compliance status, hardware inventory, app deployment status, Windows Update compliance
- **A remediation platform** - Intune can push PowerShell remediation scripts, app deployments, and configuration policies without requiring a technician to touch the device
- **A change execution platform** - approved standard changes (ITIL v5 Change Enablement) can be executed as Intune configuration profiles or scripts

### 4.2 Intune Remediations (Proactive Remediation Scripts)

**Intune Remediations** (formerly *Proactive Remediations*) are pairs of PowerShell scripts - a *Detection* script and a *Remediation* script - that Intune runs on managed devices on a schedule. This is the primary mechanism for implementing **Stage 3 automated remediation** from the Automation Maturity Ladder.

The detection script exits with code `0` (healthy) or `1` (needs remediation). If it exits `1`, Intune automatically runs the remediation script and reports the outcome.

#### 4.2.1 Print Spooler Health Remediation

```powershell
# DETECTION: Check-PrintSpoolerHealth.ps1
# Exit 0 = healthy, Exit 1 = needs remediation

$spooler = Get-Service -Name Spooler -ErrorAction SilentlyContinue

if (-not $spooler) {
    Write-Host 'Spooler service not found'
    exit 1
}

if ($spooler.Status -ne 'Running') {
    Write-Host "Spooler is $($spooler.Status)"
    exit 1
}

# Check for stuck jobs in default print queue
$jobs = Get-PrintJob -PrinterName * -ErrorAction SilentlyContinue |
        Where-Object { $_.JobStatus -match 'Error|Deleting' }
if ($jobs.Count -gt 0) {
    Write-Host "Stuck print jobs: $($jobs.Count)"
    exit 1
}

Write-Host 'Print spooler healthy'
exit 0
```

```powershell
# REMEDIATION: Fix-PrintSpoolerHealth.ps1

try {
    # Stop spooler and clear stuck jobs
    Stop-Service -Name Spooler -Force -ErrorAction Stop
    
    # Clear the spool directory
    $spoolDir = "$env:SystemRoot\System32\spool\PRINTERS"
    Get-ChildItem -Path $spoolDir -ErrorAction SilentlyContinue | Remove-Item -Force -Recurse

    # Restart spooler
    Start-Service -Name Spooler -ErrorAction Stop
    
    Write-Host 'Print spooler remediated successfully'
    exit 0
} catch {
    Write-Host "Remediation failed: $_"
    exit 1
}
```

#### 4.2.2 OneDrive Sync Health Remediation

```powershell
# DETECTION: Check-OneDriveSyncHealth.ps1

$odProcess = Get-Process -Name OneDrive -ErrorAction SilentlyContinue
if (-not $odProcess) {
    Write-Host 'OneDrive not running'
    exit 1
}

# Check registry for sync error state
$odKey = 'HKCU:\Software\Microsoft\OneDrive\Accounts\Business1'
if (Test-Path $odKey) {
    $syncState = (Get-ItemProperty $odKey -ErrorAction SilentlyContinue).SyncState
    if ($syncState -ne 0) {
        Write-Host "OneDrive sync error state: $syncState"
        exit 1
    }
}

Write-Host 'OneDrive sync healthy'
exit 0
```

```powershell
# REMEDIATION: Fix-OneDriveSyncHealth.ps1

try {
    # Reset OneDrive sync client
    $odExe = "$env:LocalAppData\Microsoft\OneDrive\OneDrive.exe"
    
    if (Test-Path $odExe) {
        # Kill existing OneDrive processes
        Get-Process -Name OneDrive -ErrorAction SilentlyContinue | Stop-Process -Force
        
        # Reset OneDrive (clears cached state, does NOT delete files)
        Start-Process $odExe -ArgumentList '/reset' -Wait -ErrorAction Stop
        
        # Wait for reset to complete, then restart
        Start-Sleep -Seconds 5
        Start-Process $odExe -ErrorAction Stop
        
        Write-Host 'OneDrive sync reset successfully'
        exit 0
    } else {
        Write-Host 'OneDrive executable not found'
        exit 1
    }
} catch {
    Write-Host "Remediation failed: $_"
    exit 1
}
```

#### 4.2.3 NIC Driver Event Log Health Check

```powershell
# DETECTION: Check-NICDriverHealth.ps1
# Detects NIC driver errors in the last 24 hours

$cutoff = (Get-Date).AddHours(-24)
$nicErrors = Get-WinEvent -FilterHashTable @{
    LogName   = 'System'
    StartTime = $cutoff
    Id        = 4001, 4004, 5010, 5011  # NIC link/driver events
    Level     = 2, 3                    # Error, Warning
} -ErrorAction SilentlyContinue

if ($nicErrors.Count -ge 3) {
    Write-Host "NIC driver errors in 24h: $($nicErrors.Count)"
    exit 1
}

Write-Host "NIC driver healthy (errors: $($nicErrors.Count))"
exit 0
```

```powershell
# REMEDIATION: Fix-NICDriverHealth.ps1

try {
    # Identify the primary network adapter
    $nic = Get-NetAdapter | Where-Object { $_.Status -eq 'Up' -and $_.Virtual -eq $false } |
           Sort-Object -Property Speed -Descending | Select-Object -First 1

    if (-not $nic) {
        Write-Host 'No active physical NIC found'
        exit 1
    }

    # Disable and re-enable the adapter (clears driver state without rebooting)
    Disable-NetAdapter -Name $nic.Name -Confirm:$false -ErrorAction Stop
    Start-Sleep -Seconds 3
    Enable-NetAdapter -Name $nic.Name -Confirm:$false -ErrorAction Stop

    # Flush DNS resolver cache as a secondary fix
    Clear-DnsClientCache

    Write-Host "NIC '$($nic.Name)' reset and DNS cache flushed"
    exit 0
} catch {
    Write-Host "Remediation failed: $_"
    exit 1
}
```

### 4.3 Intune Remediation Reporting via Graph API

Intune Remediation results are accessible via the Graph API, enabling the MIT to pull remediation outcomes and surface them in JSM:

```python
# intune_remediation_monitor.py
# Polls Intune remediation script results via Graph API and raises JSM tickets
# when remediations are failing at scale (indicating a systemic issue)

from mit_graph_client import graph_get
from jsm_client import JSMClient
from adf_builder import adf_doc, adf_heading, adf_paragraph, adf_table

jsm = JSMClient()

REMEDIATION_SCRIPT_IDS = {
    'print-spooler':   'your-intune-remediation-id-1',
    'onedrive-sync':   'your-intune-remediation-id-2',
    'nic-driver':      'your-intune-remediation-id-3',
}

FAILURE_THRESHOLD = 0.20  # 20% failure rate triggers a problem record

def check_remediation_health():
    for name, script_id in REMEDIATION_SCRIPT_IDS.items():
        results = graph_get(
            f'/deviceManagement/deviceHealthScripts/{script_id}/deviceRunStates'
            '?$select=id,detectionState,remediationState,lastStateUpdateDateTime,managedDevice'
        )

        states = results.get('value', [])
        if not states:
            continue

        failures = [s for s in states if s.get('remediationState') == 'remediationFailed']
        failure_rate = len(failures) / len(states)

        if failure_rate >= FAILURE_THRESHOLD:
            _raise_systemic_remediation_problem(name, failures, failure_rate)

def _raise_systemic_remediation_problem(script_name: str, failures: list,
                                         rate: float):
    rows = [[f['managedDevice']['deviceName'],
             f.get('lastStateUpdateDateTime', 'Unknown')]
            for f in failures[:15]]

    desc = adf_doc(
        adf_heading('Systemic Intune Remediation Failure Detected', 2),
        adf_paragraph(
            f'The Intune remediation script "{script_name}" is failing on '
            f'{rate:.0%} of devices. This indicates a systemic issue rather '
            f'than individual device faults.'
        ),
        adf_heading('Affected Devices (sample)', 3),
        adf_table(['Device Name', 'Last Attempt'], rows),
        adf_heading('Recommended Actions', 3),
        adf_paragraph(
            '1. Review remediation script logic for compatibility with recent '
            'OS or app updates. '
            '2. Check if a recent Windows Update or policy change has affected '
            'the target component. '
            '3. Test the remediation script manually on an affected device. '
            '4. If a known driver or OS version issue is confirmed, raise a '
            'Change Request for a targeted fix deployment.'
        )
    )

    jsm.create_issue(
        project_key='HW',
        issue_type='Problem',
        summary=f'Systemic Intune remediation failure: {script_name} ({rate:.0%} devices)',
        description_adf=desc,
        priority='High',
        labels=['intune', 'remediation-failure', 'auto-raised', script_name]
    )
```

### 4.4 Intune Compliance Policy Alignment with ITIL v5

Intune **Compliance Policies** define what *healthy* means for a managed device. Aligning these policies with ITIL v5's endpoint health model ensures consistency between Intune's reporting and the IASD health score:

| Intune Compliance Setting | IASD Health Score Signal | Recommended Policy |
| --- | --- | --- |
| OS version minimum | Patch compliance (10%) | Minimum: Windows 11 23H2 / macOS Sonoma |
| BitLocker encryption required | Storage security | Required for all device classifications |
| Antivirus required | Security posture | Required; Defender or approved third-party |
| Firewall enabled | Security posture | Required |
| No jailbreak / root (mobile) | Device integrity | Required |
| Secure Boot enabled | Boot integrity | Required for corporate-owned Windows |
| TPM required | Hardware security | Required for new device enrolments |
| Maximum inactive days before non-compliance | Orphaned device detection | 90 days |

Devices that fall out of compliance in Intune should trigger an Intune Compliance Change event in JSM. This can be automated via the Graph API polling pattern from the MIT:

```python
# intune_compliance_monitor.py
# Detects devices newly non-compliant and raises JSM incidents

from mit_graph_client import graph_get
from jsm_client import JSMClient
from adf_builder import adf_doc, adf_paragraph, adf_heading, adf_table

jsm = JSMClient()

def check_noncompliant_devices():
    result = graph_get(
        '/deviceManagement/managedDevices'
        '?$filter=complianceState eq \'noncompliant\''
        '&$select=id,deviceName,userDisplayName,complianceState,'
        'lastSyncDateTime,operatingSystem,osVersion'
    )

    devices = result.get('value', [])
    if not devices:
        return

    # De-duplicate: only raise incident if not already open in JSM
    for device in devices:
        existing = jsm.search_issues(
            jql=(f'project = HW AND labels = intune-noncompliant '
                 f'AND summary ~ "{device["deviceName"]}" AND status != Done'),
            fields=['summary', 'status']
        )
        if not existing:
            _raise_noncompliant_incident(device)

def _raise_noncompliant_incident(device: dict):
    desc = adf_doc(
        adf_heading('Intune Compliance Failure', 2),
        adf_paragraph(
            f'Device {device["deviceName"]} assigned to {device.get("userDisplayName", "Unknown")} '
            f'has been flagged as non-compliant by Intune. Conditional Access policies may '
            f'be blocking this device from accessing corporate resources.'
        ),
        adf_table(
            ['Field', 'Value'],
            [
                ['Device Name', device['deviceName']],
                ['User', device.get('userDisplayName', 'Unknown')],
                ['OS', f'{device["operatingSystem"]} {device["osVersion"]}'],
                ['Last Sync', device.get('lastSyncDateTime', 'Unknown')],
                ['Compliance State', device['complianceState']]
            ]
        )
    )

    jsm.create_issue(
        project_key='HW',
        issue_type='Incident',
        summary=f'Intune non-compliance: {device["deviceName"]} ({device.get("userDisplayName","Unknown")})',
        description_adf=desc,
        priority='Medium',
        labels=['intune', 'intune-noncompliant', 'auto-detected']
    )
```

---

## Part 5 · Desktop Office Environment Monitoring

### 5.1 The Desktop Office Monitoring Gap

The existing custom tools cover printers, POS hardware, and M365 cloud services. The broader category of **desktop office environment** issues - covering standard Windows and macOS workstations, peripheral connectivity, docking stations, and local network problems - requires a complementary monitoring approach. These issues are the highest-volume category by ticket count in most office-based service desks.

Common desktop office issues amenable to automated detection and remediation:

| Issue Category | Volume | Automated Detection | IASD Stage |
| --- | :---: | --- | :---: |
| Print spooler failure | High | Intune Remediation | 3 |
| USB peripheral not recognised | High | Windows Event Log (Event ID 2003) | 2–3 |
| Audio device misconfigured | Medium | Windows Audio Event Log | 2–3 |
| Monitor not detected / resolution wrong | Medium | Display driver events | 2–3 |
| Docking station not charging | Medium | Power event log (Event ID 84) | 2 |
| Slow disk / high SMART errors | Medium | SMART telemetry via Intune | 3–4 |
| Application crash loops | Medium | Windows Error Reporting (WER) | 2–3 |
| Windows Update failure | High | Intune compliance / Windows Update logs | 3–4 |
| Certificate expiry (machine certs) | Low | Certificate Store polling | 3 |
| Local profile corruption | Low | Windows Event ID 1509/1500 | 4 |

### 5.2 Endpoint Event Collection with WEC

**Windows Event Forwarding (WEF)** with a central **Windows Event Collector (WEC)** server allows collecting events from all managed endpoints without deploying a third-party agent. This is ITIL v5-aligned - it uses infrastructure that is already present in most Windows domain environments.

Configure subscription via Group Policy (`Computer Configuration → Administrative Templates → Windows Components → Event Forwarding`):

```powershell
# Configure WEC subscription via PowerShell on the WEC server
# Run once; endpoints subscribe automatically via GPO

$subscriptionXml = @'
<Subscription xmlns="http://schemas.microsoft.com/2006/03/windows/events/subscription">
  <SubscriptionId>IASD-Desktop-Monitoring</SubscriptionId>
  <SubscriptionType>SourceInitiated</SubscriptionType>
  <Description>IASD desktop health monitoring events</Description>
  <Enabled>true</Enabled>
  <Uri>http://schemas.microsoft.com/wbem/wsman/1/windows/EventLog</Uri>
  <ConfigurationMode>Custom</ConfigurationMode>
  <Delivery Mode="Push">
    <Batching><MaxItems>20</MaxItems><MaxLatencyTime>300000</MaxLatencyTime></Batching>
    <PushSettings><Heartbeat Interval="1800000"/></PushSettings>
  </Delivery>
  <Query>
    <![CDATA[
    <QueryList>
      <!-- USB device errors -->
      <Query Id="0" Path="System"><Select>*[System[(EventID=2003 or EventID=2100 or EventID=7026)]]</Select></Query>
      <!-- Application crashes (WER) -->
      <Query Id="1" Path="Application"><Select>*[System[(EventID=1000 or EventID=1001) and Provider[@Name="Application Error" or @Name="Windows Error Reporting"]]]</Select></Query>
      <!-- Print spooler events -->
      <Query Id="2" Path="System"><Select>*[System[(EventID=7034 or EventID=7036) and Provider[@Name="Service Control Manager"]]]</Select></Query>
      <!-- NIC link events -->
      <Query Id="3" Path="System"><Select>*[System[(EventID=4001 or EventID=4004)]]</Select></Query>
      <!-- User profile events -->
      <Query Id="4" Path="Application"><Select>*[System[(EventID=1509 or EventID=1500)]]</Select></Query>
      <!-- SMART disk health -->
      <Query Id="5" Path="Microsoft-Windows-Disk/Operational"><Select>*[System[Level=2 or Level=3]]</Select></Query>
    </QueryList>
    ]]>
  </Query>
  <AllowedSourceDomainComputers>O:NSG:NSD:(A;;GA;;;DC)</AllowedSourceDomainComputers>
</Subscription>
'@

wecutil cs $subscriptionXml
```

### 5.3 WEC Event Processor and JSM Integration

A Python service running on the WEC server processes forwarded events and creates JSM issues for patterns that exceed thresholds:

```python
# wec_processor.py
# Reads Windows Event Collector forwarded events and raises JSM issues

import subprocess, json, datetime
from collections import defaultdict
from jsm_client import JSMClient
from adf_builder import adf_doc, adf_heading, adf_paragraph, adf_table

jsm = JSMClient()

# Configurable thresholds
THRESHOLDS = {
    'usb_error':     {'count': 3,  'window_hours': 24},
    'app_crash':     {'count': 3,  'window_hours': 24},
    'spooler_crash': {'count': 1,  'window_hours': 1},   # immediate
    'nic_error':     {'count': 5,  'window_hours': 24},
    'disk_error':    {'count': 1,  'window_hours': 168}, # immediate for disk
}

EVENT_CLASS_MAP = {
    2003: 'usb_error', 2100: 'usb_error',
    1000: 'app_crash', 1001: 'app_crash',
    7034: 'spooler_crash',
    4001: 'nic_error', 4004: 'nic_error',
}

def get_forwarded_events(hours_back: int = 1) -> list:
    """Read events from the forwarded events log via Get-WinEvent."""
    cutoff = (datetime.datetime.utcnow() -
              datetime.timedelta(hours=hours_back)).strftime('%Y-%m-%dT%H:%M:%SZ')
    ps_cmd = f'''
    Get-WinEvent -LogName "ForwardedEvents" -ErrorAction SilentlyContinue |
    Where-Object {{ $_.TimeCreated -gt [datetime]::Parse("{cutoff}") }} |
    Select-Object TimeCreated, Id, MachineName, Message |
    ConvertTo-Json -Depth 3
    '''
    result = subprocess.run(['powershell', '-Command', ps_cmd],
                            capture_output=True, text=True, timeout=60)
    try:
        return json.loads(result.stdout) or []
    except json.JSONDecodeError:
        return []

def process_and_raise(events: list):
    # Group events by machine + event class
    groups = defaultdict(list)
    for ev in events:
        event_class = EVENT_CLASS_MAP.get(ev.get('Id'), 'unknown')
        if event_class != 'unknown':
            key = (ev['MachineName'], event_class)
            groups[key].append(ev)

    for (machine, event_class), evs in groups.items():
        threshold_cfg = THRESHOLDS.get(event_class)
        if threshold_cfg and len(evs) >= threshold_cfg['count']:
            _raise_desktop_incident(machine, event_class, evs)

def _raise_desktop_incident(machine: str, event_class: str, events: list):
    label_map = {
        'usb_error':     'USB peripheral fault',
        'app_crash':     'Application crash loop',
        'spooler_crash': 'Print spooler crash',
        'nic_error':     'NIC driver error',
        'disk_error':    'Disk health warning',
    }
    label = label_map.get(event_class, event_class)
    summary = f'{label} on {machine} ({len(events)}× detected)'

    rows = [[e['TimeCreated'], str(e['Id']), e['Message'][:80] + '...']
            for e in events[:8]]

    desc = adf_doc(
        adf_heading('Automated Detection — WEC Desktop Monitor', 2),
        adf_paragraph(
            f'The Windows Event Collector detected {len(events)} {label} events '
            f'on {machine}, exceeding the auto-raise threshold.'
        ),
        adf_heading('Event Sample', 3),
        adf_table(['Timestamp', 'Event ID', 'Message (truncated)'], rows)
    )

    jsm.create_issue(
        project_key='HW',
        issue_type='Incident',
        summary=summary,
        description_adf=desc,
        priority='Medium' if event_class != 'disk_error' else 'High',
        labels=['auto-detected', 'desktop', event_class.replace('_', '-')]
    )
```

---

## Part 6 · Unified JSM Dashboard and Reporting

### 6.1 JSM Dashboards for the IASD Model

JSM provides built-in dashboard gadgets for displaying queue status, SLA performance, and issue counts. For the IASD model, the following dashboard layout is recommended for the service desk team:

**Service Desk Operations Dashboard:**

| Gadget | Data Source | Purpose |
| --- | --- | --- |
| *Issues in Queue by Priority* | JQL: `project in (HW, M365, POS) AND status = Open` | Live queue visibility |
| *Auto-Raised Incidents (24h)* | JQL: `labels = auto-detected AND created > -24h` | Volume of automated detections |
| *SLA Breach Risk* | SLA breach gadget, filter by Priority | SLA monitoring |
| *Problems Under Investigation* | JQL: `issuetype = Problem AND status != Done` | Open problem records |
| *M365 Service Health* | Custom: MIT webhook feed | Current MS service incidents |
| *Intune Non-Compliance Count* | JQL: `labels = intune-noncompliant AND status != Done` | Compliance posture |

### 6.2 JSM Reports for Management

JSM's built-in reporting, supplemented by Confluence integration, provides the following management-level views:

- **Weekly Automated Resolution Rate**: percentage of closed tickets that were opened and resolved automatically without technician action (label: `auto-detected` → resolution: `Automated Fix`)
- **Recurrence Rate by Category**: Problem records raised in the period vs. unique CI count - measures effectiveness of problem management
- **XLA Trend**: average satisfaction score from post-resolution surveys (configured via JSM's Customer Satisfaction surveys) plotted against ticket volume
- **Top Offending CIs**: which printers, devices, or M365 users generated the most incidents in the reporting period - drives proactive hardware refresh decisions

---

## Part 7 · Security Considerations for JSM Integration

### 7.1 Credential and Secret Management

All custom tool integrations with JSM must follow secure credential management practices:

- **API tokens** must be stored in a secrets manager (Azure Key Vault, HashiCorp Vault, AWS Secrets Manager) and injected as environment variables at runtime - never hardcoded in scripts or committed to source control
- **JSM service accounts** should be dedicated accounts (not personal accounts) with the minimum required permissions: typically *Service Desk Agent* role scoped to the relevant projects
- **Webhook URLs** from JSM are effectively bearer tokens - rotate them quarterly and treat them as secrets
- **Intune remediation scripts** must be signed with a code signing certificate issued from your organisation's PKI. Unsigned scripts represent an execution risk and may be blocked by Intune policy

### 7.2 Audit and Compliance

All automated JSM actions should be traceable to a change record:

- Every automation rule that creates, transitions, or modifies issues should include a **comment identifying the automation rule name and trigger** - this provides an audit trail distinguishing automated from manual actions
- Intune Remediation outcomes are logged in the Intune portal (Device → Monitor → Remediations) and these logs should be retained for a minimum of 90 days
- JSM issue history is immutable - every comment, transition, and edit is logged with timestamp and actor, satisfying ITIL v5's audit trail requirement for standard changes

### 7.3 Scope Limitation for Automated Actions

Automated integrations must be scoped to prevent unintended privilege escalation:

- The JSM service account used by custom tools should **not** have permission to approve Change Requests or modify project configuration - these actions must remain human-controlled
- Intune Remediation scripts should run under the **SYSTEM** account with the minimum permissions required - avoid granting broader admin rights than necessary
- Graph API permissions for the MIT application should be reviewed against the **principle of least privilege** - use application permissions only for the specific endpoints required, not `Directory.ReadWrite.All` or other broad scopes

