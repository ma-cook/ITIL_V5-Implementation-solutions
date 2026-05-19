<div align="center">

# ITIL Version 5 and the Active Service Desk

### Applying Proactive Monitoring, Automated Remediation, and Experience-Driven Service Management to Hardware, Peripheral, and Connectivity Troubleshooting

**RESEARCH PAPER** · *IT Service Management Practice Series*
**May 2026**

</div>

---

## Abstract

> The release of **ITIL Version 5** (February–April 2026) marks the most significant evolution of IT service management guidance in seven years. Building on ITIL 4's value-driven foundation, ITIL v5 natively integrates **artificial intelligence governance**, a **Product and Service Lifecycle Model (PSLM)**, **Experience Level Agreements (XLAs)**, and mandatory sustainability principles. Critically for operations professionals, it elevates the **Monitor, Support and Fulfil (MSF)** practice cluster - encompassing Service Desk, Incident Management, Problem Management, Monitoring and Event Management, and Service Request Management - to a formal certification pathway.
>
> This paper examines how the converging philosophies of ITIL v5 and modern customer success methodologies can be applied concretely to an active service desk responsible for hardware troubleshooting, peripheral management, and basic network connectivity. The paper proposes an **Integrated Active Service Desk (IASD)** model that uses proactive monitoring, AI-assisted triage, automated first-line remediation, and experience-centred measurement to shift the service desk from a reactive cost centre to a proactive value engine.

**Keywords:** `ITIL v5` · `ITSM` · `active service desk` · `proactive monitoring` · `automated remediation` · `XLA` · `AI governance` · `hardware support` · `peripheral troubleshooting` · `connectivity`

---

## 1. Introduction

### 1.1 Background and Motivation

For more than three decades, IT service desks have operated on a fundamentally **reactive model**: a user experiences a problem, raises a ticket, and waits for a technician. While this model served organisations well in an era of stable, manually administered infrastructure, it is poorly aligned with today's distributed, hybrid, and AI-enabled workplaces. Laptops, docking stations, wireless peripherals, VPN clients, and multi-SSID Wi-Fi networks all interact in complex ways. Failures are frequent, often subtle, and disproportionately disruptive to end-user productivity.

ITIL Version 5, launched in February 2026 with advanced modules rolling out through April and May 2026, provides a timely framework upgrade. Its explicit embrace of AI governance, experience-driven measurement, and a living Product and Service Lifecycle Model offers service desk leaders principled guidance for implementing the proactive, automated capabilities that modern infrastructure demands.

### 1.2 Scope

This paper focuses on the physical-layer and near-physical-layer issues that constitute the majority of service desk workload in endpoint-heavy organisations:

- **Hardware faults** - desktops, laptops, workstations, docking stations, monitors
- **Peripheral issues** - keyboards, mice, headsets, webcams, printers, scanners, USB hubs
- **Basic connectivity** - wired Ethernet, Wi-Fi association and authentication (WPA2/WPA3-Enterprise), VPN client failures, DNS resolution, DHCP lease problems

> Higher-layer application support and server infrastructure management are noted where they interact with the above but are not the primary focus.

### 1.3 Research Questions

1. How does ITIL v5's Product and Service Lifecycle Model reframe the service desk's role in hardware and connectivity support?
2. Which proactive monitoring and automated remediation techniques are sanctioned by ITIL v5's MSF practice cluster?
3. How can Experience Level Agreements replace or augment traditional SLAs for physical-layer support?
4. What AI governance guardrails does ITIL v5 prescribe for automated service desk actions?
5. What does a practical implementation roadmap look like for an organisation moving toward an Integrated Active Service Desk model?

---

## 2. ITIL Version 5: Key Changes Relevant to Service Desk Operations

### 2.1 From Service Value Chain to Product and Service Lifecycle Model

