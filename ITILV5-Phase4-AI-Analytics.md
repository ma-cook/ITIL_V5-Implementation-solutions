<div align="center">

# UIWP Phase 4: AI Training on Service Desk Data and Advanced Analytics

### Practical Implementation of ML-Based Ticket Classification, Predictive Device Health, PSLM Feedback Loops, Conversational Interfaces, and Management Reporting

**FOLLOW-UP PAPER**
*Extension to the ITIL v5 Practical Solutions Series — Unified Platform Architecture*

**May 2026** · IT Service Management Practice Series

</div>

---

## Abstract

> The Unified ITSM Web Platform (UIWP) papers established a governed, API-integrated operational platform for the service desk. The final phase of that roadmap — **Phase 4: Advanced Analytics and AI Assist (Months 13–24)** — transforms the UIWP from a sophisticated aggregation layer into a learning system. Where Phase 1–3 made data visible, Phase 4 makes data *actionable through intelligence*.
>
> This paper provides the practical implementation specification for the five Phase 4 outcomes: ML-based ticket classification trained on accumulated service desk data; trend-based predictive device health alerting; a PSLM feedback loop dashboard that routes Support-stage intelligence back into the Discover stage; a conversational bot interface exposing key UIWP operations via Microsoft Teams and Slack; and Power BI or Confluence export integration for management reporting. Each outcome is grounded in the data collection architecture established in Phases 1–3 and governed by ITIL v5's 6C AI Capability Model and the IASD change management framework.

**Keywords:** `machine learning` · `ticket classification` · `predictive analytics` · `PSLM feedback loop` · `Teams bot` · `Slack bot` · `Power BI integration` · `Confluence export` · `ITIL v5` · `AI governance` · `service desk data` · `NLP`

---

## 1. Introduction

### 1.1 The Data Asset Created by Phases 1–3

Phases 1–3 of the UIWP deployment did more than surface operational data. They created something far more valuable: a **structured, correlated, multi-source dataset** accumulating continuously in the UIWP's PostgreSQL, Redis, and Elasticsearch data tier.

By the end of month 12, a typical mid-market deployment will have accumulated:

| Dataset | Source | Estimated Volume (12 months) |
| --- | --- | --- |
| JSM ticket records (incidents, problems, requests) | JSM REST API + webhooks | 5,000–50,000 records |
| Per-ticket resolution paths and time-to-resolve | JSM fields + audit trail | Attached to every ticket |
| Device context snapshots at ticket creation | UIWP `device_context_cache` | Per-ticket enrichment record |
| Endpoint health scores over time | Intune + WEC + WLAN data | ~50–500 readings per device per month |
| Printer health event log | PLA API | ~20–200 events per printer per month |
| POS terminal fault history | PHMS API | ~10–100 events per terminal per month |
| WLAN client metrics time-series | WLAN Controller API | Continuous per-client metrics |
| KB article access patterns | JSM knowledge base API | Per-article view and resolution link counts |
| Technician actions taken via UIWP | `audit_log` table | Every UIWP-initiated API call |

This accumulated dataset is the training foundation for every Phase 4 capability. No external data purchase or synthetic data generation is required — the operational history of the service desk is itself the corpus.

### 1.2 Relationship to ITIL v5 Governance

Phase 4 capabilities fall squarely within ITIL v5's **6C AI Capability Model**:

| 6C Dimension | Phase 4 Capability |
| --- | --- |
| **Cognition** | ML ticket classification; trend-based predictive device health; anomaly detection in PSLM feedback data |
| **Creation** | AI-suggested KB article recommendations from resolved ticket history; AI-drafted management summary narratives |
| **Curation** | Human review gates for ML model outputs before they influence technician decisions; confidence thresholds below which AI suggestions are suppressed |
| **Clarification** | Natural language bot commands disambiguated via intent classification before action; in-context explanations of why a prediction was made |
| **Communication** | Teams/Slack bot delivers proactive alerts and status updates; predicted resolutions surfaced with confidence scores so technicians understand the AI's basis for suggestions |
| **Coordination** | Bot interface orchestrates multi-step UIWP actions via conversational flow; PSLM dashboard coordinates the handoff of Support-stage intelligence to procurement and vendor management |

> 💡 All Phase 4 capabilities must pass through the ITIL v5 Normal Change process in JSM before activation, as specified in Section 7.2 of the UIWP Architecture paper. AI capabilities that influence human decisions require a **full CAB review with 6C checklist**.

### 1.3 Scope of This Paper

This paper covers:

1. Data preparation and feature engineering from the UIWP data tier
2. ML-based ticket classification: model selection, training pipeline, integration, and governance
3. Predictive device health alerting: trend modelling over endpoint telemetry
4. PSLM feedback loop dashboard: routing Support data to Discover and Design stages
5. Teams and Slack bot interface: architecture, command set, and safety controls
6. Power BI and Confluence export integration: automated management reporting

---

## 2. Data Preparation and Feature Engineering

Before any ML model can be trained, the accumulated UIWP data must be prepared into clean, labelled, and feature-engineered datasets. This is the step most commonly underestimated in operational ML projects.

### 2.1 Ticket Data Corpus Preparation

The JSM ticket history, enriched with UIWP `device_context_cache` data, forms the primary training corpus for ticket classification.

**Step 1 — Export and labelling**

The training label for each ticket is its **verified resolution category** — a field that must be consistently populated by technicians from Phase 1 onward. Before Phase 4 ML work begins, run a data quality audit:

```sql
-- Identify tickets missing resolution categories (training label gap)
SELECT
    issue_key,
    summary,
    created_at,
    resolved_at,
    resolution_category
FROM jsm_ticket_cache
WHERE resolved_at IS NOT NULL
  AND (resolution_category IS NULL OR resolution_category = '')
ORDER BY resolved_at DESC;
```

> ⚠️ **Governance requirement:** Resolution categories must be a controlled vocabulary — a finite list agreed by the service desk manager and implemented as a required JSM field. Free-text resolution notes cannot serve as ML labels without a labelling step. If resolution categories are not already mandatory in JSM, this must be implemented as a JSM configuration change (Normal Change) **before Phase 4 begins**.

**Step 2 — Feature extraction**

Each training record is a combination of text features and structured features:

| Feature | Source | Type |
| --- | --- | --- |
| Ticket summary (title) | JSM `summary` field | Text (NLP) |
| Ticket description | JSM `description` field | Text (NLP) |
| Reporter job role | Entra ID / JSM user profile | Categorical |
| Device model | Intune `deviceModel` | Categorical |
| OS version | Intune `osVersion` | Categorical |
| Endpoint health score at ticket creation | UIWP `device_context_cache` | Numeric |
| WLAN RSSI at ticket creation | WLAN metrics cache | Numeric |
| Time of day / day of week | Ticket `created_at` | Temporal |
| Number of prior tickets for same device (30 days) | Computed from `jsm_ticket_cache` | Numeric |
| KB articles viewed before ticket creation | JSM KB access log | Numeric (count) |
| M365 service health at ticket creation | M365 health cache | Categorical (OK/Degraded/Incident) |
| **Label:** Resolution category | JSM `resolution_category` field | Categorical (target) |

