# Wazi

Wazi is a configurable, multi-account, non-custodial payment collection, confirmation, reconciliation, reporting, and integration platform.

It enables account holders to define categories, funds, and campaigns; collect payments through payment links, QR codes, administrator-led STK Push, and USSD; confirm payments through provider callbacks; create categorized ledger entries; send optional notifications; generate reports; and integrate verified payment events with external systems.

---

## Table of Contents

- [What Wazi Does](#what-wazi-does)
- [What Wazi Is Not](#what-wazi-is-not)
- [Operating Modes](#operating-modes)
- [Payment Flow](#payment-flow)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Key Data Models](#key-data-models)
- [Payment Channels](#payment-channels)
- [Web Application Routes](#web-application-routes)
- [Integration Layer](#integration-layer)
- [Running Locally](#running-locally)
- [Docs](#docs)

---

## What Wazi Does

- **Configurable categories and campaigns** — account holders define what payments are for (fund, fee, ticket, donation, invoice line, etc.) and group them into time-bound campaigns with optional targets.
- **Multiple payment-initiation channels** — QR code, payment link, hosted checkout page, USSD, and administrator-led STK Push.
- **Confirmed-payment ledger** — a ledger entry is only created after a successful provider callback. Pending STK prompts and failed attempts are never counted as collected funds.
- **Reconciliation** — compares payment intents, attempts, provider callbacks, and ledger entries to produce `MATCHED`, `FAILED`, `EXPIRED`, `UNKNOWN`, `AMOUNT_MISMATCH`, `DUPLICATE`, `UNMATCHED_PROVIDER_PAYMENT`, `CORRECTED`, `REVERSED`, and `REFUNDED` statuses.
- **Corrections** — incorrectly categorized payments can be corrected with a full audit trail; the original record is never silently overwritten.
- **Optional notifications** — payer SMS and account-holder SMS sent only after confirmed payment success.
- **Reports and exports** — daily, weekly, and monthly collection reports; category and campaign breakdowns; CSV and PDF exports.
- **Integration webhooks** — signed `payment.confirmed` and related events sent to external systems after confirmed payment.

---

## What Wazi Is Not

Wazi is not a wallet, bank, lender, full accounting suite, CRM, inventory system, POS replacement, school-management system, church-management system, or tax-filing system. It does not hold, pool, or settle funds on behalf of account holders.

---

## Operating Modes

### Standalone Mode
For account holders without their own management software. Wazi is their primary collections and payment-record tool — categories, campaigns, payment links, QR codes, USSD, ledger, corrections, reports, and exports.

### Integration-Layer Mode
For account holders that already run another system (ERP, school system, membership platform, etc.). The external system owns its domain data. Wazi handles:

- Payment-intent creation via API
- Hosted payment links and QR codes
- STK Push
- USSD collection flow
- Provider callback handling and confirmation
- Reconciliation
- Signed `payment.confirmed` webhook events

---

## Payment Flow

```
Purpose selected
      ↓
Payment intent created
      ↓
Payment attempt initiated (STK Push)
      ↓
Provider callback received
      ↓
Payment confirmed
      ↓
Ledger entry created
      ↓
Notification / report / integration event generated
```

Internal payment states: `CREATED` → `PUSH_SENT` → `PAID | FAILED | UNKNOWN | EXPIRED | REVERSED | REFUNDED`

A ledger entry is only created on `PAID`. Unknown outcomes are queued for status verification before any notification or ledger write.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | SvelteKit + TypeScript |
| Styling | Tailwind CSS |
| Validation | Zod |
| Backend | Go + Gin |
| Database | PostgreSQL |
| Database access | pgx + SQLc |
| Migrations | Atlas or golang-migrate |
| Cache & jobs | Redis + Asynq |
| Object storage | S3-compatible (MinIO in dev) |
| Payment adapter | Daraja / approved provider |
| USSD adapter | USSD aggregator/provider |
| SMS adapter | Kenyan SMS provider |
| WhatsApp | Meta Cloud API (later) |
| API docs | OpenAPI 3.1 |
| Monitoring | OpenTelemetry, Sentry, Prometheus/Grafana |
| Deployment | Docker |
| CI/CD | GitHub Actions |
| Reverse proxy | Caddy or Nginx |
| Secrets | Secret manager |

---

## Architecture

Wazi starts as a **modular monolith**: one Go API process, one Go worker process, one PostgreSQL database, one Redis instance, one SvelteKit frontend.

```
SvelteKit Web App
      │ HTTPS
      ▼
Go + Gin API
  ├── Accounts, Categories, Campaigns
  ├── Payment intents, STK Push, Callbacks
  ├── Ledger, Reconciliation
  ├── Notifications, Reports, USSD
  └── Integrations, Webhooks, Audit
      │                        │
      ▼                        ▼
PostgreSQL               Redis
(durable records)        (sessions, jobs, cache)
                              │
                              ▼
                         Go Worker
                   (notifications, reports,
                    webhook retries, scheduled jobs)
                              │
          ┌───────────────────┼──────────────────┐
          ▼                   ▼                  ▼
  Payment provider      USSD provider      SMS / WhatsApp
  (M-Pesa / Daraja)    / aggregator         providers
```

Every account-owned record is scoped by `account_id`. The backend resolves account ownership from the authenticated session, verified API key, payment-link token, or USSD alias — the client never decides ownership.

---

## Key Data Models

### Amount Modes

| Mode | Payer experience |
|---|---|
| `FIXED` | Amount displayed; cannot be changed |
| `SUGGESTED` | Preset choices shown; "other" optional if enabled |
| `OPEN` | Payer enters amount within configured limits |
| `ADMIN_SET` | Administrator sets amount; payer cannot change it |
| `EXTERNAL_SET` | External system sets amount; payer cannot change it |

### Ledger Entry (core, immutable facts)

```
id · account_id · payment_intent_id · payment_attempt_id
provider_name · provider_transaction_reference
amount_minor · currency · payment_channel · initiated_by
payment_confirmed_at · payment_status
category_id · category_name_snapshot
campaign_id · campaign_name_snapshot
payer_phone_encrypted · payer_phone_masked · payer_name
external_reference · platform_reference · created_at
```

Financial amounts are stored as `BIGINT` (`amount_minor`) + `CHAR(3)` (`currency`). Floating-point is never used for money.

### Transaction Correction

```
id · ledger_entry_id
original_category_id · corrected_category_id
original_campaign_id · corrected_campaign_id
reason · corrected_by · created_at
```

The original ledger entry is preserved. Reports use the corrected category while keeping the full correction history.

---

## Payment Channels

### Customer-Led
Payer opens a QR code, payment link, hosted checkout page, or USSD menu → selects category/campaign → follows configured amount mode → enters phone number → STK Push → PIN → provider callback → ledger entry.

### Administrator-Led
Administrator selects category/campaign → enters or confirms amount → enters payer phone → STK Push → payer enters PIN → provider callback → ledger entry. No smartphone, mobile data, or payment link required from the payer.

### USSD
Shared shortcode + account alias → curated menu (max 12–15 items, 20–24 char labels) → category/campaign selection → configured amount flow → STK Push → provider callback → ledger entry. Sessions held in Redis; confirmed records in PostgreSQL.

---

## Web Application Routes

| Route | Purpose |
|---|---|
| `/login` | Administrator sign-in |
| `/dashboard` | Overview |
| `/categories` | Manage categories |
| `/campaigns` | Manage campaigns |
| `/payment-links` | Create and manage links |
| `/collect` | Administrator-led STK Push |
| `/transactions` | Confirmed transaction ledger |
| `/reports` | Collection reports |
| `/exports` | CSV / PDF exports |
| `/ussd` | USSD configuration and usage |
| `/integrations` | API keys, webhooks, logs |
| `/settings` | Account configuration |
| `/pay/[token]` | Public hosted payment page |

---

## Integration Layer

External systems create a payment intent via the REST API and receive a hosted payment link, QR code URL, and later a signed webhook event.

**Example `payment.confirmed` webhook event:**
```json
{
  "event_id": "evt_01JABC",
  "event": "payment.confirmed",
  "occurred_at": "2026-09-18T14:30:00Z",
  "payment_intent_id": "pi_01JABC",
  "external_reference": "INV-2026-104",
  "provider_transaction_reference": "MPE123ABC",
  "amount": 2500,
  "currency": "KES",
  "category_code": "CATEGORY_A",
  "campaign_code": "CAMPAIGN_2026",
  "channel": "HOSTED_CHECKOUT",
  "paid_at": "2026-09-18T14:29:53Z"
}
```

Webhooks are signed with HMAC-SHA256, include timestamp headers and event IDs, support retries with exponential backoff, and have full delivery logs with manual resend.

---

## Running Locally

The local development environment uses Docker Compose with mock providers.

```bash
docker compose up
```

Services started:
- SvelteKit dev server
- Go API
- Go worker
- PostgreSQL
- Redis
- MinIO (S3-compatible object storage)
- Mock payment provider
- Mock USSD provider
- Mock SMS provider

---

## Docs

| Document | Description |
|---|---|
| [`ARCHITECTURE.md`](docs/ARCHITECTURE.md) | System design and component overview |
| [`DATA_MODEL.MD`](docs/DATA_MODEL.MD) | Full database schema |
| [`API_SPECIFICATIONS.md`](docs/API_SPECIFICATIONS.md) | REST API reference |
| [`WEBHOOK_SPECIFICATION.md`](docs/WEBHOOK_SPECIFICATION.md) | Webhook events and security |
| [`PAYMENT_LINKS_AND_QR.md`](docs/PAYMENT_LINKS_AND_QR.md) | Link and QR code behavior |
| [`PAYMENT_STATE_MACHINE.md`](docs/PAYMENT_STATE_MACHINE.md) | Internal payment states |
| [`USSD_FLOW.md`](docs/USSD_FLOW.md) | USSD session and billing model |
| [`SMS_AND_NOTIFICATIONS.md`](docs/SMS_AND_NOTIFICATIONS.md) | Notification rules and templates |
| [`SECURITY.md`](docs/SECURITY.md) | Auth, API keys, and infrastructure security |
| [`PRIVACY_AND_DATA_HANDLING.md`](docs/PRIVACY_AND_DATA_HANDLING.md) | Data protection controls |
| [`DEPLOYMENT.md`](docs/DEPLOYMENT.md) | Staging and production setup |
| [`RUNBOOK.md`](docs/RUNBOOK.md) | Operational runbook |
| [`BACKUP_AND_RECOVERY.md`](docs/BACKUP_AND_RECOVERY.md) | Backup and restoration procedures |
| [`INCIDENT_RESPONSE.md`](docs/INCIDENT_RESPONSE.md) | Incident response process |
| [`TESTING_STRATEGY.md`](docs/TESTING_STRATEGY.md) | Test approach |
| [`CHANGELOG.md`](docs/CHANGELOG.md) | Release history |
