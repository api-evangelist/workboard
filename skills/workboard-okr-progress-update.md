---
name: Update OKR progress in WorkBoard
description: Find a key result (metric), update its value, and set its RAG confidence using the WorkBoard External API v1.
api: openapi/external-v1-openapi-original.yml
operations:
  - "GET /metric"
  - "GET /metric/{metric_id_path}"
  - "PUT /metric/{metric_id_path}"
  - "PUT /metric/{metric_id_path}/confidence"
---

# Update OKR progress in WorkBoard

The v1 spec publishes no operationIds; operations are referenced by method +
path (stable ids are proposed in `overlays/workboard-external-v1-overlay.yaml`).

## Auth

Send `Authorization: Bearer <token>` on every request (instant token or OAuth
2.0 access token — see `authentication/workboard-authentication.yml`). API
root: `https://www.myworkboard.com/wb/apis`.

## Steps

1. **Locate the key result.** `GET /metric` returns metrics (key results) the
   token owner can access; filter with query parameters, or `GET
   /metric/{metric_id_path}` for a single metric. WorkBoard calls key results
   "metrics".
2. **Update the value.** `PUT /metric/{metric_id_path}` updates metric
   information. Since 2025-10-17 the endpoint documents `update_start_date`
   and `update_end_date` parameters for dating the progress update.
3. **Set confidence (RAG).** `PUT /metric/{metric_id_path}/confidence`
   updates the confidence score (red/amber/green) of the key result. This
   operation can return `500` — treat non-200 as failure and retry with
   caution (no idempotency keys are supported, see
   `conventions/workboard-conventions.yml`).

## Rules

- Expect `400` with no structured body on validation failures (v1 has no
  published error schema — `errors/workboard-problem-types.yml`).
- No documented rate limits; be conservative with polling.
- Automated, recurring updates are better wired through a Datastream
  (`POST /stream`, `POST /stream/{stream_id}`) or the generated webhook
  (`POST /hook/{webhook_hash}`) than repeated PUTs.
