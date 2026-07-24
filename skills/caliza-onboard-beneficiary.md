---
name: Onboard and verify a beneficiary with a virtual account
description: Register an individual or business beneficiary, run KYC/KYB verification, and issue a USD virtual account on the Caliza Core API.
api: Caliza Core API
base_url: https://api.caliza.com/core-api/v1
auth: OAuth 2.0 Bearer token (see authentication/caliza-authentication.yml)
operations: [savebeneficiary, createverification, getverificationstatus, createvirtualaccount]
---

# Onboard and verify a beneficiary with a virtual account

## Preconditions
- OAuth 2.0 access token in the `Authorization: Bearer` header.
- Your integrator has a webhook callback URL configured — creating beneficiaries
  before that returns `callback_url.not_set` (412).

## Steps
1. **Register the beneficiary** — `savebeneficiary` (`POST /v1/beneficiaries`).
   Set `integratorBeneficiaryId` (your external id) and `type`
   (`INDIVIDUAL`/`PERSON` or `BUSINESS`). Provide the matching `person` or
   `business` block plus a valid `address` (country/state must follow ISO
   3166-2). Phone numbers must match `+` then 6–20 digits.
2. **Start verification** — `createverification`. This returns a hosted
   `verificationUrl`; redirect the end user there to complete KYC (individuals)
   or KYB (businesses). For businesses, add owners/directors via related-person
   operations first.
3. **Poll status** — `getverificationstatus` until approved, or handle the
   `BENEFICIARY_APPROVED` / `BENEFICIARY_FAILED` webhook. In sandbox you may
   short-circuit with `approveVerificationSandbox`.
4. **Issue a virtual account** — `createvirtualaccount` for the beneficiary in
   the target currency/region to receive deposits over supported rails
   (ACH, WIRE, PIX, SPEI, ...).

## Rules and error handling
- Beneficiaries can only be updated while `PENDING`
  (`beneficiary.operation_not_permitted`).
- A duplicate `integratorBeneficiaryId` returns `beneficiary.conflict` (409).
- Transacting before KYX completes returns
  `flexiblekyx.missing-kyx-requirements` (422).
- See data-model/caliza-data-model.yml for the entity graph and
  errors/caliza-problem-types.yml for the full catalog.
