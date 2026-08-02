---
name: Build a team OKR report from WorkBoard
description: Pull a team's goals (objectives), their key results, and alignment to assemble a status report using the WorkBoard External API v1.
api: openapi/external-v1-openapi-original.yml
operations:
  - "GET /team"
  - "GET /team/{team_id_path}"
  - "GET /goal"
  - "GET /goal/{goal_id_path}"
  - "GET /goal/{goal_id_path}/alignment"
  - "GET /goal/{goal_id_path}/pillars"
  - "GET /metric/{metric_id_path}"
---

# Build a team OKR report from WorkBoard

The v1 spec publishes no operationIds; operations are referenced by method +
path (stable ids are proposed in `overlays/workboard-external-v1-overlay.yaml`).

## Auth

`Authorization: Bearer <token>` on every request; the token's user must be a
member of the teams being read (a Data-Admin token widens access on user
endpoints). API root: `https://www.myworkboard.com/wb/apis`.

## Steps

1. **Resolve the team.** `GET /team` lists teams the token owner belongs to;
   `GET /team/{team_id_path}` fetches one.
2. **Pull goals.** `GET /goal` with filter parameters, or `GET
   /goal/{goal_id_path}` for a specific objective. WorkBoard calls
   objectives "goals".
3. **Expand key results.** For each goal, read its metrics via `GET
   /metric/{metric_id_path}` — the `metric_automation` object (added
   2023-07-11) shows whether a value is fed automatically, and
   `metric_mit_start_date`/`metric_mit_end_data` give multi-interval-period
   durations.
4. **Add strategic context.** `GET /goal/{goal_id_path}/pillars` returns the
   goal's strategies and pillars; `GET /goal/{goal_id_path}/alignment` lists
   aligned and dependent goals for the dependency section of the report.

## Rules

- All list endpoints return `400` on invalid filters; there is no structured
  error body in v1 (`errors/workboard-problem-types.yml`).
- Larger member lists paginate with `limit`/`offset` (documented for
  `include=org_members` on the user resource).
- For agent-driven reporting, the published MCP server
  (`https://www.myworkboard.com/wb/mcp`, see `mcp/workboard-mcp.yml`)
  exposes richer read tools (get_team_objectives, get_key_result_details,
  scorecard_get_csv) behind per-user authorization.
