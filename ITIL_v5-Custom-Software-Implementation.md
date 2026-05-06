<div align="center">

# Practical Custom Software for Recurring Issue Tracking

### Printer Faults · Point-of-Sale Hardware · Microsoft 365 User Issues

**SUPPLEMENTARY CHAPTER**
*Companion to: ITIL Version 5 and the Active Service Desk Research Paper*

**May 2026** · IT Service Management Practice Series

</div>

---

## Introduction and Purpose

The main research paper established the theoretical and operational framework for an **Integrated Active Service Desk (IASD)** under ITIL v5. This supplementary chapter moves from framework to implementation, focusing on the three highest-volume recurring issue categories that benefit most from custom software tooling:

1. 🖨️ **Printer and print-service faults** — the single largest category of *"device is broken"* contacts in most endpoint-heavy environments
2. 🛒 **Point-of-Sale (POS) hardware issues** — mission-critical in retail and hospitality contexts, where downtime directly translates to lost revenue
3. ☁️ **Microsoft 365 user issues** — the most diverse category, spanning authentication, licensing, Teams, Exchange, SharePoint, and OneDrive

For each category, this chapter covers: native log sources, a purpose-built custom software architecture using accessible technologies (PowerShell, Python, and open-source ITSM platforms), practical code patterns, recurrence detection logic, and integration touchpoints with ITIL v5's **Monitor, Support and Fulfil (MSF)** practices.

> 💡 All custom solutions described here are designed to **complement — not replace** — a primary ITSM platform such as GLPI, iTop, ManageEngine ServiceDesk Plus, or Freshservice. They act as specialised log ingestors, pattern detectors, and ticket generators that feed structured, enriched data into whichever ITSM platform the organisation already runs.

---

## Part 1 · Printer Issue Tracking and Recurrence Detection

### 1.1 The Printer Problem Landscape

Printer issues account for a disproportionate share of service desk contacts relative to the business impact they cause. Common recurring patterns include:

- **Print spooler crashes** causing queue-wide failure (Event ID `7034` in System log)
- **Driver corruption** following Windows Update rollouts (Event ID `808`/`843` in `PrintService/Admin`)
- **Network printers dropping offline** due to IP address changes or DHCP lease expiry
- **Paper jam and low-toner states** not surfaced until a user contacts the service desk
- **Shared print queue backlogs** causing apparent *"printer not working"* reports from multiple users

> ⚠️ Most of these are logged in Windows Event Logs that are either **disabled by default**, not centralised, or not correlated across endpoints. A custom **Printer Log Aggregator** fills this gap.

### 1.2 Native Windows Log Sources

Windows exposes two PrintService event logs, both under *Applications and Services Logs → Microsoft → Windows → PrintService*:

| Log | Default State | Key Event IDs |
| --- | :---: | --- |
| `PrintService/Admin` | ✅ Enabled | `808` (driver install), `843` (driver import fail), `316` (driver blocked) |
| `PrintService/Operational` | ❌ **Disabled** | `307` (job printed), `316` (job error), `820` (spooler crash), `353` (job status) |
| `System` | ✅ Enabled | `7034` (spooler service crash), `7036` (service state change) |

> 🔑 **Enabling `PrintService/Operational` is the most important first step.** This can be deployed via Group Policy (GPO) or PowerShell:

```powershell
# Enable PrintService Operational log and expand to 50MB (deploy via GPO or startup script)
$log = New-Object System.Diagnostics.Eventing.Reader.EventLogConfiguration `
        'Microsoft-Windows-PrintService/Operational'
$log.IsEnabled = $true
$log.MaximumSizeInBytes = 52428800   # 50 MB
$log.SaveChanges()
```

### 1.3 Custom Printer Log Aggregator: Architecture

The **Printer Log Aggregator (PLA)** is a lightweight Python service that runs on each print server (or is deployed as an agent on endpoints in a peer-to-peer print environment). It performs three functions: **collection**, **normalisation**, and **recurrence detection**.

| Component | Description |
| --- | --- |
| **Event Collector** | PowerShell script scheduled every 5 minutes; exports new PrintService events to JSON |
| **Python Ingestor** | Reads JSON, normalises fields (printer name, user, error code), writes to SQLite or PostgreSQL |
| **Recurrence Engine** | Queries DB for same printer + same error code ≥ N times in rolling 7-day window |
| **Ticket Generator** | Calls ITSM REST API to raise a Problem record when recurrence threshold is met |
| **Dashboard** | Flask web app: per-printer error rate, top-offending printers, trend charts |

#### 1.3.1 PowerShell Event Exporter

Deploy this script as a Windows Scheduled Task on print servers, running every 5 minutes under a service account with Event Log read permissions:

```powershell
# PrintEventExport.ps1 — runs on print server every 5 minutes
$lastRun = (Get-Date).AddMinutes(-6)
$events = Get-WinEvent -FilterHashTable @{
    LogName   = 'Microsoft-Windows-PrintService/Operational','Microsoft-Windows-PrintService/Admin'
    StartTime = $lastRun
    Id        = 307,316,808,820,843,353
} -ErrorAction SilentlyContinue