**Step 3 — Text preprocessing**

Ticket summaries and descriptions require NLP preprocessing before they can serve as model inputs:

1. **Tokenisation** — split raw text into tokens
2. **Stop-word removal** — remove low-information words ("the", "a", "please", "help")
3. **Lemmatisation** — reduce inflected forms to their base form ("printers" → "printer", "failing" → "fail")
4. **Domain vocabulary normalisation** — map common service desk synonyms to canonical terms ("laptop" = "notebook" = "device"; "not working" = "fault" = "failure")
5. **Embedding generation** — convert preprocessed text to numeric vectors using a pre-trained transformer model (see Section 3.2)

**Step 4 — Class balance assessment**

Service desk ticket distributions are rarely balanced. Hardware faults, password resets, and printer issues typically dominate, with rare but important categories (network outages, POS failures) underrepresented. Before training:

```python
from collections import Counter
import pandas as pd

df = pd.read_csv('ticket_training_data.csv')
label_dist = Counter(df['resolution_category'])
print(pd.Series(label_dist).sort_values(ascending=False))
```

Apply **class weighting** (not oversampling) during training to avoid penalising minority categories disproportionately.

### 2.2 Device Health Time-Series Preparation

For predictive device health modelling (Section 4), the endpoint health score history and its component signals must be restructured as **time-series sequences** rather than individual point-in-time readings.

```sql
-- Retrieve health score time-series per device (last 90 days)
SELECT
    device_id,
    device_model,
    recorded_at,
    health_score,
    smart_status,
    battery_cycle_count,
    ram_error_rate,
    nic_error_count,
    wifi_rssi_avg,
    patch_compliance,
    bsod_count_7d
FROM device_health_history
WHERE recorded_at >= NOW() - INTERVAL '90 days'
ORDER BY device_id, recorded_at;
```

Each device becomes a **sequence of feature vectors** over time. The prediction target is a **binary label**: did this device generate a hardware-related incident ticket within the following 14 days?

---

## 3. ML-Based Ticket Classification

### 3.1 Business Case

Today, an incoming ticket is manually reviewed by a first-line analyst who assigns it a category, priority, and resolver group before it enters the active queue. This classification step takes 2–5 minutes per ticket and is subject to analyst fatigue and inconsistency.

ML-based classification delivers three distinct capabilities within the UIWP ticket queue:

1. **Predicted category** — the most likely resolution category, displayed alongside the queue entry
2. **Suggested KB article** — the knowledge base article most frequently associated with this ticket type and device profile, surfaced before the analyst opens the ticket
3. **Predicted resolution time** — an estimated time-to-resolve based on the historical distribution for tickets of this predicted category and device context

All three are presented as **suggestions with a confidence score**, never as automatic assignments. The analyst confirms, modifies, or overrides. This is the 6C Curation requirement made concrete.

### 3.2 Model Architecture

A two-stage pipeline is recommended:

**Stage 1 — Text embedding**

Use a **sentence transformer** model (e.g., `all-MiniLM-L6-v2` from the `sentence-transformers` library) to convert ticket summaries and descriptions into 384-dimensional dense vectors. This model is:

- Small enough to run on a CPU-only inference server (no GPU required)
- Pre-trained on general English text and fine-tunable on service desk vocabulary
- Available as an open-source model with no per-inference API cost

```python
from sentence_transformers import SentenceTransformer

encoder = SentenceTransformer('all-MiniLM-L6-v2')
ticket_embeddings = encoder.encode(df['summary'] + ' ' + df['description'])
```

**Stage 2 — Classification**

Concatenate the text embedding vector with the structured feature vector (device model, health score, RSSI, etc.) and pass through a **gradient-boosted classifier** (XGBoost or LightGBM). These models:

- Handle mixed numeric and categorical input well
- Are interpretable (feature importance ranking)
- Train quickly on datasets of 5,000–50,000 records
- Do not require GPU infrastructure

```python
import lightgbm as lgb
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
import numpy as np

# Combine text embeddings with structured features
X = np.hstack([ticket_embeddings, structured_features])
y = labels  # encoded resolution categories

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = lgb.LGBMClassifier(
    n_estimators=500,
    class_weight='balanced',
    learning_rate=0.05,
    num_leaves=63,
    random_state=42
)
model.fit(X_train, y_train)

print(classification_report(y_test, model.predict(X_test)))
```

**KB article recommendation** is handled as a separate retrieval step: for the predicted category, retrieve the top-3 KB articles ranked by historical resolution linkage rate for that category and device model combination.

**Resolution time prediction** is a regression task, not classification: a separate LightGBM regressor predicts time-to-resolve (in hours) based on the same feature set. Outputs are presented as a range (e.g., "Typically resolves in 1–3 hours") based on the 25th–75th percentile of historical distribution for the predicted category.

### 3.3 Training Pipeline

The model training pipeline should be automated using the UIWP backend's task queue (BullMQ or Celery):

```
┌─────────────────────────────────────────────────────────────┐
│                   WEEKLY RETRAINING PIPELINE                │
│                                                             │
│  1. Data export job (Sunday 02:00)                          │
│     ├─ Query jsm_ticket_cache (resolved, last 6 months)     │
│     ├─ Join device_context_cache at ticket creation time    │
│     └─ Export labelled CSV to secure training data store    │
│                                                             │
│  2. Preprocessing job                                       │
│     ├─ Text preprocessing (tokenise, lemmatise, embed)      │
│     ├─ Structured feature normalisation                     │
│     └─ Class balance assessment + weight calculation        │
│                                                             │
│  3. Training job                                            │
│     ├─ Train LightGBM classifier (category)                 │
│     ├─ Train LightGBM regressor (resolution time)           │
│     └─ Evaluate against held-out test set                   │
│                                                             │
│  4. Validation gate                                         │
│     ├─ IF weighted F1 score < 0.70: flag for review, do     │
│     │  NOT deploy new model                                  │
│     └─ IF weighted F1 score ≥ 0.70: promote to staging      │
│                                                             │
│  5. Deployment (requires manual approval from SD Manager)   │
│     ├─ Canary deploy: apply new model to 10% of queue       │
│     ├─ Monitor override rate for 48 hours                   │
│     └─ Full deploy if override rate within acceptable range │
└─────────────────────────────────────────────────────────────┘
```

> 🎯 **Governance checkpoint:** The validation gate (Step 4) and the manual approval step (Step 5) are mandatory ITIL v5 6C Curation requirements. A model that does not meet the F1 threshold is not deployed, regardless of schedule pressure. The manual approval step ensures a human decision-maker is accountable for every model version placed in front of service desk technicians.

### 3.4 UIWP Ticket Queue Integration