ITIL 4 introduced the Service Value System (SVS) with its six-activity Service Value Chain (SVC). ITIL v5 replaces the SVC with the **Product and Service Lifecycle Model (PSLM)**, which treats every service as a living, learning system rather than a linear chain of activities. The PSLM has eight stages:

> **Discover → Design → Acquire → Build → Transition → Operate → Deliver → Support**

Critically, data generated in the **Support** stage feeds directly back into **Discover** and **Design**, creating a continuous improvement loop that is structurally enforced rather than aspirational.

For hardware and connectivity support, this means:

- Failure patterns captured at the service desk directly inform procurement decisions *(Discover)*
- Recurring peripheral compatibility issues trigger design reviews *(Design)*
- Hardware procurement and vendor contracts are governed through the Acquire stage *(Acquire)*
- Known defects identified through problem management are embedded in release notes *(Transition)*
- Day-to-day service desk activity — incident management, event monitoring, request fulfilment — lives in the Operate stage *(Operate)*
- XLA reporting and user satisfaction measurement are formalised in the Deliver stage *(Deliver)*
- Automated diagnostics generate the data that feeds all upstream stages *(Support → Discover)*

### 2.2 The Monitor, Support and Fulfil (MSF) Practice Cluster

ITIL v5 introduced three bundled Practice Manager modules. The **MSF module** is the most directly relevant to service desk professionals and covers five practices that must be understood and applied together:

| Practice | Service Desk Relevance |
| --- | --- |
| **Service Desk** | Primary point of contact; triage of hardware and connectivity issues |
| **Incident Management** | Structured response to hardware faults, outages, peripheral failure |
| **Problem Management** | Root-cause analysis of recurring hardware/connectivity patterns |
| **Monitoring & Event Management** | Proactive detection of endpoint health degradation |
| **Service Request Management** | Fulfilment of peripheral orders, driver updates, Wi-Fi credentials |

> 💡 The explicit bundling of Monitoring and Event Management with Service Desk is significant: **ITIL v5 formally positions the service desk as a consumer of monitoring data, not merely a receiver of user reports.**

### 2.3 Experience Level Agreements (XLAs)

ITIL v5 formally adopts **XLAs** as a standard element of service and product management. Where a traditional SLA measures uptime or Mean Time To Resolve (MTTR), an XLA measures *how well a service enables users to achieve their goals* and *how satisfied they are with the experience*.

> ⚠️ For a service desk this distinction matters profoundly: a ticket closed in 15 minutes by forcing a user to reboot and accept residual instability **meets an SLA but fails an XLA**.

**XLA dimensions relevant to hardware/connectivity support:**

- **Effort Score** - How much did the user have to do to get resolution?
- **Disruption Index** - How much did the incident interrupt productive work?
- **First-Contact Sentiment** - Was the user's experience of the interaction positive?
- **Recurrence Rate** - Did the same issue return within 30 days?

### 2.4 Native AI Governance: The 6C AI Capability Model

ITIL v5 introduces a structured AI capability framework organised around six dimensions (the **6C AI Capability Model**). Rather than a maturity scale, it is an audit and planning tool — organisations may be advanced in one capability while nascent in another. For an active service desk, each dimension maps to concrete practice:

| 6C Dimension | Service Desk Application |
| --- | --- |
| **Creation** | AI generates first-draft knowledge articles and candidate remediation scripts from resolved incident data and problem records |
| **Curation** | Human review of AI-generated knowledge articles before publication; governance of the approved standard change register; quality scoring of AI-suggested scripts |
| **Clarification** | Virtual agent disambiguates vague incident descriptions; NLP classifies email and phone-transcript requests to the correct queue; AI surfaces intent from incomplete diagnostic reports |
| **Cognition** | Predictive alerting from endpoint telemetry identifies degradation before user impact; ML-based triage classifies incoming tickets and recommends resolution paths; AI-assisted root-cause analysis in Problem Management |
| **Communication** | Virtual service desk agent handles Tier 1 requests autonomously; automated proactive outreach when endpoint health scores degrade; users are explicitly informed when resolution is automated vs. human-led |
| **Coordination** | AI orchestrates multi-step remediation workflows across resolver groups; automated routing of incidents to correct teams; defined escalation path when automation encounters out-of-scope conditions |