$output = $events | ForEach-Object {
    [PSCustomObject]@{
        TimeCreated  = $_.TimeCreated.ToString('o')   # ISO 8601
        EventId      = $_.Id
        Level        = $_.LevelDisplayName
        PrinterName  = $_.Properties[4].Value
        UserName     = $_.Properties[2].Value
        ComputerName = $_.Properties[3].Value
        Document     = $_.Properties[1].Value
        Pages        = $_.Properties[7].Value
        Message      = $_.Message
    }
}
$output | ConvertTo-Json | Out-File 'C:\PLA\events_export.json' -Encoding UTF8
```

#### 1.3.2 Recurrence Detection Logic (Python)

The recurrence engine queries the local SQLite database and raises a problem ticket when the same printer has the same error class **≥ 3 times in 7 days**:

```python
# recurrence_check.py
import sqlite3, datetime, requests

THRESHOLD   = 3          # occurrences to trigger problem record
WINDOW_DAYS = 7          # rolling window
ITSM_URL    = 'https://itsm.internal/api/v2/problems'

conn = sqlite3.connect('printer_events.db')
cutoff = (datetime.datetime.utcnow() -
          datetime.timedelta(days=WINDOW_DAYS)).isoformat()

rows = conn.execute('''
    SELECT printer_name, event_class, COUNT(*) AS cnt
    FROM events
    WHERE time_created > ? AND event_class != 'INFO'
    GROUP BY printer_name, event_class
    HAVING cnt >= ?
''', (cutoff, THRESHOLD)).fetchall()

for printer, error_class, count in rows:
    payload = {
        'title': f'Recurring {error_class} on {printer} ({count}x in 7d)',
        'category': 'Printer', 'impact': '3-Medium',
        'description': f'Auto-raised by Printer Log Aggregator. '
                       f'{count} {error_class} events in last {WINDOW_DAYS} days.'
    }
    requests.post(ITSM_URL, json=payload,
                  headers={'X-API-Key': 'YOUR_KEY'})
```

### 1.4 SNMP-Based Toner and Status Monitoring

For network printers that support SNMP (virtually all enterprise laser printers), toner levels, paper tray status, and device error states can be polled directly without relying on Windows logs. This is particularly valuable for catching **paper-out** and **low-toner** states *before* they generate user calls.

Standard SNMP OIDs for printer status (RFC 3805 / Printer MIB):

| Metric | OID | Alert Threshold |
| --- | --- | --- |
| Printer Status | `1.3.6.1.2.1.25.3.5.1.1` | Value ≠ 3 (idle) |
| Detected Error State | `1.3.6.1.2.1.25.3.5.1.2` | Any bit set |
| Toner Level (black) | `1.3.6.1.2.1.43.11.1.1.9.1.1` | < 15% of max |
| Paper Tray Remaining | `1.3.6.1.2.1.43.8.2.1.10.1.1` | < 50 sheets |
| Page Count | `1.3.6.1.2.1.43.10.2.1.4.1.1` | Track for PM scheduling |

> A Python poller using the `pysnmp` library can scan all printers every 15 minutes and write results to the same database used by the event aggregator, enabling unified dashboards and correlation between *"toner low"* events and subsequent *"printer not working"* tickets.

### 1.5 Integration with ITIL v5 MSF Practices

| MSF Practice | How the PLA Connects |
| --- | --- |
| **Monitoring & Event Mgmt** | SNMP poller and event exporter generate events; PLA classifies and suppresses noise |
| **Incident Management** | Error events ≥ P3 threshold auto-raise incident in ITSM with full event context |
| **Problem Management** | Recurrence engine auto-raises problem record when same fault recurs ≥ 3× in 7 days |
| **Service Request Mgmt** | Low-toner SNMP event auto-creates consumables request in ITSM |
| **Service Desk** | Technicians see enriched ticket with event history pre-populated; no manual log hunting |

---

## Part 2 · Point-of-Sale Hardware Issue Tracking

### 2.1 POS Hardware Context

Point-of-Sale environments introduce constraints not present in standard desktop support:

- **Downtime has an immediate, measurable revenue impact** (lost sales, queue abandonment)
- **Hardware is purpose-built and often proprietary** — receipt printers, barcode scanners, customer displays, card payment terminals (PIN pads), cash drawers, and POS terminals running specialised OS images
- **PCI DSS requirements apply** to any system touching card payment data, constraining what monitoring agents can be installed and what data can be logged
- **Retail and hospitality environments mean hardware is physically stressed** — heat, humidity, dust, spills, and heavy usage cycles

> 🎯 The goal of a **POS Hardware Monitoring System (PHMS)** is to detect hardware degradation and recurring fault patterns *before* they cause a register outage, and to provide technicians with structured fault history rather than anecdotal user reports.

### 2.2 POS Hardware Log Sources

Unlike standard desktops, POS devices expose fault data through a combination of sources:

| Source | Data Available |
| --- | --- |
| **Windows Event Log** | Application crashes, driver failures, USB device connect/disconnect (POS peripherals) |
| **POS Application Logs** | Transaction errors, peripheral communication failures, cash drawer open/close events |
| **Receipt Printer SNMP/USB** | Paper out, cover open, error states (OPOS driver exposes status) |
| **Barcode Scanner** | Scan failure rates via USB HID report or OPOS driver |
| **Payment Terminal Logs** | Declined transactions, comms failures *(note: access restricted by PCI DSS scope)* |
| **Network Switch SNMP** | POS register port up/down events, traffic anomalies |
| **MDM / RMM Platform** | Device health, patch state, disk usage, CPU/memory utilisation on POS terminals |

### 2.3 OPOS and JavaPOS Driver Status Monitoring

Most enterprise POS peripherals use the **OPOS** (OLE for Retail POS) or **JavaPOS** standard, which provides a common interface for peripheral status across vendors. OPOS exposes a `StatusUpdateEvent` that fires whenever a peripheral state changes (paper low, cover open, offline). A small monitoring agent can subscribe to these events and forward status changes to the PHMS.

```csharp
// OPOS Status Monitor (C# / .NET — runs as Windows service on POS terminal)
// Subscribes to receipt printer status and posts to PHMS API

