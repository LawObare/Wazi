# Payment Links and QR

The QR code is the entry point, resolving to a hosted payment link.

## Static Links
- Reusable collection point.
- Unlimited uses by default, no default expiry.
- Supports Fixed, Suggested, or Open amounts.

## Dynamic Links
- Created for one payment request (invoice, order).
- Can have fixed amount, expiry, single-use limit.
- External reference mapping.

## QR Rules
- Only contains public payment URL.
- Generated in PNG or SVG.
- Does NOT contain provider credentials, passkeys, or payer info.
