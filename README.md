# IntelyCare

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

IntelyCare is a healthcare workforce platform connecting post-acute and acute-care facilities with
W2-employed per-diem nursing professionals ("IntelyPros") — RNs, LPNs and CNAs — across per-diem
shifts, contract placements and travel assignments, plus credentialing, onboarding, continuing
education (IntelyEdu) and 24/7 clinical support. IntelyCare acquired CareRev in January 2026 and
also operates the Credenza verified-nursing-identity job board.

- https://www.intelycare.com/
- https://apidocs.intelycare.com/

## API

**IntelyCare External Scheduling API v1** — a REST API that lets a facility's own scheduling or
EHR system create, update and cancel shift requests, exchange timecards for billing
reconciliation, and post clock-in/clock-out events. Shift status flows back asynchronously over
two HMAC-SHA256-signed webhooks (ShiftAccept, ShiftRelease).

- Production: `https://api.intelycare.com/external-scheduling/v1/`
- Sandbox: `https://api.pre.prod01.platform.intelycare.com/external-scheduling/v1/`
- Auth: `X-API-KEY` header (client-scoped) plus `X-CLIENT-ID`
- 6 REST operations, 2 webhook events, no GET operation anywhere

The OpenAPI 3.0.0 document in `openapi/` was harvested verbatim from the `__redoc_state` payload
embedded in IntelyCare's Redoc docs page — IntelyCare serves no `/openapi.json`, `/swagger.json`
or `/openapi.yaml` on any host.

## Artifacts

| Dir | What |
|---|---|
| `openapi/` | Harvested OpenAPI 3.0.0 (+ original JSON in `_original/`) |
| `asyncapi/` | AsyncAPI 3.0.0 projection of the spec's `x-webhooks` event surface |
| `overlays/` | OpenAPI Overlay 1.0.0 of our enhancements and observed spec defects |
| `authentication/` | Auth profile — API key + HMAC webhook signing |
| `conventions/` | Cross-cutting semantics: versioning, tracing, error envelope, timestamps |
| `errors/` | Error catalogue derived from the 4xx responses |
| `data-model/` | Entity graph: Client, Facility, Shift, HealthcareProfessional, Timecard, ClockEvent |
| `examples/` | Every request/response/webhook example IntelyCare publishes |
| `sandbox/` | Environments and how credentials are provisioned |
| `lifecycle/` | Versioning; deprecation, SLA and status-page absence |
| `conformance/` | Standards conformance assessment |
| `security/` | Probed TLS/HSTS/DNSSEC/CAA/SPF/DMARC posture |
| `well-known/` | `/.well-known/` probe results across every host |
| `agentic-access/` | Recommended `x-agentic-access` execution contracts |
| `skills/` | Two packaged Agent Skills covering all 6 operations and both events |
| `mcp/` | Candidate MCP tool surface — **not** published by IntelyCare |
| `llms/` | Generated `llms.txt` |

## Not published by IntelyCare

Probed 2026-08-01 and confirmed absent: SDKs/client libraries in any package registry, a CLI, a
Postman collection, an MCP server, an A2A agent card, a `security.txt`, OAuth/OIDC discovery, a
status page, a public API changelog, a deprecation policy, a trust center, published compliance
certifications, and a vulnerability-disclosure program.