OPOSPrinter printer = new OPOSPrinter();
printer.Open("ReceiptPrinter");
printer.Claim(1000);
printer.DeviceEnabled = true;

printer.StatusUpdateEvent += (s, e) => {
    var payload = new {
        terminal_id   = Environment.MachineName,
        device        = "receipt_printer",
        status_code   = e.Data,
        timestamp     = DateTime.UtcNow.ToString("o")
    };
    HttpClient.PostAsJsonAsync("https://phms.internal/api/events", payload);
};
```

**Common OPOS status codes to monitor:**

| Status Code | Meaning | Recommended Action |
| --- | --- | --- |
| `11` *(SUE_COVER_OPEN)* | Printer cover open | Alert terminal operator; if sustained > 2 min, create incident |
| `21` *(SUE_IDLE)* | Printer idle and ready | No action — reset fault flag if previously set |
| `41` *(SUE_REC_EMPTY)* | Receipt paper empty | Auto-create consumables request; alert store supervisor |
| `51` *(SUE_REC_NEAREMPTY)* | Paper near-empty | Proactive alert to stock paper; schedule before queue forms |
| `107` *(SUE_POWER_OFF)* | Printer offline / power off | **P2 incident** if not resolved within 5 minutes |

### 2.4 POS Hardware Monitoring System (PHMS): Architecture

The **PHMS** is a centralised service that aggregates events from all POS terminals across one or more sites, stores them in a structured database, detects recurring fault patterns, and exposes a real-time operations dashboard.

| Component | Technology & Purpose |
| --- | --- |
| **Terminal Agent** | .NET service on each POS terminal: OPOS events + Windows Event Log + RMM integration |
| **API Gateway** | FastAPI (Python) or Express (Node.js): receives events, validates, stores to database |
| **Database** | PostgreSQL: `events`, `terminals`, `fault_patterns`, `incidents` tables |
| **Pattern Engine** | Python scheduled job: detects recurring faults, calculates terminal health score (0–100) |
| **ITSM Connector** | REST client: auto-raises incidents/problems in ITSM when thresholds exceeded |
| **Operations Dashboard** | React + Chart.js: floor plan view of terminals with colour-coded health, live fault feed |
| **Alert Routing** | Teams / Slack webhook + email: site supervisor and service desk notified instantly |

#### 2.4.1 Terminal Health Score for POS

Each POS terminal is assigned a health score updated every 15 minutes. The score drives proactive intervention thresholds:

| Signal | Weight | Alert Trigger |
| --- | :---: | --- |
| Receipt printer error rate (last 24h) | **25%** | > 3 errors → −15 points |
| Cash drawer open anomalies | **15%** | Unscheduled opens > 5 → −10 points |
| Barcode scanner miss rate | **15%** | > 10% miss rate → −10 points |
| Transaction decline rate (non-card) | **15%** | > 5% → −10 points |
| Windows application crash count | **10%** | > 2 in 24h → −15 points |
| Network packet loss to POS server | **10%** | > 1% → −10 points |
| Disk space (POS terminal) | **5%** | < 10 GB free → −10 points |
| Days since last patch | **5%** | > 60 days → −5 points |

#### 2.4.2 Recurrence Detection for POS

POS fault recurrence is particularly important because a terminal that fails repeatedly is a **reliability risk that needs hardware replacement**, not repeated reactive fixes. The PHMS recurrence engine applies escalating logic:

| Fault Frequency | PHMS Action | ITIL v5 Practice Triggered |
| --- | --- | --- |
| 1st occurrence | Log event, update health score | Monitoring & Event Management |
| 2nd same fault (7 days) | Notify supervisor + service desk | Incident Management |
| 3rd same fault (30 days) | Auto-raise Problem record in ITSM | Problem Management |
| 5th same fault (90 days) | Flag terminal for hardware replacement review | PSLM: *Discover* stage |
| Pattern across multiple terminals | Raise Vendor/Model defect problem record | Supplier Management |

### 2.5 POS-Specific Considerations

#### 2.5.1 PCI DSS Compliance

Any monitoring solution touching POS systems must be assessed against PCI DSS requirements. Key constraints:

> 🔒 **Do not log or transmit raw cardholder data** (Primary Account Numbers) under any circumstances.

- Payment terminal logs (PIN pad, acquiring connection) are generally **outside scope** for IT service desk monitoring — access is typically restricted to the acquiring bank or payment gateway provider
- Ensure monitoring agents are included in your **annual penetration test** scope
- **Network segmentation** between POS monitoring traffic and general IT monitoring infrastructure should be maintained

#### 2.5.2 Integration with Store Operations

Unlike standard service desk environments, POS downtime affects **store operations teams**, not just IT. The PHMS should therefore:

- Post alerts to a **store supervisor Teams or Slack channel**, not only the IT service desk queue
- Provide a **simple mobile-accessible view** for store managers to see terminal status without needing ITSM access
- Include **estimated resolution times** based on historical fault data so managers can make real-time decisions about opening additional terminals or checkout lanes

---

## Part 3 · Microsoft 365 User Issue Tracking

### 3.1 The M365 Support Landscape

Microsoft 365 generates the most diverse category of service desk contacts. Issues span identity and authentication (Entra ID / MFA), email (Exchange Online), collaboration (Teams, SharePoint, OneDrive), licensing, and increasingly AI-powered services (Copilot). Recurring M365 issues that are amenable to custom tracking and automation include:

- **MFA prompt storms** — users repeatedly prompted for MFA due to Conditional Access misconfigurations or token cache issues
- **Shared mailbox permission drift** — users losing access to shared mailboxes after licence or group membership changes
- **Teams meeting quality degradation** — recurring poor-quality meeting experience for specific users or locations
- **OneDrive sync client errors** — OneDrive repeatedly entering a *"paused"* or *"needs attention"* state
- **Licence assignment failures** — users losing access to specific M365 apps due to group-based licence conflicts
- **Exchange Online message delivery failures** — recurring NDRs or delayed delivery for specific users or domains

> A custom **M365 Issue Tracker (MIT)** uses the **Microsoft Graph API** as its primary data source, supplemented by Windows event logs on endpoints and the M365 Service Health API.

### 3.2 Microsoft Graph API as the Data Backbone

The Microsoft Graph API provides programmatic access to virtually all M365 service data relevant to user issue tracking. Key endpoints for service desk automation:

| Graph API Endpoint | Service Desk Use Case |
| --- | --- |
| `/reports/getTeamsUserActivityUserDetail` | Detect users with zero Teams activity (possible sign-in failure) |
| `/reports/getEmailActivityUserDetail` | Identify users with mail delivery anomalies |
| `/users/{id}/authentication/signInActivity` | Last sign-in date — detect orphaned or locked accounts |
| `/users/{id}/licenseDetails` | Current licence assignments — detect missing licences after group changes |
| `/identity/conditionalAccessPolicies` | Audit CA policy changes that may explain MFA prompt storms |
| `/communications/callRecords` | Teams call quality: jitter, packet loss, latency per user per session |
| `/admin/serviceAnnouncement/healthOverviews` | M365 Service Health — check if issue is a known Microsoft incident |
| `/admin/serviceAnnouncement/issues` | Active service incidents — auto-correlate user complaints with MS incidents |

> ⚠️ **Important note on Graph API stability:** the Graph PowerShell SDK has experienced significant quality issues through 2025, with the SDK team working through a substantial bug backlog. For production automation, **pin SDK version explicitly** in your automation environment and test new releases in a staging tenant before promoting to production.

### 3.3 M365 Issue Tracker (MIT): Architecture

The MIT is a Python-based service that polls the Graph API on scheduled intervals, stores results in a time-series database, correlates patterns into recurring issue records, and integrates with the ITSM platform via REST API.

| Component | Function |
| --- | --- |
| **Graph Poller** | Scheduled Python jobs using the MSAL library; polls user activity, licence, sign-in, call quality |
| **Service Health Monitor** | Polls `/admin/serviceAnnouncement` every 10 minutes; creates awareness records in ITSM |
| **Pattern Detector** | Queries database for recurring issues per user, per service, per location |
| **Ticket Enricher** | When a user raises a ticket, MIT auto-attaches last 7 days of Graph data to the ticket |
| **Self-Heal Actions** | Azure Automation runbooks triggered by MIT for safe auto-remediation *(see §3.5)* |
| **M365 Health Dashboard** | Power BI or Grafana dashboard: per-service health, top-affected users, call quality heatmap |

#### 3.3.1 Authentication Pattern — MSAL for Graph Access

The MIT authenticates to the Graph API using a registered Entra ID application with **application permissions** (not delegated), allowing it to run without a signed-in user:

```python
# mit_graph_client.py — MSAL authentication for Graph API
import msal, requests