The Phase 4 ticket queue view extends the Phase 1–3 ticket queue with three additional columns:

| New Column | Source | Display |
| --- | --- | --- |
| **AI Category** | ML classifier output | Predicted label + confidence badge (e.g., "Hardware — Disk ● 87%") |
| **Suggested KB** | KB retrieval module | Linked article title; click opens article in UIWP KB panel |
| **Est. Resolution** | Regression model output | "~2 hrs" or "1–4 hrs" range |

Technicians can dismiss, accept, or override any AI suggestion. Override actions are logged to the `audit_log` table and fed back into the next retraining cycle as correction signals.

The AI Category and Suggested KB columns are **hidden by default** and enabled per technician role via UIWP user preferences — acknowledging that not all technicians will benefit equally from the AI assist, and ensuring experienced analysts are not disrupted by suggestions they do not need.

### 3.5 Confidence Thresholds and Fallback Behaviour

| Confidence | Display Behaviour |
| --- | --- |
| ≥ 85% | Category shown in queue; KB article link shown; resolution time shown |
| 70–84% | Category shown with amber badge; KB link shown; resolution time suppressed |
| 50–69% | Category shown with grey "Low confidence" label; no KB link; no resolution time |
| < 50% | No AI suggestion displayed; ticket enters queue as standard unclassified item |

This graduated approach ensures that low-quality predictions do not create noise in the technician's workflow.

---

## 4. Predictive Device Health Alerting

### 4.1 From Threshold Alerts to Trend Alerts

The Phase 2 UIWP endpoint health score operates on **threshold-based** logic: alert when a metric crosses a fixed value (e.g., health score < 50). This approach has two well-documented limitations:

1. **False positives** — a device may dip below threshold briefly due to a transient event (e.g., a Windows Update BSOD) but recover immediately without indicating an underlying fault
2. **Late detection** — a device degrading slowly over weeks may sit just above the threshold for months before triggering, by which time failure may be imminent

**Trend-based alerting** — Phase 4's contribution — detects *trajectory*, not just current state. A device whose health score has fallen from 85 to 72 over four weeks, even though 72 is above the threshold, is showing a concerning downward trend that warrants proactive investigation.

### 4.2 Trend Detection Model

**Approach: Isolation Forest + Linear Trend Coefficient**

Two complementary techniques are applied to each device's 90-day health score time-series:

**Technique 1 — Linear trend coefficient**

Fit a simple linear regression to the device's health score over the rolling 30-day window. The slope coefficient indicates trajectory:

```python
from sklearn.linear_model import LinearRegression
import numpy as np

def calculate_health_trend(score_series: list[float]) -> float:
    """Returns slope (points/day). Negative = declining."""
    if len(score_series) < 7:
        return 0.0
    X = np.arange(len(score_series)).reshape(-1, 1)
    y = np.array(score_series)
    model = LinearRegression().fit(X, y)
    return float(model.coef_[0])
```

**Alert threshold:** If the 30-day trend slope is ≤ −0.5 (health score declining by more than 0.5 points per day on average), AND the current health score is < 75, trigger a predictive health alert at the UIWP level.

**Technique 2 — Isolation Forest anomaly detection**

For detecting sudden-but-not-yet-threshold anomalies in individual component signals (e.g., an unusual pattern in SMART attribute changes or NIC error rates that has not yet affected the composite health score):

```python
from sklearn.ensemble import IsolationForest

# Feature matrix: one row per daily snapshot, columns = individual health components
iso_forest = IsolationForest(contamination=0.05, random_state=42)
iso_forest.fit(historical_feature_matrix)

# For a new device snapshot:
anomaly_score = iso_forest.decision_function(new_snapshot.reshape(1, -1))
is_anomalous = iso_forest.predict(new_snapshot.reshape(1, -1)) == -1
```

If a device's individual component snapshot is flagged as anomalous by Isolation Forest, even when its composite health score has not yet triggered a threshold, a **"Component anomaly detected"** indicator is shown on the device's context panel in the UIWP.

### 4.3 Predictive Alert Taxonomy

Phase 4 introduces a new alert category in the UIWP distinct from the reactive threshold alerts of Phases 1–3:

| Alert Type | Trigger | UIWP Display | Suggested Action |
| --- | --- | --- | --- |
| **Trend Alert — Gradual Decline** | 30-day slope ≤ −0.5 AND score < 75 | Amber trend indicator on device tile | Schedule proactive technician review within 7 days |
| **Trend Alert — Accelerating Decline** | 7-day slope ≤ −1.5 (accelerating) | Red trend indicator | Priority 2 proactive ticket suggested |
| **Component Anomaly** | Isolation Forest flags component pattern | Grey anomaly badge on device tile | Investigate component signals in device context panel |
| **Failure Prediction** | Model predicts ticket within 14 days with ≥ 70% confidence | Orange "At risk" badge | Proactive outreach to user suggested |

**Failure Prediction** is the most advanced alert type. It requires a **binary classifier** trained on the historical dataset: given a device's feature vector today, does it generate a hardware incident ticket within the next 14 days? This classifier is trained offline using the same LightGBM framework as the ticket classifier (Section 3.2), using device health history as input features and "ticket raised within 14 days" as the binary label.

### 4.4 Model Retraining and Drift Monitoring

Device health models are retrained **monthly** (not weekly like the ticket classifier) because hardware failure patterns change more slowly than ticket language patterns.

Drift monitoring checks whether the distribution of incoming device health feature vectors has shifted significantly from the training distribution:

```python
from scipy.stats import ks_2samp

# KS test: has the distribution of health scores drifted?
statistic, p_value = ks_2samp(training_health_scores, recent_health_scores)
if p_value < 0.05:
    trigger_drift_alert("Device health score distribution has drifted — retraining recommended")
```

---

## 5. PSLM Feedback Loop Dashboard

### 5.1 The Problem: Support Data Stranded at the Support Stage

ITIL v5's Product and Service Lifecycle Model defines a closed loop: data from the **Support** stage should continuously feed back into the **Discover** and **Design** stages, informing procurement decisions, vendor evaluations, and service configuration updates.

In practice, this loop rarely closes in most organisations because Support-stage data lives in JSM, endpoint agents, and monitoring tools that procurement teams, vendor managers, and service designers never access. Phase 4 addresses this directly with a purpose-built **PSLM Feedback Loop Dashboard** within the UIWP.

### 5.2 Dashboard Design

The PSLM Feedback Loop Dashboard is a dedicated UIWP view, accessible to **IT Management, Service Desk Manager, Hardware Procurement Lead, and Vendor Relationship Manager** roles. It presents Support-stage data in the language of Discover and Design decision-making.

#### 5.2.1 Hardware Failure Trends by Model

**Data source:** `jsm_ticket_cache` (resolution category = hardware fault), joined with `device_context_cache` (device model, age, site).

**Visualisation:** A sortable table with the following columns:

| Column | Description |
| --- | --- |
| Device Model | e.g., "Dell Latitude 5440", "Lenovo ThinkPad T14s Gen 4" |
| Fleet Size | Number of devices of this model currently managed |
| Total Tickets (12M) | Total hardware fault tickets raised for this model |
| Tickets per Device | Fault rate normalised by fleet size |
| Most Common Fault | Top resolution category for this model |
| Avg. Resolution Time | Mean time to resolve for this model's tickets |
| Avg. Age at First Fault | Mean device age (months) when first hardware ticket was raised |
| PSLM Signal | 🔴 High fault rate / 🟡 Watch / 🟢 Normal |

This table directly informs the **Discover** stage by identifying device models that are underperforming relative to their fleet peers — actionable intelligence for the next hardware refresh procurement cycle.

```sql
-- Hardware failure rate by device model
SELECT
    dc.device_model,
    COUNT(DISTINCT dc.device_id) AS fleet_size,
    COUNT(t.issue_key) AS total_tickets_12m,
    ROUND(COUNT(t.issue_key)::numeric / NULLIF(COUNT(DISTINCT dc.device_id), 0), 2) AS tickets_per_device,
    MODE() WITHIN GROUP (ORDER BY t.resolution_category) AS most_common_fault,
    ROUND(AVG(EXTRACT(EPOCH FROM (t.resolved_at - t.created_at)) / 3600), 1) AS avg_resolution_hours
FROM device_context_cache dc
LEFT JOIN jsm_ticket_cache t ON t.device_id = dc.device_id
    AND t.created_at >= NOW() - INTERVAL '12 months'
    AND t.resolution_category ILIKE '%hardware%'
GROUP BY dc.device_model
ORDER BY tickets_per_device DESC;
```

#### 5.2.2 Vendor Defect Pattern Detection

Beyond per-model metrics, the dashboard identifies **cross-model patterns** attributable to specific vendors or component suppliers:

| Pattern | Detection Method | Discover/Design Action |
| --- | --- | --- |
| High NIC failure rate across multiple Dell models | `nic_error_count` spikes in `device_health_history` grouped by vendor | Review Dell NIC driver versions; raise with Dell account manager |
| Battery degradation faster than rated lifecycle for Lenovo models | Battery cycle count vs. age trend | Raise warranty claim pattern; adjust refresh cycle schedule |
| SSD failures clustering in devices with a specific firmware version | SMART event log + Intune `modelFirmwareVersion` join | Issue firmware update standard change; flag to procurement |
| USB hub failures concentrated at a single site | `device_context_cache` site field + USB event log | Investigate site power quality; raise facilities ticket |

These patterns are surfaced as **Problem Records suggestions**: when the dashboard detects a statistically significant pattern (using a simple chi-squared test for frequency anomalies), it displays a "Raise Problem Record" prompt that pre-populates a JSM problem record with the pattern evidence.

#### 5.2.3 Peripheral and Consumables Lifecycle Data

The PLA and PHMS data, previously used only for operational alerts, contributes to the PSLM dashboard as lifecycle intelligence:

- **Printer consumables consumption rate by model** — informs consumables procurement schedules
- **POS terminal fault frequency by terminal age** — informs POS hardware replacement scheduling
- **Printer recurrence rate by model** — identifies models with chronic reliability issues

#### 5.2.4 Support-to-Discover Intelligence Summary (Weekly Export)

The dashboard generates a weekly **Support-to-Discover Intelligence Summary** — a structured JSON payload that can be consumed by:

- Power BI (via the integration described in Section 7)
- Confluence (via the integration described in Section 7)
- Email digest to the IT Management distribution list

```json
{
  "week_ending": "2026-05-17",
  "hardware_fault_highlights": [
    {
      "device_model": "Dell Latitude 5440",
      "fault_rate_this_week": 0.12,
      "fault_rate_baseline_90d": 0.04,
      "delta": "+200%",
      "most_common_fault": "NIC Driver Error",
      "pslm_signal": "RED",
      "recommended_action": "Investigate NIC driver version; consider Discover-stage review"
    }
  ],
  "vendor_patterns": [...],
  "consumables_forecast": [...],
  "new_problem_records_suggested": 2
}
```

### 5.3 Governance: Closing the Loop

The PSLM Feedback Loop Dashboard is not merely a reporting tool — it must produce **traceable outcomes** to satisfy ITIL v5's requirement that Support-stage data genuinely influences upstream stages. The dashboard includes a **PSLM Action Register** panel:

| PSLM Action | Triggered By | Assigned To | Target Stage | Status |
| --- | --- | --- | --- | --- |
| Review Dell NIC driver policy | Vendor defect pattern (NIC, Q2 2026) | Desktop Engineering Lead | Design | In Progress |
| Update HP printer model on approved list | High fault rate (HP LaserJet M404) | Procurement Lead | Discover | Completed |
| Adjust Lenovo refresh cycle to 36 months | Battery lifecycle data | Asset Manager | Acquire | Not Started |

These action items are created as JSM tasks via the UIWP's standard "Raise JSM Task" API call, providing the full JSM workflow (assignment, progress tracking, SLA, approval) for PSLM actions.

---

## 6. Teams and Slack Bot Interface

### 6.1 Design Principles

The Teams/Slack bot is not a replacement for the UIWP web interface — it is a **tactical extension** that makes the most common, time-sensitive UIWP operations accessible without requiring the technician or manager to open a browser. Its scope is deliberately narrow:

> The bot **surfaces** status information and **triggers** standard, pre-approved actions. It does not grant arbitrary API access or bypass the UIWP's RBAC and audit log controls.

Every bot action maps to an existing UIWP API call that is already governed, rate-limited, and logged. The bot is a thin natural language front-end over those existing endpoints.

### 6.2 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│               Teams / Slack Channel or DM                   │
└──────────────────────────┬──────────────────────────────────┘
                           │ Message event (HTTPS POST)
┌──────────────────────────▼──────────────────────────────────┐
│                    BOT SERVICE                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Intent Classifier (LightGBM / rule-based fallback) │   │
│  │  Entity Extractor (device name, ticket ID, user)    │   │
│  └──────────────────────┬──────────────────────────────┘   │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐   │
│  │  Session Context Manager (per-user conversation)    │   │
│  └──────────────────────┬──────────────────────────────┘   │
│                         │                                   │
│  ┌──────────────────────▼──────────────────────────────┐   │
│  │  UIWP Internal API Client                           │   │
│  │  (calls existing UIWP backend endpoints with        │   │
│  │   bot service account credentials + user identity   │   │
│  │   forwarding for audit log attribution)             │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**The bot service is a new microservice** deployed alongside the UIWP backend. It handles:

1. Receiving message events from Teams (via Azure Bot Framework / Bot Service) or Slack (via Slack API Events API)
2. Classifying the intent of the message and extracting relevant entities
3. Calling the appropriate UIWP backend API endpoint
4. Formatting the API response as an adaptive card (Teams) or Block Kit message (Slack)
5. Returning the formatted response to the channel or DM

