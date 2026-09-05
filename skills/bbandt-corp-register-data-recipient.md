---
name: truist-register-data-recipient
description: Dynamically register, update and delete an OAuth data-recipient client with Truist under RFC 7591, the prerequisite for aggregator access to the retail open-banking APIs.
api: Truist Retail Dynamic Client Registration
provider: BB&T Corp (Truist)
providerId: bbandt-corp
generated: '2026-09-04'
method: generated
source: openapi/bbandt-corp-retail-register-recipient-openapi.yml
operations:
  - createRecipient
  - getRecipient
  - updateRecipient
  - deleteRecipient
base_url: https://api-secure.truist.com/retail
sandbox_url: https://api-sandbox.truist.com/retail
---

# Register a data recipient with Truist

Truist implements RFC 7591 OAuth 2.0 Dynamic Client Registration for data recipients. This product is
flagged `aggregatorOnly` in the Truist catalog — it is the aggregator / data-access-platform path, not
the ordinary application path.

## Steps

1. **Register.** `createRecipient` (`POST /v1/register`) returns the recipient's `clientId` and the
   Basic Auth credentials for that recipient.
2. **Read.** `getRecipient` (`GET /v1/register/{clientId}`).
3. **Update.** `updateRecipient` (`PUT /v1/register/{clientId}`).
4. **Delete.** `deleteRecipient` (`DELETE /v1/register/{clientId}`) — registration is fully reversible.

## The credential trap

The Basic Auth credentials returned by dynamic registration are **not** the credentials you use on the
OAuth token endpoint. Truist states it plainly in the OAuth spec: Data Access Platforms and Direct Data
Recipients must use the DAP/DDR `client_id` and `client_secret` from the application they created in the
Truist Developer Center, and *cannot* use the Data Recipient credentials returned from
`POST /v1/register`. Mixing the two produces `401`, code `603`, "Authentication failed", which reads
identically to an expired token.
