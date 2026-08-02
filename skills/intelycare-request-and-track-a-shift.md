---
name: Request and track an IntelyCare shift
description: >-
  Create a per-diem nursing shift request on the IntelyCare Staffing platform from a facility's
  own scheduling system, then track it to fill via the ShiftAccept / ShiftRelease webhooks, and
  amend or cancel it if the need changes.
api: openapi/intelycare-external-scheduling-openapi.yml
operations:
  - shift_create_api_v1
  - shift_update_api_v1
  - shift_delete_api_v1
events:
  - shift_accept_webhook_v1
  - shift_release_webhook_v1
generated: '2026-08-01'
method: generated
source: openapi/intelycare-external-scheduling-openapi.yml
---

# Request and track an IntelyCare shift

Use this when a facility has an unfilled nursing shift and wants IntelyCare to source a
qualified professional for it.

## Before you start

- Base URL (production): `https://api.intelycare.com/external-scheduling/v1/`
- Base URL (sandbox): `https://api.pre.prod01.platform.intelycare.com/external-scheduling/v1/`
- Every request needs **both** headers:
  - `X-API-KEY` — the key IntelyCare issued to this client
  - `X-CLIENT-ID` — IntelyCare's unique identifier for this client
- The key is scoped to one client and **cannot be used outside that scope**. A key belonging to
  another client returns `401`, not `403`.
- Everything is `application/json`. **Bulk is not supported — one object per request.**

## Step 1 — Create the shift request

`POST /api/shifts` → `shift_create_api_v1`

Required body fields:

| Field | Notes |
|---|---|
| `externalShiftId` | **Your** id for the shift. This is the key you will match webhooks and timecards on — pick it deliberately and store it. |
| `shiftStartTime` | ISO 8601 |
| `shiftEndTime` | ISO 8601, must be later than `shiftStartTime` and must not be in the past |
| `shiftUnit` | Facility unit where care is delivered, e.g. `1st Floor` |
| `healthcareProfessionalType` | Qualification needed — `CNA`, `LPN`, `RN` |
| `shiftSource` | Where the shift originated |

Optional: `timezone` (e.g. `ET`), `shiftBoostPercentage` (integer, default `0`) to raise the
pay rate and improve fill odds.

### Timestamps — get this right

Append `Z` for UTC (`2024-01-24T05:04:34Z`). **Omit the `Z` for facility-local time**
(`2024-01-24T05:04:34`) — local time resolves against the facility's configured timezone in the
IntelyCare system. Silently dropping the `Z` shifts the whole shift by the UTC offset, and
nothing in the API will tell you. Decide one convention per integration and stick to it.

The `200` response returns `data.internalShiftId` — **IntelyCare's** id for the shift. Persist
it: it is the `{shiftId}` path parameter for update and delete. `externalShiftId` will not work
there.

## Step 2 — Wait for the shift to fill

There is **no GET operation anywhere in this API.** You cannot poll a shift's status. Fill is
learned only from webhooks delivered to your configured HTTPS endpoint.

**`ShiftAccept`** — a professional took the shift. Payload carries `clientId`,
`externalShiftId`, `shiftStatus: Accepted`, `statusTimestamp`, and a `healthcareProfessional`
object with `id`, `externalId`, `firstName`, `lastName`, `healthcareProfessionalType`,
`phoneNumber`, `email` and optionally a base64 `profilePicture`.

**`ShiftRelease`** — the shift was given back. Payload carries `clientId`, `externalShiftId`,
`shiftStatus: Released`, `statusTimestamp`. **The shift is unfilled again** — put it back into
whatever queue your scheduling system uses. Do not assume `Accepted` is terminal.

### Verify the signature before you trust the body

Every webhook carries `X-Signature-IC`: an **HMAC hex digest (SHA256) of the body payload,
computed with the webhook secret**. Compute the same digest over the raw body and compare in
constant time. Reject on mismatch. There is no other authentication on the callback.

### Handle the PII

The `ShiftAccept` payload contains a named individual's phone number, email address and
photograph. Your receiver is a PII processor. Log the ids, not the person.

## Step 3 — Amend or cancel

- `PUT /api/shifts/{shiftId}` → `shift_update_api_v1` — send the full body again (start, end,
  unit, professional type, source; optionally boost and timezone). `{shiftId}` is the
  `internalShiftId` from step 1.
- `DELETE /api/shifts/{shiftId}` → `shift_delete_api_v1` — withdraws the request. If a
  professional has already accepted, this cancels on a real person's schedule. **Confirm with a
  human before an agent calls this.**

## Errors

Envelope is a bare `{"message": "..."}` — no error codes, no `type` URIs, no
`application/problem+json`.

| Status | Meaning | Do |
|---|---|---|
| `400` | Malformed input, e.g. *"Shift end time must be later than the shift start time."* | Fix the body; do not retry unchanged |
| `401` | Bad/missing `X-API-KEY`, or a key outside its client scope | Fix credentials; do not retry |
| `422` | Body parsed but broke a business rule, e.g. *"shiftEndTime must be a date in the present or in the future."* | Fix the semantics |

`400` and `422` return an **`IC-CorrelationID`** response header — a uuid for debugging. Capture
it into your logs and quote it to `apisupport@intelycare.com`.

## Retries — read this before you build any

**IntelyCare publishes no idempotency contract.** There is no `Idempotency-Key` header and no
documented replay behaviour. `externalShiftId` is a natural key, so a repeat create *may* be
rejected or reconciled — but that is not guaranteed anywhere, and a blind retry could
double-book a shift. Do not retry writes automatically. Confirm the actual behaviour with
`apisupport@intelycare.com` before adding any retry policy.