The bot authenticates to the UIWP backend using a dedicated **bot service account** with a fixed, minimal RBAC role. The user's identity is forwarded via a request header so that the UIWP audit log records the *human* who initiated the action, not the bot service account.

### 6.3 Intent Classification

For the initial release, a **hybrid intent classification** approach is used: a curated rule-based matcher handles the most common commands (avoiding ML dependency for straightforward cases), with an ML classifier as fallback for more natural language queries:

```python
INTENT_RULES = {
    r'\bdevice status\b.*\b(?P<device>[A-Z]{2,}-\d{4,})\b': 'get_device_status',
    r'\bticket\b.*\b(?P<ticket>ITSD-\d+)\b': 'get_ticket_status',
    r'\bopen tickets?\b.*\b(?P<user>[\w.]+@[\w.]+)\b': 'get_tickets_for_user',
    r'\bqueue (depth|count|size)\b': 'get_queue_depth',
    r'\bremediat\w+\b.*\b(?P<device>[A-Z]{2,}-\d{4,})\b': 'trigger_remediation',
    r'\bprinter\b.*\b(?P<printer>[\w-]+)\b.*\bstatus\b': 'get_printer_status',
    r'\bhealth\b.*\b(?P<device>[A-Z]{2,}-\d{4,})\b': 'get_device_health',
    r'\bmy tickets?\b': 'get_my_tickets',
    r'\bP1 incidents?\b': 'get_p1_incidents',
}
```

Unmatched messages fall through to the ML intent classifier (a fine-tuned `all-MiniLM-L6-v2` model, same as Section 3.2, but fine-tuned on a corpus of labelled bot command examples).

### 6.4 Supported Command Set

The Phase 4 bot supports the following operations in its initial release:

| Command Pattern | Intent | UIWP API Called | RBAC Requirement |
| --- | --- | --- | --- |
| `device status LAPTOP-0142` | `get_device_status` | `GET /api/devices/{id}/context` | Service Desk Agent |
| `ticket ITSD-4821` | `get_ticket_status` | `GET /api/tickets/{key}` | Service Desk Agent |
| `open tickets for john.smith@corp.com` | `get_tickets_for_user` | `GET /api/tickets?reporter={email}` | Service Desk Agent |
| `queue depth` | `get_queue_depth` | `GET /api/metrics/queue` | Any authenticated user |
| `P1 incidents` | `get_p1_incidents` | `GET /api/tickets?priority=1&status=open` | Service Desk Agent |
| `my tickets` | `get_my_tickets` | `GET /api/tickets?assignee={caller}` | Any authenticated user |
| `printer status PRINTER-SITE-B-01` | `get_printer_status` | `GET /api/printers/{id}` | Service Desk Agent |
| `device health LAPTOP-0142` | `get_device_health` | `GET /api/devices/{id}/health` | Service Desk Agent |
| `trigger remediation LAPTOP-0142` | `trigger_remediation` | `POST /api/devices/{id}/remediate` | Desktop Engineer (elevated) |
| `raise ticket for LAPTOP-0142` | `raise_ticket` | `POST /api/tickets` | Service Desk Agent |
| `XLA summary` | `get_xla_summary` | `GET /api/metrics/xla` | Service Desk Manager |
| `PSLM alerts` | `get_pslm_alerts` | `GET /api/pslm/alerts` | IT Management |

> ⚠️ **Write actions** (`trigger_remediation`, `raise_ticket`) require the user's UIWP RBAC role to permit the action. If the user lacks the required role, the bot returns a "You don't have permission to trigger this action" response and logs the attempt. The bot never silently ignores a permission boundary.

### 6.5 Confirmation Flows for Write Actions

Write actions require a confirmation step before the UIWP API is called. This is mandatory under ITIL v5's 6C Communication requirement (users are explicitly informed when actions are automated vs. manually initiated):

```
User:  trigger remediation LAPTOP-0142

Bot:   ⚠️ Confirm action
       Trigger Intune Remediation: NIC Driver Fix
       Device: LAPTOP-0142 (John Smith)
       Current health score: 61 (declining trend)
       
       This will run the "NIC Driver Fix" remediation script
       via Microsoft Intune. The device will not restart
       automatically.
       
       [Confirm]  [Cancel]

User:  [clicks Confirm]

Bot:   ✅ Remediation triggered
       Intune script "NIC Driver Fix" queued for LAPTOP-0142.
       Reference: UIWP-AUDIT-20260519-4821
       
       The script typically completes within 5–10 minutes.
       You can check status with: device status LAPTOP-0142
```

### 6.6 Microsoft Teams Implementation

Teams bot registration uses **Azure Bot Framework** with an App Registration in Entra ID:

1. Register a new Bot in Azure Bot Service (or Azure AI Bot Service)
2. Create an Entra ID App Registration for the bot with scopes limited to the UIWP API
3. Configure the bot service URL to point to the UIWP bot microservice endpoint
4. Deploy the bot as a Teams app via Microsoft Teams Admin Centre (or App Studio)
5. Configure proactive notifications: the UIWP backend can POST alerts to a Teams channel via the Bot Framework proactive messaging API when critical events occur (P1 incident, trend alert, PSLM RED signal)

**Proactive notification example — P1 incident:**
```
🚨 P1 Incident Raised — ITSD-5012
Summary: Network switch failure — Site B — affecting 47 users
Raised: 2026-05-19 09:14 UTC
Assigned: Network Engineering
Health Impact: POS terminals at Site B offline

[View in UIWP]  [Add Comment]
```

### 6.7 Slack Implementation

Slack bot registration uses the **Slack API** with a Slack App:

1. Create a Slack App at api.slack.com/apps
2. Enable Events API; set request URL to the bot microservice `/slack/events` endpoint
3. Enable Slash Commands (e.g., `/uiwp device status LAPTOP-0142`) as a more explicit alternative to natural language
4. Enable Incoming Webhooks for proactive notifications from the UIWP backend
5. Use Block Kit to format responses consistently

Slash commands provide a discoverable, consistent interface that is preferred over relying entirely on natural language recognition for operational use cases.

---

## 7. Power BI and Confluence Export Integration

### 7.1 Management Reporting Requirements

IT management typically requires two types of reporting not well served by the UIWP's real-time operational views:

1. **Periodic narrative reports** — monthly/quarterly summaries presented to stakeholders in a document format that can be commented on, distributed, and archived (Confluence)
2. **Trend dashboards for non-technical stakeholders** — visual analytics in a tool familiar to business analysts and executives without UIWP access (Power BI)

Phase 4 adds both channels without requiring management to log into the UIWP or manually export data.

### 7.2 Power BI Integration

**Architecture: Push Dataset + Scheduled Refresh**

The UIWP backend exposes a **reporting data export endpoint** that serialises the key XLA and operational metrics into a structured JSON payload. Power BI connects to this endpoint in one of two modes:

