# Final Research Paper: Unified Web Platform for IT Service Management

**Repository Synthesis Paper**  
**Date:** 2026-05-17  
**Series Context:** Integrative synthesis of all papers in this repository

---

## Abstract

This paper presents a unified technical and operating vision for consolidating service desk, endpoint, cloud productivity, and field-device operational data into a custom-built web platform. It synthesises findings from four repository papers: **ITILV5-active-service-desk-paper.md**, **ITIL_v5-Custom-Software-Implementation.md**, **ITILV5-Jira-Intune-Integration.md**, and **ITILV5-Change-Management-Policy-Alignment.md**. Together, those papers establish the ITIL v5-driven Integrated Active Service Desk (IASD) model, practical custom telemetry and recurrence tooling (PLA, PHMS, MIT), enterprise integration patterns for Jira Service Management and Microsoft Intune, and policy/change governance required for sustainable rollout.

The synthesis in this final paper answers a central implementation question: *which services and tools can route or embed their data into a unified web platform, by which methods, and at what complexity?* The findings show that most ecosystem components expose integration surfaces through REST APIs, Graph APIs, webhooks, event forwarding, and script-based agents. Service management platforms (Jira Service Management, GLPI, iTop) provide incident/problem/change APIs and workflow engines. Microsoft Intune and Microsoft Graph provide device compliance, remediation state, service health, and identity telemetry. Endpoint and network sources provide event/log telemetry through Windows Event Forwarding, SNMP, and platform-native logs.

A reference architecture is proposed using a federated connector model: role-based web UI, API gateway and identity broker, connector/adaptor layer, normalisation and correlation services, XLA/ITIL analytics, and governed automation orchestration. The roadmap recommends phased implementation from baseline data unification through proactive engagement and approved auto-remediation to AI-assisted optimisation under ITIL v5 6C governance. Expected outcomes include improved mean time to detect, reduced recurrence, stronger first-contact resolution, and improved user effort scores (XLA), while maintaining policy traceability, security boundaries, and auditability.

---

## 1. Introduction

### 1.1 Background and problem statement

Modern service environments are fragmented by design. IT support teams operate across multiple control planes: ITSM ticketing systems, endpoint management platforms, identity providers, cloud productivity suites, network telemetry tools, and local custom scripts. While each tool is valuable, operational fragmentation causes recurring issues:

- duplicate triage and re-keying of context between systems,
- delayed root-cause recognition for cross-domain incidents,
- inconsistent governance of automated actions,
- weak end-user experience visibility beyond SLA timing,
- limited closed-loop learning from support data to service design.

The papers in this repository argue for a proactive ITIL v5 operating model that merges monitoring, support, fulfilment, and governed remediation. This final paper translates that body of work into a unified web platform blueprint capable of ingesting, embedding, and orchestrating data from those tools into one operational plane.

### 1.2 Motivation for unification

The practical motivation is not “replace every tool,” but “unify decision-making.” A custom platform should:

1. Present a consolidated operational view for service desk, engineering, and management.
2. Preserve specialist systems of record (JSM, Intune, Graph, CMDB) while abstracting their complexity.
3. Enable event-to-incident-to-problem flow with recurrence intelligence.
4. Surface ITIL v5 and XLA outcomes in one place.
5. Enforce policy and change governance for automation.

### 1.3 Scope

This paper focuses on service and data domains covered in the repository:

- Service desk and ITSM workflow data,
- Endpoint compliance/remediation and device telemetry,
- Custom software telemetry pipelines for printer, POS, and M365 patterns,
- Desktop environment event monitoring,
- Identity and access controls relevant to multi-tool integration,
- Policy, risk, and change governance for automated operations.

---

## 2. Review of Previous Research

This section cross-references repository papers and identifies what each contributes to a unified platform design.

### 2.1 Paper 1: `ITILV5-active-service-desk-paper.md`

The foundational paper introduces the **Integrated Active Service Desk (IASD)** model and frames ITIL v5 as the governing architecture. Major contributions include:

- **PSLM lifecycle loop** (Discover → Design → Transition → Support → Improve) as the structural basis for support data feeding service improvement.
- **MSF practice convergence** (Service Desk, Incident, Problem, Monitoring & Event, Service Request).
- **XLA metrics** (user effort, disruption, recurrence, proactive resolution) as core performance dimensions.
- **Automation Maturity Ladder** (Detect, Notify, Safe Remediate, Complex Remediate).
- **6C AI governance** for accountable automation.

