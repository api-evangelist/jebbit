---
name: Build a Jebbit dynamic product feed
description: Create a Jebbit product feed, define its columns, and load product rows so experiences can render dynamic product recommendations.
api: openapi/jebbit-openapi-original.json
generated: '2026-07-19'
method: generated
operations:
  - POST /api/v1/auth
  - POST /api/v1/feeds
  - GET /api/v1/feeds/{feed_id}/feed_columns
  - POST /api/v1/feed_columns
  - POST /api/v1/feed_rows
---

# Build a Jebbit dynamic product feed

A Jebbit product feed powers dynamic product/recommendation experiences. You create a feed, describe its columns, then load rows.

## Conventions
- Base URL `https://api2.jebbit.com`; media type `application/vnd.api+json`.
- Bearer JWT from `POST /api/v1/auth` (see `jebbit-create-webhook-integration.md`, step 1).
- Multi-business tokens must send `x-jebbit-business: <business_id>`.
- List endpoints for columns/rows require the `filter[feed_id]` query parameter.

## Steps
1. **Create the feed** — `POST /api/v1/feeds` with `{"data":{"type":"feeds","attributes":{"name":"My Product Feed"}}}`. Keep the returned feed `id`.
2. **Inspect existing columns** — `GET /api/v1/feeds/{feed_id}/feed_columns` (or `GET /api/v1/feed_columns?filter[feed_id]=<feed_id>`).
3. **Add custom columns** — `POST /api/v1/feed_columns`, referencing the `feed_id`, one per product attribute (e.g. `title`, `price`, `image_url`, `product_url`).
4. **Load rows** — `POST /api/v1/feed_rows` with a `product_identifier`, the `feed_id`, and a `data` object of the column values. One row per product.
5. **Maintain** — use the `PATCH`/`DELETE` variants on `/api/v1/feed_columns/{id}` and `/api/v1/feed_rows/{id}` to keep the feed current.

## Errors
- `400 Bad Request` — usually a missing `filter[feed_id]` or malformed JSON:API body.
- `403 Forbidden` — scope / `x-jebbit-business` issue.
- See `errors/jebbit-problem-types.yml`.