TENANT_ID = 'your-tenant-id'
CLIENT_ID = 'your-app-registration-client-id'
CLIENT_SECRET = 'stored-in-key-vault'  # never hardcode
SCOPE = ['https://graph.microsoft.com/.default']

app = msal.ConfidentialClientApplication(
    CLIENT_ID,
    authority=f'https://login.microsoftonline.com/{TENANT_ID}',
    client_credential=CLIENT_SECRET
)

def get_token():
    result = app.acquire_token_silent(SCOPE, account=None)
    if not result:
        result = app.acquire_token_for_client(scopes=SCOPE)
    return result['access_token']

def graph_get(endpoint):
    headers = {'Authorization': f'Bearer {get_token()}'}
    return requests.get(f'https://graph.microsoft.com/v1.0{endpoint}',
                        headers=headers).json()
```

#### 3.3.2 Recurring Issue Detection — MFA Prompt Storm Example

MFA prompt storms — where a user is asked to authenticate repeatedly — generate identifiable patterns in Entra ID sign-in logs (accessible via Graph). The following pattern identifies affected users:

```python
# mfa_storm_detector.py
import datetime
from mit_graph_client import graph_get

def detect_mfa_storms(threshold=5, window_hours=24):
    '''Flag users with >= threshold MFA interrupts in window_hours'''
    cutoff = (datetime.datetime.utcnow() -
              datetime.timedelta(hours=window_hours)).strftime('%Y-%m-%dT%H:%M:%SZ')

    # Graph: sign-ins with MFA required but session not persistent
    signins = graph_get(
        f'/auditLogs/signIns?$filter=createdDateTime ge {cutoff}'
        '&$select=userDisplayName,userPrincipalName,status,mfaDetail,createdDateTime'
    )

    from collections import Counter
    mfa_counts = Counter()
    for signin in signins.get('value', []):
        if signin.get('mfaDetail'):
            mfa_counts[signin['userPrincipalName']] += 1

    return {upn: cnt for upn, cnt in mfa_counts.items()
            if cnt >= threshold}
