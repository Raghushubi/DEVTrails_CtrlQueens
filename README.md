# DEVTrails_CtrlQueens: VeriPay
### AI-Enabled Parametric Income Protection for Gig Workers

**Team:** CtrlQueens
**Hackathon:** Guidewire DEVTrails 2026

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Persona and Scenario](#3-persona-and-scenario)
4. [How VeriPay Works](#4-how-veripay-works)
5. [Core Architecture](#5-core-architecture)
6. [Adversarial Defense and Anti-Spoofing Strategy](#6-adversarial-defense-and-anti-spoofing-strategy)
7. [Pricing Engine](#7-pricing-engine)
8. [User Experience Design](#8-user-experience-design)
9. [Technical Architecture](#9-technical-architecture)
10. [Implementation Plan](#10-implementation-plan)
11. [Scope Exclusions](#11-scope-exclusions)
12. [Competitive Differentiation](#12-competitive-differentiation)
13. [Appendix: Device Signal Schema](#13-appendix-device-signal-schema)

---

## 1. Project Overview

VeriPay is a parametric income protection product designed specifically for food delivery partners on Zomato and Swiggy platforms in India. In case of an external shock such as heavy rain, high pollution, platform downtime, or civic events that prevent a delivery partner from earning, VeriPay automatically detects such events and sends money directly to the food delivery partner without requiring a claim to be filed.

VeriPay is a completely parametric product, and food delivery partners do not need to prove their income loss in terms of rupees and cents. Using external data and platform data, income loss is estimated through a pre-agreed formula.

**Design philosophy:** VeriPay is designed to ensure minimum basis risk, maximum trust among the workers, and resistance to fraud without compromising the accessibility of the system. The system is designed to prevent "coordinated payout spikes" that drain the liquidity pool by using "multi-signal validation," "graded payouts," and "group-level anomaly detection."

**Core promise:** If you were actively trying to earn and the system can verify you were disrupted, you get paid automatically.

**Example:** Rani usually works Friday evenings in Zone A and earns around ₹180 per hour. A sudden cloudburst crosses the rainfall threshold and platform order volume drops 60%. Her effort score is high — she was online before and during the event. Fraud checks pass. ₹320 reaches her UPI within 30 minutes. No forms, no calls, no waiting.

---

## 2. Problem Statement

India's gig delivery workforce operates without income continuity protections. The structural gaps are:

- No sick pay, no weather pay, no platform downtime compensation
- Traditional insurance requires lengthy claims processes that workers cannot afford in time or documentation
- Existing parametric products pay flat rates regardless of actual lost opportunity, causing systematic over- or under-compensation
- Workers in high-risk zones (flood-prone areas, high-AQI corridors) are either priced out or excluded
- Fraud is a documented and growing problem: GPS spoofing, coordinated Telegram-based fraud rings, and fake inactivity claims are active attack vectors in the Indian gig economy

---

## 3. Persona and Scenario

**Primary persona:** Food delivery partners on Zomato and Swiggy operating in tier-1 Indian cities.

**Why this segment:** They are the ones who experience the strongest effects of weather-related disruptions (orders drop sharply for them when it rains), who are most directly impacted by platform outages, and who work in dense urban areas where civic outages such as a strike in their neighborhood or a curfew can instantly shut off access to restaurants and customers. Their earnings are the ones that are most time-sensitive like a rainstorm for two hours on a Friday night can comprise 30–40% of their earnings for the day.

**Typical worker profile:**
- Works 6–8 hours per day across 2–3 time bands (morning, evening peak, late night)
- Earns ₹800–₹1,400 per day depending on zone, time, and platform incentives
- Operates across 2–4 zones within a city
- Pays for fuel, phone data, and vehicle maintenance from earnings — no buffer for zero-income days

**Workflow for a covered week:**

1. Worker onboards via mobile app: completes KYC stub, links UPI, selects home zone and typical working hours
2. Each Sunday, reviews auto-generated weekly premium quote and activates coverage with one tap
3. During the week, the app runs silently in the background, sending periodic device heartbeats
4. If a disruption triggers in their zone during a covered window, the system processes automatically
5. Worker receives a push notification with payout confirmation or a soft-review prompt if signals are unclear
6. All decisions and breakdowns are visible in the worker's dashboard at any time

---

## 4. How VeriPay Works

Workers subscribe every Sunday. They choose the types of hazards they are interested in (weather, pollution, platform outage, civic disruption). The system continuously monitors external data sources. When a disruption of the relevant type occurs in the active zone of the worker, the system:

1. Confirms the parametric trigger threshold was crossed
2. Validates the zone-level earnings index dropped materially
3. Checks whether the worker was actively trying to work (effort condition)
4. Cross-references market supply and demand to confirm a real disruption occurred
5. Runs individual fraud signal scoring
6. Checks group-level fraud via graph analysis
7. Calculates a graded payout proportional to disruption severity
8. Initiates automatic UPI payout if all checks pass

The entire pipeline runs without worker intervention for legitimate cases.

---

## 5. Core Architecture

### 5.1 Parametric Trigger Engine

Triggers are based on objective, publicly verifiable indexes — not self-reported claims.

| Trigger Type | Category | Threshold | Data Source |
|---|---|---|---|
| Rainfall | Environmental | Zone-specific mm threshold | IMD / Open-Meteo API |
| Air Quality | Environmental | AQI > 250 sustained 2+ hours | CPCB / AQI India API |
| Extreme Heat | Environmental | Temperature > 42°C sustained 2+ hours | IMD / Open-Meteo API |
| Platform Outage | Technical | Activity index drops 40%+ vs. baseline | Platform API (mocked) |
| Civic Disruption | Social | Zone access blocked: curfew, strike, or emergency closure | Traffic / civic alert API (mocked) |

Zone granularity is sub-city level (5–10 km radius clusters). Triggers are evaluated per zone per event window, not city-wide. A localized storm does not generate payouts across an entire metropolitan area.

The platform outage baseline is calculated as a day of week and hour of day-adjusted rolling average over the past 4 weeks, with a minimum sustained time of 20 minutes before the trigger fires. This prevents brief API issues from being treated as qualifying events.

---

### 5.2 Income Loss Modeling

A key failure of generic parametric products is paying a flat rate regardless of actual lost opportunity. VeriPay uses a **Counterfactual Income Loss** model instead.

#### Expected Earnings Corridor (EEC)

For each worker and each event window, the system computes expected earnings using:

- The worker's historical earnings rate for that zone, day-of-week, and hour-of-day band (rolling 4-week window)
- Zone-level demand index: median earnings per active hour for comparable workers in normal conditions
- Platform incentive layer: active surge or bonus multipliers during the window (mocked)

**Cold-start handling:** Those workers with less than 4 weeks of history will use the zone's median earnings for that particular time band. These accounts will also be set to the Bronze trust tier, causing them to receive a higher number of soft review prompts. This ensures that these accounts are taken care of from the first day.

**Payout formula:**
```
Payout = min(Cap, Expected Earnings/hr × Eligible Hours × Zone Loss Factor)
```

Zone Loss Factor is not a binary factor but a graded one. A 30% drop in the zone earnings index means there is a 0.3 loss factor, and a 70% drop means there is a 0.7 loss factor. This means that payments will vary according to the actual extent of disruption, rather than being triggered as soon as any disruption occurs. This prevents cliff-edge payments, where a small difference in rainfall leads to a 100% difference in payout costs.

A disruption on a Friday night, as opposed to one on a Tuesday morning, will result in a much higher payout due to the expected earning potential. This prevents systematic overpayment for part-time workers.
---

### 5.3 Disruption Validation Layer

A trigger event does not automatically confirm that the disruption prevented work. VeriPay adds a market equilibrium check before finalizing any payout decision.

When a parametric trigger fires, the system queries the platform activity API for the zone's supply-demand state during the event window:

- If active partners remain at ≥ 80% of historical average AND order volume is normal → disruption did not prevent work → payout denied
- If supply is low AND demand is also low → slow day, not a disruption → payout denied
- Payout approved only when supply drops significantly AND the zone earnings index confirms a demand-side disruption

This prevents a valid weather event that had no measurable economic impact from triggering payouts at scale.

---

### 5.4 System Resilience and Fallback Logic

Production systems face data gaps. VeriPay handles them explicitly rather than failing silently.

| Failure Scenario | Fallback Behavior |
|---|---|
| Weather API unavailable | Use last known reading + reduce payout confidence score; route to soft review |
| Platform activity API missing | Skip DisruptionValidator check; apply conservative Zone Loss Factor cap of 0.5 |
| Conflicting GPS and cell-tower signals | Do not auto-deny; route to soft review with explanation to worker |
| New worker with no earnings history | Default to zone median earnings for that time band (cold-start handling) |
| Graph analysis incomplete at batch time | Individual fraud score used as sole basis; Ring Score treated as neutral |

The system defaults toward soft review rather than hard denial when data is incomplete. Denying a legitimate claim due to a data pipeline failure is treated as a worse outcome than routing it to human review.

---

## 6. Adversarial Defense and Anti-Spoofing Strategy

This directly addresses the coordinated spoofing attack described in the 24-hour market shift scenario.

### 6.1 The Differentiation: Genuine Worker vs. Spoofer

A genuine stranded delivery worker and a fraud actor sitting at home will produce fundamentally different signal profiles, even if their GPS coordinates appear identical.

| Signal Dimension | Genuine Stranded Worker | GPS Spoofer at Home |
|---|---|---|
| GPS vs. cell tower | Consistent — both place them in the rain zone | Mismatch — GPS spoofed to zone, cell tower places them elsewhere |
| Device temperature | Elevated — phone warm from outdoor use, charging on bike | Low — phone cold, plugged into wall charger |
| Route history | Plausible movement: shelter-seeking micro-movements, speed drops | Static or perfectly straight GPS trace with no turns |
| App state | Delivery app foregrounded, recent order interactions | App open but no UI interaction, screen dimmed |
| Accelerometer | Jittery — movement consistent with being on a bike or walking | Flat — device sitting on a desk |
| Cell tower history | Tower transitions consistent with zone geography | No tower transitions or transitions inconsistent with claimed location |

No single signal is treated as a binary kill-switch. The system requires multiple signals to align before escalating a fraud score. This is deliberate: a genuine worker in a poor-signal area may fail one or two checks without being a fraudster.

### 6.2 The Data: Detecting Coordinated Fraud Rings

Coordinated rings exploit the fact that each individual member can look legitimate in isolation. VeriPay detects them by analyzing relationships between users, not just individual profiles.

**What the system looks for across a group of payout candidates during a single trigger event:**

- Multiple accounts sharing the same hashed Wi-Fi environment fingerprint (same physical location) while claiming to be spread across a zone
- Synchronized login and logout timestamps across accounts within narrow time windows around the trigger — behavior consistent with a coordinated script, not independent workers
- Shared device IDs, SIM card hashes, or UPI handles across multiple user accounts
- Spike in new account registrations (under 14 days old) all active during the same trigger event
- Multiple accounts showing identical GPS noise signatures — a fingerprint of the same spoofing application running on different devices
- Accounts that appear only during high-payout trigger windows and are otherwise inactive

**Graph construction:** At each trigger event, a fraud graph is built where nodes are payout-eligible users and edges are weighted by the shared entities above. Connected components are analyzed for size, shared-entity concentration, and synchronization score. A high Ring Score flags the entire component for review, not just individual members.

**Why this catches rings that individual checks miss:** A fraud ring member is designed to look individually legitimate. The ring score catches the collective behavior — 50 accounts in one apartment cannot individually explain why they all share the same Wi-Fi hash, same device noise pattern, and logged in within 30 seconds of each other.

### 6.3 The UX Balance: Flagged Claims Without Punishing Honest Workers

The hardest design problem in fraud detection is minimizing false positives. A genuine worker experiencing a network drop in bad weather may fail some of the same individual checks as a spoofer.

VeriPay's response to this is a tiered workflow, not a binary block:

**Soft review (Fraud Score 0.3–0.7):**
- Payout is not denied. It is held for up to 24 hours.
- The worker receives a specific, plain-language notification explaining which signals were unclear (e.g., "We couldn't confirm your location matched your usual cell tower. Keep your delivery app open for 10 more minutes to release your payout automatically.")
- The worker can resolve most soft flags passively by remaining active on the platform.
- This creates a natural filter: legitimate workers stay online and resolve the flag; fraudsters running scripts or sitting at home cannot replicate the persistence of genuine work behavior.

**Hard block (Fraud Score > 0.7 or high Ring Score):**
- Payout is denied with a specific reason.
- Worker is shown their activity log for the event window and offered a single-tap appeal that creates a review record with full signal evidence attached.
- The appeal is reviewed against the evidence — workers with a clean history who were caught by a noisy data point are recovered without friction.

**Trust tier protection for honest workers:**
- Workers at Silver or Gold tier receive a higher fraud score tolerance before being routed to soft review, because their history provides context that a new account cannot.
- A one-time flag does not downgrade a worker's tier. Sustained patterns do.

---

## 7. Pricing Engine

### 7.1 Hybrid Premium Model

Pure dynamic pricing creates two problems: volatility that erodes worker trust, and adverse selection where workers only buy coverage in high-risk weeks. VeriPay uses a two-component hybrid model.

| Component | Definition | Update Frequency |
|---|---|---|
| Zone Risk Base Rate | Historical event frequency per zone over rolling 3 months | Monthly |
| Weekly Volatility Load | 0–20% surcharge based on 7-day forecast; capped to prevent gouging | Weekly (±10% cap) |
| Trust Tier Discount | Bronze / Silver / Gold tiers based on clean event history and behavioral consistency | Rolling (event-based) |

**Formula:**
```
Weekly Premium = Base Rate + Volatility Load - Trust Discount
```

Week-on-week change is capped at 10% to prevent sticker shock. Additionally, total weekly premium is capped at 5% of the worker's expected weekly earnings for their zone and typical hours, ensuring the product never prices out the workers it is designed to protect.

### 7.2 Trust Tiers and No-Claim Bonus

Trust tiers are determined by clean payout history, consistently low fraud scores, and stable working patterns — the same logic that telematics-based vehicle insurance uses to reward safe driving with lower premiums.

| Tier | Requirements | Benefit |
|---|---|---|
| Bronze | New accounts or recent soft fraud flags | No discount |
| Silver | 4+ weeks consistent behavior, 2+ clean event records | 10% premium discount |
| Gold | 8+ weeks clean history, no anomalies | 20–25% discount + No-Claim Bonus |

**No-Claim Bonus (NCB):** Gold tier workers who experience no payout events in a given week receive 10–15% of their premium credited toward the following week's cost. This shifts the worker's perception from insurance as a sunk cost to a financial tool that rewards regular, honest participation.

Tier downgrades are explained with specific reasons, not generic notifications.

---

## 8. User Experience Design

### 8.1 Design Principles

- **Explainability over precision:** Every number has a plain-language reason. Workers never encounter an unexplained premium change or an unexplained payout denial. Internally, fraud scores are translated into user-facing explanations via the ExplainerAPI to ensure no decision reads as a black box.
- **Recovery paths over hard blocks:** When fraud signals are elevated, the system prompts for lightweight verification rather than silently denying. Legitimate workers get a path out; fraudsters face higher friction.
- **Background operation:** During normal weeks the app requires near-zero active engagement. Workers notice VeriPay when it helps them, not constantly.

### 8.2 Platform Choice

VeriPay is built as a mobile application. Food delivery workers in India operate exclusively on mobile devices throughout their working day. A mobile-first design also enables the device signal collection (GPS, accelerometer, app state, battery) that powers the fraud detection system. A web platform would forfeit this entirely.

### 8.3 Core Screens

**Onboarding**

Designed to complete in under 3 minutes:
- Step 1: Phone number + OTP (no document upload at this stage)
- Step 2: Confirm primary platform (Zomato / Swiggy), typical working zones, and usual shift times
- Step 3: Link UPI for payouts
- Step 4: Consent screen for device telemetry, with plain-language explanation of what is collected, why, and what never leaves the device
- Step 5: First weekly quote presented immediately — one tap to activate

**Weekly Cover and Pricing Breakdown**

Workers see a plain breakdown before paying their weekly premium:
- Base zone risk: amount with brief explanation (e.g., "Zone A has had 4 rain events in the last 3 months")
- Exposure adjustment: amount (e.g., "You usually work 28 hours in high-risk slots this week")
- Trust discount: amount and tier (e.g., "Silver tier — 10% discount applied")
- Final premium: total with one-tap payment

A What-If toggle shows the worker what their premium would be at the next tier, creating a behavioral incentive without being prescriptive.

**Payout Confirmation Screen**

When an event triggers and payout is approved, the worker sees:
- The specific event: date, time, zone, index value that crossed the threshold
- Their effort record: hours online matched against their historical pattern
- The payout calculation: expected earnings rate × eligible hours × zone loss factor
- Fraud check summary: confirmation that all signals were verified

**Denied or Reduced Payout Screen**

- The specific reason drawn directly from the fraud or effort scoring system — not a generic message
- The worker's activity log for the event window, showing exactly what data the system used
- A single-tap appeal path that creates a review record with evidence logs attached
- Actionable guidance for next time

**Worker Dashboard**

- Earnings protected this week (total payout received vs. premium paid)
- Active coverage status and current zone risk level
- Trust tier and trend
- History of past events, decisions, and payouts

**Admin / Insurer Dashboard**

- Loss ratios per zone and per hazard type
- Fraud flag rates and ring detection activity
- Weekly premium pool health and reserve levels
- Predictive analytics: next-week expected claim volume based on weather and AQI forecasts
- Worker trust tier distribution across the pool

---

## 9. Technical Architecture

### 9.1 Tech Stack

| Layer | Technology |
|---|---|
| Mobile frontend | React Native (iOS and Android from single codebase) |
| Backend API | Node.js with Express |
| Database | PostgreSQL (relational data) + Redis (session and scoring cache) |
| ML models | Python (scikit-learn for GLMs and decision trees; batch jobs) |
| Graph analysis | NetworkX (Python, weekly batch) |
| Weather / AQI | Open-Meteo API, CPCB AQI India API |
| Traffic / civic | Google Maps Platform (mocked for disruption events) |
| Platform activity | Mocked REST API simulating Zomato/Swiggy delivery activity |
| Payments | Razorpay test mode / UPI simulator |
| Hosting | Any cloud provider (AWS / GCP free tier sufficient for demo) |

### 9.2 Core Services

| Service | Responsibility | Key Output |
|---|---|---|
| TriggerEvaluator | Polls environmental and civic APIs, evaluates zone-level thresholds | TriggerEvent records |
| IncomeModelService | Computes expected earnings per worker per event window | IncomeExpectationSnapshot |
| EffortScoreEngine | Evaluates worker activity against historical patterns | EffortScore (0–1) per window |
| DisruptionValidator | Cross-checks market supply/demand at trigger time | Payout approve / deny |
| FraudScoringEngine | Aggregates individual multi-signal fraud indicators | FraudScore + flag type |
| FraudGraphService | Weekly batch graph analysis for coordinated ring detection | RingScore, cluster flags |
| PricingEngine | Computes hybrid weekly premiums per worker | Premium + tier + NCB |
| PayoutPipeline | Final payout decision and UPI transfer initiation (mocked) | PayoutDecision record |
| ExplainerAPI | Maps internal scores to plain-language user-facing strings | Decision explanations |

### 9.3 Key Data Entities

| Entity | Purpose |
|---|---|
| PolicyWeek | Weekly coverage record: hazards, cap, premium, tier at issue |
| EventWindow | Per-trigger record: zone, hazard type, index values, severity |
| IncomeExpectationSnapshot | Expected vs. observed earnings model output per user per event |
| EffortScoreWindow | Effort score, reason code, and raw metrics per user per event |
| FraudSignalSnapshot | All individual fraud sub-scores and flags per user per event |
| FraudGraphComponent | Group-level ring detection: size, shared-entity ratios, ring score |
| PayoutDecision | Final decision: gross payout, adjusted payout, adjustment reasons |
| TrustTierHistory | Tier changes with timestamps and reasons — full audit trail |
| PricingSnapshotWeek | Full pricing breakdown per worker per week for explainability |

### 9.4 External Integrations

| Integration | Purpose | Status |
|---|---|---|
| IMD / Open-Meteo | Weather triggers (rainfall, temperature) | Mocked with real API structure |
| CPCB / AQI India | Pollution triggers (AQI index) | Mocked with real API structure |
| Google Maps Platform | Traffic and civic disruption signals | Mocked |
| Platform Activity API | Delivery partner counts, order volumes, surge state per zone | Fully mocked |
| Razorpay test mode | UPI payout initiation | Sandbox |

---

## 10. Implementation Plan

| Week | Phase | Deliverables |
|---|---|---|
| Week 1 | Scale | Mocked API integrations (weather, AQI, platform activity, civic). Device heartbeat schema. TriggerEvaluator. IncomeModelService trained on synthetic earnings data (structure mirrors real platform data; no PII, models and scoring flows are fully live). |
| Week 2 | Scale | EffortScoreEngine. DisruptionValidator. FraudScoringEngine with soft/hard flag routing. Fallback logic for API failures. PayoutPipeline with mocked UPI transfer. Registration and onboarding flow. |
| Week 3 | Soar | FraudGraphService built on a digital twin simulation dataset, including a demonstrable ring-detection scenario with graph visualization. PricingEngine with hybrid model and trust tiers. NCB logic. ExplainerAPI. |
| Week 4 | Soar | Frontend: onboarding, weekly quote, payout confirmation, denial screen, worker dashboard, admin dashboard. Full end-to-end demo covering a clean payout scenario and a caught fraud ring scenario. Final documentation and demo video. |

All ML components use explainable, lightweight models — GLMs, decision trees, and rule-based scoring — to ensure auditability and compatibility with insurance regulatory standards where black-box decisions face increasing scrutiny.

### Worker Data and Consent

Workers opt in to device telemetry collection at signup. The consent flow explains which signal categories are collected, that proximity data is hashed on-device before transmission, and that raw network identifiers never leave the device. This aligns with India's Digital Personal Data Protection Act (DPDP).

### Success Metrics

| Metric | Target |
|---|---|
| Payout latency | Under 30 minutes from trigger to UPI transfer for clean cases |
| False-positive fraud rate | Legitimate workers incorrectly blocked: below 5% |
| Pool sustainability | Share of premium weeks with no payout event, indicating reserve health |
| Trust tier distribution | Share of workers reaching Silver or Gold after 8 weeks |

---

## 11. Scope Exclusions

VeriPay explicitly does not cover:

- Health or accident insurance
- Vehicle insurance or damage claims
- Life insurance or disability products
- Any insurance product requiring manual claim adjudication

These are deliberate product decisions, not limitations. Parametric income protection is the mechanism that makes zero-touch, instant payouts possible.

---

## 12. Competitive Differentiation

| Dimension | Generic Parametric Insurance | VeriPay |
|---|---|---|
| Income Loss Model | Flat payout on trigger | Counterfactual EEC: expected earnings × eligible hours × graded zone loss factor |
| Trigger Coverage | Environmental only | Environmental + technical (outage) + social (civic disruption) |
| Fraud Defense | GPS check or none | 6-signal weighted scoring + effort gate + graph-based coordinated ring detection |
| Pricing | Zone rate or dynamic only | Hybrid: stable base + capped weekly load + trust tier discount + NCB + earnings-relative premium cap |
| UX Transparency | Approval or denial message | Full breakdown at every touchpoint: pricing, payout math, fraud results, denial reasons, evidence logs, recovery path |
| System Resilience | Not addressed | Explicit fallback logic for every data failure scenario |
| Liquidity Protection | Not addressed | Graded payouts + ring detection + effort gate prevent coordinated pool drain |

---

## 13. Appendix: Device Signal Schema

The following JSON payload is sent by the worker's device every 5–10 minutes and consumed by the FraudScoringEngine and EffortScoreEngine. Proximity fields are hashed on-device before transmission. Raw network identifiers never leave the device.
```json
{
  "user_id": "GIG_88291",
  "timestamp": "2026-03-20T18:45:00Z",
  "session_id": "sess_9901_alpha",
  "location": {
    "lat": 12.9716,
    "lng": 77.5946,
    "accuracy_meters": 12.5,
    "is_mocked": false,
    "is_stationary": false
  },
  "device_vitals": {
    "battery_level": 74,
    "battery_temp_c": 38.5,
    "is_charging": true,
    "screen_brightness": 0.85,
    "foreground_app": "com.zomato.delivery"
  },
  "proximity_signals": {
    "wifi_environment_hash": "a3f9c12e",
    "nearby_bt_device_count": 3,
    "avg_signal_strength": -65
  },
  "trust_telemetry": {
    "app_uptime_minutes": 120,
    "last_ui_interaction_ts": "2026-03-20T18:42:10Z"
  }
}
```

**Fraud detection logic applied to this payload:**

- **Cold phone detection:** `battery_temp_c` below 30°C while `foreground_app` is active and outdoor temperature is 35°C+ is a weak signal of indoor stationary fraud. It is weighted at 0.15 and only elevates the fraud score when combined with other anomalies such as `is_stationary = true`.
- **Static ghost detection:** `is_stationary = true` for more than 30 minutes during a high-demand EEC window, combined with no app interaction, indicates the worker is not actively working.
- **Sybil cluster detection:** Matching `wifi_environment_hash` values across multiple `user_id`s in the same zone with zero platform order activity indicates a coordinated fraud ring. This feeds directly into the FraudGraphService edge-weight calculation.

---

*VeriPay — CtrlQueens — Guidewire DEVTrails 2026*
*Protecting the people who keep the city moving.*
