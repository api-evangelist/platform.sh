---
name: platform.sh-create-preview-environment
description: Branch a Platform.sh / Upsun environment from a parent, activate it, and wait for the deployment activity to complete before reading the environment's public URLs.
api: Platform.sh REST API
base_url: https://api.upsun.com
generated: '2026-08-26'
method: generated
source: openapi/platform.sh-rest-api-openapi.json
operations:
  - list-orgs
  - list-org-projects
  - list-projects-environments
  - branch-environment
  - activate-environment
  - get-projects-environments-activities
  - list-projects-environments-routes
---

# Create a preview environment

A preview environment is a full copy of its parent — code, services and data. Branching is a
long-running operation: the write returns an **Activity**, not a finished environment.

## Before you start

- Authenticate with an OAuth 2.0 bearer token. Exchange an API token for one:
  `POST https://auth.upsun.com/oauth2/token` with `grant_type=api_token&api_token=<token>`,
  HTTP basic user `platform-api-user`. The access token lives 900 seconds — refresh long runs.
- Send `Authorization: Bearer <access_token>` on every call.

## Steps

1. **Find the project.** `list-orgs` (`GET /organizations`), then `list-org-projects`
   (`GET /organizations/{organization_id}/projects`). Both accept cursor pagination:
   `page[size]`, then `page[after]` from `_links.next.href`.
2. **Pick the parent environment.** `list-projects-environments`
   (`GET /projects/{projectId}/environments`). The environment ID is the branch name.
3. **Branch.** `branch-environment`
   (`POST /projects/{projectId}/environments/{environmentId}/branch`) against the PARENT
   environment. Capture the returned activity `id`.
4. **Poll the activity.** `get-projects-environments-activities`
   (`GET /projects/{projectId}/environments/{environmentId}/activities/{activityId}`).
   `state` moves through `pending` → `scheduled`/`staged` → `in_progress` → `complete` or
   `cancelled`; `result` is `success` or `failure`. Do not proceed until `state` is `complete`.
5. **Activate if needed.** A branched environment may be inactive. `activate-environment`
   (`POST /projects/{projectId}/environments/{environmentId}/activate`) returns another activity —
   poll it the same way.
6. **Read the URLs.** `list-projects-environments-routes`
   (`GET /projects/{projectId}/environments/{environmentId}/routes`). There is no dedicated
   "environment URLs" operation; the route list is the URL set.

## Rules

- **Not idempotent.** No `Idempotency-Key` header exists on this API. If step 3 times out, do NOT
  blind-retry — call `list-projects-environments-activities` and look for an in-flight branch
  activity first, or you will create a second environment.
- **403 vs 404.** A caller without the environment-type role may get `404 Not Found` where you
  expected `403 Forbidden`. Check the role before assuming the resource is missing.
- Errors are RFC 9457 `application/problem+json`; read `detail` for per-field validation errors.
  See `errors/platform.sh-problem-types.yml`.
- To abort a running branch, `action-projects-environments-activities-cancel`
  (`POST .../activities/{activityId}/cancel`) works only while the activity is in progress.
