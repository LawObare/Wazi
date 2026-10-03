# Data Model

Core entities:
- **Account:** id, legal_name, public_display_name, etc.
- **Administrator:** id, account_id, email, password_hash, etc.
- **Category:** What a payment is for (Fund, Fee, etc.).
- **Campaign:** Time-bound or goal-based collection.
- **PaymentLink:** Represents public tokens for QR/Link payments.
- **PaymentIntent:** The requested payment.
- **PaymentAttempt:** The request sent to the provider.
- **PaymentTransaction:** Confirmed outcome.
- **LedgerEntry:** Final categorized record for the account holder.
- **TransactionCorrection:** For correcting categorized payments.
