<div align="center">

# ITIL v5 Change Management, Policy Alignment & Implementation Phases

### Comparing Current Practices to ITIL v5 · Change Workflows · Policy Update Methodology

**SUPPLEMENTARY CHAPTER**
*Companion to: ITIL Version 5 and the Active Service Desk Research Paper*

**May 2026** · IT Service Management Practice Series

</div>

---

## Introduction and Purpose

The preceding chapters of this series focused on *what* to build: monitoring tools, automated remediations, and ITSM integrations. This chapter addresses the organisational change challenge that must accompany technical implementation: **how to compare your current policies and procedures against ITIL v5, identify gaps, and drive controlled updates through a change management workflow that itself demonstrates ITIL v5 principles**.

An ITSM transformation that deploys sophisticated automation without updating the policies that govern it creates a compliance gap. Equally, a policy update programme that does not account for ITIL v5's new constructs - the Product and Service Lifecycle Model, Experience Level Agreements, and the 6C AI governance framework - will miss the structural improvements the framework offers.

> 💡 This chapter is designed to be practical and actionable. It provides a gap analysis framework, a policy inventory template, change management workflow patterns for both JSM and generic ITSM platforms, and a phased implementation roadmap that ties the technical chapters together into a governed transition plan.

---

## Part 1 · Understanding the ITIL v5 Changes That Require Policy Updates

### 1.1 Key Structural Changes from ITIL 4 to ITIL v5

Organisations that have implemented ITIL 4 will find that ITIL v5 is an evolution, not a replacement. However, certain structural changes have direct policy implications:

| ITIL 4 Construct | ITIL v5 Evolution | Policy Impact |
| --- | --- | --- |
| Service Value Chain (SVC) | Product and Service Lifecycle Model (PSLM) | Service design and improvement procedures need updating to reference PSLM stages |
| Service Value System (SVS) | SVS retained but extended | Existing SVS-based policy documents remain valid; supplement with PSLM references |
| General Management Practices (34) | Practice Manager modules (PIC, MSF, SVX) | Training, role definitions, and certification requirements should reference module structure |
| Continual Improvement practice | Embedded in each PSLM stage | CI records and procedures should reference PSLM stage outputs |
| Change Control practice | Change Enablement practice (in PIC module) | Change policy naming and scope may need updating |
| No native AI governance | 6C AI Governance Model | New: AI governance policy required where any AI-assisted or automated actions are used |
| SLAs as primary metric | XLAs formally mandated alongside SLAs | SLA documents need XLA companion metrics added |
| Sustainability: optional | Sustainability: mandatory principle | Procurement, asset management, and disposal policies need sustainability clauses |

### 1.2 The PSLM and What It Means for Your Procedures

The shift from Service Value Chain to **Product and Service Lifecycle Model (PSLM)** is the most structurally significant change. The PSLM treats every service as moving through five stages:

```
Discover → Design → Transition → Support → Improve
```

Unlike the SVC's parallel activities, the PSLM implies a **deliberate lifecycle gate** between each stage. This has direct procedural implications:

- **Discover stage procedures** should capture how telemetry data from *Support* (incident trends, hardware failure rates, XLA scores) feeds into decisions about new services or service changes
- **Design stage procedures** should include a step for reviewing endpoint health score data and known recurring issues before finalising a service design
- **Transition stage** should reference ITIL v5's *change enablement* process and require an automated remediation impact assessment for changes to device configurations
- **Support stage procedures** - the largest set - should reference the IASD model's four operational modes (Silent Monitoring, Proactive Engagement, Automated Remediation, Human-Led Response)
- **Improve stage procedures** should mandate XLA review alongside SLA review at defined intervals

### 1.3 New Policies Required by ITIL v5

The following policy documents are **required by ITIL v5** that most ITIL 4 organisations will not have:

| Policy Document Required | Triggering ITIL v5 Element | Urgency |
| --- | --- | :---: |
| AI and Automation Governance Policy | 6C AI Governance Model | 🔴 High |
| Experience Level Agreement (XLA) Policy | XLA formal adoption | 🔴 High |
| Automated Remediation Standard Change Register | Change Enablement / 6C Control | 🔴 High |
| Endpoint Telemetry Data Handling Policy | GDPR / 6C Compliance | 🟠 Medium |
| Sustainability Principles in IT Procurement | ITIL v5 Sustainability pillar | 🟡 Low–Medium |
| PSLM Service Entry and Exit Criteria | PSLM governance | 🟠 Medium |

---

## Part 2 · Policy Gap Analysis Framework

### 2.1 How to Conduct a Policy Gap Analysis

A structured gap analysis compares your current documented policies and procedures against ITIL v5 requirements. The output is a prioritised list of policy actions: **retain** (no change needed), **update** (existing document needs revision), or **create** (no existing document covers this requirement).

**Step 1: Inventory current policies**

Collect all current IT policy and procedure documents. Assign each a category from the ITIL v5 practice area it most closely relates to:

| Policy Category | Examples |
| --- | --- |
| Service Desk | Incident categorisation procedure, escalation matrix, working hours policy |
| Incident Management | Priority matrix, P1 major incident procedure, communication templates |
| Problem Management | Problem record procedure, known error database policy, post-incident review template |
| Change Management / Enablement | RFC template, CAB terms of reference, standard change register, emergency change procedure |
| Monitoring & Event Management | Alerting thresholds document, noise suppression rules, on-call rota |
| Asset & CMDB | Asset classification policy, CI naming convention, CMDB update procedure |
| Service Request Management | Service catalogue items, SLA per request type, fulfilment workflow |
| Procurement | Hardware procurement policy, vendor assessment procedure |
| Security | Endpoint security baseline, acceptable use policy, data handling policy |

**Step 2: Map each policy to ITIL v5 constructs**

For each policy document, record:
- Which ITIL v5 practice(s) it relates to
- Whether it references ITIL 4 or older ITIL versions explicitly
- Whether it contains SLA-only metrics (no XLA equivalent)
- Whether it addresses automated actions at all
- Whether it addresses AI governance

**Step 3: Identify gaps**

Apply the gap analysis matrix below:

| Gap Type | Indicator | Action |
| --- | --- | --- |
| **Missing construct** | No policy covers ITIL v5 requirement | Create new policy |
| **Outdated reference** | Document references ITIL 4 SVC or older constructs | Update references |
| **SLA-only metrics** | Document defines success metrics without XLA equivalent | Add XLA supplement |
| **Automation void** | Document describes manual process that IASD model automates | Update to document automated path |
| **AI governance absent** | Any automated or AI-assisted process without 6C controls | Add 6C governance section or create AI governance policy |
| **Sustainability absent** | Procurement or disposal procedures lack sustainability clauses | Update with ITIL v5 sustainability principle |

### 2.2 Gap Analysis Scoring Matrix

Assign each identified gap a **Priority Score** using this two-axis scoring:

$$\text{Priority} = \text{Impact} \times \text{Effort}^{-1}$$

Where:
- **Impact** (1–5): risk of non-compliance, XLA failure, or audit finding if the gap is not closed
- **Effort** (1–5): estimated effort to close the gap (1 = trivial update, 5 = new policy from scratch)

| Impact | Effort | Priority Score | Recommended Timeline |
| :---: | :---: | :---: | --- |
| 5 | 1–2 | High | Close within 30 days |
| 5 | 3–5 | High | Close within 60 days |
| 3–4 | 1–3 | Medium | Close within 90 days |
| 3–4 | 4–5 | Medium | Close within 120 days |
| 1–2 | Any | Low | Close within 180 days |

### 2.3 Sample Gap Analysis Output

Below is a sample gap analysis result for a typical mid-market ITIL 4 organisation:

| Policy Document | Current State | ITIL v5 Gap | Action | Priority |
| --- | --- | --- | --- | :---: |
| Incident Management Procedure | ITIL 4 aligned | No XLA metrics; no automated resolution path | Update | High |
| Change Control Policy | ITIL 4 Change Control | Rename to Change Enablement; add standard change automation register | Update | High |
| Service Desk Operating Procedure | ITIL 4 aligned | No reference to IASD modes; no proactive monitoring workflow | Update | High |
| Problem Management Procedure | ITIL 4 aligned | No automated problem-raising pathway | Update | Medium |
| Acceptable Use Policy | Current | No telemetry monitoring consent clause | Update | Medium |
| AI/Automation Governance Policy | Does not exist | Entire 6C framework absent | Create | High |
| XLA Policy and Measurement Guide | Does not exist | XLA entirely absent | Create | High |
| Automated Remediation Standard Change Register | Does not exist | No standard change register for automated scripts | Create | High |
| Hardware Procurement Policy | Current | No sustainability clause | Update | Low |
| CMDB Naming Convention | Current | No reference to PSLM stage data | Update | Low |

---

## Part 3 · Writing and Updating Policies for ITIL v5

### 3.1 Policy Document Structure for ITIL v5 Alignment

Every ITIL v5-aligned policy document should follow a consistent structure. This structure ensures reviewers can quickly locate the relevant ITIL v5 construct and assess compliance:

```markdown
# [Policy Name]

**Version:** x.x  
**Effective Date:** [date]  
**Owner:** [Role, not person]  
**Review Cycle:** Annual  
**ITIL v5 Practice Area(s):** [e.g., Change Enablement (PIC module)]  
**Related Policies:** [links to related documents]  

---

## 1. Purpose
[Why this policy exists and what behaviour it governs]

## 2. Scope
[Who and what is covered; explicit exclusions]

## 3. Definitions
[Key terms, especially any ITIL v5 constructs used]

## 4. Policy Statements
[Numbered, specific, measurable policy statements]

## 5. Procedures
[Step-by-step workflows; reference to supporting procedure documents]

## 6. Roles and Responsibilities
[RACI matrix or role descriptions]

## 7. Metrics and Measurement
[SLA metrics and XLA companion metrics; review frequency]

## 8. Exceptions and Escalations
[How to request an exception; escalation path]

## 9. Compliance and Audit
[How compliance is verified; consequence of non-compliance]

## 10. Review History
[Version table: version, date, author, change summary]
```

### 3.2 Template: AI and Automation Governance Policy

