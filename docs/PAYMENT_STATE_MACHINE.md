# Payment State Machine

## Internal States
- `CREATED`: Intent/Attempt initialized.
- `PUSH_SENT`: Request sent to provider.
- `PAID`: Confirmed successful by callback.
- `FAILED`: Definitive failure (cancelled, insufficient funds).
- `UNKNOWN`: No callback received after timeout.
- `REVERSED` / `REFUNDED`

## Payer-Facing Status
- Payment prompt sent.
- Payment received.
- Payment not completed.
- Payment link unavailable.

## Reconciliation States
- MATCHED, FAILED, EXPIRED, UNKNOWN, AMOUNT_MISMATCH, DUPLICATE, UNMATCHED_PROVIDER_PAYMENT, CORRECTED, REVERSED, REFUNDED.