**Mode A — Power BI REST API Push Dataset (recommended for real-time dashboards):**

The UIWP backend publishes metric snapshots to a **Power BI Push Dataset** via the Power BI REST API on a scheduled basis (hourly or daily, depending on the metric):

```typescript
// UIWP backend: Power BI push dataset publisher
async function pushMetricsToPowerBI(metrics: UiwpMetricsSnapshot): Promise<void> {
  const accessToken = await getPowerBIAccessToken(); // via MSAL client credentials
  const datasetId = config.powerbi.datasetId;
  
  await axios.post(
    `https://api.powerbi.com/v1.0/myorg/datasets/${datasetId}/rows`,
    {
      rows: [
        {
          snapshot_date: metrics.date,
          total_tickets_open: metrics.queueDepth,
          p1_count: metrics.p1Count,
          avg_resolution_hours: metrics.avgResolutionHours,
          first_contact_resolution_rate: metrics.fcrRate,
          ues_score: metrics.uesScore,
          recurrence_rate: metrics.recurrenceRate,
          automated_remediation_rate: metrics.autoRemediationRate,
          devices_at_risk: metrics.devicesAtRisk,
          pslm_red_signals: metrics.pslmRedSignals
        }
      ]
    },
    { headers: { Authorization: `Bearer ${accessToken}` } }
  );
}
```

**Mode B — Power BI Dataflow / OData feed (recommended for historical analysis):**

The UIWP backend exposes an OData-compatible endpoint (`/api/export/odata`) that Power BI can connect to directly using the **OData** connector in Power Query. This allows Power BI report authors to build custom queries against the full UIWP data model without requiring the UIWP team to pre-define every report.

**Required Power BI App Registration scopes:**
- `Dataset.ReadWrite.All` (for Push Dataset mode)
- The UIWP OData endpoint is protected by the same Entra ID auth as the main UIWP

**Recommended Power BI report structure for ITIL v5 reporting:**

| Report Page | Key Visuals | Data Source |
| --- | --- | --- |
| Executive Summary | Total tickets this month, XLA score, FCR rate, top 3 failure categories | XLA cache |
| Ticket Volume & Trends | Line chart: tickets by week × category; heat map: tickets by day of week × hour | JSM ticket cache |
| Device Health Overview | Bar chart: devices by health band; trend line: fleet average health score | Device health history |
| PSLM Signals | Table: hardware failure rates by model; vendor defect alerts | PSLM feedback data |
| Technician Performance | Avg resolution time by assignee; SLA compliance rate (anonymised or role-visible) | JSM ticket cache |
| Automation Effectiveness | Remediation actions triggered; auto-resolved vs. escalated | Audit log |

### 7.3 Confluence Export Integration

**Architecture: Confluence REST API + Scheduled Page Updates**

The UIWP backend uses the **Confluence REST API** to create and update report pages in a designated Confluence space (e.g., `IT Service Management > Monthly Reports`).

```typescript
// UIWP backend: Confluence report publisher
async function publishMonthlyReportToConfluence(
  report: MonthlyReport,
  spaceKey: string,
  parentPageId: string
): Promise<void> {
  const pageTitle = `Service Desk Report — ${report.monthYear}`;
  const confluenceBase = config.confluence.baseUrl;
  const auth = Buffer.from(`${config.confluence.email}:${config.confluence.apiToken}`).toString('base64');

  // Check if the page already exists
  const searchRes = await axios.get(
    `${confluenceBase}/rest/api/content?title=${encodeURIComponent(pageTitle)}&spaceKey=${spaceKey}`,
    { headers: { Authorization: `Basic ${auth}` } }
  );

  const pageContent = renderReportAsConfluenceStorageFormat(report);

  if (searchRes.data.results.length > 0) {
    // Update existing page
    const existingPage = searchRes.data.results[0];
    await axios.put(
      `${confluenceBase}/rest/api/content/${existingPage.id}`,
      {
        version: { number: existingPage.version.number + 1 },
        title: pageTitle,
        type: 'page',
        body: { storage: { value: pageContent, representation: 'storage' } }
      },
      { headers: { Authorization: `Basic ${auth}`, 'Content-Type': 'application/json' } }
    );
  } else {
    // Create new page
    await axios.post(
      `${confluenceBase}/rest/api/content`,
      {
        type: 'page',
        title: pageTitle,
        space: { key: spaceKey },
        ancestors: [{ id: parentPageId }],
        body: { storage: { value: pageContent, representation: 'storage' } }
      },
      { headers: { Authorization: `Basic ${auth}`, 'Content-Type': 'application/json' } }
    );
  }
}
```

**Report content (rendered as Confluence Storage Format):**

The `renderReportAsConfluenceStorageFormat` function converts the `MonthlyReport` object into Confluence's XML-based storage format, producing a page containing:

1. **Executive summary section** — Key metrics table (total tickets, FCR, XLA score, remediation rate) generated from UIWP metric cache
2. **Ticket volume and trends** — Inline data table (Confluence macros for charts are not used due to data-binding complexity; instead, a table of weekly volumes is provided alongside a linked Power BI dashboard URL)
3. **Top issues this month** — Auto-generated list of the top 5 resolution categories by volume, with YoY or MoM comparison
4. **PSLM signals** — Hardware failure trend table and vendor pattern highlights from the PSLM dashboard
5. **Actions raised from PSLM data** — The PSLM Action Register entries from this period (see Section 5.3)
6. **Next month focus** — Template section for the service desk manager to add manual commentary

The monthly report page is published on the **first business day of each month** via a scheduled BullMQ/Celery task, covering the prior calendar month's data.

### 7.4 Export Governance

Both the Power BI and Confluence export integrations are subject to the UIWP's data governance framework (Section 7.3 of the Architecture paper):

| Governance Concern | Control |
| --- | --- |
| Personal data in exports | XLA and reporting exports use **role-aggregate data** only; per-user data (e.g., tickets by user) is suppressed in exports or pseudonymised unless explicitly authorised by the DPIA |
| Confluence access control | Report pages published to a Confluence space accessible only to IT Management and Service Desk Manager roles; individual technician-level metrics are not included |
| Power BI access control | Power BI workspace shared with named IT Management group; dataset permissions controlled via Power BI workspace roles |
| API token security | Confluence API token and Power BI client secret stored in Azure Key Vault / HashiCorp Vault; rotated per the credential rotation schedule in Section 4.5 of the Architecture paper |
| Audit trail | Every export action logged to `audit_log` with timestamp, target system, and data range |

---

## 8. Phase 4 Implementation Sequence

Phase 4 spans months 13–24. The following sequence prioritises value delivery while managing the change risk of introducing AI capabilities:

### Month 13–15: Data Foundation and Classifier Training

- [ ] Audit resolution category field completeness across all JSM tickets; enforce mandatory field via JSM configuration change (Normal Change)
- [ ] Build and test the weekly ticket data export pipeline
- [ ] Train initial ticket classifier using months 1–12 historical data
- [ ] Validate classifier performance (weighted F1 ≥ 0.70) against held-out test set
- [ ] Internal review by Service Desk Manager: review model predictions on 200 sample historical tickets
- [ ] CAB Normal Change: approve deployment of ticket classifier to UIWP ticket queue (canary mode, 10% of queue)
- [ ] Run 4-week canary with override rate monitoring

### Month 16–18: Classification Full Deploy + Trend Alerting

- [ ] Full deploy of ticket classifier to 100% of queue (confirmed via normal change after canary review)
- [ ] Build and test device health time-series export pipeline
- [ ] Train Isolation Forest anomaly detector on months 1–15 device health data
- [ ] Train failure predictor binary classifier; validate precision/recall against acceptable thresholds
- [ ] Deploy trend alert indicators to UIWP device context panel and endpoint health heatmap view
- [ ] 4-week review: service desk manager and desktop engineering lead review trend alert precision and actionability

### Month 19–21: PSLM Dashboard and Reporting Exports

- [ ] Build PSLM Feedback Loop Dashboard (hardware model table, vendor pattern detection, consumables data)
- [ ] Implement PSLM Action Register panel and JSM task creation from dashboard
- [ ] Normal Change: deploy PSLM dashboard to IT Management and Procurement Lead UIWP roles
- [ ] Implement Power BI Push Dataset export; build initial Power BI report template
- [ ] Implement Confluence monthly report publisher; publish first automated report
- [ ] Review automated report accuracy with IT Management — adjust metric selection and commentary template as needed

### Month 22–24: Bot Interface

- [ ] Register Teams bot application in Azure Bot Service and Entra ID
- [ ] Build bot microservice: intent classifier, entity extractor, UIWP API client, Teams adaptive card renderer
- [ ] Implement and test all Tier 1 command set (read-only queries)
- [ ] Normal Change: deploy read-only bot to service desk team channel (pilot group: 5 technicians)
- [ ] 4-week pilot review; expand to all service desk agents
- [ ] Implement write actions (`trigger_remediation`, `raise_ticket`) with confirmation flows
- [ ] Normal Change (medium risk, full CAB): deploy write-action capabilities
- [ ] Optional: Register Slack bot (if organisation uses Slack) using same microservice with Slack-specific event handling

---

## 9. Governance and ITIL v5 Alignment

### 9.1 6C AI Capability Model Audit for Phase 4

Each Phase 4 capability must be assessed against the 6C AI Capability Model before deployment:

| Capability | Creation | Curation | Clarification | Cognition | Communication | Coordination |
| --- | :---: | :---: | :---: | :---: | :---: | :---: |
| Ticket Classifier | ✅ Generates suggestions | ✅ Human review gate; confidence thresholds | ✅ Confidence scores; override logging | ✅ LightGBM + embeddings | ✅ Suggestions labelled as AI | ✅ Override feeds retraining |
| KB Recommender | ✅ Retrieves articles | ✅ Frequency-based ranking | ✅ Article shown with link count context | ✅ Retrieval from historical data | ✅ Shown as "Suggested" | ✅ Click-through logged |
| Resolution Time Predictor | ✅ Generates estimate | ✅ Shown as range; suppressed below threshold | ✅ Range format communicates uncertainty | ✅ Regression on historical data | ✅ Labelled as estimate | ⚠️ Not decision-binding |
| Trend-Based Health Alerts | — | ✅ Alert taxonomy; graduated confidence | ✅ Trend slope shown with alert | ✅ Linear trend + Isolation Forest | ✅ Alert type clearly distinguished from threshold alerts | ✅ Proactive ticket suggestion |
| PSLM Dashboard | — | ✅ Pattern detection threshold; human action required | ✅ Statistical basis shown | ✅ Chi-squared anomaly detection | ✅ PSLM action is always human-initiated | ✅ Action Register links to JSM tasks |
| Teams/Slack Bot | — | ✅ Write-action confirmation required | ✅ Ambiguous intent prompted for clarification | ✅ ML intent classification | ✅ Bot identity declared; every response shows what was done | ✅ RBAC enforced; audit log attributed to human |

### 9.2 Change Management Requirements

| Phase 4 Capability | ITIL v5 Change Type | CAB Gate |
| --- | --- | --- |
| Ticket classifier (canary, read-only suggestions) | Normal Change (low risk) | Technical review only |
| Ticket classifier (full deploy) | Normal Change (low risk) | Technical review only |
| Write-action bot capabilities | Normal Change (medium risk) | Full CAB + 6C checklist |
| PSLM Problem Record auto-suggest | Normal Change (low risk) | Technical review only |
| Power BI push dataset | Normal Change (low risk) | Technical review only |
| Confluence report publisher | Normal Change (low risk) | Technical review only |
| Failure predictor deployed to alert surface | Normal Change (medium risk) | Full CAB + 6C checklist |

### 9.3 Model Versioning and Rollback

Every trained model is versioned and stored alongside its evaluation metrics in the UIWP model registry (a designated path in the application's object storage or a lightweight MLflow tracking server):

```
model_registry/
├── ticket_classifier/
│   ├── v1.0.0_20260601/
│   │   ├── model.pkl
│   │   ├── encoder.pkl
│   │   ├── evaluation_metrics.json    # F1: 0.74; accuracy: 0.78
│   │   └── training_data_snapshot.csv.sha256
│   └── v1.1.0_20260701/
│       └── ...
├── device_health_trend/
│   └── ...
└── failure_predictor/
    └── ...