The most commonly absent policy in ITIL 4 organisations is an AI and Automation Governance Policy. The following template provides the essential content, aligned with ITIL v5's 6C model:

---

**AI and Automation Governance Policy**
**Version:** 1.0 | **ITIL v5 Practice Area:** Change Enablement, Service Desk (MSF module) | **Owner:** Head of IT Operations

---

**1. Purpose**

This policy governs the development, deployment, and ongoing operation of all automated and AI-assisted actions taken on managed devices, cloud services, and the ITSM platform by the IT service desk and engineering functions.

**2. Scope**

This policy applies to all automated scripts, remediation workflows, AI-assisted triage tools, and ITSM automation rules that initiate actions on user devices, cloud service configurations, or ITSM records without direct human initiation at the point of action.

**3. The 6C Framework**

All automation and AI deployments must demonstrate compliance with each of the following six dimensions prior to production deployment:

| Dimension | Requirement |
| --- | --- |
| **Control** | An accountable owner (a named role) is designated for each automated action. A human override mechanism exists and is documented. |
| **Compliance** | Automated actions are registered as Standard Changes in the Change Register. All actions are logged with full audit trail. |
| **Culture** | Technicians are informed of all automation in their operational scope. Automation is introduced in consultation with affected teams. |
| **Competence** | Automated scripts are tested in a non-production environment before deployment. Detection and remediation logic is validated against current hardware and software catalogue. |
| **Communication** | Users receive a notification when an automated action is taken on their device. Notification templates are approved by the service desk manager. |
| **Continual Improvement** | Automated action outcomes are reviewed monthly against XLA metrics. Actions with negative XLA outcomes are suspended pending review. |

**4. Standard Change Classification for Automated Actions**

Automated remediation scripts may be classified as Standard Changes when all of the following criteria are met:

1. The script has been tested in a controlled environment and produces a documented, consistent outcome
2. A rollback procedure exists and has been tested
3. The action scope is bounded - it cannot escalate its own permissions or modify systems outside the defined target
4. The script is stored in version control with access restricted to the automation service account and named engineers
5. The CAB sub-committee has reviewed and approved the classification

**5. Metrics and Review**

- Monthly: automated action volume, success rate, user complaint rate, XLA survey scores for automated closures
- Quarterly: 6C compliance review for all active automation; retire or remediate non-compliant automations
- Annually: full policy review against updated ITIL v5 guidance and organisational risk appetite

---

### 3.3 Template: XLA Policy and Measurement Guide

---

**Experience Level Agreement (XLA) Policy and Measurement Guide**
**Version:** 1.0 | **ITIL v5 Practice Area:** Service Desk, Service Level Management (MSF module) | **Owner:** Service Desk Manager

---

**1. Purpose**

This policy establishes the organisation's commitment to measuring and improving the quality of IT service experiences, not merely technical SLA metrics. XLAs exist alongside - not instead of - SLAs.

**2. Core XLA Metrics**

| Metric | Definition | Target | Measurement Method |
| --- | --- | :---: | --- |
| **User Effort Score (UES)** | How much effort did the user expend to reach resolution? (1=high effort, 5=effortless) | ≥ 4.0 | Post-resolution survey (5-point scale) |
| **Disruption Index** | Total minutes of productive work disrupted per incident | < 20 min | Calculated: ticket open time × reported impact level |
| **First-Contact Resolution (FCR)** | Percentage of tickets resolved at first contact without escalation | ≥ 75% | ITSM platform report |
| **Recurrence Rate** | Percentage of resolved incidents that recur within 30 days for the same user/device | < 10% | ITSM linkage report |
| **Proactive Resolution Rate** | Percentage of incidents detected and resolved before the user raised a ticket | ≥ 30% | ITSM label: auto-detected AND closed before user report |
| **Automated Remediation Rate** | Percentage of incidents resolved entirely through automated means | ≥ 40% (18-month target) | ITSM resolution code: Automated Fix |

**3. Survey Deployment**

- Post-resolution surveys are sent automatically via the ITSM platform within 1 hour of ticket closure
- Survey content: single question - *"How easy was it to get your IT issue resolved today?"* (1–5 scale) - plus optional free-text comment
- Surveys are not sent for auto-closed tickets where no user interaction occurred; instead, a 24-hour follow-up is sent: *"Your IT issue was resolved automatically. Did this fix your problem?"* (Yes / No / Still having issues)

**4. XLA Review Cadence**

- **Weekly**: Service Desk Manager reviews average UES and any scores below 3.0 for root-cause identification
- **Monthly**: Full XLA dashboard review with management; comparison against previous month and rolling 12-month trend
- **Quarterly**: XLA performance included in IT steering group report alongside SLA performance

---

## Part 4 · Change Management Workflows

### 4.1 ITIL v5 Change Types and Their IASD Equivalents

ITIL v5's **Change Enablement** practice (in the PIC module) defines three change types. In the IASD context, these map to specific change scenarios:

| Change Type | ITIL v5 Definition | IASD Examples |
| --- | --- | --- |
| **Standard Change** | Pre-approved, low-risk, follows a documented procedure | Automated print spooler restart; DNS cache flush; NIC driver reset; OneDrive sync reset; Intune remediation script deployment |
| **Normal Change** | Assessed and approved through normal process; goes to CAB | New Intune remediation script for a new issue type; changes to IASD health score weightings; new JSM automation rule |
| **Emergency Change** | Rapid approval required; risk accepted to restore service quickly | Rolling back a script that caused widespread issues; emergency driver update for a newly discovered vulnerability |

### 4.2 Standard Change Register for IASD Automated Actions

Every automated action in the IASD model must be registered as a Standard Change before it can execute in production. The register should be maintained in the ITSM platform (as a JSM Standard Change template or equivalent) and include:

| Field | Content |
| --- | --- |
| **Change ID** | SC-IASD-001 (sequential) |
| **Name** | Print Spooler Restart and Spool Queue Clear |
| **Category** | Endpoint Remediation |
| **Trigger Condition** | Print spooler in stopped/error state OR ≥ 3 stuck print jobs |
| **Automated Action** | Stop Spooler service; delete files in `%SYSTEMROOT%\System32\spool\PRINTERS`; Start Spooler service |
| **Target Scope** | Any Windows endpoint in scope of Intune Remediation policy `PR-PrintSpooler` |
| **Rollback Procedure** | Spooler service manual start: `Start-Service Spooler`; re-add persistent print connections from GPO |
| **User Notification** | Email: *"Your print service was automatically restarted. If issues persist, please contact the service desk."* |
| **Test Evidence** | Test results on 5 devices across 3 OS versions; link to test record |
| **CAB Approval Date** | [Date] |
| **Review Date** | [Date + 12 months] |
| **Owner** | Endpoint Support Lead |

### 4.3 Normal Change Workflow in JSM

New IASD components - new Intune remediation scripts, new JSM automation rules, changes to health score weightings - should go through a Normal Change workflow in JSM:

```
RFC Submitted → Initial Assessment → Technical Review → 
CAB Review Meeting → Approved / Rejected → 
Implementation (scheduled) → Post-Implementation Review → Closed
```

**JSM Change Request configuration for IASD Normal Changes:**

Configure a JSM Change Request screen with the following fields relevant to IASD:

| Field Name | Field Type | ITIL v5 Mapping |
| --- | --- | --- |
| Change Category | Dropdown | Standard / Normal / Emergency |
| Automation Impact | Dropdown | No automated action / Updates existing auto / Adds new auto |
| 6C Compliance Checklist | Checkbox group | Control, Compliance, Culture, Competence, Communication, Continual Improvement |
| Affected CIs | CI picker (CMDB) | Configuration Management |
| Rollback Plan | Text area | Change Enablement |
| Test Results | File attachment | Change Enablement |
| XLA Impact Assessment | Text area | Service Level Management |
| Risk Score | Calculated (Low/Medium/High) | Risk Management |

**JSM automation rule for 6C compliance gate:**

```
Trigger: Issue Transitioned (to 'CAB Review')
Condition: Issue Type = Change Request AND Automation Impact != 'No automated action'
Action: 
  Check: all six 6C Compliance Checklist items are checked
  If not all checked:
    Add comment: "This change includes automated actions. All 6C compliance 
                  checkboxes must be completed before CAB review."
    Transition back to: Technical Review
```

### 4.4 Emergency Change Procedure for IASD

Emergency changes in the IASD context are most likely triggered by:

1. An automated script causing unintended widespread impact (e.g., a remediation script deleting files it should not have)
2. A critical security vulnerability requiring immediate deployment of an updated Intune configuration
3. A JSM automation rule causing runaway ticket creation

**Emergency Change Checklist:**

```markdown
## IASD Emergency Change Checklist

**Triggered by:** [Incident key / description]  
**Requestor:** [Name and role]  
**Approver (emergency):** [Head of IT Operations or delegate]  
**Time approved:** [timestamp]  

### Immediate Actions
- [ ] Disable/suspend the offending automated action (Intune, JSM, PLA/PHMS/MIT)
- [ ] Assess scope of impact (how many devices/users/tickets affected)
- [ ] Notify affected users and service desk team
- [ ] Document the timeline of events

### Remediation Plan
- [ ] Root cause identified
- [ ] Fix tested in non-production
- [ ] Rollback available if fix fails
- [ ] Senior approver sign-off obtained

### Post-Emergency Actions (within 72 hours)
- [ ] Post-Incident Review scheduled
- [ ] Normal Change raised to formalise the emergency fix
- [ ] 6C compliance review for the affected automation
- [ ] Update Standard Change Register if the trigger condition was incorrect
- [ ] Lessons learned shared with CAB sub-committee
```

---

## Part 5 · Implementation Phases - From Current State to ITIL v5 IASD

### 5.1 Phase Model Overview

The implementation phases below build on the roadmap in the main research paper, adding the **policy and change management dimension** that is necessary for a governed transition. Each phase has technical deliverables (from the earlier chapters), policy deliverables (from this chapter), and governance gates that must be passed before moving to the next phase.

