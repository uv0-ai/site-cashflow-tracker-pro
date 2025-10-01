# GiftCare – Diagram Pack, API Patterns, and Sprint 1 Plan (v1)

Owner: Orchestrator  
Complements: PRD and Handoff docs

---

## 1) Mermaid Diagram Pack

### 1.1 System Context (C4‑ish)
```mermaid
flowchart LR
  subgraph Client
    A["Web App (SPA)"]
    M["Mobile App (later)"]
  end

  subgraph Edge
    G["API Gateway\nAuthZ/Rate limits\nIdempotency"]
  end

  subgraph Core Services
    U["User Service"]
    R["Recipient Service"]
    C["Calendar Service"]
    K["Catalog Service"]
    P["Policy Engine"]
    Pay["Payments Service"]
    F["Fulfilment Service"]
    N["Notification Service"]
    S["Scheduler/Orchestrator"]
    AU["Audit Log"]
    EB[(Event Bus)]
  end

  subgraph Data Stores
    DB[(Postgres)]
    REDIS[(Redis)]
    OBJ[(Object Store)]
    VAULT[(Secrets Vault)]
  end

  subgraph External
    GC["Google Calendar"]
    PG["Payment Gateway\n(UPI Autopay/Card/NACH)"]
    VEND["Gift Vendors\n(e-gift/physical)"]
    MSG["Email/SMS/WhatsApp"]
  end

  A --> G
  M --> G
  G --> U
  G --> R
  G --> C
  G --> K
  G --> P
  G --> Pay
  G --> F
  G --> N
  G --> S

  U --> DB
  R --> DB
  C --> DB
  K --> DB
  P --> DB
  Pay --> DB
  F --> DB
  N --> DB
  S --> DB
  G --> REDIS
  Pay --> REDIS
  S --> REDIS
  AU --> DB
  VAULT --- G
  VAULT --- Pay

  C <--> GC
  Pay <--> PG
  F <--> VEND
  N <--> MSG

  S --> EB
  EB --> Pay
  EB --> F
  EB --> N
  EB --> AU

```

### 1.2 Sequence – Onboarding with UPI Autopay Mandate
```mermaid
sequenceDiagram
  participant User
  participant SPA as Web App
  participant API as API Gateway
  participant PAY as Payments Service
  participant PG as Payment Gateway
  participant NOTIF as Notification
  participant USER as User Service

  User->>SPA: Start UPI Autopay setup
  SPA->>API: POST /mandates{rail=upi,cap,frequency}
  API->>PAY: createMandate()
  PAY->>PG: Create UPI mandate
  PG-->>PAY: auth_link/qr, mandate_id (pending)
  PAY-->>API: {mandate_id, auth_link}
  API-->>SPA: Show deeplink/QR
  Note over User,PG: User approves in UPI app (one‑time AFA)
  PG-->>PAY: webhook mandate_activated
  PAY->>USER: update instrument status=Active
  PAY->>NOTIF: send setup success
  NOTIF-->>User: "UPI Autopay active"
```

### 1.3 Sequence – Scheduled Digital Gift Execution
```mermaid
sequenceDiagram
  participant S as Scheduler
  participant POL as Policy Engine
  participant PAY as Payments
  participant F as Fulfilment
  participant N as Notification
  participant V as Vendor
  participant PG as Payment Gateway

  S->>POL: Decide(event, level, expected_price, caps)
  POL-->>S: decision (proceed/swap/alt_rail/skip)
  S->>N: Pre‑debit notification T‑24h
  S->>PAY: Charge at T‑0 10:00
  PAY->>PG: debit(mandate_id, amount)
  PG-->>PAY: payment_succeeded
  PAY-->>S: charge_id status=success
  S->>F: request e‑code(order)
  F->>V: issue e‑code
  V-->>F: code, expiry
  F->>N: deliver via channel
  N-->>Recipient: e‑code delivered
  S->>Audit: record all states
```

### 1.4 Sequence – Google Calendar Import
```mermaid
sequenceDiagram
  participant User
  participant SPA as Web App
  participant API as API Gateway
  participant CAL as Calendar Service
  participant GC as Google Calendar

  User->>SPA: Connect Google Calendar
  SPA->>API: POST /calendar/google/connect {code}
  API->>CAL: exchange code→tokens
  CAL->>GC: subscribe/pull events
  GC-->>CAL: events payload
  CAL->>CAL: parse birthdays/anniversaries, dedupe
  CAL-->>SPA: imported items & mapping suggestions
```

