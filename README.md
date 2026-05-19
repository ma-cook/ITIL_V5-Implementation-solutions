<div align="center">

# ITIL v5 Practical Solutions

### A Research and Implementation Series for the Modern IT Service Desk

**IT Service Management Practice Series** · May 2026

</div>

---

## About This Series

This repository contains a series of research papers and implementation guides exploring how **ITIL Version 5** (released February–May 2026) can be applied practically to IT service desk operations. The series focuses on on-site and remote environments - desktop office hardware, Microsoft 365 cloud services, Microsoft Intune device management, and point-of-sale hardware. It provides working code patterns, policy templates, and integration guides.

The series is structured as a foundation training document, a primary research paper, and supplementary implementation chapters. Each chapter builds on the last;

---

## Papers in This Series

### 1. [ITIL® (Version 5) Foundation Training Programme](ITIL_Version5_Training_Document.md)

**Type:** Foundation Training Document  
**Audience:** IT professionals, service desk staff, ITSM practitioners preparing for ITIL v5 certification

The first paper in this series and the recommended starting point for all readers. A structured learning guide covering the complete ITIL® (Version 5) Foundation syllabus, providing the conceptual grounding required to understand the applied papers that follow.

**Key topics covered:**
- What is ITIL (Version 5)? Evolution from ITIL 4; content composition (40% retained, 24% updated, 36% new)
- Certification pathway: Foundation, Bridge, Advanced modules, Managing Professional Transition
- The 8-Stage Product and Service Lifecycle Model (Discover → Design → Acquire → Build → Transition → Operate → Deliver → Support)
- The 6C AI Capability Model (Creation, Curation, Clarification, Cognition, Communication, Coordination)
- The 7 Guiding Principles (unchanged from ITIL 4)
- Core management practices: Incident, Problem, Change Enablement, Service Desk, and 8 others
- Foundation exam format, topic weighting, study path, and key terms glossary

---

### 2. [ITIL Version 5 and the Active Service Desk](ITILV5-active-service-desk-paper.md)

**Type:** Primary Research Paper  
**Audience:** Service desk managers, ITSM practitioners, IT leadership

The primary research paper of this series. Examines how ITIL v5's new constructs - the **Product and Service Lifecycle Model (PSLM)**, **Experience Level Agreements (XLAs)**, and the **6C AI Capability Model** - apply to an active service desk responsible for hardware, peripheral, and connectivity support. Proposes the **Integrated Active Service Desk (IASD)** model.

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

### 3. [Practical Custom Software for Recurring Issue Tracking](ITIL_v5-Custom-Software-Implementation.md)

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

### 4. [Jira Service Management & Microsoft Intune Integration](ITILV5-Jira-Intune-Integration.md)

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

### 5. [ITIL v5 Change Management, Policy Alignment & Implementation Phases](ITILV5-Change-Management-Policy-Alignment.md)

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

### 6. [Unified ITSM Web Platform: Architecture and Integration Guide](ITILV5-Unified-Platform-Architecture.md)

**Type:** Final Synthesis Paper  
**Audience:** IT architects, service desk managers, IT directors, senior engineers

The capstone paper of this series. Synthesises all preceding papers and proposes a **Unified ITSM Web Platform (UIWP)** — a custom-built web application that acts as a single operational pane of glass, consolidating data from every tool and service described in the series. Covers which services are suitable for data routing, iframe embedding, or webhook push, and specifies the full system architecture.

**Key topics covered:**
- Service-by-service integration suitability assessment (JSM, Intune, M365, Graph API, PLA, PHMS, MIT, WEC, WLAN controllers, RADIUS/NPS, SIEM/Grafana)
- Integration modes explained: API pull, webhook push, iframe embedding — and why most enterprise portals block embedding
- Full system architecture: presentation tier (React/TypeScript), application tier (Node.js/FastAPI), data tier (PostgreSQL, Redis, Elasticsearch), integration layer
- Caching strategy, WebSocket real-time updates, webhook receiver design
- Credential management and network security architecture
- Key UI views: Device Context Panel, Printer Fleet Health, POS Operations View, XLA Reports, Change Register, Endpoint Health Heatmap
- Illustrated user journeys: device investigation, printer fleet morning check, POS incident response
- **Four-phase implementation plan** aligned to IASD deployment phases
- ITIL v5 governance alignment: UIWP as a PSLM-managed service; 6C AI Capability Model compliance for UIWP-initiated actions; data governance and DPIA obligations
- Full technology stack recommendation with rationale

**Technologies:** React, TypeScript, Node.js, FastAPI, PostgreSQL, Redis, Elasticsearch, Grafana, Docker/Kubernetes, Azure Key Vault, Entra ID SSO

---

## Quick Reference: How the Series Fits Together

```
Foundation Training (Paper 1)
│   ITIL v5 concepts · 8-stage lifecycle · 6C AI Capability Model · practices · exam guide
│
Research Paper (Paper 2)
│   ITIL v5 framework · IASD model · XLA · 6C AI Capability · roadmap
│
├── Custom Software (Paper 3)
│   Printer (PLA) · POS (PHMS) · M365 (MIT) · open-source ITSM integration
│
├── Jira & Intune Integration (Paper 4)
│   JSM API patterns · Intune Remediations · WEC desktop monitoring · security
│
├── Change Management & Policy (Paper 5)
│   Gap analysis · policy templates · change workflows · phased roadmap · CAB governance
│
└── Unified Platform Architecture (Paper 6)  ← Final Synthesis
    Integration assessment · system architecture · UI views · implementation phasing
```

---

## Technologies Referenced

| Category | Technologies |
| --- | --- |
| **Languages** | Python, PowerShell, C#/.NET, TypeScript |
| **Device Management** | Microsoft Intune, Windows Event Collector/Forwarding, Group Policy |
| **Cloud Services** | Microsoft 365, Microsoft Graph API, Microsoft Entra ID (Azure AD), MSAL, Azure Key Vault |
| **ITSM Platforms** | Jira Service Management, GLPI, iTop |
| **Monitoring Protocols** | SNMP (RFC 3805 Printer MIB), Windows Event Log, OPOS/JavaPOS |
| **Databases** | SQLite, PostgreSQL, Redis, Elasticsearch / OpenSearch |
| **Web / API** | React, Node.js, FastAPI, Flask, REST API, Webhooks, WebSockets, Atlassian Document Format (ADF) |
| **Dashboards** | Power BI, Grafana, JSM Dashboards |
| **Infrastructure** | Docker, Kubernetes, Azure Container Apps, nginx |

---

## ITIL v5 Practices Covered

| ITIL v5 Practice | Papers Where Covered |
| --- | --- |
| Service Desk | 1, 2, 3, 4, 5, 6 |
| Incident Management | 1, 2, 3, 4, 5, 6 |
| Problem Management | 1, 2, 3, 4, 5, 6 |
| Monitoring & Event Management | 1, 2, 3, 4, 6 |
| Service Request Management | 1, 2, 3, 6 |
| Change Enablement (PIC module) | 1, 2, 5, 6 |
| Service Level Management / XLA | 1, 2, 5, 6 |
| AI Governance (6C AI Capability Model) | 1, 2, 5, 6 |
| IT Asset Management / CMDB | 3, 4, 5, 6 |
| Supplier Management | 3 |
| 8-Stage Product and Service Lifecycle (PSLM) | 1, 2, 6 |

---

## Licence

See [LICENSE](LICENSE) for terms of use.