```

### 3.4 M365 Service Health Auto-Correlation

One of the most impactful **quick wins** available through the Graph API is correlating user-reported tickets with active Microsoft service incidents. When a technician receives a *"Teams not working"* ticket, the MIT should automatically check whether Microsoft has an active incident for the Teams service in the tenant's region and attach that context to the ticket.

```python
# service_health_check.py
from mit_graph_client import graph_get

def get_active_m365_incidents():
    issues = graph_get('/admin/serviceAnnouncement/issues'
                       '?$filter=isResolved eq false'
                       '&$select=id,title,service,classification,startDateTime')
    return issues.get('value', [])

def match_ticket_to_incident(service_name):
    '''Returns active MS incident for a given service, or None'''
    for issue in get_active_m365_incidents():
        if service_name.lower() in issue['service'].lower():
            return issue
    return None
```

When a match is found, the MIT updates the ITSM ticket with a note such as:

> *"Microsoft has an active service incident for Exchange Online (`INC123456`) started at 14:30 UTC. Resolution is in progress. This ticket will be auto-updated when the Microsoft incident is resolved."*

This single automation reduces unnecessary technician investigation and improves XLA **User Effort Scores**.

### 3.5 Safe Auto-Remediation Actions for M365 Issues

The following M365 remediation actions are candidates for classification as **Standard Changes** under ITIL v5's Change Enablement practice — low-risk, well-understood, and reversible:

| Issue Type | Automated Action | Trigger Condition |
| --- | --- | --- |
| **MFA prompt storm** | Revoke user refresh tokens (force clean sign-in) | ≥ 5 MFA prompts in 24h |
| **Licence assignment missing** | Re-trigger group-based licence via Graph PATCH | `LicenseDetails` shows missing app |
| **OneDrive sync paused** | Push Intune remediation script to reset sync client | OneDrive health event in Intune |
| **Orphaned account** (no sign-in 90d) | Flag for HR confirmation before disabling | `signInActivity.lastSignInDateTime` |
| **Shared mailbox permissions lost** | Re-add via Exchange Online PS from authorised group list | Ticket keyword match + Graph confirm |
| **Teams meeting quality poor** | Push Network QoS policy via Intune; reset Teams cache | ≥ 3 call records with jitter > 30 ms |

### 3.6 M365 Recurrence Dashboard: Key Metrics

The MIT dashboard should surface the following metrics for the service desk team and management:

- **Top 10 users by M365 incident frequency** *(last 30 days)* — identify users needing targeted support or device refresh
- **Top 5 recurring issue types by volume** — feed directly into Problem Management records
- **Microsoft Service Health timeline** — overlay MS incidents with ticket volume spikes to validate correlation
- **Teams call quality heatmap by office location** — identify network or infrastructure issues causing meeting degradation
- **Licence compliance rate** — percentage of users with correct licence assignments vs. entitlements
- **MFA registration completeness** — percentage of users with SSPR and MFA methods configured (reduces auth-related tickets)
- **Average time from Graph event detection to ticket creation** — measures MIT responsiveness

---

## Part 4 · Choosing and Integrating an Open-Source ITSM Platform

### 4.1 Platform Evaluation for Custom Integration

The three custom tools described above (PLA, PHMS, and MIT) all depend on a central ITSM platform to receive and manage the tickets, incidents, and problem records they generate. The ideal platform for custom integration has a **stable REST API**, **strong ITIL v5 alignment**, and **reasonable cost** for organisations also investing in custom tooling.

| Platform | REST API | ITIL Alignment | Asset / CMDB | Best For |
| --- | :---: | --- | :---: | --- |
| **GLPI** | ✅ Full REST | ITIL 4 / v5 ready | ✅ Native CMDB | SMB to mid-market; strong asset management |
| **iTop (Combodo)** | ✅ REST + JSON | ITIL 4 certified | ✅ Full CMDB | Mid-market; excellent CMDB for POS asset tracking |
| **osTicket** | ✅ REST API | Basic ITIL | ❌ Plugin only | Ticket-only; good for simple environments |
| **Faveo Helpdesk** | ✅ REST API | ITIL 4 practices | ✅ Basic asset | SMB with multi-channel ticket needs |

> 🏆 For organisations implementing all three custom tools described in this chapter, **GLPI or iTop are recommended**. Both provide native CMDB integration — enabling the custom tools to link printers, POS terminals, and M365 user accounts to Configuration Items (CIs) — and both expose stable REST APIs that the Python and .NET agents can call directly.

### 4.2 Unified API Integration Pattern

All three custom tools should use a consistent API integration pattern to create tickets, incidents, and problem records in the central ITSM platform. The following Python helper class provides a reusable client for GLPI's REST API:

```python
# itsm_client.py — reusable GLPI REST API client
import requests