### 1.5 Data Model – High Level Entities
```mermaid
erDiagram
  USER ||--o{ PAYMENTINSTRUMENT : has
  USER ||--o{ RECIPIENT : owns
  USER ||--o{ EVENT : schedules
  EVENT ||--|| RECIPIENT : for
  EVENT ||--o{ ORDER : triggers
  ORDER ||--o{ CHARGE : results_in
  ORDER ||--o{ NOTIFICATION : informs
  USER ||--o{ POLICY : defines

  USER {
    uuid id
    string name
    string email
    string phone
    string timezone
  }
  PAYMENTINSTRUMENT {
    uuid id
    uuid user_id
    enum rail_type
    string mandate_id
    string token_ref
    money cap_amount
    enum status
  }
  RECIPIENT {
    uuid id
    uuid user_id
    string name
    enum relationship
    jsonb contacts
  }
  EVENT {
    uuid id
    uuid user_id
    uuid recipient_id
    enum type
    date date
    string recurrence
    string timezone
  }
  ORDER {
    uuid id
    uuid event_id
    uuid vendor_id
    uuid sku_id
    money expected_amount
    enum status
  }
  CHARGE {
    uuid id
    uuid order_id
    uuid instrument_id
    money amount
    enum status
    string pg_txn_id
  }
  NOTIFICATION {
    uuid id
    uuid order_id
    enum channel
    string template_id
    timestamptz sent_at
  }
  POLICY {
    uuid id
    uuid user_id
    bool downgrade_allowed
    bool alternate_rail_allowed
    bool skip_allowed
  }
```

### 1.6 Infrastructure Runtime View
```mermaid
flowchart TB
  subgraph K8s[Cluster]
    GW[API Gateway]
    US[User]
    RS[Recipient]
    CS[Calendar]
    KS[Catalog]
    PE[Policy]
    PS[Payments]
    FS[Fulfilment]
    NS[Notification]
    SCH[Scheduler]
    EB[(Event Bus)]
    AUL[Audit]
  end
  DB[(Postgres HA)]
  RE[(Redis Cluster)]
  OBJ[(Object Store/S3)]
  VA[Vault]
  EX1[Google Calendar]
  EX2[Payment Gateway]
  EX3[Vendor APIs]
  EX4[Email/SMS/WhatsApp]

  GW-->US & RS & CS & KS & PE & PS & FS & NS & SCH
  US & RS & CS & KS & PE & PS & FS & NS & SCH --> DB
  PS & SCH --> RE
  VA --- GW
  VA --- PS
  CS<-->EX1
  PS<-->EX2
  FS<-->EX3
  NS<-->EX4
  SCH-->EB
  EB-->PS & FS & NS & AUL
```

---

## 2) API Error Codes & Pagination Patterns

### 2.1 Standard Error Envelope
```json
{
  "error": {
    "code": "string",          // machine-readable (e.g., VALIDATION_FAILED)
    "message": "human summary", // safe for UI
    "field": "optional.path",   // pointer for validation errors
    "retryable": false,          // hint for clients
    "details": {                 // optional structured details
      "constraint": "...",
      "limit": 1000,
      "remaining_window_sec": 7200
    },
    "trace_id": "uuid"
  }
}
```

### 2.2 HTTP Status Mapping & Codes
- **400 BAD_REQUEST**
  - `VALIDATION_FAILED` – payload/schema invalid; include `field` and reason.
  - `UNSUPPORTED_OPERATION` – not allowed in current plan/phase.
- **401 UNAUTHORIZED**
  - `AUTH_REQUIRED` – missing/expired token.
  - `AFA_REQUIRED` – only during initial mandate creation if PG signals interactive step.
- **403 FORBIDDEN**
  - `INSUFFICIENT_SCOPE` – token lacks scope.
  - `MANDATE_NOT_ACTIVE` – attempt to charge without active mandate.
- **404 NOT_FOUND**
  - `RESOURCE_NOT_FOUND` – id not present or not owned by caller.
- **409 CONFLICT**
  - `IDEMPOTENCY_CONFLICT` – same key different payload.
  - `DUPLICATE_RESOURCE` – e.g., duplicate recipient by unique constraint.
- **422 UNPROCESSABLE_ENTITY**
  - `POLICY_BLOCKED` – amount > cap and policy forbids swap/alternate.
  - `VENDOR_UNAVAILABLE` – chosen vendor down or out of stock.
- **429 TOO_MANY_REQUESTS**
  - `RATE_LIMITED` – backoff + `Retry-After` header.
- **500 INTERNAL_SERVER_ERROR**
  - `INTERNAL_ERROR` – unexpected; include `trace_id`.
- **502/503/504 Upstream**
  - `PG_UNAVAILABLE`, `VENDOR_UNAVAILABLE`, `CALENDAR_UNAVAILABLE` – transient retriable.