```
Phase 0: Assessment and Baseline
        ↓
Phase 1: Foundation (Months 1–3)
        ↓
Phase 2: Proactive Engagement (Months 4–6)
        ↓
Phase 3: Automated Remediation (Months 7–12)
        ↓
Phase 4: Optimisation and AI Integration (Months 13–24)
```

### 5.2 Phase 0 - Assessment and Baseline *(Weeks 1–4)*

**Objective:** Understand current state. Establish baselines. Produce the gap analysis.

#### Technical Assessment

- [ ] Inventory all current endpoint management tools (SCCM, Intune, RMM, manual)
- [ ] Assess current event log collection: what is centralised? what is siloed?
- [ ] Review current ITSM platform configuration: ticket types, workflows, automation rules
- [ ] Document current printer fleet, POS terminal count, M365 tenant configuration
- [ ] Measure baseline metrics: average ticket volume by category, MTTR, FCR, repeat ticket rate

#### Policy Assessment

- [ ] Inventory all current IT policy documents (see Section 2.1 template)
- [ ] Conduct gap analysis against ITIL v5 requirements (see Section 2.2 matrix)
- [ ] Produce prioritised policy action list
- [ ] Identify policy owner for each document to be created or updated

#### Governance Gate: Phase 0 → Phase 1

> ✅ Baseline metrics documented and agreed  
> ✅ Gap analysis complete and signed off by IT Management  
> ✅ Policy action list prioritised and owners assigned  
> ✅ JSM project structure agreed (or existing ITSM project structure reviewed)

### 5.3 Phase 1 - Foundation *(Months 1–3)*

**Objective:** Deploy monitoring infrastructure. Create highest-priority policy documents. Establish the CAB sub-committee for IASD governance.

#### Technical Deliverables

- [ ] Deploy/verify Microsoft Intune coverage to ≥ 95% of managed devices
- [ ] Enable `PrintService/Operational` event log via GPO across all print servers
- [ ] Configure Windows Event Collector (WEC) and deploy WEF subscription (see Jira/Intune chapter)
- [ ] Deploy baseline Intune compliance policies aligned to ITIL v5 health model (see Section 4.4 of Jira/Intune chapter)
- [ ] Configure JSM projects: HW, M365, POS, ITSM (or review equivalent in existing ITSM platform)
- [ ] Implement endpoint health score calculation (manual or semi-automated) using Intune compliance + event log data
- [ ] Deploy initial JSM dashboard for service desk team

#### Policy Deliverables - Phase 1

- [ ] **Create**: AI and Automation Governance Policy (v1.0 - automation scope limited to detection only at this stage)
- [ ] **Create**: XLA Policy and Measurement Guide (v1.0 - define metrics; begin baseline data collection)
- [ ] **Update**: Incident Management Procedure - add reference to IASD operational modes and health score triggers
- [ ] **Update**: Service Desk Operating Procedure - add proactive notification section (even though not yet active)
- [ ] **Create**: Initial Standard Change Register template (blank; populated in Phase 3)
- [ ] **Establish**: IASD CAB Sub-Committee: terms of reference, meeting cadence (monthly), membership (IT Ops Lead, Service Desk Manager, Security representative, one end-user representative)

#### Governance Gate: Phase 1 → Phase 2

> ✅ Endpoint agent/Intune coverage ≥ 95%  
> ✅ Event log data flowing to central WEC or equivalent  
> ✅ JSM projects configured with correct issue types and basic automation rules  
> ✅ Health score dashboard operational (even if manually calculated)  
> ✅ AI/Automation Governance Policy v1.0 approved  
> ✅ XLA baseline established (first 30 days of XLA survey data collected)  
> ✅ IASD CAB Sub-Committee first meeting held

### 5.4 Phase 2 - Proactive Engagement *(Months 4–6)*

**Objective:** Activate proactive monitoring and notifications. Deploy the PLA, initial PHMS data collection, and MIT service health correlation. Transition manual health scoring to automated.

#### Technical Deliverables

- [ ] Deploy **Printer Log Aggregator (PLA)** on all print servers; connect to JSM via REST API
- [ ] Enable SNMP polling for printer fleet; connect toner/status alerts to PLA
- [ ] Deploy **MIT** Graph API polling: service health, sign-in activity, licence compliance
- [ ] Activate **M365 service health auto-correlation** in JSM (Rule 2 from Jira/Intune chapter)
- [ ] Automate endpoint health score calculation from Intune + WEC event data
- [ ] Activate proactive user notifications for health scores < 70
- [ ] Deploy WEC event processor; activate JSM incident creation for USB errors, app crash loops, spooler crashes (detection only - no automated remediation yet)
- [ ] Publish self-service diagnostic guides in JSM knowledge base for top-10 issue types

#### Policy Deliverables - Phase 2

- [ ] **Update**: Problem Management Procedure - add automated problem-raising section; reference PLA and MIT recurrence thresholds
- [ ] **Update**: Monitoring and Event Management Policy - add SNMP printer monitoring, WEC scope, and WEF subscription details
- [ ] **Update**: Acceptable Use Policy - add telemetry monitoring consent clause; inform users that endpoint health scoring is active
- [ ] **Create**: Proactive Notification Policy - defines when automated outreach is sent, what it contains, and user opt-out rights
- [ ] **Update**: AI/Automation Governance Policy - expand to cover automated notification scope (Stage 2 of maturity ladder)
- [ ] **Update**: XLA Policy - add Proactive Resolution Rate metric now that proactive detection is active

