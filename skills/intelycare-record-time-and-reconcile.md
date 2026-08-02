---
name: Record time worked and reconcile an IntelyCare shift for billing
description: >-
  Post clock-in and clock-out punches for a nursing professional on an IntelyCare shift, then
  submit and correct the timecard that drives billing reconciliation between the facility and
  IntelyCare.
api: openapi/intelycare-external-scheduling-openapi.yml
operations:
  - check_in_out_api_v1
  - timecard_create_api_v1
  - update_timecard_api_v1
generated: '2026-08-01'
method: generated
source: openapi/intelycare-external-scheduling-openapi.yml
---

# Record time worked and reconcile an IntelyCare shift

Use this after a shift has been accepted (see *Request and track an IntelyCare shift*) to record
attendance and close the shift out for billing.

There are two distinct surfaces here and they are easy to confuse:

- **Clock events** (`/api/clockinout/{platform_type}`) — the raw punch, one per arrival and one
  per departure.
- **Timecards** (`/api/timecards`) — the reconciled total for the shift: check-in, check-out and
  break duration. This is what billing reads.

Posting a clock event does **not** create a timecard. Submit both.

## Headers

Same on every call:

- `X-API-KEY` — client-scoped key
- `X-CLIENT-ID` — IntelyCare's identifier for this client

All bodies are `application/json`. One object per request; bulk is not supported.

## Step 1 — Post the clock-in

`POST /api/clockinout/{platform_type}` → `check_in_out_api_v1`

`{platform_type}` is IntelyCare's identifier for the platform calling the API — the docs
illustrate `intelycare`. Confirm the value for your integration with API support.

Required body:

| Field | Notes |
|---|---|
| `nurseId` | IntelyCare's id for the caregiver — the `healthcareProfessional.id` you received on the `ShiftAccept` webhook |
| `shiftId` | IntelyCare's shift id (`internalShiftId` from shift creation) |
| `facilityId` | Identifier for the facility |
| `clockTime` | ISO 8601, **UTC** — e.g. `2024-02-23T07:34:00.000Z` |
| `event` | `CLOCK_IN` or `CLOCK_OUT` — nothing else is accepted |

The `200` echoes your fields plus an `additionalProperties` block containing a confirmation
message (*"Thanks for Checking in."*), `providerId`, `dateCreated` and `clientId`.

## Step 2 — Post the clock-out

Identical call with `event: CLOCK_OUT` and the departure `clockTime`.

## Step 3 — Submit the timecard

`POST /api/timecards` → `timecard_create_api_v1`

All four fields are required:

| Field | Notes |
|---|---|
| `externalShiftId` | **Your** shift id, not IntelyCare's. The timecard has no id of its own — it is keyed entirely on this. |
| `checkIn` | ISO 8601, UTC |
| `checkOut` | ISO 8601, UTC |
| `breakDuration` | Integer **minutes**, e.g. `30` |

Note the identifier switch: clock events key on IntelyCare's `shiftId`, timecards key on **your**
`externalShiftId`. Keep both ids on your shift record from creation onward or you cannot complete
this flow.

The `200` returns `clientId`, the four fields you sent, `createdAt`, and an `errors` field
(`null` in the published example). Check `errors` — it is the only per-record failure channel on
a `200`.

## Step 4 — Correct a timecard

`PUT /api/timecards` → `update_timecard_api_v1`

Same body shape, same required fields, no path parameter — the record is located by
`externalShiftId` in the body. Send the full corrected set, not a delta.

One spec inconsistency to code around: `externalShiftId` is declared `type: string` on timecard
**create** and `type: integer` on timecard **update**, while every published example sends it as
a JSON string. Send it as a string in both directions.

## Errors

Both timecard operations and the clock-in/out operation declare only `200` and `401` in the
spec. There is no documented validation-failure response despite four required fields on each.
Assume a `400`-class response is possible and handle a bare `{"message": "..."}` body
defensively.

`401` means a missing, wrong, or out-of-scope `X-API-KEY`. Do not retry it.

## Care with money and hours

Timecards drive billing reconciliation between the facility and IntelyCare, and the hours are a
real person's pay. Two rules for any automated caller:

1. **Do not auto-retry a timecard write.** There is no idempotency key on this API and no
   documented replay behaviour. A duplicate submission may double-count a shift.
2. **Do not derive `breakDuration` from a guess.** Send what the facility's own timekeeping
   recorded, or do not send the timecard at all.