class ITSMClient:
    def __init__(self, base_url, app_token, user_token):
        self.base = base_url
        self.session_token = self._init_session(app_token, user_token)

    def _init_session(self, app_token, user_token):
        r = requests.get(f'{self.base}/initSession',
            headers={'App-Token': app_token,
                     'Authorization': f'user_token {user_token}'})
        return r.json()['session_token']

    def create_ticket(self, name, content, itilcategory_id,
                       urgency=3, impact=3, priority=3):
        payload = {'input': {
            'name': name, 'content': content,
            'itilcategories_id': itilcategory_id,
            'urgency': urgency, 'impact': impact, 'priority': priority
        }}
        return requests.post(f'{self.base}/Ticket', json=payload,
            headers={'Session-Token': self.session_token,
                     'App-Token': 'YOUR_APP_TOKEN'}).json()

    def create_problem(self, name, content, ci_ids=[]):
        payload = {'input': {'name': name, 'content': content}}
        resp = requests.post(f'{self.base}/Problem', json=payload,
            headers={'Session-Token': self.session_token,
                     'App-Token': 'YOUR_APP_TOKEN'}).json()
        # Link to Configuration Items if provided
        for ci_id in ci_ids:
            requests.post(f'{self.base}/Problem_Item',
                json={'input': {'problems_id': resp['id'], 'items_id': ci_id}},
                headers={'Session-Token': self.session_token,
                         'App-Token': 'YOUR_APP_TOKEN'})
        return resp