#### Governance Gate: Phase 2 → Phase 3

> ✅ PLA operational and raising JSM incidents automatically  
> ✅ MIT service health correlation active and reducing unnecessary technician investigation  
> ✅ Proactive notifications sent and user feedback collected (Effort Score ≥ 4.0 target tracking)  
> ✅ Problem Management Procedure updated and signed off  
> ✅ Acceptable Use Policy updated and re-circulated to all staff  
> ✅ No unresolved complaints from users about unexpected automated notifications  
> ✅ CAB Sub-Committee Month 4 review: approve proceeding to automated remediation

### 5.5 Phase 3 - Automated Remediation *(Months 7–12)*

**Objective:** Deploy automated remediation scripts. Classify approved scripts as Standard Changes. Begin measuring Automated Remediation Rate as an XLA metric. Expand to POS monitoring where applicable.

#### Technical Deliverables

- [ ] Deploy **Intune Remediations** for print spooler, OneDrive sync, NIC driver (from Jira/Intune chapter)
- [ ] Deploy **Intune non-compliance monitor** with automatic JSM incident creation
- [ ] Deploy **Intune Remediation failure detector** with automatic JSM problem creation for systemic failures
- [ ] Activate JSM Rule 3 (escalate unacknowledged auto-raised incidents) and Rule 4 (link printer duplicates to open problem)
- [ ] Deploy **PHMS** terminal agent and API on POS estate (if applicable)
- [ ] Activate **M365 safe auto-remediation** actions: token revocation for MFA storms, licence re-trigger, Teams cache reset via Intune
- [ ] Implement JSM Rule 5 (auto-close M365 incidents when MS incident resolves)
- [ ] Activate post-resolution XLA surveys for all automated ticket closures

#### Policy Deliverables - Phase 3

- [ ] **Populate**: Standard Change Register (SC-IASD-001 through SC-IASD-010+) for all deployed automated actions - each with test evidence, rollback procedure, and CAB approval date
- [ ] **Create**: Automated Remediation Testing and Deployment Standard - defines test environment requirements, test case structure, rollback test requirement, and promotion criteria
- [ ] **Update**: Change Enablement Policy - rename from Change Control if required; add standard change fast-track path for IASD-classified automations; add 6C checklist requirement for all Normal Changes involving automation
- [ ] **Update**: AI/Automation Governance Policy - expand to cover Stage 3 (automated remediation); add scope boundaries for each deployed script
- [ ] **Update**: Incident Management Procedure - document the automated resolution path (event → Intune remediation → auto-close → XLA survey); distinguish from manual path
- [ ] **Update**: Service Request Management Policy - document that toner and consumables requests may be auto-generated by SNMP events; define fulfilment workflow

#### Governance Gate: Phase 3 → Phase 4

> ✅ All deployed automated actions registered as Standard Changes with full documentation  
> ✅ Automated Remediation Rate ≥ 20% (progress toward 40% target)  
> ✅ No automated action has caused a user-reported negative outcome in the last 60 days  
> ✅ XLA UES for automated closures ≥ 4.0  
> ✅ Recurrence Rate ≤ 12% (progress toward 10% target)  
> ✅ CAB Sub-Committee Month 10 review: approve proceeding to AI-assisted triage  
> ✅ Change Enablement Policy updated, approved by IT Management, and circulated

### 5.6 Phase 4 - Optimisation and AI Integration *(Months 13–24)*

**Objective:** Introduce AI-assisted triage. Implement predictive hardware replacement. Mature XLA framework. Pursue ITIL v5 Practice Manager certification. Close remaining policy gaps.

#### Technical Deliverables

- [ ] Deploy ML-based ticket classification using 12+ months of training data from JSM
- [ ] Implement predictive device refresh scheduling based on health score trends
- [ ] Expand MIT to cover Teams call quality heatmaps and QoS policy auto-deployment
- [ ] Implement **PSLM feedback loop**: monthly report from Support stage data → Discover stage (hardware procurement recommendations, vendor defect reports)
- [ ] Expand PHMS recurrence engine to flag vendor/model defect patterns across POS terminal fleet
- [ ] Integrate JSM reports with Power BI or Confluence for management XLA dashboard
- [ ] Begin ITIL v5 Practice Manager (MSF) certification pathway for senior analysts

#### Policy Deliverables - Phase 4

- [ ] **Close all remaining gap analysis items** identified in Phase 0
- [ ] **Update**: AI/Automation Governance Policy - expand to cover AI-assisted triage; add accuracy validation requirement; add demographic/device-class bias review
- [ ] **Create**: PSLM Feedback Procedure - documents how Support stage data (incident trends, health scores, XLA outcomes) is formally submitted to IT leadership for service and procurement decisions (Discover stage gate)
- [ ] **Update**: Problem Management Procedure - add predictive problem pattern detection section (ML-identified patterns)
- [ ] **Create**: IT Service Desk AI Model Governance Register - tracks all ML models in use, training data sources, validation results, and review schedule (ITIL v5 6C Competence dimension)
- [ ] **Update**: Asset and CMDB Policy - add lifecycle stage tagging aligned to PSLM; add health score threshold for end-of-life flag
- [ ] **Create**: Sustainability Principles in IT Procurement - hardware procurement checklist referencing ITIL v5 sustainability pillar; includes energy efficiency, repairability, and disposal considerations

