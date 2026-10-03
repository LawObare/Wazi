# Architecture

The system operates in two modes:
1. **Standalone Mode:** Direct use by account holders.
2. **Integration-Layer Mode:** Used by external systems via API.

## High-Level Data Flow

Configuration -> Payment Intent -> Payment Attempt -> Provider Callback -> Confirmed Payment Transaction -> Ledger Entry -> Receipt/Report/Integration Event.

## Multi-Account Architecture

- One shared application, database, backend, and frontend.
- `account_id` used for data isolation (not exposed).

## Key Components
- SvelteKit (Dashboard / Mobile App)
- Go/Gin API (Backend)
- PostgreSQL (Database)
- Redis (USSD Session State)
- Worker (for async tasks like Receipts, Reports, Webhooks)
