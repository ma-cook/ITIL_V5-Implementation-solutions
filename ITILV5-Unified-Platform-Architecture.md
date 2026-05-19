<div align="center">

# Unified ITSM Web Platform: Architecture and Integration Guide

### Consolidating Service Desk Tooling, Endpoint Management, and Custom Monitoring into a Single Operational Interface

**FINAL SYNTHESIS PAPER**
*Capstone to the ITIL v5 Practical Solutions Series*

**May 2026** · IT Service Management Practice Series

</div>

---

## Abstract

> The preceding papers in this series established a complete **Integrated Active Service Desk (IASD)** model under ITIL v5: a proactive monitoring architecture, custom software implementations for printer, POS, and Microsoft 365 issue tracking, integration patterns for Jira Service Management (JSM) and Microsoft Intune, and a governed change management framework. Each paper introduced a new layer of tooling — endpoint agents, event forwarding, Graph API polling, SNMP monitoring, ITSM automation — that collectively spans multiple vendor platforms, administrative interfaces, and data sources.
>
> This paper addresses the natural consequence of that breadth: **operational fragmentation**. When technicians must navigate Jira for tickets, the Intune portal for device compliance, a bespoke PLA dashboard for printer health, a Power BI report for XLA metrics, and the Microsoft 365 admin centre for service health, the cognitive overhead undermines the productivity gains that each individual tool provides.
>
> This paper proposes and architecturally specifies a **Unified ITSM Web Platform (UIWP)** — a custom-built web application that acts as a single operational pane of glass, routing and embedding data from every tool and service described in this series. It covers: which services are suitable for **data routing** (API-pulled data displayed natively), which are suitable for **iframe embedding** (vendor UI rendered inside the platform), which require **webhook push** patterns, and how to design the underlying system architecture to support all three integration modes securely and sustainably under ITIL v5 governance.

**Keywords:** `unified platform` · `ITSM integration` · `single pane of glass` · `system architecture` · `Microsoft Intune` · `Jira Service Management` · `Microsoft Graph API` · `ITIL v5` · `web application` · `data aggregation` · `XLA dashboard`

---

## 1. Introduction

### 1.1 The Integration Problem

The IASD model, as fully specified across this series, draws on at minimum **twelve distinct data sources and platforms**:

| Platform / Tool | Primary Role in IASD |
| --- | --- |
| Jira Service Management (JSM) | Incident, problem, change, and request management |
| Microsoft Intune | Device compliance, MDM configuration, remediation scripts |
| Microsoft Entra ID (Azure AD) | Identity, MFA, Conditional Access, licence management |
| Microsoft 365 Service Health API | Tenant-level service incident correlation |
| Microsoft Graph API (M365 workloads) | Teams call quality, mailbox activity, sign-in data |
| Printer Log Aggregator (PLA) | Windows PrintService event log + SNMP printer polling |
| POS Hardware Monitoring System (PHMS) | POS terminal health scoring, OPOS fault detection |
| M365 Issue Tracker (MIT) | Graph API-based M365 issue pattern detection |
| Windows Event Collector (WEC) | Centralised Windows event log aggregation |
| WLAN Controller (Cisco/Aruba/UniFi) | Per-client Wi-Fi signal, authentication, roaming events |
| RADIUS / NPS | 802.1X EAP authentication logs |
| SIEM / Observability (optional) | Correlation engine, event enrichment, SNMP aggregation |

Operating across twelve platforms is sustainable for specialists. It is not sustainable for a service desk team operating at speed, responding to incidents across all of these domains simultaneously. Technicians lose context switching between tabs; dashboards in one tool do not reflect the state of another; and management reporting requires manual data extraction from multiple sources.

The goal of the Unified ITSM Web Platform (UIWP) is to resolve this without replacing any of the underlying tools — preserving their specialist capabilities while surfacing the data and actions that service desk staff actually need in one interface.

### 1.2 Design Philosophy

The UIWP is not an ITSM platform. It does not store tickets, manage changes, or replace any of the systems listed above. It is a **data aggregation and presentation layer** with three responsibilities:

1. **Surface** — Pull data from source systems via APIs and present it in a unified interface
2. **Contextualise** — Correlate data across sources to present a coherent picture (e.g., a device view that shows Intune compliance, JSM open tickets, WEC event log summary, and printer/M365 health on one screen)
3. **Act** — Provide direct-action shortcuts that call the appropriate source system API (raise a JSM incident, trigger an Intune remediation, open an M365 admin URL) without requiring the operator to navigate to the source system

> 💡 This philosophy of **consume-correlate-act** maps directly to ITIL v5's **Monitor, Support and Fulfil** practice cluster. The UIWP is the operational front-end that makes all MSF practices accessible from a single interface, while the underlying tools retain their authoritative role.

### 1.3 Scope of This Paper

This paper covers:

- A service-by-service assessment of integration suitability (data routing vs. embedding vs. push)
- The proposed UIWP system architecture (frontend, backend, data layer, integration layer)
- Authentication and security architecture
- Key UI views and their data sources
- Implementation phasing
- Governance alignment with ITIL v5 and the IASD change management framework

---

## 2. Integration Suitability Assessment

### 2.1 Integration Modes

Before assessing each service, it is important to distinguish the three integration modes available to the UIWP:

