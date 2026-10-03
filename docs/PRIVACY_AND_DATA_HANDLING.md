# Privacy and Data Handling

## Phone Number Handling
- Normalize Kenyan phone numbers server-side.
- Encrypt or protect phone numbers at rest.
- Do not expose full payer numbers publicly.
- Masked payer phone numbers in account-holder ledger (e.g. `2547**9210`).
- Do not include phone numbers in QR codes or public URLs.

## Data Ownership
- Account holder owns: Categories, Campaigns, Payment-purpose config, Reports, Domain records.
- Platform owns: Intents, Attempts, Callbacks, Confirmed records, Ledger entries, Receipt events.
- Payment Provider owns: M-Pesa payment execution, Payer PIN entry, Settlement.
- Platform does NOT own: Customer funds, M-Pesa wallet balances, Customer PINs.
