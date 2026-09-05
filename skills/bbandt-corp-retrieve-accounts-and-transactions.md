---
name: truist-retrieve-accounts-and-transactions
description: Retrieve a Truist personal or small-business customer's accounts, account detail and transaction history over the FDX open-banking APIs, using a consented OAuth 2.0 access token.
api: Truist Personal and Small Business Accounts / Transactions
provider: BB&T Corp (Truist)
providerId: bbandt-corp
generated: '2026-09-04'
method: generated
source: openapi/bbandt-corp-retail-accounts-openapi.yml, openapi/bbandt-corp-retail-accounts-transaction-openapi.yml, openapi/bbandt-corp-retail-auth-oauth-openapi.yml
operations:
  - authorize
  - token
  - searchForAccounts
  - getAccount
  - searchForAccountTransactions
base_url: https://api-secure.truist.com/retail
sandbox_url: https://api-sandbox.truist.com/retail
---

# Retrieve Truist accounts and transactions

Truist's retail open-banking surface follows the Financial Data Exchange (FDX) Core API. Nothing here works
without a **customer consent grant** — the token is per-end-user, not per-application.

## Preconditions

- A registered application in the Truist Developer Center with a Client Key and Client Secret.
- Sandbox first: `https://api-sandbox.truist.com/retail`. Sandbox credentials do **not** work in production.
- Production access requires an approved "Promote to production" request.

## Steps

1. **Get customer consent.** Send the customer to `authorize` (`GET /v3/authorize` on
   `https://api.truist.com/retail/auth/oauth`). Request only the scopes you need:
   `ACCOUNT_BASIC` to list accounts, `ACCOUNT_DETAILED` for account detail, `TRANSACTIONS` for
   transaction history.
2. **Exchange the code.** Call `token` (`POST /v3/token`) with HTTP Basic
   `Base64(client_id:client_secret)`. You get an access token, a refresh token and an `id_token`.
3. **List accounts.** Call `searchForAccounts` (`GET /v2/accounts`). Pass `resultType=lightweight`
   (the default) for `AccountDescriptor` metadata, or `resultType=details` for full `Account` bodies.
4. **Read one account.** Call `getAccount` (`GET /v2/accounts/{accountId}`). `accountId` is an opaque
   token, not the account number — Truist tokenizes it deliberately, so store the token, never the PAN.
5. **Read transactions.** Call `searchForAccountTransactions`
   (`GET /v2/accounts/{accountId}/transactions`).

## Rules that apply to every call

- **Send `x-fapi-interaction-id`** on the request — it is a required header and Truist echoes it on every
  response, including errors. Log it; it is the only handle Truist support has on a failed call.
- Set `FDX-API-Actor-Type: BATCH` when the customer is not present, `USER` when they are.
- **There is no idempotency mechanism.** These are reads, so retry freely — but see the payments skill
  before retrying anything that writes.
- **No rate-limit headers exist.** A 429 with code `1207` is an Apigee spike arrest; `1207`/`1208` with
  message "Quota violation" is a quota. Back off exponentially with jitter; there is no `Retry-After`.
- Errors are the FDX `Error` envelope — `{ "code": "...", "message": "..." }` — not RFC 9457
  `application/problem+json`. See `errors/bbandt-corp-problem-types.yml` for the 119 published codes.
- `403` with code `602` means the consent does not authorize this data, not that the token is bad.
  `401` with code `603` means the token is bad. They are different failures; do not retry the first.

## Consent can disappear underneath you

Subscribe to `CONSENT_REVOKED` and `CONSENT_UPDATED` through
`createNotificationSubscription` (`POST /v1/notification-subscriptions`) so you learn the moment access
ends, rather than discovering it as a 403 mid-batch.