| Mode | Description | When to Use | Constraints |
| --- | --- | --- | --- |
| **Data Routing (API Pull)** | Backend calls source system REST/Graph API; data is stored or cached in UIWP and rendered natively | Full control over display; correlation across sources; supports aggregation | Requires valid API credentials for each source; subject to rate limits; data may be slightly stale (poll interval) |
| **Webhook Push** | Source system sends events to UIWP endpoint in real time | Low-latency alerts; high-volume events (e.g., per-event SNMP) | Source system must support outbound webhooks; requires a public or tunnelled endpoint |
| **iframe Embedding** | Vendor portal is rendered inside the UIWP using an HTML `<iframe>` or similar | Quick access to complex vendor UI without rebuilding it | Dependent on vendor allowing embedding (many enterprise portals block cross-origin framing via `X-Frame-Options: DENY`); session management is separate; limited correlation capability |

> ⚠️ Many major vendor admin portals — including Microsoft 365 Admin Centre and Jira Service Management — **explicitly block iframe embedding** via HTTP security headers. This is the most common misconception when planning unified portal builds. The integration strategy for each service must reflect what the vendor's portal actually permits.

### 2.2 Service-by-Service Integration Assessment

#### 2.2.1 Jira Service Management

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing (REST API v3) |
| **iframe Embedding** | ❌ Blocked — JSM sets `X-Frame-Options: SAMEORIGIN`; cannot be embedded |
| **Webhook Push** | ✅ Supported — JSM can send outbound webhooks on issue events |
| **Available Data** | Incidents, problems, change requests, service requests, SLA status, queue depth, open/closed counts, assignee, labels, custom fields, comments |
| **Actions via API** | Create issue, transition status, add comment, link issues, assign, update fields |
| **Rate Limits** | Atlassian Cloud: ~200 requests per minute per API token; use caching aggressively |
| **UIWP Use Cases** | Live ticket queue; per-device open incidents; problem record summary; XLA metric display; quick-raise incident form; auto-populated from device context |

**Integration approach:** The UIWP backend polls JSM on a configurable interval (default: 60 seconds) using the JSM REST API v3. Ticket data is cached in the UIWP's own database. JSM outbound webhooks push real-time issue-created and issue-updated events to the UIWP webhook receiver, providing near-real-time updates without continuous polling pressure.

#### 2.2.2 Microsoft Intune (via Microsoft Graph API)

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing (Microsoft Graph API) |
| **iframe Embedding** | ❌ Blocked — Microsoft Intune admin centre (`intune.microsoft.com`) blocks embedding |
| **Webhook Push** | ✅ Supported — Microsoft Graph Change Notifications (webhooks) for device compliance state changes |
| **Available Data** | Device inventory, compliance state, OS version, last check-in, assigned user, app deployment status, remediation script run states, BitLocker status, antivirus status |
| **Actions via API** | Trigger device sync, trigger remediation script run, retire/wipe device (high-risk; requires elevated permissions and UIWP-level approval gate), assign compliance policy |
| **Auth Model** | Entra ID App Registration; application permissions (device read/write); client credential flow |
| **UIWP Use Cases** | Device health panel; compliance dashboard; non-compliant device list with auto-raise; remediation script status monitor; per-device Intune data in device context view |

**Integration approach:** A registered Entra ID application with `DeviceManagementManagedDevices.Read.All` (minimum) and `DeviceManagementConfiguration.Read.All` scopes provides read access. Write actions (sync, remediation trigger) require `DeviceManagementManagedDevices.PrivilegedOperations.All`. Graph Change Notifications (subscriptions) deliver real-time compliance state changes to the UIWP webhook receiver for immediate dashboard updates.

#### 2.2.3 Microsoft 365 Service Health

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing (Graph API: `/admin/serviceAnnouncement`) |
| **iframe Embedding** | ❌ Blocked |
| **Webhook Push** | ✅ Graph Change Notifications available for service health |
| **Available Data** | Active service incidents per M365 workload (Exchange, Teams, SharePoint, OneDrive, Entra, Intune, etc.), incident severity, start time, estimated resolution, status updates, post-incident reviews |
| **Actions via API** | Read-only; no administrative actions available via this endpoint |
| **UIWP Use Cases** | Service health status banner at top of UI; per-workload health indicator; auto-correlation banner when M365 incidents are active; auto-suppress user ticket creation prompts when M365 incident explains the reported symptom |

**Integration approach:** Poll `/admin/serviceAnnouncement/healthOverviews` every 5 minutes. Active incidents trigger a banner in the UIWP alerting operators that certain user complaints may relate to a Microsoft-side event. This data feeds the JSM Rule 2 equivalent within the UIWP (display-level, not ticket-creation-level).

#### 2.2.4 Microsoft Graph API — M365 Workload Data (MIT)

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing (Microsoft Graph API; MIT acts as a microservice feeding the UIWP) |
| **Available Data** | Teams call quality (jitter, packet loss, latency per user/session), user sign-in activity, OneDrive sync state, mailbox activity, licence assignment status, MFA registration status, Conditional Access policy state |
| **Actions via API** | Revoke sign-in sessions, trigger licence re-assignment, initiate OneDrive reset via Intune, reset MFA methods (with appropriate permissions and approval gate) |
| **UIWP Use Cases** | Per-user M365 health panel (populated from MIT data); call quality heatmap by office location; licence anomaly alerts; MFA storm detection indicator; user context panel showing last 7 days of Graph data when viewing a related ticket |