> The 6C Capability Model provides the vocabulary and structure to audit existing AI use and identify governance gaps before they become incidents — particularly important when automated scripts act on user hardware configurations or network settings.

---

## 3. Customer Success Methodologies and Their Convergence with ITIL v5

### 3.1 What Customer Success Brings to ITSM

Customer success (CS) as a discipline emerged in SaaS organisations to proactively ensure customers achieved value from products rather than waiting for churn signals. Its core techniques - health scoring, proactive outreach, lifecycle stage management, and outcome-based success metrics - are now being absorbed into enterprise ITSM, a trend ITIL v5 both acknowledges and accelerates through XLAs and the PSLM.

**Key CS concepts that translate to service desk practice:**

- **Health Scoring** - aggregate signal from multiple telemetry sources into a single endpoint health score, triggering proactive intervention before a user notices a problem
- **Lifecycle Stage Awareness** - understanding whether an endpoint is newly provisioned, mid-lifecycle, or approaching end-of-life, and applying different support postures accordingly
- **Proactive Outreach** - contacting users before they raise tickets, based on detected anomalies
- **Success Plans** - defining what *good* looks like for a user's device and measuring against it

### 3.2 Endpoint Health Scoring

Borrowing the CS health score model, an **endpoint health score** aggregates telemetry into a 0–100 index that service desks can act on proactively. Sample components for a hardware-focused score:

| Signal | Weight | Monitoring Source |
| --- | :---: | --- |
| Disk health (SMART) | **20%** | Endpoint agent / SCCM |
| Battery cycle count | **15%** | BIOS telemetry / MDM |
| RAM error rate | **15%** | Windows Reliability Monitor |
| NIC driver event log errors | **15%** | Windows Event Log |
| Wi-Fi RSSI & SNR | **10%** | WLAN controller / NPS |
| Peripheral connectivity drop rate | **10%** | USB event log |
| Patch compliance | **10%** | WSUS / Intune |
| Recent BSOD / kernel panic count | **5%** | WER / Crashplan |

**Action thresholds:**

- 🟢 **Score > 70** - silent monitoring
- 🟡 **Score < 70** - triggers proactive outreach
- 🟠 **Score < 50** - triggers an automated diagnostic workflow
- 🔴 **Score < 30** - triggers a technician-dispatched hardware review

---

## 4. Proactive Monitoring Architecture for the Active Service Desk

### 4.1 Monitoring Layers

An effective proactive monitoring architecture for hardware, peripherals, and connectivity operates across **four interdependent layers**:

#### Layer 1: Endpoint Agents

Lightweight agents (e.g., Microsoft Intune, Tanium, Nexthink, or open-source equivalents) collect raw telemetry from managed endpoints. Key collection targets:

- **Hardware:** CPU/GPU thermals, memory error counters, storage SMART attributes, power events
- **Peripherals:** USB connect/disconnect events (Event ID `2003`/`2100`), display hotplug events, audio device state changes
- **Connectivity:** DHCP lease events, DNS resolution failures (Event ID `1014`), Wi-Fi association state changes, VPN tunnel establishment and teardown

#### Layer 2: Network-Side Monitoring

Monitoring should not rely solely on the endpoint. Network infrastructure provides a complementary perspective:

- **WLAN Controllers** (Cisco WLC, Aruba Central, Ubiquiti UniFi) - per-client RSSI, SNR, retry rate, roaming events, and authentication failures
- **RADIUS/NPS Logs** - 802.1X authentication attempts, EAP failures, certificate errors
- **DHCP Server Logs** - lease exhaustion, rogue client detection, scope utilisation thresholds
- **DNS Server Logs** - NXDOMAIN rates, resolution latency spikes