```

### 4.3 Deployment Architecture Overview

The following describes how the three custom tools and the central ITSM platform connect in a typical deployment:

```text
┌──────────────────────────────┐        ┌───────────────────────────┐
│  Print Servers / Endpoints   │        │      POS Terminals        │
│  PrintService events         │        │  OPOS status events       │
│  (PowerShell exporter)       │        │  (.NET agent)             │
└────────────┬─────────────────┘        └────────────┬──────────────┘
             │                                       │
             ▼                                       ▼
   ┌───────────────────┐                  ┌───────────────────────┐
   │  PLA Python       │                  │  PHMS API Gateway     │
   │  ingestor         │                  │  + PostgreSQL         │
   └────────┬──────────┘                  └────────────┬──────────┘
            │                                          │
            │                                          │
            │       ┌──────────────────────────────┐   │
            │       │  Microsoft 365 / Entra ID    │   │
            │       │  Graph API polls             │   │
            │       │  (Python / MSAL)             │   │
            │       └──────────────┬───────────────┘   │
            │                      │                   │
            │                      ▼                   │
            │           ┌────────────────────────┐     │
            │           │  MIT pattern engine    │     │
            │           │  + Azure Automation    │     │
            │           └────────────┬───────────┘     │
            │                        │                 │
            └────────────┐           │           ┌─────┘
                         ▼           ▼           ▼
                ┌────────────────────────────────────────┐
                │     Central ITSM Platform              │
                │     GLPI / iTop                        │
                │     Incidents · Problems · CIs ·       │
                │     XLA surveys · Reports · KB         │
                └────────────────────────────────────────┘
