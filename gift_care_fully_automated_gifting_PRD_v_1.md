# GiftCare – Fully Automated Gifting PRD (v1) [Product Requirements Document]

**Doc owner:** Product Manager
**Stakeholders:** Analyst, UX Architect, Architect, Product Owner  
**Target market (phase 1):** India, consumer users in WhatsApp/UPI-heavy segments  
**Release target:** MVP in 8–10 weeks (Manual)  & 3–5 weeks (Auto-Code Platform)
**Time zone default:** Asia/Kolkata

---

## 1) Problem & Vision
**Problem**: People forget or scramble to send gifts on recurring occasions (birthdays, anniversaries), and even when they remember, checkout, OTP/AFA, and logistics make it a chore.

**Vision**: GiftCare is a “set-and-forget” gifting autopilot. After initial setup (recipients, preferences, mandate authorization), the system selects, pays, and delivers gifts on time with zero day‑of human intervention.

**North Star**: % of scheduled gifts that are paid and delivered on-time without any user action after initial setup.

---

## 2) Goals & Non‑Goals
**Goals (MVP)**
- Automate the entire lifecycle for recurring occasions: plan → select → pay → fulfil → notify, hands‑free.
- Integrate with Google Calendar to import birthdays/anniversaries and avoid double entry.
- Support in‑app event creation with recurrence.
- Enable compliant, OTP‑less recurring payments via mandates (UPI Autopay / Card e‑mandate / NACH), with pre‑debit notifications.
- Start with **digital gifts** (e-gift cards/donations) plus optional limited physical SKUs in tier‑1 cities.
- Provide a policy engine to keep charges within mandate caps automatically (fallback rails, SKU swap, or skip per user policy).

**Non‑Goals (MVP)**
- One‑off, ad‑hoc gifts outside the calendar.
- International shipping or multicurrency settlement.
- Marketplace for user‑curated baskets; we use pre‑approved vendors/SKUs.
- Manual approvals on the day of gifting (breaks the “no interaction” principle).

---

## 3) Personas & Primary Use Cases
**Priya (28, Consultant)** – Busy, forgetful with birthdays. Wants a reliable autopilot for close family and friends (10–15 recipients). Prefers WhatsApp updates.

**Arjun (35, Product Manager)** – Organized but lazy about checkout frictions. Wants a single monthly cap and predictable spend.

**Dr. Meera (41, Pediatrician)** – Values punctuality. Wants professional gifts for colleagues and staff. Requires downloadable invoices.

**Top Use Cases**
1. Import birthdays from Google Calendar, set gift levels by relationship, and never touch the app again except for FYI notifications.  
2. Add a new colleague with a recurring work anniversary and assign a corporate e‑gift provider with GST invoice.  
3. Configure a high‑cap mandate and a fallback rail; if a vendor price changes, auto‑swap to a similar SKU under the same level.

---

## 4) Scope & Prioritization (MVP → V1.1)
**P0 (MVP) Must‑haves**
- Google Calendar OAuth + event import (birthdays/anniversaries/custom) and in‑app event creation.
- Recipient management: name, birthday, relationship type, gift level; delivery channel (email/SMS/WhatsApp) and optional address.
- Gift levels mapped to price bands; vendor catalog for **digital gifts** per level.
- Policy engine: forecast vs. mandate cap, fallback rules (swap SKU / alternate rail / skip). No transaction splitting.
- Mandate onboarding: UPI Autopay primary; Card e‑mandate secondary; NACH optional. Tokenized storage via PG.
- Pre‑debit notifications; automatic payment execution; automated fulfilment; post‑delivery confirmation.
- Scheduler with lead‑time logic; idempotency; retries; audit trail.
- Security: encryption at rest/in transit; PII and PCI responsibilities (via PG tokenization).

**P1 (V1.1) Nice‑to‑have**
- Limited **physical gifts** in top 6 metros with SLA and tracking.
- Secondary funding source with automatic fallback.
- Invoices (GST), monthly statements export, and basic reconciliation view.
- Basic personalization templates per event type.

**P2 (Future)**
- Cross‑border gifting; multicurrency; broader catalog; recipient preferences learning.

---

## 5) User Stories & Acceptance Criteria (AC)

### 5.1 Onboarding & Mandate Setup
- **Story:** As a user, I can complete onboarding in one flow to enable fully automated gifting.  
**AC:**
  - Can sign in, set time zone, add payment rail (UPI Autopay or card e‑mandate) with one‑time AFA.
  - Mandate shows status=Active with cap and frequency; stored token/mid/mandate_id present.
  - Pre‑debit notification channel verified (email/SMS/WhatsApp).