#### Layer 3: Correlation and Alerting

Raw telemetry from agents and network infrastructure is ingested into a SIEM or dedicated ITSM observability platform (e.g., ServiceNow ITOM, Dynatrace, or Datadog). Correlation rules translate raw events into meaningful service signals:

> **Example Rule**
> ```
> IF (Wi-Fi RSSI < -75 dBm
>     AND DNS latency > 800ms
>     AND VPN reconnect count > 3 in 30 min)
> THEN raise 'Probable Connectivity Degradation' event
>      with Priority 2, auto-assign to Network Tier 1.
> ```

#### Layer 4: Intelligent Event Management

ITIL v5's Monitoring and Event Management practice explicitly calls for *"systematically observing services and service components, recording, reporting, and responding to selected changes of state identified as events."* In practice this means **event noise suppression**, **correlation into incidents**, and - where governance permits - **automated response**.

### 4.2 ITIL v5 Alignment: Event → Incident → Problem

ITIL v5 preserves but refines the event-incident-problem relationship:

- **Event** - a detected change of state (e.g., SMART status changes from `OK` to `Pre-fail`)
- **Incident** - a disruption to normal service (e.g., drive degradation causing application crashes)
- **Problem** - the underlying cause of one or more incidents (e.g., a specific SSD firmware version causing early failure)

The MSF practice cluster mandates that Monitoring and Event Management feeds directly into Incident Management, which in turn feeds Problem Management. In an active service desk, this means:

| Trigger | Action |
| --- | --- |
| Events below threshold | Logged, no ticket |
| Events meeting threshold | Auto-ticket raised, user notified proactively |
| Pattern of events | Problem record raised, root-cause investigation scheduled |
| Confirmed problem with known workaround | Knowledge article published, automated remediation script enabled |

---

## 5. Automated Remediation Frameworks

### 5.1 The Automation Maturity Ladder

Not all service desks are ready for the same level of automation. ITIL v5's 6C AI Capability Model implicitly requires organisations to establish capability maturity before expanding automation scope. The following four-stage ladder provides a structured progression:

| Stage | Capability | Example Actions | Governance Gate |
| :---: | --- | --- | --- |
| **1 – Detect** | Monitoring only | Generate health score, raise alert | Observation; no action |
| **2 – Notify** | Proactive outreach | Email user with self-help link, push KB article | Communication policy approval |
| **3 – Remediate (Safe)** | Low-risk auto-fix | Flush DNS, reset NIC, update driver silently | CAB (standard change) |
| **4 – Remediate (Complex)** | High-impact auto-fix | Remote wipe/re-enrol, schedule hardware swap | Individual change approval + audit trail |

> 🎯 Most organisations should target **Stage 3 within 12 months** of adopting an active service desk model, with Stage 4 actions reserved for well-tested, high-frequency scenarios.

### 5.2 Hardware Troubleshooting Automation

#### 5.2.1 Storage Diagnostics

When a SMART attribute crosses a threshold (e.g., `Reallocated Sectors > 5`):

1. Agent raises event; correlation engine creates **Priority 2** incident.
2. Automated script runs `chkdsk /scan` (non-destructive) and captures output.
3. If errors found: user is notified, backup client is triggered to run immediately, technician dispatched for drive replacement within **2 hours**.
4. Technician marks resolution; PSLM data feeds procurement (device flagged for next refresh cycle).

#### 5.2.2 Memory Fault Detection

Windows Memory Diagnostic and Linux `memtester` can be scheduled during low-activity windows (outside business hours, detected by endpoint idle state):

- Results logged centrally; repeated failures auto-escalate to hardware replacement request
- Known-good RAM configurations are maintained in the CMDB to guide fast like-for-like replacement

#### 5.2.3 Thermal Management

