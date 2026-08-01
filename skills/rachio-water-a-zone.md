---
name: Start and stop watering a zone
description: Authenticate, find a device and its zones, start watering a zone for a set duration, and stop all watering.
api: docs
operations:
  - GET /public/person/info
  - GET /public/person/{id}
  - GET /public/device/{id}
  - PUT /public/zone/start
  - PUT /public/device/stop_water
---

# Start and stop watering a Rachio zone

Use the Rachio Public API (base `https://api.rach.io/1`) to control irrigation.

## Auth
Every request sends `Authorization: Bearer <API_TOKEN>`. The token is one per
Rachio account, copied from the Rachio mobile app (Profile -> API key).

## Steps
1. `GET /public/person/info` — returns the authenticated person's `id`.
2. `GET /public/person/{id}` — returns the person and their `devices[]`.
3. `GET /public/device/{id}` — returns the device with its `zones[]`; each zone
   has an `id` and `zoneNumber`.
4. `PUT /public/zone/start` — body `{ "id": "<zoneId>", "duration": <seconds> }`
   starts watering that zone. `duration` is in seconds.
5. `PUT /public/device/stop_water` — body `{ "id": "<deviceId>" }` stops all
   watering on the device.

## Rules
- Respect the daily quota of 3,500 requests/token; watch `X-RateLimit-Remaining`.
- Zone/device control is state-setting; there is no idempotency-key header, so do
  not blindly retry a start — re-read device state first (see
  `../conventions/rachio-conventions.yml`).
- Handle `401` (bad token), `404` (unknown id) and `429` (rate limited) per
  `../errors/rachio-problem-types.yml`.
