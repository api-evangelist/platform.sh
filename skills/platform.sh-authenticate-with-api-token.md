---
name: platform.sh-authenticate-with-api-token
description: Exchange a Platform.sh / Upsun API token for a bearer access token, confirm the caller identity, and manage the token set for a machine user.
api: Platform.sh REST API
base_url: https://api.upsun.com
generated: '2026-08-26'
method: generated
source: openapi/platform.sh-rest-api-openapi.json
operations:
  - get-current-user
  - list-api-tokens
  - create-api-token
  - delete-api-token
---

# Authenticate with an API token

Every call to this API needs an OAuth 2.0 bearer access token. For automation, the token you hold
is an **API token**, which is not itself a bearer token — it must be exchanged.

## Exchange

```
POST https://auth.upsun.com/oauth2/token
Authorization: Basic <base64 of "platform-api-user:">
Content-Type: application/x-www-form-urlencoded

grant_type=api_token&api_token=<YOUR_API_TOKEN>
```

The response is `{"access_token": "...", "expires_in": 900, "token_type": "bearer"}`. Send it as
`Authorization: Bearer <access_token>` on every subsequent request and refresh before 900 seconds.

The authorization server for the REST API is `https://auth.api.platform.sh` — its discovery
documents (`/.well-known/openid-configuration`, `/.well-known/oauth-authorization-server`,
`/.well-known/oauth-protected-resource`) are saved in `well-known/` in this repository. Interactive
clients use `authorization_code` with PKCE `S256`; the `client_credentials` flow carries a single
`admin` scope.

## Steps

1. **Confirm who you are.** `get-current-user` (`GET /users/me`). Note that `GET /me` still exists
   but is marked **deprecated** in the contract — use `/users/me`.
2. **List tokens.** `list-api-tokens` (`GET /users/{user_id}/api-tokens`).
3. **Create a token.** `create-api-token` (`POST /users/{user_id}/api-tokens`). The secret is
   returned once and cannot be retrieved again.
4. **Revoke.** `delete-api-token` (`DELETE /users/{user_id}/api-tokens/{token_id}`).

## Rules

- **Never echo a token.** Do not print, log, or write an API token or an access token into any file,
  artifact, commit or message. Rotate immediately if one is exposed.
- Create a **machine user** per automated task rather than reusing a human account, and grant it the
  narrowest environment-type roles that let the task succeed.
- Machine users under an SSO domain can be excluded from SSO enforcement so automation does not
  break on an identity-provider redirect.
- A `403` on a project operation almost always means a missing environment-type role, not a bad
  token; `get-current-user` succeeding proves the token itself is valid.
