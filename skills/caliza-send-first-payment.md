---
name: Simulate and send a cross-border payment
description: Preview costs with a simulation, then execute the transaction and track it to completion on the Caliza Core API.
api: Caliza Core API
base_url: https://api.caliza.com/core-api/v1
auth: OAuth 2.0 Bearer token (see authentication/caliza-authentication.yml)
operations: [createsimulation, createtransaction, gettransactionbyid]
---

# Simulate and send a cross-border payment

Caliza uses a mandatory **simulate-then-execute** pattern. You must create a
simulation first; its single-use `simulationId` is required to create the
transaction. Simulations are valid ~24 hours.

## Preconditions
- You hold an OAuth 2.0 access token: `Authorization: Bearer {access_token}`
  (obtain via the Keycloak token endpoint, `grant_type=password`).
- The source beneficiary exists and is verified (see the onboarding skill).

## Steps
1. **Create a simulation** — `createsimulation` (`POST /v2/simulations`). Provide
   the `beneficiaryId`, source `from` and/or destination `to` amount objects
   (`{currencyCode, value}`), and the destination. The response returns the
   `simulationId`, itemized fees, `exchangeRate`, and estimated settlement.
   Show these costs to the user before proceeding.
2. **Execute the transaction** — `createtransaction` (`POST /v1/transactions`).
   Pass the `simulationId` (required) plus `beneficiaryIp`. The transaction is
   created in `CREATED`/`PROCESSING`.
3. **Track it** — poll `gettransactionbyid` (`GET /v1/transactions/{id}`) or,
   preferably, subscribe to `TRANSACTION_COMPLETED` / `TRANSACTION_FAILED`
   webhooks (see skills/caliza-verify-webhook.md).

## Rules and error handling
- A `simulationId` is single-use; reusing an executed one returns
  `simulation.is_already_executed` and an expired one returns
  `simulation.is_expired` — create a fresh simulation.
- `flow_of_funds.insufficient-funds` and `limit.daily_limit_reached` are 400s;
  surface the message to the user.
- Amounts are objects: `{"currencyCode":"USD","value":97.52}`.
- No idempotency key exists — rely on the single-use simulation to prevent
  double execution. See conventions/caliza-conventions.yml and
  errors/caliza-problem-types.yml.
