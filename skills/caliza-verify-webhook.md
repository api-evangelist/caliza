---
name: Configure and verify Caliza webhooks
description: Configure the webhook endpoint and verify the HMAC-SHA256 signature on every inbound Caliza webhook before processing it.
api: Caliza Core API
base_url: https://api.caliza.com/core-api/v1
auth: OAuth 2.0 Bearer token (see authentication/caliza-authentication.yml)
operations: [updatewebhookconfig, getintegratorwebhooksecret]
---

# Configure and verify Caliza webhooks

## Steps
1. **Configure the endpoint** — `updatewebhookconfig`
   (`POST /v1/integrators/{integratorId}/webhooks`). Set the HTTPS callback URL
   and the event subscriptions. Your endpoint must return `200 OK` within
   5 seconds. A callback URL must be configured before you can create
   beneficiaries.
2. **Fetch the signing secret** — `getintegratorwebhooksecret`
   (`GET /v1/integrators/{integratorId}/webhook-secret`). Store it in a secrets
   manager. Rotate it with `updateintegratorwebhooksecret` when needed (all
   prior signatures become invalid immediately after rotation).
3. **Verify every request** — read the **raw** request body (do not re-serialize
   the JSON), compute `HMAC-SHA256(rawBody, secret)`, Base64-encode it, and
   constant-time compare it to the `X-Caliza-Webhook-Signature` header. Reject
   any request whose signature does not match.
4. **Dispatch on `data`** — the envelope is shared; branch on the event type
   (e.g. `TRANSACTION_COMPLETED`, `TRANSACTION_FAILED`, `PAYMENT_IN_COMPLETED`,
   `BENEFICIARY_APPROVED`, `SWEEP_PAYOUT_COMPLETED`). See
   asyncapi/caliza-webhooks.yml for the full event catalog and `data` shapes.

## Rules
- Always verify before processing; use constant-time comparison to avoid timing
  attacks.
- In sandbox, trigger events with the mock deposit simulators and
  `createFakePixOutConfirmation` (see sandbox/caliza-sandbox.yml).
