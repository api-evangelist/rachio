---
name: Subscribe to device and schedule events
description: Register a webhook for a Rachio device to receive real-time schedule and zone-run events, and verify callback signatures.
api: docs
operations:
  - GET /public/person/info
  - GET /public/device/{id}
  - GET /webhook/listWebhookEventType
  - POST /webhook/createWebhook
---

# Subscribe to Rachio events via webhooks

## Auth
Send `Authorization: Bearer <API_TOKEN>` on every request.

## Steps
1. `GET /public/person/info` then `GET /public/device/{id}` to get the
   `resourceId` (device id) you want events for.
2. `GET /webhook/listWebhookEventType` — list the available event types.
3. `POST /webhook/createWebhook` — body
   `{ "resourceId": "<deviceId>", "url": "https://your-callback", "eventTypes": ["SCHEDULE_STARTED_EVENT", "DEVICE_ZONE_RUN_COMPLETED_EVENT"] }`.
   Up to 10 webhooks may be registered per resource.

## Handling callbacks
- Each callback is a JSON envelope: `eventId`, `eventType`, `payload`,
  `resourceId`, `resourceType`, `timestamp`.
- Verify the `x-signature` header: HMAC-SHA256 of the raw body using your API
  token as the shared secret. Reject on mismatch.
- Full event list and envelope in `../asyncapi/rachio-webhooks.yml`.

## Rules
- Return 2xx quickly and process asynchronously.
- Deduplicate on `eventId` — deliveries may repeat.
