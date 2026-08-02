---
name: Query marketing data (sync and async)
description: Pull marketing data from a connected Supermetrics data source, using the async pattern for large or long-running queries.
api: https://api.supermetrics.com/enterprise/v2
operations:
  - queries.execute
  - query.status
  - query.results
source: https://docs.supermetrics.com/apidocs/making-requests
---

# Query marketing data with the Supermetrics API

Use this skill to extract marketing data (Google Ads, Facebook Ads, GA4, and 100+
sources) from a Supermetrics account.

## Authentication
Send `Authorization: Bearer <api key>` (a valid Supermetrics API license is required),
or use an OAuth2 access token from the `authorization_code` flow with the
`ds_queries_run` scope. See `authentication/supermetrics-authentication.yml`.

## Steps

1. **Build the query.** Assemble the query parameters: `ds_id` (data source, e.g. `GAWA`
   for Google Ads), `ds_accounts`, `start_date`, `end_date`, `fields`, and optionally
   `filter`, `order_rows`, `max_rows`, `settings`.

2. **Execute (synchronous) — `queries.execute`.**
   `GET https://api.supermetrics.com/enterprise/v2/query/data/json?json=<url-encoded JSON>`
   or `POST` the same path with an `application/json` body. Add `pretty=true` for readable
   JSON. Output formats: `json`, `csv`, `parquet`, `html`.

3. **For large/slow queries, go async.** Send the same execute request with
   `sync_timeout=0`. A `202 Accepted` returns `"status_code":"QUEUED"` and a `schedule_id`.

4. **Poll — `query.status`.**
   `GET https://api.supermetrics.com/enterprise/v2/query/status?schedule_id=<id>`
   until the query stops. Max runtime is four hours; status/results are retained up to one
   hour after it stops (missing → `STATUS_NOT_FOUND`).

5. **Fetch results — `query.results`.**
   `GET https://api.supermetrics.com/enterprise/v2/query/results?schedule_id=<id>`.

## Rules
- **Rate limit:** stay under 20 requests/second per license and IP; a `429` means back off.
- **Concurrency:** queries run with limited per-license concurrency; identical simultaneous
  queries are coalesced (only the first runs, up to 10 wait) — this is not a client
  idempotency key.
- **Errors:** parse the envelope `{meta.request_id, error:{code,message,description}, links}`.
  Capture `meta.request_id` for support. See `errors/supermetrics-problem-types.yml`.

## CLI equivalent
`supermetrics queries execute --ds-id GAWA --fields sessions --start-date 2024-01-01 --end-date 2024-01-31 --ds-accounts list.all_accounts --all`
