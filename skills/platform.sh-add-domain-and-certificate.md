---
name: platform.sh-add-domain-and-certificate
description: Attach a custom domain to a Platform.sh / Upsun project and install or inspect its TLS certificate.
api: Platform.sh REST API
base_url: https://api.upsun.com
generated: '2026-08-26'
method: generated
source: openapi/platform.sh-rest-api-openapi.json
operations:
  - list-projects-domains
  - create-projects-domains
  - get-projects-domains
  - update-projects-domains
  - delete-projects-domains
  - list-projects-certificates
  - create-projects-certificates
  - get-projects-certificates
  - delete-projects-certificates
---

# Add a custom domain and its certificate

Domains exist at two levels and the API keeps them separate. Project domains
(`/projects/{projectId}/domains`) apply to the production route set; environment domains
(`/projects/{projectId}/environments/{environmentId}/domains`) are scoped to one environment.
Choose deliberately — they are different operations, not variants of one.

## Steps

1. **Check what is already attached.** `list-projects-domains`
   (`GET /projects/{projectId}/domains`). Re-adding an existing domain returns `409 Conflict`.
2. **Add the domain.** `create-projects-domains` (`POST /projects/{projectId}/domains`).
3. **Verify.** `get-projects-domains` (`GET /projects/{projectId}/domains/{domainId}`); update with
   `update-projects-domains` (`PATCH`).
4. **Certificates.** `list-projects-certificates` (`GET /projects/{projectId}/certificates`) shows
   what is installed. To install your own, `create-projects-certificates`
   (`POST /projects/{projectId}/certificates`); inspect with `get-projects-certificates`.
5. **Remove.** `delete-projects-domains` and `delete-projects-certificates`. Both are irreversible
   and take the domain offline immediately.

## Rules

- Certificate material is a secret. Never log a request body for `create-projects-certificates`,
  and never write one into an artifact or a transcript.
- Errors are RFC 9457 `application/problem+json`; `detail` carries the field-level reasons a domain
  was rejected (ownership, format, already-claimed).
- Domain claims are a separate resource (`/projects/{projectId}/domain-claims`) used to assert
  ownership; do not confuse a claim with an attached domain.
- No `Idempotency-Key` exists. On a timeout, re-list before re-posting.