Unified platform implication: architecture must support lifecycle feedback, not just dashboarding. Data models should represent event, incident, problem, change, and experience outcomes in linked form.

### 2.2 Paper 2: `ITIL_v5-Custom-Software-Implementation.md`

This paper provides concrete integration-ready telemetry and remediation building blocks:

- **PLA (Printer Log Aggregator):** Windows print events + SNMP status + recurrence detection.
- **PHMS (POS Hardware Monitoring System):** OPOS/JavaPOS and terminal health scoring with recurrence escalation.
- **MIT (M365 Issue Tracker):** Graph API polling, anomaly detection (e.g., MFA storms), service health correlation.
- **Open-source ITSM integration patterns:** reusable REST client patterns and CI linking.

Unified platform implication: custom tools already produce structured outputs that can be absorbed through connectors and normalisation pipelines. These tools are ideal early integration candidates because they were designed around machine-readable records.

### 2.3 Paper 3: `ITILV5-Jira-Intune-Integration.md`

This implementation chapter operationalises enterprise integration patterns:

- **JSM API + ADF patterns** for rich automated issue creation and linking.
- **JSM automation rules** for routing, escalation, duplicate linking, vendor hold, and closure logic.
- **Microsoft Intune remediations** with detection/remediation script pairs.
- **Graph polling for remediation failure and non-compliance detection**.
- **WEC/WEF desktop event collection** and processing for high-volume endpoint faults.
- **Security guidance** (secrets, least privilege, script signing, audit trail).

Unified platform implication: bidirectional orchestration is feasible—events from endpoints/Graph can create ITSM actions; ITSM states can drive updates back to user-facing dashboards and automation governance views.

### 2.4 Paper 4: `ITILV5-Change-Management-Policy-Alignment.md`

This governance-focused paper addresses adoption risk and control design:

- **Policy gap analysis framework** for ITIL 4 → ITIL v5 transition.
- **Templates for AI/automation governance and XLA policy**.
- **Standard/Normal/Emergency change patterns** for automated actions.
- **Phase model with governance gates** from baseline through AI optimisation.
- **CAB sub-committee terms** for recurring automation oversight.

Unified platform implication: technical unification must embed compliance metadata and change controls. A modern platform that does not expose audit lineage and policy adherence will fail operationally even if data integration succeeds.

### 2.5 Synthesis summary

Across all papers, a clear architecture principle emerges:

> **Retain distributed systems of execution; centralise intelligence, governance visibility, and user-facing operational context.**

This is the design center for the unified web platform proposed below.

---

## 3. Candidate Services and Tools for Integration

The table below assesses data exposure, integration method, and feasibility for each major tool/service in the paper series.

| Service / Tool | Data / Capability Exposed | Integration Method(s) | Complexity | Feasibility Notes |
| --- | --- | --- | --- | --- |
| Jira Service Management (JSM) | Incidents, Problems, Changes, comments, statuses, SLA data, automation triggers | REST API, webhook triggers, dashboard embedding (links/iframes where policy allows), OAuth/API token auth | Medium | Strong fit as ITSM system of record; mature API and automation engine |
| GLPI / iTop | Tickets, Problems, CMDB/CI data, categories, users | REST API ingestion/push, CI synchronization | Medium | Good alternative where JSM unavailable; CI-centric workflows useful for asset correlation |
| Microsoft Intune | Device inventory, compliance state, remediation scripts, update posture | Graph API polling, event-driven checks, remediation status ingestion | Medium-High | Requires app registration and permission governance; highly valuable for endpoint actionability |
| Microsoft Graph (M365/Entra) | Service health incidents, sign-in activity, licence details, call quality metadata | Graph API polling, scheduled ETL, anomaly detectors | Medium | High value for M365 issue correlation and proactive support |
| PLA (custom) | Printer events, SNMP states, recurrence counts | Agent/data-push API, file/queue ingestion, DB connector | Low-Medium | Structured, purpose-built data source; quick-win connector |
| PHMS (custom) | POS terminal health score, OPOS status events, recurrence outputs | API ingestion, webhook callbacks, dashboard embed | Medium | Strong business value in retail/hospitality; needs PCI-aware logging boundaries |
| MIT (custom) | M365 anomaly detections, incident correlation metadata, remediation recommendations | API ingestion, scheduled sync, webhook updates | Medium | Useful orchestration bridge between Graph and ITSM workflows |
| Windows Event Collector/Forwarding (WEC/WEF) | Forwarded endpoint events (USB, spooler, crash, profile, NIC) | Event subscriber service, parser pipeline, threshold engine | Medium | Scalable desktop visibility; requires event schema normalisation |
| SNMP polling (printer/network) | Device state, consumables, status and error bits | Poller integration, trap receiver, metric normaliser | Medium | Mature but heterogeneous device OIDs need catalog mapping |
| Entra ID / SSO | Identity, groups, role claims, sign-in policies | OIDC/SAML federation, JWT claim mapping | Medium | Critical for RBAC and least-privilege access to unified portal |
| Power BI / Grafana | Visualisation layer for analytics | Embedded dashboards, API-backed data sources | Low-Medium | Useful for executive/ops reporting while platform matures |
| Teams / Slack | Operational notifications | Webhooks / bot integration | Low | Fast route for proactive outreach and escalations |

