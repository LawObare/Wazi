# Webhook Specification

## Provider Callbacks
- M-Pesa callbacks sent to: `POST /webhooks/payment-provider/mpesa`
- Processed idempotently (e.g. `UNIQUE(provider_name, provider_transaction_reference)`).

## Outbound Webhooks
- Signed `payment.confirmed` events sent to external systems for dynamic link payments.
- Sent via background worker from outbox table.
