---
name: Run a batch payout to saved recipients
description: Register payout recipients, simulate costs, and execute a batch payout to many parties in a single call on the Caliza Core API.
api: Caliza Core API
base_url: https://api.caliza.com/core-api/v1
auth: OAuth 2.0 Bearer token (see authentication/caliza-authentication.yml)
operations: [createrecipient, createsimulation, payouts, getalltransactions]
---

# Run a batch payout to saved recipients

Use this for payroll, marketplace payouts, or affiliate commissions — one call
distributes funds to many recipients, typically at lower fees than individual
transactions.

## Preconditions
- OAuth 2.0 access token in the `Authorization: Bearer` header.
- A funded source beneficiary/account.

## Steps
1. **Register recipients** — `createrecipient` (`POST` under a beneficiary) for
   each payout destination: bank details (account/routing or SWIFT/BIC), type
   (individual/business), currency, and address. Reuse saved recipients across
   runs; list them with `getrecipients`.
2. **Simulate** — `createsimulation` to preview per-payout fees and FX before
   committing. When a payout is involved, the `recipientId` is required (V2).
3. **Execute the batch** — `payouts` (`POST /v1/transactions/payouts`). Provide
   `source`, `sourceType`, and a `transactions` list where each item carries
   `amount`, `target`, `targetType`, and `paymentRailType`. Missing any of these
   returns a validation `errors[]` list.
4. **Reconcile** — list results with `getalltransactions` (filter by status /
   date / beneficiary) or handle `TRANSACTION_COMPLETED` /
   `SWEEP_PAYOUT_COMPLETED` webhooks.

## Rules and error handling
- `payout.unprocessable` (422) means the payout request is missing required
  configuration.
- Batch amounts below the rail minimum raise `flow_of_funds.invalid-amount`;
  insufficient balance raises `flow_of_funds.insufficient-funds`.
- For treasury consolidation across many beneficiaries, use the atomic
  `sweeppayouts` operation instead. See conventions/caliza-conventions.yml.