### 2.3 Idempotency
- All POST/PUT/PATCH support `Idempotency-Key` header (UUIDv4 recommended).
- Server stores (key, request_hash, response, ttl=24h). If key reused with different hash → 409 `IDEMPOTENCY_CONFLICT`.

### 2.4 Pagination Pattern (Cursor‑based)
- Request params: `limit` (1–100, default 25), `cursor` (opaque, base64), `order` (asc|desc by created_at).
- Response envelope:
```json
{
  "data": [ /* items */ ],
  "page": {
    "next_cursor": "opaque-or-null",
    "prev_cursor": "opaque-or-null",
    "limit": 25,
    "order": "desc"
  }
}
```
- Cursors include `(created_at, id)` tuple to avoid duplicates across time.

### 2.5 Problem‑Specific Errors
- `MANDATE_CAP_EXCEEDED` (422): Attempted charge exceeds cap and cannot be adjusted by policy.
- `MANDATE_PAUSED_OR_EXPIRED` (403): Rail inactive; requires user action outside event day.
- `COMPLIANCE_VIOLATION` (403): Operation would break scheme rules (e.g., transaction splitting).
- `SCHEDULE_IN_PAST` (400): Event date invalid/elapsed.
- `RECIPIENT_CONTACT_INVALID` (400): No deliverable channel.

---

## 3) Sprint 1 – Backlog, Story Points, Dependencies

### 3.1 Scope (Phase A focus)
- UPI Autopay mandate onboarding (happy path + webhooks)
- Google Calendar connect + import (read‑only)
- Recipient CRUD + validation
- Scheduler skeleton + pre‑debit notifications
- Error envelope, idempotency, and cursor pagination foundations

### 3.2 Sprint Board (Stories & Points)
| Key | Story | Points | Owner |
|---|---|---:|---|
| S1 | API gateway idempotency + error envelope | 5 | BE |
| S2 | UPI mandate: create + deeplink + webhook activate | 8 | BE |
| S3 | Notification service: send pre‑debit (Email/SMS) | 5 | BE |
| S4 | Google OAuth connect + token store | 5 | BE |
| S5 | Calendar import (birthdays/anniv parse + dedupe) | 8 | BE |
| S6 | Recipient CRUD + validations | 3 | BE |
| S7 | Scheduler skeleton (T‑24h, T‑0 triggers) | 5 | BE |
| S8 | Minimal UI: onboarding flow + mandate status | 8 | FE |
| S9 | Minimal UI: recipients + calendar views | 5 | FE |
| S10| Observability: metrics + trace + dashboards (core) | 5 | DevOps |
| S11| Security: Vault wiring + secret rotation hooks | 3 | DevOps |
| S12| E2E path: seed data + test harness | 5 | QA |

**Total**: 65 pts (2‑week sprint, 6‑8 dev capacity; adjust if team velocity < 55)

### 3.3 Dependency Graph
```mermaid
flowchart LR
  S1[Idempotency + Error Envelope]
  S2[UPI Mandate Flow]
  S3[Pre‑debit Notifications]
  S4[Google OAuth Connect]
  S5[Calendar Import]
  S6[Recipient CRUD]
  S7[Scheduler Skeleton]
  S8[UI: Onboarding + Mandate]
  S9[UI: Recipients + Calendar]
  S10[Observability]
  S11[Security: Vault]
  S12[E2E Harness]

  S1 --> S2
  S1 --> S3
  S1 --> S5
  S4 --> S5
  S6 --> S5
  S2 --> S7
  S3 --> S7
  S5 --> S7
  S2 --> S8
  S6 --> S9
  S5 --> S9
  S10 --> S12
  S11 --> S2
```

### 3.4 Gantt (2 weeks)
```mermaid
gantt
    dateFormat  YYYY-MM-DD
    title Sprint 1 – Timeline
    section Backend
    S1:done, 2025-10-01, 1d
    S2:active, 2025-10-02, 4d
    S3: 2025-10-06, 2d
    S4: 2025-10-02, 2d
    S5: 2025-10-07, 3d
    S7: 2025-10-10, 2d
    section Frontend
    S8: 2025-10-05, 4d
    S9: 2025-10-09, 3d
    section DevOps/QA
    S10: 2025-10-01, 3d
    S11: 2025-10-02, 2d
    S12: 2025-10-11, 2d
```

### 3.5 Definition of Done (Sprint 1)
- End‑to‑end demo: Onboarding → mandate activated → calendar import → pre‑debit sent → (mock) charge executed → digital fulfilment stub delivered.
- Logs/traces visible; error envelope returned by all services; cursor pagination on list endpoints.

---

## 4) Next Steps
- Confirm PG webhook event names and signatures.
- Lock WhatsApp sender (for later) and ensure SMS templates cover pre‑debit.
- Create staging vendor stub with variable price responses for policy testing in Sprint 2.