**Integration approach:** The MIT (from the Custom Software chapter) operates as a background microservice. It writes its aggregated, normalised data to a shared PostgreSQL database that the UIWP backend reads. This avoids the UIWP having to duplicate MIT's polling logic. The UIWP exposes MIT data through its own API layer, enriching ticket views and device context panels.

#### 2.2.5 Printer Log Aggregator (PLA)

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing (PLA exposes a REST API; UIWP queries it) + Webhook Push (PLA posts recurrence alerts to UIWP) |
| **iframe Embedding** | ✅ **Conditionally possible** — the PLA's Flask dashboard is a custom-built application; it can be built to allow embedding if the UIWP and PLA share the same origin or the PLA omits restrictive `X-Frame-Options` headers. However, data routing is preferred for full correlation capability |
| **Available Data** | Per-printer error rate, error history, toner levels (SNMP), paper tray status, recurrence alerts, health score per printer, top-fault printers |
| **Actions via API** | Acknowledge alert, suppress printer from alerts (maintenance window), trigger ad-hoc SNMP poll |
| **UIWP Use Cases** | Printer fleet health panel; per-printer detail view; recurrence alert feed; toner/status widget; integration with JSM ticket view (show printer history when viewing a printer incident) |

**Integration approach:** Since the PLA is a custom application under the organisation's control, its REST API design should be built with UIWP consumption in mind from the start. A standardised internal API contract (OpenAPI specification) enables the UIWP to query printer data without coupling to PLA internals.

#### 2.2.6 POS Hardware Monitoring System (PHMS)

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing (PHMS REST API) + Webhook Push (real-time terminal fault events) |
| **iframe Embedding** | ✅ **Conditionally possible** — the PHMS React dashboard can be embedded if built to allow it; however, data routing into the UIWP provides deeper integration including correlation with JSM and Intune data for the same terminal |
| **Available Data** | Per-terminal health score, OPOS fault history, receipt printer status, barcode scanner miss rate, cash drawer anomalies, transaction decline rates, recurrence patterns, floor plan view data |
| **Actions via API** | Acknowledge fault, set terminal maintenance mode, trigger health re-evaluation |
| **UIWP Use Cases** | POS operations view; site-level terminal health; real-time fault feed for store operations teams; per-terminal detail with JSM open tickets in context; escalation shortcuts |

**Integration approach:** The PHMS API is the same pattern as PLA — custom application, REST API, webhook push for critical events. The UIWP aggregates POS data alongside JSM and Intune data to provide a unified per-site view accessible to both IT service desk and store operations managers.

#### 2.2.7 Windows Event Collector (WEC) / Event Log Data

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing — WEC collects events from all enrolled endpoints; a log processor (Elasticsearch/OpenSearch, or direct SQL store) makes them queryable by the UIWP |
| **Available Data** | Application crashes, NIC driver errors, USB device events, print spooler events, security events, BSOD/kernel dumps, DHCP client events, DNS failures — per device |
| **Actions** | Read-only from UIWP perspective; actions are dispatched via Intune |
| **UIWP Use Cases** | Per-device event timeline; recent error summary on device context panel; anomaly indicators in device health score; pre-populated diagnostic context when raising a JSM ticket from the UIWP |

**Integration approach:** The WEC processor writes normalised events to Elasticsearch (or OpenSearch/PostgreSQL). The UIWP backend queries this store for per-device event summaries, providing a rich event timeline in the device context panel without exposing raw log data to the main UI. Search and filtering capabilities allow technicians to investigate device-level events without leaving the platform.

#### 2.2.8 WLAN Controller (Cisco WLC / Aruba Central / Ubiquiti UniFi)

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing — WLAN controller APIs provide per-client metrics; some controllers support webhook/SNMP trap push |
| **iframe Embedding** | ⚠️ **Varies by vendor** — Ubiquiti UniFi allows some embedding; Aruba Central and Cisco WLC generally block it |
| **Available Data** | Per-client RSSI, SNR, retry rate, authentication state, roaming history, AP association, channel utilisation, interference data |
| **Actions via API** | Deauthenticate client (force roam), change SSID power level, view AP health — vendor-specific |
| **UIWP Use Cases** | Wi-Fi health panel per user/device; site-level RF health heatmap; alert when client RSSI drops below threshold; correlation between Wi-Fi degradation events and JSM connectivity tickets |

**Integration approach:** A lightweight wireless metrics microservice polls the WLAN controller API (REST for Aruba/UniFi; XML/REST for Cisco WLC) on a 5-minute interval. Per-client data is stored in time-series format (InfluxDB or Prometheus) and exposed to the UIWP via an internal metrics API. This microservice acts as an abstraction layer, making the UIWP controller-agnostic.

#### 2.2.9 RADIUS / NPS Logs

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing — NPS logs written to Windows Event Log (Event IDs 6272–6280) or to a SIEM; UIWP queries the log store |
| **Available Data** | 802.1X authentication attempts, success/failure per user and device, EAP type, certificate errors, reason codes for failures |
| **UIWP Use Cases** | Authentication failure indicator in device context panel; alert when a specific device or user has repeated 802.1X failures; correlate with Wi-Fi health data for comprehensive connectivity view |

**Integration approach:** NPS log events are forwarded to the WEC or directly to the SIEM. The UIWP queries authentication summary data via the same log query service used for WEC event data, filtered to authentication-relevant event IDs.

#### 2.2.10 SIEM / Observability Platform (Optional)

