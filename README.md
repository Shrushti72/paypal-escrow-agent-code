# Autonomous Escrow Agent (AEA)

PayPal AI Hackathon demo: milestone funds are **authorized and held** with PayPal Orders v2 (`intent: AUTHORIZE`). A later human action will capture. The AI evaluator (later) only recommends; it must never capture or pay out.

**This is not legally guaranteed escrow.** It is a Sandbox demonstration of authorize-then-human-capture.

## Current step

**Step 1 — create AUTHORIZE orders only**

Included:

- Server-side OAuth (client credentials) with token caching
- `POST /paypal-api/checkout/orders/create` → PayPal `POST /v2/checkout/orders` with `intent: AUTHORIZE`
- `GET /health`

Not included yet: frontend, AI SLA evaluator, capture, void, Payouts v1, Invoicing v2 (Sandbox Invoicing returned 403 insufficient permissions).

## Prerequisites

- Node.js 18+
- A [PayPal Developer](https://developer.paypal.com/home/) Sandbox app with **Client ID** and **Client Secret**
- Sandbox REST calls use `https://api-m.sandbox.paypal.com`

## Setup

```bash
cd C:\Users\shrus\Projects\paypal-escrow-agent
copy .env.example .env
```

Edit `.env` and set:

- `PAYPAL_CLIENT_ID`
- `PAYPAL_CLIENT_SECRET`

Do not put secrets in frontend code or git. `.env` is gitignored.

```bash
npm install
npm start
```

Dev auto-reload:

```bash
npm run dev
```

## Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health` | Process up; whether credentials are set (never returns secrets) |
| POST | `/paypal-api/checkout/orders/create` | Create a Sandbox order with `intent: AUTHORIZE` |

Capture, authorize-after-approval, and payout routes are intentionally absent.

### Create order body

```json
{
  "amount": "25.00",
  "currency": "USD",
  "description": "Milestone 1 hold",
  "contractId": "contract_demo_001"
}
```

- `amount` is required, must be &gt; 0, at most 2 decimal places
- `currency` must be `USD` in this demo
- `intent` from the client is ignored except that non-`AUTHORIZE` values are rejected; the server always sends `AUTHORIZE` to PayPal
- Optional header `PayPal-Request-Id` (UUID) for idempotent retries

Success (`201`) returns PayPal `id`, `status` (typically `CREATED`), `intent`, and HATEOAS `links`. Buyer approval and capture are later steps.

## Postman

1. Start the server (`npm start`).
2. New GET request: `http://localhost:3000/health`
3. New POST request: `http://localhost:3000/paypal-api/checkout/orders/create`
4. Headers: `Content-Type: application/json`
5. Optional: `PayPal-Request-Id: <uuid>`
6. Body → raw JSON using the example above
7. Send

Without `.env` credentials, create-order returns `503 PAYPAL_NOT_CONFIGURED`. Health still returns `200` with `credentialsConfigured: false`.

On PayPal API errors, the JSON may include `debugId` (`paypal-debug-id`). Use that when contacting PayPal support. Do not retry the same funding instrument on `INSTRUMENT_DECLINED` (not expected until a later capture/authorize-after-approval step).

## What was not live-tested in this step

Code was written against PayPal Orders v2 / OAuth docs and the PayPal AI Toolkit best-practices snippets. **Creating a real Sandbox order requires your Client ID and Secret in `.env`.** If those are not set in this environment, treat the implementation as code review only, not a passing API test.

## Later phases (not built)

- Human-approved capture via Payments v2 authorizations
- AI SLA evaluator (proposed 85% threshold; remediation; never auto-settle)
- Ledger UI (`HELD`, `INSPECTING`, `REMEDIATION`, `SETTLED`)
- Render, Postman collection file, CSV export
- Optional Payouts / Invoicing **only if** Sandbox permissions allow and the payment sequence is verified

## License / disclaimer

Hackathon demo. Not a licensed escrow service and not legal advice.