Chronic overheating is a leading cause of hardware failure. Automated thermal monitoring can:

- Alert users to clear vents or elevate laptops
- Log events for trend analysis; repeated thermal events trigger a proactive clean/service ticket
- Correlate with high-CPU processes and suggest termination of rogue processes

### 5.3 Peripheral Troubleshooting Automation

Peripheral issues - USB devices not recognised, audio not functioning, monitors flickering - are among the highest-volume service desk contacts and are amenable to scripted diagnosis:

> **Common Peripheral Automation - USB device not recognised**
> 1. Query Event Log for USB error codes
> 2. Run automated driver reinstall via `DevCon` or `PnPUtil`
> 3. If unresolved, push self-service diagnostic URL
> 4. If user confirms failure, raise fulfilment request for replacement peripheral

- **Printer issues** - automated port re-binding, print spooler restart, driver version verification against approved list
- **Webcam / headset** - automated device manager scan, audio endpoint reset via Windows Core Audio API
- **Monitor** - EDID data capture to confirm display recognised; automated resolution reset to native

### 5.4 Connectivity Troubleshooting Automation

#### 5.4.1 Wi-Fi Diagnosis and Remediation

Wireless connectivity is the most reported category of *"I can't connect"* incidents. Automated Wi-Fi diagnosis should follow a structured test path:

1. **Physical layer** - Confirm radio enabled (not hard/soft blocked), check RSSI via `netsh wlan show interfaces`
2. **Association** - Verify SSID association; if failed, trigger profile deletion and rejoin
3. **Authentication** - Check EAP status; if certificate expired, trigger MDM certificate re-enrolment
4. **IP layer** - Confirm DHCP lease; if `169.254.x.x` (APIPA), release/renew, escalate to network team if DHCP scope exhausted
5. **DNS** - Run `nslookup` against internal resolver; if failing, flush resolver cache, test alternative DNS
6. **Gateway** - Ping default gateway; if failing, raise network incident, auto-notify NOC
7. **Internet** - Ping external (`8.8.8.8`); if internal OK but external failing, suspect firewall/proxy → auto-raise Priority 2 for network team

#### 5.4.2 VPN Client Automation

- Detect tunnel state via client API or process monitoring
- Automated reconnection attempt with logging
- Certificate validity check; auto-renew via MDM if within 30-day expiry window
- Split-tunnel route verification; correct routing table injection if misconfigured

#### 5.4.3 Wired Ethernet

- Physical link detection (Event ID `4001`/`4004` for NIC link up/down)
- Automated cable fault isolation: prompt user to reseat cable, check switch port LED state
- Switch port admin state verification via SNMP polling; auto-alert NOC if port in `err-disabled` state
- 802.1X supplicant state machine monitoring; automated credential cache refresh

---

## 6. The Integrated Active Service Desk (IASD) Model

### 6.1 Model Overview

The IASD model combines ITIL v5's MSF practices with customer success methodologies and the automation frameworks described above into a coherent operational model. It has **four operational modes** that activate based on endpoint health score and event severity:

| Mode | Trigger | Primary Actor | Actions |
| --- | --- | --- | --- |
| 🟢 **Silent Monitoring** | Score > 70 | Automated system | Collect telemetry, update health score, trend analysis |
| 🟡 **Proactive Engagement** | Score 50–70 | Automated + Self-service | Notify user, push KB, offer guided fix |
| 🟠 **Automated Remediation** | Score 30–50 or P3 event | Automation + Tier 1 oversight | Run safe scripts, update ticket, confirm with user |
| 🔴 **Human-Led Response** | Score < 30 or P1/P2 event | Technician (+ AI assist) | Dispatch, diagnose, escalate, replace hardware |

### 6.2 Workflow: From Event to Resolution

The following end-to-end workflow illustrates the IASD model for a laptop with a deteriorating Wi-Fi connection:

1. WLAN controller detects client retry rate exceeding **15% over 10 minutes**. Event raised.
2. Correlation engine matches event to endpoint; health score drops from **74 → 61**.
3. System sends proactive notification:
   > *"We've noticed your Wi-Fi may be experiencing issues. Click here to run an automated check."*
4. User clicks link; automated diagnostic script runs Steps 1–7 from Section 5.4.1.
5. Script identifies DHCP lease is stale; performs release/renew. **Connectivity restored.**
6. Ticket auto-closed with resolution code `DHCP Lease Refresh – Automated`. User satisfaction survey sent (XLA data collection).
7. *If script fails:* ticket escalates to Tier 1 with full diagnostic output pre-populated. Technician avoids re-running diagnostics already completed.
8. Problem Management reviews weekly report of DHCP-related resolutions; identifies scope exhaustion pattern. Network team engaged.

### 6.3 XLA Measurement Framework

The IASD model tracks the following XLA dimensions for hardware, peripheral, and connectivity support:

| XLA Metric | Measurement Method | Target |
| --- | --- | :---: |
| **User Effort Score (UES)** | Post-resolution survey (1–5 scale) | `> 4.0` average |
| **Disruption Index** | Minutes of disrupted work per incident | `< 20 min` average |
| **Proactive Resolution Rate** | % of incidents resolved before user report | `> 30%` within 12 months |
| **First-Contact Resolution (FCR)** | Tickets closed without escalation | `> 75%` |
| **Recurrence Rate** | Same issue within 30 days | `< 10%` |
| **Automated Remediation Rate** | % resolved by automation | `> 40%` within 18 months |

---

## 7. AI Governance and Change Management

### 7.1 Applying ITIL v5's 6C AI Capability Model to Automated Remediation

Every automated remediation action on a user's device constitutes a **change**. ITIL v5's Change Enablement practice (in the PIC module) and its 6C AI Capability Model together define what responsible automation looks like in practice:

| 6C Dimension | Automated Remediation Obligation |
| --- | --- |
| **Creation** | Remediation scripts and knowledge articles generated or suggested by AI must be created through a documented process — including test coverage, rollback procedures, and scope definitions |
| **Curation** | Every AI-generated script must pass human review and formal standard change classification before autonomous execution is permitted; the approved script library is a curated, governed artefact |
| **Clarification** | Diagnostic automation must accurately interpret telemetry without false positives; ambiguous signals must escalate to human review rather than trigger remediation |
| **Cognition** | ML-based triage and predictive alerting must be validated against historical data; accuracy, false positive rate, and any device-class bias must be tracked and reviewed regularly against XLA outcomes |
| **Communication** | Every automated action must notify the affected user with a clear explanation of what ran and what changed; users and technicians must always be able to suppress automation on specific devices |
| **Coordination** | Automation must never exceed the scope of its standard change record; any out-of-scope condition must halt and escalate to a human technician; a full audit trail (timestamp, action, device, trigger event, outcome) is mandatory |

**Core governance controls required for all automated remediation:**

- **Standard Change Classification** — automated scripts that have been tested, approved, and documented must be classified as standard changes, removing the need for individual approval and enabling autonomous execution
- **Audit Trail** — every automated action must be logged with timestamp, action taken, user device, outcome, and the monitoring event that triggered it
- **Rollback Capability** — all standard changes must have a documented rollback procedure (e.g., driver rollback via `DISM`)
- **Human Override** — users and technicians must be able to suppress automated actions on specific devices
- **Scope Boundaries** — automation must never exceed the scope of the standard change record; any action outside scope must halt and escalate

### 7.2 Ethical and Compliance Considerations

Automated access to user endpoints raises privacy and compliance questions:

- **Informed Consent** - users should be aware that endpoint agents collect telemetry and may initiate automated actions; this should be stated in the acceptable use policy
- **Data Minimisation** - collect only the telemetry required to calculate health scores and diagnose issues; do not retain raw logs beyond **90 days** unless required for problem management
- **Regulated Environments** - in healthcare, financial services, or government, automated changes may require additional approval workflows under frameworks such as **HIPAA**, **SOX**, or **NZISM**