### 3.1 Integration pattern categories

From the repository evidence, the most practical integration categories are:

1. **REST/Graph API ingestion** (structured state and transactional records).
2. **Webhook/event ingestion** (low-latency trigger pathways).
3. **Agent-based push** from custom tooling and endpoint monitors.
4. **Embedded visual context** (dashboards/portals where direct API build-out is unnecessary in phase 1).
5. **Identity federation** to unify role-based user experience across tools.

### 3.2 Feasibility ranking for phase sequencing

- **Phase 1 quick wins:** JSM read/write integration, Graph service health ingestion, PLA/MIT connectors.
- **Phase 2 operational expansion:** Intune remediation and compliance, WEC desktop event ingestion.
- **Phase 3 advanced correlation:** PHMS cross-site recurrence, CI-level risk scoring, predictive lifecycle inputs.

---

## 4. Unified Web Platform Architecture

### 4.1 Architecture goals

The architecture must satisfy five goals:

- Consolidate multi-system operational context without replacing systems of record.
- Support both real-time event handling and scheduled data sync.
- Enforce identity, role, and action-level governance.
- Preserve audit lineage from source event to platform decision and downstream action.
- Produce ITIL v5 and XLA-aligned analytics for operational and executive roles.

### 4.2 High-level architecture diagram (Mermaid)

```mermaid
flowchart LR
  subgraph Sources[Source Systems]
    JSM[Jira Service Management / GLPI / iTop]
    INTUNE[Microsoft Intune]
    GRAPH[Microsoft Graph / M365 / Entra]
    PLA[PLA Printer Tool]
    PHMS[PHMS POS Tool]
    MIT[MIT M365 Tool]
    WEC[WEC/WEF Event Stream]
    SNMP[SNMP Pollers]
  end

  subgraph Integration[Integration & Middleware]
    APIGW[API Gateway]
    CONN[Connector/Adapter Layer]
    BUS[Event Bus / Queue]
    NORM[Normalizer + Correlation Engine]
    RULES[Policy/Rule Engine + Change Guardrails]
  end

  subgraph Data[Data Services]
    ODS[Operational Data Store]
    TS[Time-Series / Metrics Store]
    CACHE[Cache Layer]
    AUDIT[Audit & Lineage Store]
  end

  subgraph App[Unified Web Platform]
    UI[Role-Based Web UI]
    OPS[Ops Console]
    XLA[XLA & ITIL Analytics]
    AUTO[Automation Orchestration Panel]
  end

  subgraph IAM[Identity & Access]
    SSO[SSO (OIDC/SAML)]
    RBAC[RBAC + ABAC Policy]
    SECRETS[Secrets Vault]
  end

  JSM --> APIGW
  INTUNE --> APIGW
  GRAPH --> APIGW
  PLA --> CONN
  PHMS --> CONN
  MIT --> CONN
  WEC --> BUS
  SNMP --> BUS

  APIGW --> CONN
  CONN --> NORM
  BUS --> NORM
  NORM --> RULES
  NORM --> ODS
  NORM --> TS
  RULES --> AUDIT
  RULES --> JSM
  RULES --> INTUNE

  ODS --> UI
  TS --> XLA
  CACHE --> UI
  AUDIT --> OPS

  SSO --> UI
  RBAC --> UI
  SECRETS --> CONN
```

### 4.3 Frontend layer (dashboard, portal, role-based views)

The frontend should provide distinct role-oriented views over a shared data model:

- **Service Desk Analyst View:** active incidents, correlated source events, suggested diagnostics/remediations, linked problems.
- **Engineering View:** connector health, failed automations, recurrence hotspots, script efficacy.
- **Manager/Leadership View:** XLA trends, proactive resolution rate, recurrence reduction, risk/compliance indicators.
- **CAB/Governance View:** automation change register status, 6C checklist adherence, exceptions and emergency actions.

### 4.4 Backend and middleware

Core middleware services:

- **API Gateway:** request authentication, rate limiting, schema enforcement.
- **Connector/adaptor services:** source-specific clients for JSM/GLPI/Graph/Intune/WEC/SNMP/custom tools.
- **Normalisation service:** maps source-specific payloads to canonical event/incident/problem/change schema.
- **Correlation engine:** links events by CI, user, service, location, timeframe, and recurrence signatures.
- **Policy/rule engine:** evaluates conditions for auto-ticketing, escalation, remediation eligibility, and suppression.

### 4.5 Identity and access management

Identity must be federated, not duplicated:

- SSO via Entra ID or equivalent IdP.
- Claims-based RBAC (e.g., ServiceDesk.Agent, Ops.Manager, CAB.Reviewer, Security.Auditor).
- Fine-grained permissions for action controls (view-only vs execute remediation vs approve automation).
- Just-in-time elevated actions for sensitive workflows.

### 4.6 Data storage and caching strategy

A hybrid storage pattern is recommended:

- **Operational store (relational):** incidents, problems, mappings, CI links, workflow states.
- **Time-series store:** telemetry metrics, remediation success rates, XLA trend lines.
- **Audit store (append-only):** action lineage, policy decisions, actor context, timestamps.
- **Cache layer:** low-latency dashboard reads and expensive query result caching.

Key principle: treat external ITSM and endpoint tools as source-of-truth for their native domains while maintaining a canonical read model for cross-system analytics and orchestration.

### 4.7 Connector/adaptor strategy per tool

- **JSM/GLPI/iTop connectors:** bidirectional (ingest state + create/update records).
- **Intune/Graph connectors:** mostly pull with controlled write actions for approved remediations.
- **PLA/PHMS/MIT connectors:** event push + periodic state snapshots.
- **WEC/SNMP connectors:** stream or micro-batch processing with thresholding.

### 4.8 Security considerations

Aligned to repository guidance:

- Centralized secret management; no hardcoded credentials.
- Least-privilege service accounts and Graph scopes.
- Script signing and controlled rollout for remediation scripts.
- Webhook rotation and inbound signature validation where possible.
- Tenant, device-group, and action-scope boundaries to prevent blast radius.
- Full audit trail tying each automated action to policy basis and change classification.

---

## 5. Implementation Roadmap

A phased roadmap should align technical rollout with policy maturity.

### Phase 0: Assessment and baseline (Weeks 1–4)

- Inventory tool landscape, integrations, and data availability.
- Define canonical event/incident/problem/change schema.
- Build policy gap baseline (AI governance, XLA, change enablement).
- Confirm initial KPIs: MTTD, MTTR, recurrence, FCR, user effort.

### Phase 1: Foundation (Months 1–3)

- Implement SSO and RBAC framework in the platform.
- Build connectors for JSM + Graph + one custom source (PLA or MIT).
- Deliver first unified dashboard (read-only plus deep links).
- Establish governance board/cadence for automation approvals.

### Phase 2: Proactive engagement (Months 4–6)

- Add Intune compliance/remediation telemetry.
- Integrate WEC desktop event pipeline and recurrence tagging.
- Enable proactive notification workflows for threshold events.
- Begin XLA survey and dashboard reporting in unified portal.

### Phase 3: Governed automation (Months 7–12)

- Implement rule engine pathways for approved safe remediations.
- Enforce standard-change registry checks before automation execution.
- Add PHMS and expanded SNMP/device health integrations.
- Activate auto-linking event→incident→problem with dedupe logic.

### Phase 4: Optimisation and AI-assisted operations (Months 13–24)

- Add predictive models for recurrence and lifecycle risk.
- Improve prioritisation with confidence scoring and explainability.
- Expand governance analytics (6C compliance scorecards).
- Feed support insights into Discover/Design procurement decisions.

---

## 6. Benefits and Business Value

### 6.1 Operational benefits

