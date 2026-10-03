# API Specifications

## Endpoints (Examples)

- `POST /v1/categories`: Create categories.
- `POST /v1/payment-intents`: Create intent (Administrator/External).
- `GET /public/payment-links/{token}`: Load public config for hosted page.
- `POST /public/payment-links/{token}/pay`: Create payment intent from public page.
- `GET /public/payment-intents/{id}/status`: Poll for payment status.
