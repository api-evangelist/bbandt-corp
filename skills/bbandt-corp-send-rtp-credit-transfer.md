---
name: truist-send-rtp-credit-transfer
description: Initiate, approve, track and cancel a real-time payment (RTP) credit transfer from a Truist commercial account, including the duplicate and limit failure modes.
api: Truist Credit Transfers
provider: BB&T Corp (Truist)
providerId: bbandt-corp
generated: '2026-09-04'
method: generated
source: openapi/bbandt-corp-commercial-credit-transfers-oas-v2-openapi.yml
operations:
  - getEligibleCreditTransfer
  - createRtpCreditTransfer
  - getRealTimePaymentApprovals
  - createRtpCreditTransferApprove
  - getRealTimePaymentTransactions
  - updatePaymentStatus
  - createRtpPaymentAcknowledgment
base_url: https://api.truist.com/commercial
sandbox_url: https://api-sandbox.truist.com/commercial
---

# Send an RTP credit transfer from a Truist commercial account

This is the only write surface in the Truist estate that moves money. Read the reversal section
**before** you call `createRtpCreditTransfer`.

## Steps

1. **Check eligibility.** `getEligibleCreditTransfer` (`GET /v2/payments/rtp/credit-transfers/accounts`)
   returns the accounts that may originate a transfer. An ineligible account fails late, at code `8037`
   `CLOSED_ACCOUNT`, so check first.
2. **Submit the transfer.** `createRtpCreditTransfer` (`POST /v2/payments/rtp/credit-transfers`).
   Required in the body: `amount`, `fromAccountId`, `paymentDate`, `correlationId` and the full `creditor`
   block (account number, routing number, address line, city, state, zip). Every one of those has its own
   400 code — `7000`, `7001`, `7002`, `7003`, `7005`, `7006`, `7008`, `7009`, `7010`, `7011` — so validate
   locally rather than discovering them one call at a time.
3. **Approve, if the payment lands in `needsApproval`.** List with `getRealTimePaymentApprovals`
   (`GET /v2/payments/rtp/credit-transfers/approvals`), then `createRtpCreditTransferApprove`
   (`POST /v2/payments/rtp/credit-transfers/approvals`) with status `APPROVE` or `REJECT`.
4. **Track.** `getRealTimePaymentTransactions` (`GET /v2/payments/rtp/credit-transfers`) accepts
   `paymentId` or `correlationId` plus date and amount ranges, and pages with `offset`/`limit`.
   Status moves through `needsApproval`, `scheduled`, `processing`, `approved`, `completed`, or lands on
   `rejected`, `rejectedByApprover`, `canceled`, `failed`, `expired`.
5. **Acknowledge inbound transfers.** `createRtpPaymentAcknowledgment`
   (`POST /v2/payments/rtp/credit-transfers/{paymentId}/acknowledgements`). There is no un-acknowledge.

## Retrying is not safe

Truist publishes **no `Idempotency-Key` header and no replay semantics**. If `createRtpCreditTransfer`
times out you do not know whether the payment was created. Do not blind-retry.

- Truist detects duplicates server-side and returns **HTTP 409, code `908 DUPLICATE_PAYMENT_REQUEST`**.
  Treat a 409 as "the first attempt probably landed".
- The safe recovery is to **query, not retry**: call `getRealTimePaymentTransactions` filtered on the
  `correlationId` you sent. That is exactly why `correlationId` is a required field — supply your own
  unique value on every submission and keep it.

## Reversal

- **Cancel:** `updatePaymentStatus` (`PUT /v2/payments/rtp/credit-transfers`) with
  `{"status":"CANCEL","payments":["<paymentId>"]}`. The spec summary is literally "Credit transfers cancel".
- **Reject before approval:** `createRtpCreditTransferApprove` with status `REJECT`.
- **Truist publishes no cancellation window.** Do not promise a customer a payment can be pulled back.
  RTP settles in real time and settlement is final on the network; a cancel that arrives after `completed`
  should be expected to fail. Check status first.

## Limits you will hit

`8027 TRANSACTION_LIMIT_EXCEEDED`, `8026 DAILY_LIMIT_EXCEEDED`, `8029 USER_DAILY_LIMIT_EXCEEDED` — the
thresholds are set per client and are not published. `8002 AMOUNT_MORE_THAN_10MILLION` is the one stated
ceiling: 10 million USD.