---

## Part 6 · Managing Resistance and Organisational Change

### 6.1 Common Sources of Resistance

ITIL v5 transitions, particularly those involving automation and proactive monitoring, encounter resistance from multiple directions. Understanding these in advance enables targeted change management communication:

| Stakeholder Group | Common Concern | Mitigation Approach |
| --- | --- | --- |
| **End users** | *"IT is watching what I do on my device"* | Transparent communication about what telemetry is collected and why; clear opt-out for personal devices; update AUP before monitoring goes live |
| **Service desk technicians** | *"Automation is replacing my job"* | Emphasise that automation handles routine tasks, freeing technicians for complex problem-solving and user relationship work; involve technicians in script design and governance |
| **IT management** | *"This is expensive and uncertain"* | Show ROI model: reduced ticket handling time × ticket volume = staff hours recovered; reference XLA improvement projections |
| **Security team** | *"Automated scripts are a privilege escalation risk"* | Present 6C governance model, scope limitations, code signing requirements, and audit trail; invite security to IASD CAB Sub-Committee |
| **CAB / Change Advisory** | *"We don't want to rubber-stamp automation"* | Give the CAB real governance authority via the sub-committee model; frame their role as oversight and improvement, not blocking |

### 6.2 Communication Plan for IASD Rollout

Each phase transition should be accompanied by structured communications:

**Before Phase 2 (proactive notifications go live):**
> Send to all staff from IT Director or CTO level:
> *"From [date], our IT team will be using automated health monitoring to proactively identify and fix common issues on your device before they cause problems. You may occasionally receive a notification that your device has been checked or a small fix applied automatically. This is intentional and designed to reduce the time you spend without a working device. You can find more information at [link to self-service portal]."*

**Before Phase 3 (automated remediation goes live):**
> Send to all staff with a description of what will be automated, what user notification they will receive, and how to report if an automated action caused an unexpected problem.

**Monthly manager briefing:**
> Brief format: XLA trend (UES, FCR, recurrence rate), automated resolution rate progress, top-3 open problems, and any upcoming changes. This keeps stakeholders informed and builds confidence in the programme.

### 6.3 Measuring Organisational Change Adoption

Beyond the technical XLA metrics, track the following adoption indicators:

- **Self-service article usage rate** - are users reading knowledge base articles before raising tickets?
- **Ticket reopen rate** - declining reopen rate indicates users trust automated resolutions
- **Technician survey score** - quarterly survey asking technicians whether the IASD model helps or hinders their work
- **CAB Sub-Committee engagement** - are members attending and contributing, or is governance becoming a rubber stamp?
- **Policy acknowledgment rate** - percentage of staff who have acknowledged updated AUP and telemetry policies

---

## Part 7 · Aligning with ITIL v5 Certification Pathways

### 7.1 ITIL v5 Certification Structure

ITIL v5 introduced three **Practice Manager modules** as formal certification pathways. Each module bundles a set of practices relevant to a specific operational role:

| Module | Practices Covered | Relevant To |
| --- | --- | --- |
| **MSF (Monitor, Support & Fulfil)** | Service Desk, Incident Mgmt, Problem Mgmt, Monitoring & Event Mgmt, Service Request Mgmt | Service desk analysts, team leads, ITSM managers |
| **PIC (Plan, Implement & Control)** | Change Enablement, Release Mgmt, Deployment Mgmt, IT Asset Mgmt, Service Configuration Mgmt | Change managers, release engineers, CMDB owners |
| **SVX (Service Value & Experience)** | Service Level Mgmt, Relationship Mgmt, Supplier Mgmt, Continual Improvement | Service managers, CIOs, vendor managers |

For an IASD implementation team, the recommended certification path is:

1. **All service desk team members**: ITIL Foundation v5 - establishes common language and framework understanding
2. **Service desk team leads and ITSM managers**: ITIL Practice Manager MSF - deep dive into the five MSF practices directly governing IASD operations
3. **Change managers and engineers responsible for automation deployment**: ITIL Practice Manager PIC - covers Change Enablement and Configuration Management relevant to the standard change register
4. **IT management and service owners**: ITIL Practice Manager SVX - covers XLA and service level management at the governance level

### 7.2 Certification Timing Within Implementation Phases

| Phase | Recommended Certification Activity |
| --- | --- |
| Phase 0 | All team members: ITIL Foundation v5 (or conversion from ITIL 4 Foundation) |
| Phase 1–2 | Service desk leads: begin ITIL Practice Manager MSF preparation |
| Phase 2–3 | Change managers: begin ITIL Practice Manager PIC preparation |
| Phase 3 completion | Service desk leads: sit ITIL Practice Manager MSF examination |
| Phase 4 | IT management: ITIL Practice Manager SVX; senior analysts: MSF if not already completed |

