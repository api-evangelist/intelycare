# IntelyCare

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