### 7.3 Change Advisory Board Integration

The IASD model recommends a monthly review of automated remediation scripts by a lightweight CAB sub-committee. The agenda should cover:

1. **New scripts** proposed for standard change classification - review test results and rollback procedures
2. **Audit log review** of automated actions for the previous month - identify any unintended outcomes
3. **XLA data review** - are automated resolutions improving user experience metrics?
4. **AI model performance review** (where ML-based triage is in use) - accuracy, false positive rate, and any demographic or device-class bias

---

## 8. Implementation Roadmap

The following phased roadmap is designed for an organisation with an existing ITSM platform and basic endpoint management tooling. Timelines are indicative and should be adjusted based on organisational size and maturity.

### 📅 Phase 1 - Foundation *(Months 1–3)*

- Audit current telemetry collection capabilities across endpoints, network, and ITSM platform
- Deploy or verify endpoint agent coverage *(target: **95%** of managed devices)*
- Define endpoint health score formula and configure dashboards
- Classify and document top-10 hardware, peripheral, and connectivity issue types
- Establish baseline SLA and initial XLA metrics
- Train service desk staff on ITIL v5 MSF concepts and health score interpretation

### 📅 Phase 2 - Proactive Engagement *(Months 4–6)*

- Activate automated proactive notifications for health scores below 70
- Publish and link self-service diagnostic guides for top-10 issue types
- Integrate WLAN controller and DHCP server log feeds into ITSM correlation engine
- Begin recording proactive resolution rate as an XLA metric
- Run pilot of automated DNS flush and NIC reset scripts for Tier 1 approval as standard changes

### 📅 Phase 3 - Automated Remediation *(Months 7–12)*

- Promote approved scripts to standard change status and enable automated execution
- Expand script library to cover Wi-Fi reconnection, printer spooler reset, USB driver reinstall, VPN certificate renewal
- Implement post-resolution XLA survey for all automated closures
- Introduce monthly CAB sub-committee for script governance
- **Target:** 40% of top-10 issue types resolved without human intervention

### 📅 Phase 4 - Optimisation and AI Integration *(Months 13–24)*

- Deploy ML-based triage to classify incoming tickets and recommend scripts before technician review
- Train model on 12 months of resolved ticket data; validate against 6C AI Capability framework
- Introduce predictive replacement scheduling for devices with consistently low health scores
- Expand XLA framework to include Disruption Index and Recurrence Rate
- Pursue ITIL v5 Practice Manager (MSF) certification for senior service desk analysts

---

## 9. Discussion

### 9.1 Opportunities

The convergence of ITIL v5's structural framework with customer success monitoring philosophies creates a compelling opportunity for service desks. **Proactive monitoring** reduces mean time to detect (MTTD) from the user-report moment to the event-detection moment, often hours earlier. **Automated remediation** reduces mean time to resolve (MTTR) for well-understood issues from tens of minutes to seconds. Together, these shifts can materially reduce the **Disruption Index** - the single metric most strongly correlated with user satisfaction in endpoint support contexts.

The PSLM's feedback loop from *Support* to *Discover* also creates a natural mechanism for **evidence-based hardware procurement**. Service desks that systematically capture failure data will be able to advise procurement teams on which device models, peripheral brands, and driver versions generate disproportionate support load - a capability that has historically required manual effort and was rarely exercised.

### 9.2 Challenges and Risks

> ⚠️ **Automation confidence** - automated scripts that misconfigure network settings or roll back drivers incorrectly can cause more harm than the original issue. Rigorous testing environments and conservative initial scope are essential.

> ⚠️ **User trust** - proactive outreach can be perceived as surveillance if not communicated transparently. Clear messaging about *what* is monitored and *why* is critical to XLA outcomes.