### 7.3 Mapping This Paper Series to ITIL v5 Exam Topics

The papers in this series directly address the following ITIL v5 exam topic areas:

| Paper | ITIL v5 Exam Topics Covered |
| --- | --- |
| **Active Service Desk Research Paper** | PSLM model; MSF practice cluster; XLA framework; 6C AI governance; automation maturity ladder; IASD model |
| **Custom Software Implementation** | Monitoring & Event Management practice; Incident Management (automated raising); Problem Management (recurrence detection); Service Request Management (consumables automation) |
| **Jira & Intune Integration** | Change Enablement (standard change automation); Service Desk (ITSM platform configuration); Monitoring integration patterns |
| **This Document** | Change Enablement (policy; CAB workflows); Service Level Management (XLA policy); AI governance (6C policy); Implementation roadmap |

---

## Appendix A · Policy Gap Analysis Worksheet

Copy and complete this worksheet during Phase 0 assessment.

| # | Policy Document Name | Location | Owner | ITIL v5 Practice(s) | Contains XLA? | Covers Automation? | AI Governance? | Sustainability? | Gap Action | Priority |
| --- | --- | --- | --- | --- | :---: | :---: | :---: | :---: | --- | :---: |
| 1 | | | | | ☐ | ☐ | ☐ | ☐ | | |
| 2 | | | | | ☐ | ☐ | ☐ | ☐ | | |
| 3 | | | | | ☐ | ☐ | ☐ | ☐ | | |
| 4 | | | | | ☐ | ☐ | ☐ | ☐ | | |
| 5 | | | | | ☐ | ☐ | ☐ | ☐ | | |

*Extend as required. Gap Action: Retain / Update / Create. Priority: High / Medium / Low.*

---

## Appendix B · Standard Change Register Template

| Field | Entry |
| --- | --- |
| **Change ID** | SC-IASD-[nnn] |
| **Change Name** | |
| **Category** | Endpoint Remediation / Cloud Remediation / ITSM Automation |
| **Trigger Condition** | |
| **Automated Action Description** | |
| **Target Scope** | |
| **Intune Policy / JSM Rule / Script Name** | |
| **Rollback Procedure** | |
| **User Notification Text** | |
| **Test Environment Used** | |
| **Test Date and Results** | |
| **CAB Sub-Committee Approval Date** | |
| **Approver Name and Role** | |
| **Review Date** | |
| **Owner (role)** | |
| **Version** | 1.0 |

---

## Appendix C · IASD CAB Sub-Committee Terms of Reference

**Name:** IASD Governance and Automation Review Sub-Committee

**Purpose:** To provide governance oversight of all automated and AI-assisted actions deployed as part of the Integrated Active Service Desk model, ensuring compliance with ITIL v5 Change Enablement, the 6C AI Governance framework, and the organisation's AI and Automation Governance Policy.

**Membership:**
- IT Operations Lead (Chair)
- Service Desk Manager
- Information Security representative
- One service desk analyst (rotating, 6-month term)
- One end-user representative from a business unit (rotating, 12-month term)

**Meeting Cadence:** Monthly (can be combined with existing CAB if agenda time is protected)

**Standing Agenda Items:**
1. New automation proposals - review against 6C checklist; approve/reject standard change classification
2. Audit log review - sample of automated actions from previous month; any unintended outcomes?
3. XLA data review - are automated resolutions improving user experience metrics?
4. AI model performance (where applicable) - accuracy, false positive rate, bias review
5. Policy actions - review outstanding policy updates; sign off on completed updates
6. Any other business

**Decision Authority:**
- Approve new Standard Changes: ✅ (majority vote)
- Approve Normal Changes involving automation: ✅ (recommendation to full CAB)
- Suspend an active automated action: ✅ (Chair unilateral authority in an emergency; sub-committee ratification at next meeting)
- Approve Emergency Changes: ❌ (this authority remains with Head of IT Operations)

---

## References and Further Reading

1. **PeopleCert** (2026). *ITIL Foundation (Version 5).* PeopleCert Official Publication.
2. **PeopleCert** (2026). *ITIL Practice Manager: Monitor, Support and Fulfil (MSF) Module.*
3. **PeopleCert** (2026). *ITIL Practice Manager: Plan, Implement and Control (PIC) Module.*
4. **Atlassian** (2026). *Jira Service Management: ITSM Best Practices.* <https://www.atlassian.com/itsm>
5. **Microsoft** (2026). *Microsoft Intune documentation - Proactive Remediations.* <https://learn.microsoft.com/mem/intune>
6. **Microsoft** (2026). *Microsoft Graph API reference.* <https://learn.microsoft.com/graph>
7. **AXELOS** (2019). *ITIL 4 Foundation.* TSO (The Stationery Office). *(ITIL 4 reference for comparative gap analysis)*
8. **ISO/IEC 20000-1:2018** - *Information technology - Service management - Part 1: Service management system requirements.* *(Complementary international standard)*
9. **Incident.io** (2026). *Everything You Need to Know About ITIL 5, AI and Incident Management.* <https://incident.io/blog>