- Faster triage through unified context and pre-enriched incidents.
- Reduced duplicated effort across desk, endpoint, and cloud teams.
- Higher proactive detection and lower user-reported disruption.
- Better recurrence control through automated pattern recognition.

### 6.2 User and experience outcomes (XLA)

Expected directional improvements from repository baselines and projections:

- Higher **User Effort Score** via fewer handoffs/repeat diagnostics.
- Lower **Disruption Index** due to earlier detection and safe automation.
- Higher **First-Contact Resolution** through context-rich tickets.
- Lower **30-day recurrence rate** through problem-level interventions.

### 6.3 Strategic and governance value

- Creates measurable ITIL v5 alignment across MSF and Change Enablement.
- Improves audit posture with decision/action lineage.
- Strengthens confidence in automation through transparent controls.
- Enables evidence-based procurement and lifecycle management via PSLM feedback.

### 6.4 Financial and productivity impact

Though exact figures are environment-specific, value generally appears in:

- reclaimed technician hours from reduced repetitive diagnostics,
- reduced downtime in endpoint-heavy and POS-critical environments,
- improved service quality without linear staffing growth,
- lower operational risk from uncontrolled scripting/tool sprawl.

---

## 7. Risks and Mitigations

| Risk | Description | Mitigation |
| --- | --- | --- |
| Integration fragility | APIs, payload schemas, or auth flows change over time | Versioned connectors, contract tests, fallback parsers, change monitoring |
| Automation misfire | Incorrect remediation affects many devices/users | Pre-approval gates, scoped rollout, canary deployment, emergency stop control, rollback playbooks |
| Data quality variance | Inconsistent field semantics across tools | Canonical schema, normalisation dictionary, source confidence scoring |
| Security overreach | Excessive permissions in service accounts/app registrations | Least privilege, periodic access review, segregated accounts per connector |
| Privacy/trust concerns | Perceived surveillance from endpoint telemetry | Transparent policy communication, data minimisation, retention controls |
| Alert fatigue | Excess noise from event ingestion | Correlation thresholds, deduplication, suppression windows, priority tuning |
| Governance lag | Technical rollout outpaces policy readiness | Phase gates tied to policy deliverables and CAB review |
| Vendor dependency | Critical workflow tied to one platform behavior | Abstracted adaptor layer and portable canonical models |

---

## 8. Conclusion

The repository’s four papers collectively demonstrate that a unified web platform for IT service operations is both technically feasible and operationally justified. The required ingredients already exist: ITSM APIs, endpoint management telemetry, Graph-accessible cloud signals, custom recurrence engines, and mature governance patterns. The challenge is orchestration discipline—combining these components under a coherent architecture and change model.

The proposed platform deliberately avoids a “rip-and-replace” strategy. Instead, it establishes a federated integration fabric where source systems continue execution in their domains while the unified platform provides shared intelligence, governance visibility, and role-based action surfaces. This approach aligns with ITIL v5 principles by creating closed-loop improvement between support activity and service design decisions.

Most importantly, the model is practical. Organizations can start with low-risk read and correlation use cases, then progressively adopt proactive engagement and approved remediation as policy maturity grows. When implemented in phases and governed through 6C-aligned controls, the unified platform can shift service operations from reactive ticket handling to proactive value delivery—improving user experience, reducing disruption, and strengthening confidence in automated IT service management.

---

## 9. References

### 9.1 Repository papers (primary synthesis sources)

1. **`ITILV5-active-service-desk-paper.md`** — *ITIL Version 5 and the Active Service Desk*.  
2. **`ITIL_v5-Custom-Software-Implementation.md`** — *Practical Custom Software for Recurring Issue Tracking*.  
3. **`ITILV5-Jira-Intune-Integration.md`** — *Jira Service Management & Microsoft Intune Integration*.  
4. **`ITILV5-Change-Management-Policy-Alignment.md`** — *ITIL v5 Change Management, Policy Alignment & Implementation Phases*.  
5. **`README.md`** — repository overview and cross-paper topic map.

### 9.2 Standards and vendor documentation cited across the series

6. PeopleCert (2026). *ITIL Foundation (Version 5).*  
7. PeopleCert (2026). *ITIL Practice Manager Modules (MSF, PIC, SVX).*  
8. Atlassian Documentation (Jira Service Management REST API and Automation).  
9. Microsoft Learn (Microsoft Intune documentation and Microsoft Graph API reference).  
10. ISO/IEC 20000-1:2018. *Information technology — Service management systems requirements.*

