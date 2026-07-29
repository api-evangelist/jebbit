---
name: Create and verify a Jebbit webhook integration
description: Authenticate to the Jebbit API, create a webhook integration that streams experience session data, and validate delivery with a signed test webhook.
api: openapi/jebbit-openapi-original.json
generated: '2026-07-19'
method: generated
operations:
  - POST /api/v1/auth
  - GET /api/v1/self
  - POST /api/v1/integrations
  - POST /api/v1/integrations/{integration_id}/test
---

# Create and verify a Jebbit webhook integration

Jebbit (BlueConic Experiences) delivers user session data — identifiers, declared attributes, outcomes, opt-in — to your endpoint via signed webhooks. Use this flow to stand one up.

## Prerequisites
- A `client_id` / `client_secret` issued by your Jebbit Customer Success representative.
- A public HTTPS endpoint to receive `POST`s.

## Conventions (apply to every call)
- Base URL: `https://api2.jebbit.com`
- Media type: `application/vnd.api+json` (JSON:API — bodies use the `data.type` + `data.attributes` envelope).
- Auth: `Authorization: Bearer <jwt>`.
- If your token has scope to multiple businesses, send `x-jebbit-business: <business_id>` on every request.

## Steps
1. **Mint a token** — `POST /api/v1/auth` (Auth0 token endpoint `https://auth.jebbit.com/oauth/token`) with `grant_type=client_credentials`, `audience=public-api`, and your `client_id`/`client_secret`. The response `access_token` is a JWT valid 24 hours — cache and reuse it, do not mint per request.
2. **Confirm the account** — `GET /api/v1/self`. The returned `data.id` is your business id (use it for `x-jebbit-business`).
3. **Create the integration** — `POST /api/v1/integrations` with `{"data":{"type":"integrations","attributes":{"endpoint":"<your https endpoint>","region":"us","is_active":true}}}`. Persist the returned `attributes.shared_secret` — it is the HMAC key and is only returned here.
4. **Send a test** — `POST /api/v1/integrations/{integration_id}/test` with an empty body `{}`. Jebbit posts a sample submission to your endpoint carrying an `x-jebbit-test` header (discard these in production ingestion).
5. **Verify the signature** on every delivery: read `x-jebbit-signature: t=<ts>,v1=<hmac>`; reject if `ts` is older than ~5 minutes (replay guard); recompute `Base64(HMAC-SHA256(shared_secret, "<ts>.<raw_body>"))` and constant-time compare against `v1`.

## Errors
- `401 Unauthorized Request` — token missing/expired; mint a fresh one.
- `403 Forbidden` — missing scope or wrong/absent `x-jebbit-business`.
- See `errors/jebbit-problem-types.yml` (JSON:API `errors[]` envelope).
