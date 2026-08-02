---
name: Onboard an end user's data source with a login link
description: Create a single-use login link so an end user authenticates a marketing data source, then confirm the login and list its accounts.
api: https://api.supermetrics.com/enterprise/v2
operations:
  - login-links.create
  - login-links.get
  - logins.list
  - accounts.list
source: https://docs.supermetrics.com/apidocs/management-api
---

# Onboard a data source via a Supermetrics login link

Use this skill to let an end user connect a marketing data source under your app,
using the Management API (or the first-party CLI).

## Authentication
Use an OAuth2 access token with the `ds_login_links_write` / `ds_login_links_read`
and `ds_logins_read` / `ds_accounts_read` scopes, or an API key with a license that
permits Management operations. See `scopes/supermetrics-scopes.yml`.

## Steps

1. **Create a login link — `login-links.create`.** Request a single-use data-source
   login link (optionally branded via the Branded connection flow). The end user opens it
   and authenticates their data source (Google, Meta, etc.).

2. **Check the link / poll — `login-links.get`.** Retrieve the link to see whether the
   user has completed authentication; `login-links.list` enumerates open links and
   `login-links.close` retires one.

3. **Confirm the login — `logins.list`.** Once authenticated, the new data source login
   appears; read it to get the `login_id` and `ds_info`.

4. **List accounts — `accounts.list`.** For the connected data source, list the accounts
   now available to query (`GET .../accounts` for a given `ds_id`), then hand them to the
   query skill.

## Rules
- Treat login links as single-use secrets; close them when done.
- Errors use the standard envelope `{meta.request_id, error, links}` — licensing failures
  surface as `LICENSE_*` codes (see `errors/supermetrics-problem-types.yml`).

## CLI equivalent
```
supermetrics login-links create --dry-run
supermetrics logins list -o table
supermetrics accounts list --ds-id GAWA
```