```

If a deployed model produces a significant increase in analyst override rate (> 40% for any category over a 2-week window), the UIWP automatically rolls back to the previous model version and raises a JSM incident for the Phase 4 ML team to investigate.

---

## 10. Technology Stack Additions for Phase 4

The following additions extend the technology stack specified in Section 8 of the UIWP Architecture paper:

| Component | Technology | Purpose |
| --- | --- | --- |
| **Text embedding** | `sentence-transformers` (Python) / `@xenova/transformers` (Node.js) | Ticket text vectorisation for classifier and bot intent |
| **ML framework** | `scikit-learn` + `LightGBM` (Python) | Ticket classifier, health trend regression, failure predictor |
| **Anomaly detection** | `scikit-learn` IsolationForest | Component-level device health anomaly detection |
| **Time-series store** | TimescaleDB (PostgreSQL extension) or existing Elasticsearch | Device health score time-series at daily resolution |
| **Model registry** | MLflow (self-hosted) or simple versioned object storage | Model versioning, metrics tracking, rollback capability |
| **Bot framework (Teams)** | Azure Bot Framework SDK (Node.js or Python) | Teams bot event handling and card rendering |
| **Bot framework (Slack)** | Slack Bolt SDK (Node.js or Python) | Slack event handling and Block Kit rendering |
| **Power BI integration** | Microsoft Power BI REST API (via MSAL) | Push dataset publishing; OData feed |
| **Confluence integration** | Atlassian Confluence REST API v2 | Monthly report page creation and update |

All Phase 4 components are deployed as **additional microservices** in the existing Docker/Kubernetes deployment, independently scalable and independently deployable. The bot service, ML inference service, and reporting export service each run in their own container and communicate with the UIWP backend via its existing internal REST API.

---

## 11. Measuring Phase 4 Outcomes

The following metrics should be tracked to evaluate whether Phase 4 is delivering its intended value:

| Outcome | Metric | Target | Measurement Source |
| --- | --- | --- | --- |
| Ticket classifier accuracy | Weighted F1 score | ≥ 0.70 sustained | Model evaluation pipeline |
| Classifier adoption | % tickets where AI suggestion accepted without override | ≥ 60% | `audit_log` (override events) |
| KB recommendation relevance | % suggested KB articles rated useful by technician | ≥ 65% | UIWP KB feedback widget |
| Resolution time prediction | Mean absolute error vs. actual (hours) | ≤ 4 hours | Model evaluation pipeline |
| Predictive alert precision | % trend alerts followed by a hardware ticket within 30 days | ≥ 50% | `alert_state` × `jsm_ticket_cache` join |
| PSLM action close rate | % PSLM-triggered actions completed within SLA | ≥ 75% | PSLM Action Register JSM tasks |
| Bot adoption | Active bot users as % of service desk team | ≥ 70% | Teams/Slack bot usage telemetry |
| Reporting efficiency | Hours saved on monthly report preparation | ≥ 3 hours/month | Service Desk Manager self-report |
| Overall Phase 4 XLA impact | UES score change from Phase 3 baseline | +5 points | XLA Reports view |

These metrics are added to the UIWP's XLA Reports view in a dedicated **Phase 4 AI Performance** sub-section and included in the Confluence monthly report from Month 22 onward.

---

## 12. Conclusion

Phase 4 is the stage at which the UIWP becomes more than the sum of its integrations. The operational data accumulated through 12 months of Phase 1–3 operation represents a structured, correlated, multi-source history of every device fault, every technician action, every endpoint degradation event, and every resolution path in the organisation. Phase 4 puts that history to work.

The five capabilities specified in this paper — ticket classification, predictive health alerting, PSLM feedback loops, the Teams/Slack bot, and management reporting exports — are each independently valuable. Together, they complete the transformation of the UIWP from an operational interface into a **learning service management platform**: one that improves its own accuracy as more data accumulates, that surfaces intelligence at the right time in the right channel for each role, and that closes the ITIL v5 PSLM loop by ensuring that the signals captured at the Support stage genuinely influence decisions at the Discover and Design stages.

Critically, all five capabilities are governed under ITIL v5's 6C AI Capability Model. Every model has a confidence threshold below which it is silent. Every write action has a confirmation step. Every model version is tracked and rollback-capable. Human decision-making is never replaced — it is augmented, accelerated, and better informed.

The IASD model, as now fully specified across all seven papers in this series, is an end-to-end system: from the foundational ITIL v5 concepts, through the active monitoring architecture, the custom software implementations, the ITSM integrations, the change management governance, the unified platform, and finally the AI-powered analytics layer that makes the platform a continuously improving operational asset.

---

## Appendix A — Phase 4 Data Schema Extensions

The following tables are added to the UIWP PostgreSQL data tier in Phase 4:

```sql
-- Model registry table
CREATE TABLE model_registry (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    model_name      TEXT NOT NULL,              -- 'ticket_classifier', 'failure_predictor', etc.
    model_version   TEXT NOT NULL,              -- 'v1.0.0_20260601'
    trained_at      TIMESTAMPTZ NOT NULL,
    f1_score        NUMERIC(4,3),
    accuracy        NUMERIC(4,3),
    mae_hours       NUMERIC(6,2),               -- for regression models
    is_active       BOOLEAN NOT NULL DEFAULT FALSE,
    storage_path    TEXT NOT NULL,
    UNIQUE (model_name, model_version)
);

