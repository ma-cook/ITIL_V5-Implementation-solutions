<div align="center">

# ITIL v5 Practical Solutions

### A Research and Implementation Series for the Modern IT Service Desk

**IT Service Management Practice Series** · May 2026

</div>

---

## About This Series

This repository contains a series of research papers and implementation guides exploring how **ITIL Version 5** (released February–May 2026) can be applied practically to IT service desk operations. The series focuses on on-site and remote environments — desktop office hardware, Microsoft 365 cloud services, Microsoft Intune device management, and point-of-sale hardware — and provides working code patterns, policy templates, and integration guides throughout.

The series is structured as a primary research paper accompanied by supplementary implementation chapters. Each chapter builds on the last; reading the primary paper first is recommended.

---

## Papers in This Series

### 1. [ITIL Version 5 and the Active Service Desk](ITILV5-active-service-desk-paper.md)

**Type:** Primary Research Paper  
**Audience:** Service desk managers, ITSM practitioners, IT leadership

The foundational paper of this series. Examines how ITIL v5's new constructs — the **Product and Service Lifecycle Model (PSLM)**, **Experience Level Agreements (XLAs)**, and the **6C AI Governance Model** — apply to an active service desk responsible for hardware, peripheral, and connectivity support. Proposes the **Integrated Active Service Desk (IASD)** model.

**Key topics covered:**
- ITIL v5 vs ITIL 4: key changes for service desk operations
- The Monitor, Support and Fulfil (MSF) practice cluster
- Customer success health scoring applied to endpoint management
- Four-layer proactive monitoring architecture
- Automation Maturity Ladder (Detect → Notify → Remediate → Optimise)
- IASD operational modes and end-to-end workflow
- XLA measurement framework
- AI governance and change management
- Implementation roadmap (Phases 1–4)

---

### 2. [Practical Custom Software for Recurring Issue Tracking](ITIL_v5-Custom-Software-Implementation.md)

**Type:** Supplementary Implementation Chapter  
**Audience:** IT engineers, developers, service desk technical leads

Covers purpose-built custom software for the three highest-volume recurring issue categories. Provides working code patterns in Python, PowerShell, and C#, with architecture diagrams and ITIL v5 integration touchpoints.

**Key topics covered:**
- **Printer Log Aggregator (PLA):** Windows PrintService event log collection, SNMP toner monitoring, recurrence detection, auto-ticket generation
- **POS Hardware Monitoring System (PHMS):** OPOS/JavaPOS driver status monitoring, terminal health scoring, PCI DSS compliance considerations
- **M365 Issue Tracker (MIT):** Microsoft Graph API integration (MSAL authentication), MFA storm detection, service health auto-correlation, safe auto-remediation actions
- Open-source ITSM platform evaluation (GLPI, iTop) and unified API integration patterns

**Technologies:** Python, PowerShell, C#/.NET, SQLite/PostgreSQL, MSAL, pysnmp, Flask, FastAPI

---

### 3. [Jira Service Management & Microsoft Intune Integration](ITILV5-Jira-Intune-Integration.md)

**Type:** Supplementary Implementation Chapter  
**Audience:** IT engineers, Jira administrators, Intune administrators

Connects the custom monitoring tools from Chapter 2 to Jira Service Management (JSM) as the ITSM platform. Covers Microsoft Intune as a remediation platform, desktop office environment monitoring via Windows Event Collector, and security considerations for all integrations.

**Key topics covered:**
- JSM project structure and ITIL v5 MSF mapping
- JSM REST API client with Atlassian Document Format (ADF) helpers
- Raising enriched incidents and problem records from PLA, PHMS, and MIT
- JSM Automation rules for IASD workflows (auto-assign, escalation, vendor hold, auto-close)
- **Microsoft Intune Remediations:** print spooler, OneDrive sync, NIC driver health scripts
- Intune compliance policy alignment with IASD health scoring
- Graph API polling for Intune remediation outcomes and non-compliant devices
- **Desktop office environment monitoring** via Windows Event Forwarding (WEF) and WEC
- Security: credential management, audit trails, scope limitation

**Technologies:** Python, PowerShell, JSM REST API, Microsoft Intune (Graph API), Windows Event Collector/Forwarding

---

### 4. [ITIL v5 Change Management, Policy Alignment & Implementation Phases](ITILV5-Change-Management-Policy-Alignment.md)

**Type:** Supplementary Implementation Chapter  
**Audience:** ITSM managers, change managers, IT directors, compliance leads

Addresses the organisational and governance dimension of an ITIL v5 IASD transition. Provides a structured framework for comparing existing policies against ITIL v5 requirements, a change management workflow aligned with ITIL v5 Change Enablement, and a detailed phased implementation roadmap.

**Key topics covered:**
- ITIL 4 → ITIL v5 structural changes and their policy implications
- Policy Gap Analysis Framework with scoring matrix
- Sample gap analysis for a typical ITIL 4 organisation
- Policy document structure template for ITIL v5 alignment
- **Policy templates:** AI and Automation Governance Policy; XLA Policy and Measurement Guide
- Change management workflows in JSM: Standard, Normal, and Emergency change types
- Standard Change Register for IASD automated actions (template)
- **Phased implementation roadmap with policy and governance gates (Phases 0–4)**
- Managing organisational resistance; communication plan
- ITIL v5 certification pathways (MSF, PIC, SVX) and timing within implementation phases
- Appendices: Policy Gap Analysis Worksheet; Standard Change Register Template; IASD CAB Sub-Committee Terms of Reference

---

## Quick Reference: How the Series Fits Together

```
Research Paper (Paper 1)
│   ITIL v5 framework · IASD model · XLA · 6C governance · roadmap
│
├── Custom Software (Paper 2)
│   Printer (PLA) · POS (PHMS) · M365 (MIT) · open-source ITSM integration
│
├── Jira & Intune Integration (Paper 3)
│   JSM API patterns · Intune Remediations · WEC desktop monitoring · security
│
└── Change Management & Policy (Paper 4)
    Gap analysis · policy templates · change workflows · phased roadmap · CAB governance
```

---

## Technologies Referenced

| Category | Technologies |
| --- | --- |
| **Languages** | Python, PowerShell, C#/.NET |
| **Device Management** | Microsoft Intune, Windows Event Collector/Forwarding, Group Policy |
| **Cloud Services** | Microsoft 365, Microsoft Graph API, Microsoft Entra ID (Azure AD), MSAL |
| **ITSM Platforms** | Jira Service Management, GLPI, iTop |
| **Monitoring Protocols** | SNMP (RFC 3805 Printer MIB), Windows Event Log, OPOS/JavaPOS |
| **Databases** | SQLite, PostgreSQL |
| **Web / API** | Flask, FastAPI, REST API, Webhooks, Atlassian Document Format (ADF) |
| **Dashboards** | Power BI, Grafana, JSM Dashboards |

---

## ITIL v5 Practices Covered

| ITIL v5 Practice | Papers Where Covered |
| --- | --- |
| Service Desk | 1, 2, 3, 4 |
| Incident Management | 1, 2, 3, 4 |
| Problem Management | 1, 2, 3, 4 |
| Monitoring & Event Management | 1, 2, 3 |
| Service Request Management | 1, 2 |
| Change Enablement (PIC module) | 1, 4 |
| Service Level Management / XLA | 1, 4 |
| AI Governance (6C model) | 1, 4 |
| IT Asset Management / CMDB | 2, 3, 4 |
| Supplier Management | 2 |

---

## Licence

See [LICENSE](LICENSE) for terms of use.