### 5.2 Recipient & Event Management
- **Story:** As a user, I can add recipients with gift levels and contact/delivery details.  
**AC:** Name, relationship, birthday (no year allowed), level required; contact at least one of email/SMS/WhatsApp; validation enforced.  
- **Story:** As a user, I can import events from Google Calendar.  
**AC:** OAuth completes; events with “birthday/anniversary” keywords or metadata are listed; user can map events to recipients; duplicates deduped.  
- **Story:** As a user, I can create an event in‑app.  
**AC:** Click‑to‑add on calendar; set recurrence; link to recipient; stored with timezone.

### 5.3 Automated Gifting Execution
- **Story:** As a user, I expect the gift to be sent on time with no action needed.
**AC:** For digital gifts, e‑code delivered via selected channel by 10:00 local time on event day; status visible in timeline.
- **Story:** As a user, I expect payment to be auto‑charged without OTP if within mandate parameters.  
**AC:** Charge executes with no interactive step when amount ≤ cap; pre‑debit sent ≥24h ahead (configurable by rail).  
- **Story:** As a user, I expect the system to adapt if price > cap.  
**AC:** Based on my policy, the system either a) swaps SKU under the same level, b) uses alternate rail with adequate cap, or c) skips and notifies FYI. No same‑day approval requests.

### 5.4 Notifications & Audit
- **Story:** As a user, I receive FYI notifications (pre‑debit, post‑delivery) but never an approval prompt on event day.  
**AC:** Channels configurable; messages templated; audit trail records send time and delivery result.

---

## 6) End‑to‑End Flows

### 6.1 Onboarding Flow
1) Sign up → Verify contact → Set timezone.  
2) Add payment rail → Create mandate → Consent captured → Mandate active.  
3) Connect Google Calendar (optional).  
4) Add recipients & set gift levels; choose default delivery channel; choose fallback policies.  
5) Review summary → Finish.

### 6.2 Scheduler & Execution Flow (Digital Gift)
- T‑10 days: Forecast amount vs mandate cap; if breach predicted, apply policy (swap SKU/alternate rail/skip) and send FYI.  
- T‑24 hours (or scheme‑required): Send pre‑debit notification with amount, date, merchant, cancel window info.  
- T‑0 10:00 local: Initiate payment; on success, request e‑code from vendor; deliver via chosen channel; log and notify.  
- Exceptions: Retry policy with exponential backoff; outbox+idempotency keys for PG and vendor API.

### 6.3 Physical Gift Flow (P1)
- T‑N (lead‑time matrix): Place order with vendor; include message; track shipment; on “Delivered” or carrier event, notify.

---

## 7) Functional Specs

### 7.1 Calendar Integration
- Google OAuth 2.0; read‑only scopes; periodic sync (webhook/poll every 12h).  
- Event parser: birthday/anniversary detection; recipient matching by name/email; manual map UI.  
- In‑app calendar: month view; click‑to‑add; RRULE support (yearly); timezone aware.

### 7.2 Recipients & Levels
- Relationship types (Family, Friend, Colleague, Other).  
- Levels: L1/L2/L3 mapped to price bands (config); per‑recipient override allowed.  
- Delivery channels: Email, SMS, WhatsApp (template id), optional address (P1).

### 7.3 Catalog & Policy Engine
- Catalog service with vendors, SKUs, min/max prices, availability.  
- Selection strategies: Default by level; round‑robin or sticky vendor.  
- Policy engine inputs: expected price, mandate caps per rail, user fallback settings.  
- Decisions: proceed on primary, switch rail, downgrade SKU, or skip.

### 7.4 Payments
- Rails: UPI Autopay (primary), Card e‑mandate (tokenized), NACH (optional).  
- Store: token references/mandate_id via PG; never store raw PAN.  
- Pre‑debit notice service with templates and variables (amount, date, merchant, cancel link if scheme allows).  
- Payment execution API with idempotency key; reconciliation hooks.

### 7.5 Fulfilment (Digital)
- Vendor adapters (e‑gift aggregators, donation APIs).  
- SLA: e‑code within 60 seconds post‑capture; deliver via channel provider.  
- Delivery proof: message id, vendor code id, optional short‑link hit.

### 7.6 Observability & Admin
- Metrics: automation rate, on‑time rate, payment success, vendor fulfilment SLA, notification deliverability, refund/chargeback rate.  
- Admin: mandate status view; vendor health; replay dead‑letter.  
- Audit: immutable logs of consent, events, charges, fulfilments.

---

