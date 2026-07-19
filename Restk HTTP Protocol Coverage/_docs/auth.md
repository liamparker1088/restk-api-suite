---
id: "337171814958415872"
kind: auth
order: 3
createdAt: 1784455225
---
# Authentication

This collection exercises three authentication schemes, all in the **10. Auth (headline)** folder. Auth is set per request rather than inherited from the collection, so each request is self-contained and can be run in isolation.

## Bearer token

`GET /auth/bearer` with an `Authorization: Bearer <token>` header.

Configure the token on the request's Auth tab, or store it in an environment variable and reference it — the latter keeps the literal out of the request and out of any exported file.

The companion request **Bearer — missing** sends no Authorization header at all and expects **401 Unauthorized**. Leave its auth set to `none`: the 401 is what makes it pass.

## Basic authentication

`GET /auth/basic` with an `Authorization: Basic <base64>` header.

Set a username and password on the Auth tab; the client handles base64 encoding. To confirm the negative path, change the password to something wrong and resend — the server should return 401.

## API key

`GET /auth/apikey` with the key supplied as a request header.

Both the key value and the header name must match what the server expects. The key can also be placed in the query string, but this request specifically verifies the header path — switching the location to query is a useful way to confirm the server rejects it.

## Keeping credentials out of your repo

If this workspace is synced to git, prefer environment variables over literals typed directly into the Auth tab, and mark sensitive values as secret. Marked secrets are withheld from the exported files, so the credential stays local while the request definition remains shareable.