```

> All three custom tools write to the same ITSM platform, meaning technicians see a **unified queue**. Tickets auto-generated by the PLA, PHMS, and MIT are tagged with their source, ensuring technicians can see whether an issue was user-reported or system-detected, and whether the event log context was pre-attached.

---

## Part 5 · Implementation Summary and ITIL v5 Alignment

### 5.1 Consolidated Implementation Checklist

#### 📅 Phase 1 — Foundation *(Weeks 1–4)*

- [ ] Enable `PrintService/Operational` logs via GPO on all print servers and endpoints
- [ ] Install SNMP monitoring for all network printers; document OIDs per printer model
- [ ] Register MIT application in Entra ID with minimum required Graph API permissions
- [ ] Select and deploy central ITSM platform (GLPI or iTop recommended)
- [ ] Define ITIL v5 ticket categories: *Printer*, *POS Hardware*, *M365 – Authentication*, *M365 – Email*, *M365 – Teams*, *M365 – Licensing*

#### 📅 Phase 2 — Custom Tool Deployment *(Weeks 5–12)*

- [ ] Deploy Printer Log Aggregator on print servers; validate event ingestion
- [ ] Deploy OPOS status monitor .NET service on POS terminals (staging site first)
- [ ] Deploy M365 Issue Tracker Python service on monitoring server
- [ ] Validate ITSM API integration for all three tools: test ticket creation end-to-end
- [ ] Enable M365 Service Health auto-correlation in MIT

#### 📅 Phase 3 — Recurrence and Automation *(Weeks 13–24)*

- [ ] Enable recurrence detection engines in PLA and PHMS
- [ ] Classify printer spooler reset and SNMP-triggered consumables request as **Standard Changes**
- [ ] Classify M365 safe-remediation actions (token revoke, licence re-trigger) as **Standard Changes**
- [ ] Build XLA post-resolution surveys for automated ticket closures
- [ ] Establish monthly Problem Management review using auto-raised problem records

### 5.2 ITIL v5 Practice Mapping: Custom Tools

| ITIL v5 MSF Practice | 🖨️ PLA (Printers) | 🛒 PHMS (POS) | ☁️ MIT (M365) |
| --- | --- | --- | --- |
| **Monitoring & Event Mgmt** | PrintService events + SNMP | OPOS status + terminal health score | Graph API polls + sign-in anomalies |
| **Incident Management** | Spooler crash → P2 auto-ticket | Register offline → P1 auto-ticket | MFA storm → incident + auto-remediate |
| **Problem Management** | 3+ errors/7d → problem record | 5+ faults/90d → hardware review | Recurring issue type → problem + KB |
| **Service Request Mgmt** | Low toner → consumables request | Paper out → store supervisor alert | Licence missing → re-trigger request |
| **Service Desk** | Pre-enriched tickets for technicians | Floor-plan dashboard for store ops | MS incident context auto-attached |

### 5.3 XLA Impact Projections

Based on implementations of similar tooling in comparable environments, the following XLA improvements are projected over an **18-month period**:

| XLA Metric | Baseline (Month 0) | Target (Month 18) |
| --- | :---: | :---: |
| User Effort Score — Printers | `2.8 / 5.0` | **`> 4.2 / 5.0`** |
| User Effort Score — POS Hardware | `2.5 / 5.0` | **`> 4.0 / 5.0`** |
| User Effort Score — M365 | `3.1 / 5.0` | **`> 4.3 / 5.0`** |
| Proactive Resolution Rate | `4%` | **`> 35%`** |
| Automated Remediation Rate | `2%` | **`> 45%`** |
| Recurrence Rate (same issue 30d) | `28%` | **`< 8%`** |
| Mean Time to Detect (MTTD) | User report (avg 47 min) | **`< 5 min` (automated)** |

### 5.4 Closing Notes

The three custom tools described in this chapter — the **Printer Log Aggregator**, **POS Hardware Monitoring System**, and **M365 Issue Tracker** — are not commercial products. They are purpose-built solutions assembled from standard components (Python, PowerShell, .NET, SNMP, and REST APIs) that any IT team with intermediate scripting capability can build and maintain.

> Their value comes not from technical sophistication but from **structural discipline**: they convert noisy, fragmented log data into structured, actionable records that feed ITIL v5's MSF practice cluster. Every recurring printer fault that becomes a *problem record* rather than the fifteenth reactive ticket is a concrete demonstration of ITIL v5's Product and Service Lifecycle Model working as intended — **support data flowing back into improvement**.

The supplementary implementation roadmap in §5.1 is designed to be achievable by a team of two to three people over six months, without specialist development skills. The goal is a service desk that **knows about printer failures, POS terminal degradation, and M365 authentication storms before the phone rings** — and increasingly, one that **fixes them before the user even notices**.

---

## Appendix · Quick Reference

### A. Printer Monitoring Event IDs

| Event ID | Log Source | Meaning |
| :---: | --- | --- |
| `307` | PrintService/Operational | Document successfully printed (job completion) |
| `316` | PrintService/Operational | Print job error / job abandoned |
| `353` | PrintService/Operational | Print job status update |
| `808` | PrintService/Admin | Printer driver installed |
| `820` | PrintService/Operational | Print spooler crash / restart |
| `843` | PrintService/Admin | Driver import failed (often post-update) |
| `1000` | Application | Application error in spooler process |
| `7034` | System | Print Spooler service terminated unexpectedly |
| `7036` | System | Print Spooler service state changed (start/stop) |

### B. Microsoft 365 Graph API Endpoints for Service Desk

| Endpoint | Use Case |
| --- | --- |
| `/reports/getTeamsUserActivityUserDetail(period='D7')` | Teams usage per user (7 days) |
| `/users/{id}/authentication/signInActivity` | Last sign-in timestamp |
| `/users/{id}/licenseDetails` | Current licence assignment details |
| `/auditLogs/signIns?$filter=...` | Sign-in log with MFA detail |
| `/admin/serviceAnnouncement/issues?$filter=isResolved eq false` | Active M365 service incidents |
| `/communications/callRecords?$filter=...` | Teams call quality records |
| `/identity/conditionalAccessPolicies` | CA policy audit (change detection) |

### C. SNMP Printer MIB OIDs

| OID | Metric | Notes |
| --- | --- | --- |
| `1.3.6.1.2.1.25.3.5.1.1.1` | Printer status | `3` = idle, `4` = printing, `5` = warmup |
| `1.3.6.1.2.1.25.3.5.1.2.1` | Printer detected error state | Bitmask; `0` = no error |
| `1.3.6.1.2.1.43.10.2.1.4.1.1` | Total page count | Track for PM scheduling |
| `1.3.6.1.2.1.43.11.1.1.8.1.1` | Toner max capacity | Denominator for % calculation |
| `1.3.6.1.2.1.43.11.1.1.9.1.1` | Toner current level | Numerator; alert if < 15% of max |
