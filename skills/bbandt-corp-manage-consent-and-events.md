---
name: truist-manage-consent-and-events
description: Read and revoke a Truist customer consent grant, and subscribe to the consent lifecycle events Truist publishes to a registered callback URL.
api: Truist User Consent / Event Subscriptions / Event Notifications
provider: BB&T Corp (Truist)
providerId: bbandt-corp
generated: '2026-09-04'
method: generated
source: openapi/bbandt-corp-retail-consents-openapi.yml, openapi/bbandt-corp-retail-event-subscriptions-openapi.yml, openapi/bbandt-corp-retail-event-notifications-openapi.yml
operations:
  - getConsentGrant
  - revokeConsentGrant
  - createNotificationSubscription
  - getNotificationSubscription
  - deleteNotificationSubscription
  - publishNotification
base_url: https://api-secure.truist.com/retail
sandbox_url: https://api-sandbox.truist.com/retail
---

# Manage Truist consent and consent events

Consent is the thing that actually authorizes every retail read. These three APIs let you inspect it,
end it, and be told when it ends. They authenticate with **HTTP Basic** (`client_id:client_secret`),
not with the customer's OAuth token.

## Steps

1. **Inspect a grant.** `getConsentGrant` (`GET /v1/consents/{consentId}`).
2. **Revoke a grant.** `revokeConsentGrant` (`PUT /v1/consents/{consentId}/revocation`). This is the
   reversal path for consent, and it is available for the life of the grant — Truist publishes no window.
3. **Subscribe to consent events.** `createNotificationSubscription`
   (`POST /v1/notification-subscriptions`) with `type`, `category`, `callbackUrl`, `subscriber` and
   `subscriptionId`. Published event types are `CONSENT_REVOKED` and `CONSENT_UPDATED`.
4. **Read or remove a subscription.** `getNotificationSubscription` /
   `deleteNotificationSubscription` on `/v1/notification-subscriptions/{subscriptionId}`.
5. **Publish an event to Truist.** `publishNotification` (`POST /v1/notifications`) sends a
   `Notification` the other direction. A published notification cannot be recalled.

## What the contract does not give you

- **A subscription holds exactly one callback URL.** "Previous callback URL will be updated with latest" —
  re-creating a subscription replaces the URL rather than adding one.
- **There is no signature on inbound notifications.** Truist publishes no signing scheme, so the payload
  itself cannot prove it came from Truist. Verify out of band (mutual TLS at your edge, source allowlist)
  before you act on a `CONSENT_REVOKED`.
- **There is no retry or delivery guarantee published.** Treat the event as a hint and reconcile by
  calling `getConsentGrant`; do not treat a missing notification as proof consent still stands.
