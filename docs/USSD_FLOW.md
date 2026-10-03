# USSD Flow

USSD is a core optional collection channel for feature-phone users.

- Shares shortcode + account alias/code.
- Adapts channel logic using Redis for temporary session state.
- Stores payment intent, attempt, and ledger in PostgreSQL.
- Initiates STK push.
- Does NOT contain independent financial business logic.
- Supported billing modes: DISABLED, PAYER_PAID, ACCOUNT_PAID, HYBRID.