> ⚠️ **Tool sprawl** - effective proactive monitoring requires integration between ITSM platforms, endpoint agents, WLAN controllers, and DHCP/DNS servers. Organisations with fragmented tooling may need significant integration investment before the IASD model delivers returns.

> ⚠️ **Skill transition** - ITIL v5's MSF certification pathway requires service desk analysts to develop competency in monitoring data interpretation, script governance, and XLA management - skills not typically emphasised in traditional ITIL training.

### 9.3 Future Directions

ITIL v5's **AI Governance extension module** - currently in development as a standalone certification - will provide more detailed guidance on agentic AI in ITSM contexts. As large language model-based triage agents mature, the boundary between automated remediation and autonomous AI operation will require careful governance. The IASD model described in this paper provides a principled starting point for organisations that will need to extend their governance frameworks as AI capabilities advance.

---

## 10. Conclusion

ITIL Version 5 arrives at exactly the moment when service desks are under pressure to do more with less while simultaneously meeting rising user expectations. Its **Product and Service Lifecycle Model** provides the structural rationale for treating every hardware failure, peripheral malfunction, or connectivity drop as data that should flow back into service improvement. Its **Monitor, Support and Fulfil** practice cluster formally unites monitoring with service desk operations. Its adoption of **Experience Level Agreements** provides the measurement language to capture what traditional SLAs miss. And its **6C AI Capability Model** provides the audit and planning framework to deploy automation responsibly.

The **Integrated Active Service Desk (IASD)** model proposed in this paper translates these principles into actionable practice for the most frequent and tangible service desk workload: the physical-layer issues that affect every device-dependent worker, every day. By combining endpoint health scoring, network-side monitoring, structured automated remediation, and experience-centred measurement, organisations can shift their service desks from **cost centres that react to calls** to **proactive partners that prevent disruption**.

> The implementation roadmap provided is intentionally conservative. Automation that is introduced carefully, governed rigorously, and measured against user experience outcomes will compound in value over time. Automation introduced hastily without governance will erode trust - both user trust and the trust of the organisation in its ITSM function.
>
> **ITIL v5's framework, applied thoughtfully, makes the conservative path the winning path.**

---

## References

1. **PeopleCert** (2026). *ITIL Foundation (Version 5).* PeopleCert Official Publication.
2. **PeopleCert** (2026). *ITIL (Version 5) Explained: New Modules, AI Governance, and Certification Pathways.* <https://www.itil.com>
3. **Knowledge Academy** (2026). *ITIL Version 5: What's Changed and Why It Matters.* <https://www.theknowledgeacademy.com>
4. **Spidya** (2026). *What Is ITIL v5 Framework? New Practices & ITIL 4 Comparison.* <https://spidya.com>
5. **PassIT Exams** (2026). *ITIL v4 vs v5: Key Differences, Updates & Complete Certification Guide.* <https://passitexams.com>
6. **ServiceNow Community** (2026). *Understanding ITIL 5: What's New and How It Builds on ITIL 4.* <https://www.servicenow.com/community>
7. **Incident.io** (2026). *Everything You Need to Know About ITIL 5, AI and Incident Management.* <https://incident.io/blog>
8. **Moveworks** (2025). *ITIL AI: How AI Is Transforming IT Service Management.* <https://www.moveworks.com>
9. **Ivanti** (2026). *Navigating the Shift to Agentic AI in IT Service Management.* <https://www.ivanti.com>
10. **Serviceaide** (2026). *The Best Practices for ITIL Help Desk Success 2026.* <https://www.serviceaide.com>
11. **ITIL.org.uk** (2026). *ITIL Version 5: A Complete Guide.* <https://www.itil.org.uk>
12. **ITSM.tools** (2026). *ITIL (Version 5) Explained: Key Changes, Lifecycle, AI Governance.* <https://itsm.tools>