| Aspect | Detail |
| --- | --- |
| **Integration Mode** | ✅ Data Routing (Elasticsearch/OpenSearch API, Splunk API, Datadog API) — or iframe for some vendors |
| **iframe Embedding** | ⚠️ **Varies** — Grafana supports embedding via `allow_embedding = true`; Kibana can be embedded with appropriate configuration; enterprise SIEMs (Splunk, Sentinel) typically block it |
| **Available Data** | Correlated alerts, event volumes, anomaly detections, pre-built dashboards, trend data |
| **UIWP Use Cases** | Alert feed widget; event volume sparklines per device or site; anomaly indicator; link-out to full SIEM investigation view |

**Integration approach:** Grafana is the recommended observability front-end for the UIWP context because it natively supports embedded panels via signed URLs when `allow_embedding` is enabled. Individual Grafana panels (charts, gauges, alert lists) can be embedded as `<iframe>` components in the UIWP, providing rich visualisation without rebuilding charting logic. For SIEMs that block embedding, data is routed via their REST API and rendered natively.

---

## 3. Summary Integration Matrix

| Service / Tool | API Pull | Webhook Push | Embeddable | Primary UIWP Role |
| --- | :---: | :---: | :---: | --- |
| Jira Service Management | ✅ | ✅ | ❌ | Ticket management hub |
| Microsoft Intune | ✅ | ✅ | ❌ | Device compliance and remediation |
| M365 Service Health | ✅ | ✅ | ❌ | Incident correlation banner |
| Microsoft Graph (M365 workloads) | ✅ | ✅ | ❌ | Per-user/device M365 health |
| Printer Log Aggregator (PLA) | ✅ | ✅ | ⚠️ Conditional | Printer fleet health panel |
| POS Hardware Monitor (PHMS) | ✅ | ✅ | ⚠️ Conditional | POS operations view |
| Windows Event Collector (WEC) | ✅ | ❌ | ❌ | Per-device event timeline |
| WLAN Controller | ✅ | ⚠️ Vendor | ⚠️ Vendor | Wi-Fi health panel |
| RADIUS / NPS | ✅ | ❌ | ❌ | Authentication fault indicator |
| Grafana / Observability | ✅ | ❌ | ✅ | Embedded metric panels |
| SIEM (Splunk / Sentinel) | ✅ | ❌ | ❌ | Alert feed widget |

---

## 4. Proposed System Architecture

### 4.1 Architecture Overview

The UIWP follows a **three-tier architecture** with a dedicated integration layer:

```
┌─────────────────────────────────────────────────────────────────┐
│                     PRESENTATION TIER                           │
│         React (TypeScript) Single-Page Application              │
│   Dashboard · Ticket Views · Device Context · Alerts · Reports  │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS / WebSocket
┌────────────────────────────▼────────────────────────────────────┐
│                     APPLICATION TIER                            │
│              Node.js / Python (FastAPI) Backend                 │
│   REST API · WebSocket Gateway · Auth Middleware · Cache Layer  │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ JSM Service  │  │ Intune Svc   │  │ M365 / Graph Service │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  PLA Service │  │ PHMS Service │  │  WEC / Log Service   │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
│  ┌──────────────┐  ┌──────────────────────────────────────────┐ │
│  │ WLAN Service │  │       Webhook Receiver (inbound)         │ │
│  └──────────────┘  └──────────────────────────────────────────┘ │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                       DATA TIER                                 │
│                                                                 │
│   PostgreSQL          Redis              Elasticsearch /        │
│   (relational:        (cache:            OpenSearch             │
│    users, config,      API responses,    (event logs,           │
│    audit log,          health scores,    WEC events,            │
│    alert state)        session data)     WLAN time-series)      │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                   INTEGRATION LAYER                             │
│   (external services; all calls outbound from Application Tier) │
│                                                                 │
│  JSM REST API    │  Microsoft Graph    │  PLA REST API          │
│  Intune Graph    │  M365 Service Health│  PHMS REST API         │
│  WLAN Controller │  NPS/WEC Event Store│  SIEM API              │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Presentation Tier

**Technology:** React 18+ with TypeScript, using a component library (e.g., Ant Design, MUI, or Tailwind CSS with Shadcn/ui) for consistent UI primitives.

**Key architectural decisions:**

- **Single-Page Application (SPA):** Enables fast navigation between views without full page reloads; state management via Zustand or Redux Toolkit
- **WebSocket connection:** For real-time updates (webhook-triggered events pushed from backend to frontend without polling)
- **Role-based view rendering:** Components rendered based on user roles (e.g., Service Desk Agent, Desktop Engineer, Store Operations Manager, IT Management) — the same application serves different audiences with tailored views
- **Grafana panel embedding:** Select Grafana panels (metric charts, gauges) are embedded via signed `<iframe>` URLs in relevant views. The backend generates signed embed URLs using the Grafana API, keeping the signing key server-side

**Key UI views:**

| View | Data Sources | Primary Users |
| --- | --- | --- |
| **Operations Dashboard** | JSM queue depth, M365 health, Intune non-compliance count, open P1/P2 incidents, XLA score summary | Service Desk Managers |
| **Device Context Panel** | Intune compliance, JSM open tickets, WEC event summary, MIT M365 data, WLAN RSSI history | Service Desk Agents, Desktop Engineers |
| **Ticket Queue** | JSM incidents, problems, service requests (all projects) with unified filter/search | Service Desk Agents |
| **Printer Fleet Health** | PLA per-printer health scores, toner levels, recent errors, recurrence alerts | Desktop Engineers, Print Administrators |
| **POS Operations View** | PHMS terminal health, floor plan, live fault feed, estimated resolution times | Store Operations Managers, IT Engineers |
| **M365 Health Panel** | MIT data: Teams call quality, sign-in activity, service health per workload | M365 Administrators, Service Desk Agents |
| **XLA Reports** | JSM-derived metrics: UES scores, FCR, recurrence rate, automated remediation rate | IT Management, Service Desk Managers |
| **Change Register** | JSM change requests, ITIL v5 standard change register, 6C compliance status | Change Manager, IT Operations Lead |
| **Endpoint Health Heatmap** | Intune compliance, Intune remediation outcomes, WEC error rates, WLAN data | Desktop Engineering |

### 4.3 Application Tier

**Technology choices:**

- **Primary backend:** **Node.js (TypeScript) with Express or Fastify**, chosen for its event-driven architecture that aligns well with webhook-heavy, I/O-bound integration workloads. Python (FastAPI) is an equally valid alternative, particularly if the organisation's existing tooling (PLA, PHMS, MIT) is already Python-based and a single-language backend is preferred
- **WebSocket gateway:** Socket.IO (Node.js) or FastAPI WebSocket (Python) for pushing real-time updates from webhook events to connected browser clients
- **Task queue:** **Redis + BullMQ** (Node.js) or **Celery + Redis** (Python) for scheduled polling jobs, retry logic, and background processing of incoming webhook events
- **API gateway pattern:** Each integration is encapsulated in its own service module with a standardised interface:

```typescript
// Standardised integration service interface (TypeScript)
interface IntegrationService<T> {
  fetchData(params: QueryParams): Promise<T>;
  getLastUpdated(): Date;
  isHealthy(): Promise<boolean>;
}
```

This pattern allows individual integration services to be developed, tested, and replaced independently. If JSM is replaced with ServiceNow, only the JSM service module changes; all views consuming the normalised ticket data remain unchanged.

**Caching strategy:**

| Data Type | Cache TTL | Rationale |
| --- | :---: | --- |
| JSM ticket queue | 60 seconds | Supplemented by webhook push for real-time updates |
| Intune device compliance | 5 minutes | Graph change notifications provide real-time state changes |
| M365 service health | 5 minutes | Low change frequency; critical to cache to avoid polling pressure |
| Printer health (PLA) | 2 minutes | SNMP poll interval is 15 min; intermediate caching is sufficient |
| POS terminal health (PHMS) | 30 seconds | High operational sensitivity; lower TTL acceptable for POS context |
| WEC event summaries | 10 minutes | Events are historical; near-real-time is not required |
| WLAN client metrics | 5 minutes | Wireless conditions change quickly; shorter TTL for active investigations |
| XLA metrics | 1 hour | Report-frequency data; daily refresh is sufficient for dashboards |

**Webhook receiver:**

The UIWP exposes a dedicated `/webhooks` endpoint namespace. Each integration has its own path:

```
POST /webhooks/jsm          — JSM issue created/updated events
POST /webhooks/intune        — Graph change notifications (device compliance)
POST /webhooks/m365-health   — Graph change notifications (service health)
POST /webhooks/pla           — PLA recurrence alerts
POST /webhooks/phms          — PHMS terminal fault events
```

All webhook endpoints validate a shared secret or HMAC signature (per-source). Validated events are written to a Redis stream and processed asynchronously by the event processor, which updates the data tier and pushes WebSocket notifications to connected clients.

### 4.4 Data Tier

**PostgreSQL** (relational store):

- `users` — UIWP user accounts, roles, preferences
- `integration_config` — per-integration credentials (encrypted at rest), polling intervals, enabled state
- `alert_state` — current alert conditions, acknowledgement state, suppression windows
- `audit_log` — every action taken via the UIWP that calls a source system API (with user, timestamp, target, and result)
- `xla_cache` — pre-aggregated XLA metrics for dashboard performance
- `standard_change_register` — local mirror of ITIL v5 standard change register entries
- `device_context_cache` — normalised device records aggregating identifiers across Intune, JSM, PLA, PHMS

**Redis** (cache and message broker):

- Per-source API response caches (keyed by request parameters, TTL as specified above)
- WebSocket subscription registry (which clients are subscribed to which data streams)
- Task queues for polling jobs and webhook event processing
- Rate limit counters per source API

**Elasticsearch / OpenSearch** (event log and time-series store):

- WEC event log data (forwarded from Windows Event Collector log processor)
- WLAN client metric time-series (RSSI, SNR, retry rate per client per AP)
- NPS authentication event summaries
- Full-text search capability for event investigation within the UIWP

### 4.5 Integration Layer Security

**Credential management:**

All credentials for external services are stored in a secrets manager — **Azure Key Vault** (in Microsoft-centric environments) or **HashiCorp Vault** (cross-platform). The UIWP backend retrieves secrets at startup and rotates them on a schedule. No credentials are stored in the UIWP database or source code.

| Credential Type | Storage | Rotation |
| --- | --- | --- |
| Entra ID app client secret (Graph/Intune) | Azure Key Vault | 180 days (auto-rotate via Key Vault policy) |
| JSM API token | Secrets manager | 90 days (manual; alert 14 days before expiry) |
| PLA / PHMS API keys | Secrets manager | 180 days |
| Webhook shared secrets | Secrets manager | 365 days |
| WLAN controller API credentials | Secrets manager | 90 days |

**Network security:**

- The UIWP application tier runs in a dedicated network segment with outbound-only access to external APIs
- Inbound webhook traffic is terminated at an edge reverse proxy (nginx or Azure Application Gateway) with IP allowlisting for known source IP ranges (e.g., Atlassian IP ranges for JSM webhooks, Microsoft IP ranges for Graph notifications)
- All internal service-to-service communication uses mutual TLS (mTLS) in production deployments
- The UIWP is not exposed to the public internet; it is an internal tool accessible via VPN or corporate network only (or via Azure AD-protected Application Proxy for remote access)

**ITIL v5 6C AI Capability Model alignment for the UIWP itself:**

The UIWP, as an automation-adjacent platform, must itself be evaluated against the 6C AI Capability Model when it takes actions on behalf of users or surfaces AI-generated insights:

| 6C Dimension | UIWP Implementation |
| --- | --- |
| **Creation** | Phase 4 ML capabilities generate ticket categorisation suggestions, KB article recommendations, and predicted resolution approaches from aggregated UIWP data; AI-generated summaries in the device context panel |
| **Curation** | Data quality controls and staleness indicators on every panel; validation of source system data before display; governance review of AI-generated summaries before they influence technician decisions |
| **Clarification** | Cross-source correlation contextualises ambiguous situations (e.g., WLAN degradation + WEC NIC errors + JSM open ticket combined into a coherent device health narrative rather than three separate signals) |
| **Cognition** | Phase 4 predictive device health alerts based on trend analysis across multiple data sources; ML ticket classification for intelligent queue management; anomaly detection across correlated datasets |
| **Communication** | Every UIWP-initiated action presents a confirmation message stating which system was called and the outcome; users and technicians are explicitly informed when actions are automated vs. manually initiated |
| **Coordination** | UIWP orchestrates write actions across multiple systems (JSM, Intune) with RBAC gates, audit logging (`audit_log` table), and defined escalation paths when integration sources are unavailable |

> **Governance note:** The operational controls underpinning 6C-compliant UIWP operation — authenticated RBAC roles for all write actions, full audit logging of every API call, and scope limitation per action type — are specified in Section 4.5 (Integration Layer Security) and Section 7.2 (Change Management).

---

## 5. Key User Journeys

### 5.1 Device Context Investigation

> *A service desk agent receives a call from a user reporting intermittent connectivity and print failures.*

**Without the UIWP:** The agent opens JSM (check for open tickets), then opens Intune (check compliance), then opens the PLA dashboard (check printer history), then navigates to the WLAN controller (check signal), then checks the WEC log viewer. Total: 4–5 tabs, 10–15 minutes of context gathering.

**With the UIWP:**

1. Agent searches for the device by hostname or user in the UIWP search bar
2. The **Device Context Panel** opens, displaying in a single view:
   - Intune compliance state and last check-in time
   - Open JSM incidents linked to this device (with status and age)
   - WEC event summary: last 24 hours of error-level events (NIC, print, app crash)
   - PLA printer history: last 5 print events from this device's assigned printer(s)
   - WLAN RSSI history: signal strength over the last 4 hours (Grafana embedded panel)
   - MIT data: Teams call quality last 7 days, OneDrive sync state, licence status
3. Agent identifies from the combined view that WLAN RSSI dropped below −75 dBm three hours ago (correlating with the user's report), and that the Intune NIC driver remediation script ran and failed on this device yesterday
4. Agent clicks **Raise JSM Incident** (pre-populated with device name, user, WEC event summary, and WLAN data from context) — one click, not a manual form fill
5. Agent clicks **Trigger Intune Remediation: NIC Driver** — calls the Intune Graph API, confirmation shown in UIWP

Total time: 2–3 minutes. Context is gathered automatically; the agent focuses on diagnosis and communication rather than tool navigation.

### 5.2 Printer Fleet Morning Check

> *A desktop engineer starts their shift and needs to check printer fleet health across three office sites.*

1. Engineer opens the **Printer Fleet Health** view
2. UIWP displays a site-grouped table: each printer's health score (from PLA), toner level (SNMP), last error event, and open JSM incidents
3. Three printers are highlighted in amber (health score 45–60); one is red (score 28, paper tray empty, three print spooler errors in last 24 hours)
4. Engineer clicks the red printer: PLA event history, SNMP status, and linked JSM problem record all visible in the detail pane
5. Engineer clicks **Raise JSM Service Request: Consumables** (pre-populated for the empty paper tray) — one click
6. The existing JSM problem record for the spooler errors is shown; engineer adds a comment via the UIWP comment field (calls JSM comment API)

Total time: under 5 minutes for a full fleet health check that previously required navigating PLA dashboard, JSM, and manual email to the print team.

### 5.3 POS Incident Response (Store Operations View)

> *A store operations manager receives a UIWP push notification on their mobile-accessible browser: "Register 4 at Site B — health score dropped to 22. Receipt printer offline."*

1. Manager opens UIWP mobile view; the POS Operations view shows Site B's floor plan with Register 4 highlighted in red
2. Detail pane shows: receipt printer status `SUE_POWER_OFF`, two prior paper-out events today, open JSM P2 incident (auto-raised by PHMS), IT estimated resolution time: 45 minutes
3. Manager taps **Notify Store Supervisor** (pre-configured Teams webhook post) with the current status and ETA
4. Manager taps **Open Adjacent Lane** (informational action — adds a note to the JSM incident noting the operational workaround)

The store operations manager never had to log into JSM, navigate the PHMS dashboard, or call the IT service desk. The UIWP surfaces exactly the context needed for their role in their operational channel.

---

## 6. Implementation Phasing

The UIWP should be implemented in phases that align with the IASD deployment phases defined in the Change Management and Policy Alignment paper:

### Phase 1 — Foundation Views (Months 1–3)

Deploy the UIWP with read-only data routing from the highest-value sources:

- [ ] UIWP skeleton: authentication (SSO via Entra ID / SAML), RBAC, audit logging
- [ ] JSM integration: ticket queue view, open incidents per device
- [ ] Intune integration: device compliance list, per-device compliance panel
- [ ] M365 Service Health integration: service health banner
- [ ] Basic device search and context panel (JSM + Intune data only)
- [ ] Operations Dashboard with queue depth and compliance metrics

**Value delivered:** Technicians no longer need separate JSM and Intune tabs for the majority of device investigations.

### Phase 2 — Monitoring Data Integration (Months 4–6)

Add monitoring data sources as IASD monitoring infrastructure comes online:

- [ ] PLA integration: printer fleet health view; printer panel in device context
- [ ] WEC event log integration: per-device event timeline in device context panel
- [ ] MIT (M365 Issue Tracker) integration: per-user M365 health panel
- [ ] WLAN metrics integration: RSSI/SNR panel in device context (Grafana embed or API pull)
- [ ] Real-time WebSocket updates for alert events (PLA recurrence alerts, Intune non-compliance)
- [ ] JSM quick-raise form with pre-population from device context

**Value delivered:** Device investigation time reduced from 10–15 minutes to 2–3 minutes. Proactive alerts surfaced in real time.

### Phase 3 — Action Capabilities and POS View (Months 7–12)

Add write actions and the POS-specific view:

- [ ] UIWP-initiated Intune remediation triggers (with RBAC gate and audit log)
- [ ] UIWP-initiated JSM ticket creation, comment, and status transition
- [ ] PHMS integration: POS operations view; mobile-accessible store manager view
- [ ] Grafana panel embedding for metric visualisations
- [ ] XLA reporting view: UES scores, FCR, recurrence rate, automated remediation rate
- [ ] Standard Change Register view (local mirror from JSM)

**Value delivered:** Full IASD operational capability from a single interface. Store operations teams gain a tool appropriate to their role. XLA visibility without manual report extraction.

### Phase 4 — Advanced Analytics and AI Assist (Months 13–24)

- [ ] ML-based ticket classification integrated into the UIWP ticket queue (predicted category, suggested KB article, predicted resolution time)
- [ ] Predictive device health alerts (trend-based, not threshold-based)
- [ ] PSLM feedback loop dashboard: Support-stage data surfaced as Discover-stage inputs (hardware failure trends by model, vendor defect patterns)
- [ ] Teams / Slack bot interface: key UIWP operations accessible via chat commands
- [ ] Power BI or Confluence export integration for management reporting

---

## 7. Governance and ITIL v5 Alignment

### 7.1 The UIWP as an ITIL v5 Asset

The UIWP itself is a **service component** within the PSLM. It must be managed through the same 8-stage lifecycle model it helps to operate:

- **Discover:** Business case validated against technician productivity data (time-per-investigation baseline from Phase 0)
- **Design:** Architecture as specified in this paper; security model reviewed by security team; RBAC design reviewed with service desk manager and store operations lead
- **Acquire:** Hosting infrastructure, third-party component licences, and any contracted development resources sourced through the standard procurement process
- **Build:** UIWP developed and tested per phase plan; integration tests for each source system; acceptance criteria reviewed against original business case
- **Transition:** Phased rollout per Section 6; training materials published; Normal Change process in JSM for each phase activation
- **Operate:** UIWP in live service; incident management for integration outages; on-call runbook maintained
- **Deliver:** UIWP XLA tracking — does it make technicians faster? Does it reduce ticket handling time? Does it reduce cognitive overhead?
- **Support:** UIWP integration health dashboard (as described in Section 4.5); monthly integration health review; quarterly UX review with technician users

### 7.2 Change Management for the UIWP

Each new integration or write-action capability added to the UIWP should go through the ITIL v5 Normal Change process in JSM:

| UIWP Change Type | ITIL v5 Change Type | CAB Gate |
| --- | --- | --- |
| New read-only data integration | Normal Change (low risk) | Technical review; no full CAB |
| New write action (e.g., Intune remediation trigger) | Normal Change (medium risk) | Full CAB review; 6C checklist |
| New automation within the UIWP itself | Normal Change (medium/high risk) | Full CAB + IASD sub-committee |
| Emergency fix to a broken integration | Emergency Change | Head of IT Operations approval |

### 7.3 Data Governance

The UIWP aggregates data from multiple systems, some of which contain personal data (user sign-in activity, device location, call quality metrics). A **Data Processing Impact Assessment (DPIA)** should be conducted before Phase 1 deployment, covering:

- What personal data is processed by the UIWP (display only vs. stored in data tier)
- For data stored in the data tier: retention periods, access controls, deletion procedures
- Staff consent/notice obligations under the organisation's acceptable use policy (updated in IASD Phase 2)
- Processor agreements with cloud vendors used to host the UIWP

---

## 8. Technology Stack Summary

The following stack represents a practical, supportable choice for a mid-market organisation building the UIWP:

| Layer | Recommended Technology | Rationale |
| --- | --- | --- |
| **Frontend** | React 18 + TypeScript + Tailwind CSS | Widely supported; strong ecosystem; type safety reduces integration bugs |
| **Backend** | Node.js (TypeScript) + Fastify | Performant for I/O-bound integration workloads; same language as frontend; excellent async/WebSocket support |
| **Alternative backend** | Python 3.12 + FastAPI | Preferred if team's existing tools (PLA, PHMS, MIT) are Python and single-language stack is desired |
| **WebSocket** | Socket.IO (Node) or FastAPI WebSockets | Real-time event push to browser clients |
| **Task queue** | BullMQ + Redis (Node) or Celery + Redis (Python) | Reliable scheduled polling, retry logic, dead-letter queues |
| **Relational DB** | PostgreSQL 16 | Robust, widely supported; used by PLA/PHMS/MIT for consistency |
| **Cache / Message broker** | Redis 7 | Low latency; pub/sub for WebSocket fan-out; TTL-based API cache |
| **Event log store** | Elasticsearch 8 or OpenSearch 2 | Full-text search over WEC events; time-series for WLAN metrics |
| **Metrics visualisation** | Grafana 10 | Embeddable panels; supports multiple data sources; widely known |
| **Secrets management** | Azure Key Vault (Microsoft env) or HashiCorp Vault | Centralised secrets; rotation; audit trail |
| **Reverse proxy** | nginx or Azure Application Gateway | TLS termination; IP allowlisting for webhooks; rate limiting |
| **Container runtime** | Docker + Kubernetes (or Azure Container Apps) | Independent scaling per service module; consistent deployment |
| **CI/CD** | GitHub Actions | Integration test suite per service module; automated deployment on merge |
| **Authentication** | Entra ID (SAML 2.0 / OIDC) | SSO with existing corporate identity; MFA enforced at IdP |

---

## 9. Conclusion

The five preceding papers in this series established a complete framework for an **Integrated Active Service Desk** under ITIL v5: the foundation training that defines the core concepts, the theoretical model applied to an active service desk, the custom software implementations, the ITSM and device management integrations, and the change management governance that ties them together. The capability built across those papers is significant — but only if technicians can access it efficiently.

The **Unified ITSM Web Platform (UIWP)** proposed in this paper is the operational front-end that makes the full IASD capability accessible from a single, role-appropriate interface. Its design recognises the practical limits of vendor portal embedding (most enterprise portals block iframe embedding) and therefore prioritises **data routing via REST and Graph APIs** as the primary integration mode, supplemented by **webhook push** for real-time events and **selective Grafana panel embedding** for metric visualisations.

The architecture is deliberately **additive and replaceable**: no existing tool is removed; each integration is independently modular; and the platform grows through the same governed ITIL v5 change management process it is designed to support.

Implemented across the four phases described, the UIWP transforms the IASD from a collection of well-integrated specialist tools into a coherent operational platform — one where a service desk agent can move from alert to investigation to action in minutes rather than navigating a dozen separate systems, and where a store operations manager and an IT engineer can both use the same platform in ways appropriate to their respective roles and responsibilities.

---

## Appendix A — API Reference Quick Guide

| Service | Base URL Pattern | Auth Method | Key Docs |
| --- | --- | --- | --- |
| Jira Service Management | `https://{org}.atlassian.net/rest/api/3/` | HTTP Basic (email + API token) | developer.atlassian.com |
| Microsoft Graph | `https://graph.microsoft.com/v1.0/` | OAuth 2.0 Bearer (MSAL / client credentials) | learn.microsoft.com/graph |
| Intune (via Graph) | `https://graph.microsoft.com/v1.0/deviceManagement/` | OAuth 2.0 Bearer (same app reg as Graph) | learn.microsoft.com/intune |
| M365 Service Health | `https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/` | OAuth 2.0 Bearer | learn.microsoft.com/graph |
| Aruba Central | `https://apigw-{region}.central.arubanetworks.com/` | OAuth 2.0 Bearer | developer.arubanetworks.com |
| Ubiquiti UniFi | `https://{controller}:8443/api/` | Cookie-based session or API key | uisp.docs.apiary.io |
| Cisco WLC (9800) | `https://{wlc}/restconf/data/` | HTTP Basic or token | developer.cisco.com |
| Grafana | `https://{grafana}/api/` | API key or service account token | grafana.com/docs/grafana/api |

---

## Appendix B — Recommended Reading

- ITIL Version 5 Foundation Guide (Axelos, 2026)
- ITIL v5 Practice Manager: Monitor, Support and Fulfil (MSF) Module (Axelos, 2026)
- Microsoft Graph API documentation: `learn.microsoft.com/graph`
- Atlassian REST API documentation: `developer.atlassian.com/cloud/jira/service-desk/`
- OWASP Web Application Security Testing Guide (for UIWP security review)
- GDPR Article 35 (Data Protection Impact Assessments)

---

*This paper is the sixth and final entry in the ITIL v5 Practical Solutions series. It should be read in conjunction with the preceding five papers, which provide the conceptual foundation and detailed specifications for all tools, integrations, and policies referenced here.*

---

<div align="center">

**ITIL v5 Practical Solutions Series** · May 2026
*Papers: Foundation Training · Active Service Desk · Custom Software · Jira & Intune Integration · Change Management · Unified Platform*

</div>