-- AI suggestion log (for override rate tracking and feedback loop)
CREATE TABLE ai_suggestion_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    ticket_key      TEXT NOT NULL,
    suggested_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    model_version   TEXT NOT NULL,
    predicted_category TEXT,
    confidence      NUMERIC(4,3),
    suggested_kb_article TEXT,
    predicted_resolution_hours NUMERIC(6,2),
    analyst_accepted    BOOLEAN,           -- NULL if not yet actioned
    analyst_override    TEXT,              -- actual category if overridden
    actioned_at     TIMESTAMPTZ
);

-- PSLM action register
CREATE TABLE pslm_action_register (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    detected_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    pattern_type    TEXT NOT NULL,         -- 'hardware_fault_trend', 'vendor_defect', etc.
    affected_model  TEXT,
    affected_vendor TEXT,
    evidence_summary TEXT NOT NULL,
    pslm_stage      TEXT NOT NULL,         -- 'Discover', 'Design', 'Acquire'
    assigned_to_role TEXT NOT NULL,
    jsm_task_key    TEXT,                  -- linked JSM task
    status          TEXT NOT NULL DEFAULT 'not_started',
    completed_at    TIMESTAMPTZ
);

-- Bot interaction log
CREATE TABLE bot_interaction_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    platform        TEXT NOT NULL,         -- 'teams', 'slack'
    user_id         TEXT NOT NULL,         -- platform user ID
    uiwp_user_id    UUID REFERENCES users(id),
    message_text    TEXT NOT NULL,
    classified_intent TEXT,
    extracted_entities JSONB,
    action_taken    TEXT,
    action_result   TEXT,
    latency_ms      INTEGER,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## Appendix B — Recommended Reading

- ITIL Version 5 Foundation Guide (Axelos, 2026)
- ITIL v5 Practice Manager: Monitor, Support and Fulfil (MSF) Module (Axelos, 2026)
- ITIL v5 AI Governance Supplement: 6C AI Capability Model in Practice (Axelos, 2026)
- Sentence Transformers documentation: `sbert.net`
- LightGBM documentation: `lightgbm.readthedocs.io`
- Azure Bot Framework documentation: `learn.microsoft.com/azure/bot-service`
- Slack Bolt SDK documentation: `slack.dev/bolt-python`
- Microsoft Power BI REST API documentation: `learn.microsoft.com/power-bi/developer/embedded/rest-api-reference`
- Atlassian Confluence REST API documentation: `developer.atlassian.com/cloud/confluence/rest`
- Scikit-learn Isolation Forest documentation: `scikit-learn.org`

---

*This paper is the seventh entry in the ITIL v5 Practical Solutions series. It should be read in conjunction with the six preceding papers, which provide the complete IASD model and UIWP architecture that Phase 4 builds upon.*

---

<div align="center">

**ITIL v5 Practical Solutions Series** · May 2026
*Papers: Foundation Training · Active Service Desk · Custom Software · Jira & Intune Integration · Change Management · Unified Platform · Phase 4 AI & Analytics*

</div>