## 8) Data Model (MVP, high‑level)
- **User**(id, name, email, phone, tz, created_at)
- **PaymentInstrument**(id, user_id, rail_type, mandate_id, status, cap_amount, frequency, token_ref)
- **Recipient**(id, user_id, name, relationship, contacts[email/phone/wa], address?)
- **Event**(id, user_id, recipient_id, type, date, recurrence, source[gcal/app], timezone)
- **GiftLevel**(id, user_id?, level_code, min_amount, max_amount, default_vendor_policy)
- **Policy**(id, user_id, downgrade_allowed, alternate_rail_allowed, skip_allowed)
- **Order**(id, event_id, vendor_id, sku_id, expected_amount, decided_rail, status, timestamps)
- **Charge**(id, order_id, instrument_id, amount, status, pg_txn_id, attempts)
- **Notification**(id, type, channels, template_id, sent_at, delivery_status)

---

## 9) API (Representative)
- `POST /v1/onboarding/mandates` – start mandate; returns status & mandate_id.
- `GET /v1/mandates/:id` – retrieve mandate status/cap.
- `POST /v1/recipients` – create recipient.
- `POST /v1/events` – create event; supports RRULE.
- `POST /v1/catalog/select` – resolve SKU for (user, level, policy).
- `POST /v1/payments/charge` – execute charge (idempotent).
- `POST /v1/fulfil/digital` – request e‑code; returns code/token.
- `POST /v1/notify/send` – send message via channel provider.
- `POST /v1/scheduler/run` – daily job trigger (internal).

---

## 10) Compliance, Risk & Controls
- **Mandates**: Only charge within mandate parameters; send pre‑debit notices per rail rules; expose in‑app cancel/pause.  
- **No AFA circumvention**: Never simulate OTP or split transactions to dodge limits.  
- **PCI/PII**: Use PG tokenization; encrypt PII; access controls & secrets management.  
- **Rate limits & fraud**: Soft caps per day/month; anomaly alerts (FYI only, no approval gates).  
- **Disputes**: Chargeback workflow with PG; refund path; vendor substitution rules.

---

## 11) UX Requirements (Key Screens)
1. **Onboarding** – Progress steps: Mandate → Calendar → Recipients → Policies → Review.
2. **Home** – Upcoming events list; status chips (Ready, Needs Policy, Skipped).
3. **Recipient Detail** – Level, delivery channels, last gift, next event.
4. **Event Calendar** – Month/week view; click‑to‑add; import status.
5. **Automation Log** – Timeline of pre‑debits, charges, orders, deliveries.

**Content & Copy**
- Reinforce “No approvals needed on the day” promise.
- Clear FYI notifications with undo/cancel windows only where the rail allows and **never** as a blocking step on event day.

---

## 12) Success Metrics (MVP)
- **Automation rate**: ≥ 95% of scheduled gifts executed without user action.
- **On‑time delivery**: ≥ 97% (digital); ≥ 92% (physical P1).
- **Payment success**: ≥ 98% within mandate parameters.
- **Notification deliverability**: ≥ 99% pre‑debit sends.
- **Churn (90‑day)**: ≤ 10%.
- **CSAT/NPS**: Baseline ≥ 45 for MVP cohort.

---

## 13) Rollout Plan
- **Phase A (Weeks 0–4)**: Digital gifts only, UPI Autopay, Google Calendar import, core scheduler, notifications.
- **Phase B (Weeks 5–8)**: Card e‑mandate, reconciliation, policy engine v1, invoices.
- **Phase C (Weeks 9–12)**: Physical gifts (metros), secondary rail fallback, admin dashboards.

---

## 14) Open Questions
- Do we allow per‑recipient override of rail preference? (e.g., high‑value gifts via NACH only)
- What are initial mandate cap defaults and recommended levels?
- Should we support donation receipts in recipient’s name?
- How to handle recipients with only WhatsApp but no email/SMS?

---

## 15) Risks & Mitigations
- **Regulatory changes** → Config‑driven caps and policy updates; rails feature flags.
- **Vendor outages** → Multi‑vendor adapters; circuit breakers; auto‑swap SKUs.
- **Mandate lapses** → Watchdog to prompt renewal well before events.
- **Data privacy** → Minimize PII; strong access controls; incident plan.

---

## 16) Exit Criteria (MVP “Done”)
- End‑to‑end e2e test passes for: onboarding→import→schedule→pre‑debit→charge→fulfil→notify.
- 20 pilot users; ≥ 95% automation rate; ≥ 97% on‑time digital deliveries.
- No production PII/PCI findings in security review.

---

## 17) Appendices
- Notification templates (pre‑debit, delivery confirmation).
- Example policy matrix by level.
- Event parsing rules for Google Calendar titles/descriptions.

