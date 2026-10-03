# Security

## Links and QR
- Use long random public tokens.
- Do not expose internal IDs or credentials.
- Limit usage and expiry.

## Payments
- Backend is always authoritative for amount and configuration validation.
- Idempotency checks on callbacks.
- Passkeys and credentials stored in secret manager.

## Data Isolation
- Scoped by `account_id` based on authenticated session, verified API key, or verified token.
- No trusting frontend `account_id`.
