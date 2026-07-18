# RESTk YAML Schema Reference

This workspace holds API collections, folders, requests, environments, runner
presets, and docs exported from the RESTk app. AI assistants and external tools
can read and modify these files — changes sync back to the app automatically.

Everything here reflects the **current (v2) format**. If you have seen older
RESTk exports, note what CHANGED (see "Removed / renamed fields" at the end):
the verbose flat fields were consolidated into compact nested blocks.

## Directory Structure

```
workspace-name/
├── .restk/                      # This docs directory (DO NOT EDIT)
│   ├── AGENTS.md                # Start here
│   ├── SCHEMA.md                # Format reference (this file)
│   └── RULES.md                 # Rules for editing
├── .restk-meta.json             # Workspace metadata (DO NOT EDIT)
├── .gitignore
├── .env                         # Secret values (git-ignored)
├── .env.sample                  # Secret key names (committed)
├── environments/
│   └── {env-name}.yaml
├── {collection-slug}/
│   ├── collection.yaml
│   ├── {METHOD}_{slug}.yaml      # a request at collection root
│   ├── {folder-slug}/
│   │   ├── folder.yaml
│   │   └── {METHOD}_{slug}.yaml  # a request inside the folder
│   ├── _runners/
│   │   └── {preset-name}.runner.yaml
│   └── _docs/
│       └── {slug}.md            # documentation pages (Markdown)
└── ...
```

A request's **folder is its directory location** — there is no `folderId`
field. Move the file to re-parent the request.

## Common conventions

- **id** — a Snowflake ID, always a **quoted string**. Never change it.
- **order** — display/sort order (a number). Lower comes first.
- **{{variable}}** — resolved at runtime: request → folder → collection →
  environment scope.
- **Rows** (variables, headers, params) are compact flow-mappings:
  `{id: "h1", key: X, value: Y, order: 10, enabled: false, secret: true}`
  — `enabled` defaults true (omitted when true); `secret: true` keeps the
  value out of the committed file (stored in `.env`), so `value` is omitted.
- There is **NO `version:` field** in v2.

## Request YAML Format

File: `{METHOD}_{slug}.yaml` (e.g. `POST_login.yaml`).

```yaml
# Restk Request
id: "410001"                       # Snowflake, quoted, DO NOT CHANGE
name: Create User
method: POST                       # GET POST PUT DELETE PATCH HEAD OPTIONS
url: '{{base_url}}/users'          # supports {{variable}}
order: 1
description: ""                     # optional

params:                            # query parameters
    - {id: "p1", key: verbose, value: "true", order: 10}
    - {id: "p2", key: trace, order: 20, enabled: false}

headers:
    - {id: "h1", key: Content-Type, value: application/json, order: 10}
    - {id: "h2", key: X-Api-Key, order: 20, secret: true}   # value in .env

body:                              # ONE key selects the body type (infer-from-key)
    json: |                        #   json | text | xml — raw content, literal block
        { "name": "Ada", "email": "ada@example.com" }
    # form:    [{key: f, value: v}]          # multipart form-data
    # urlencoded: [{key: f, value: v}]       # x-www-form-urlencoded
    # graphql: {query: |…, variables: |…}    # GraphQL
    # binary:  <reference>                   # binary upload
    # (omit the whole 'body:' block for no body)

auth:                              # ONE key selects the auth type (infer-from-key)
    bearer: {token: '{{token}}'}
    # basic:   {username: '{{user}}', password: '{{pass}}'}
    # apikey:  {key: X-API-Key, value: '{{key}}', in: header}   # in: header | query
    # oauth2:  {grant: client_credentials, clientId, clientSecret, tokenUrl, scope: [read, write]}
    # digest / awsSigV4 / oauth1 / hawk / jwt / ntlm: {…}
    #
    # auth: none      → EXPLICIT override to no auth (stops inheritance)
    # (omit 'auth:' entirely → INHERIT from folder, then collection)

scripts:                           # JavaScript hooks (Nova API)
    preRequest: |-
        nova.request.headers.set("X-Trace", nova.uuid())
    postResponse: |-
        if (nova.response.status === 201)
          nova.vars.set("user_id", nova.response.json().id)
    # WS/SSE requests may also use: onOpen | onMessage | onClose | onError
skipParentScripts: false           # true = ignore folder/collection scripts

variables:                         # request-scoped variables
    - {id: "rv1", key: page_size, value: "20", order: 10}

# ── AI-facing annotation (you MAY add/edit this) ─────────────────────
intent:
    whenToUse: "Call after login to create a new user"
    preconditions: [valid_token]
    commonNextSteps: [get_user, update_profile]
    doNotUseFor: password_reset

# ── non-HTTP protocols (omit 'protocol' for plain HTTP) ──────────────
protocol: websocket                # http (default) | websocket | sse | soap | graphql
socket: {subprotocol: json, pingInterval: 30000}   # or sse: {...} / soap: {...}
templates:                         # WS/SSE saved message templates
    - {id: "1", name: Subscribe, contentType: json, payload: '{"op":"subscribe"}', order: 10}
```

## Collection YAML Format

File: `{collection-slug}/collection.yaml`.

```yaml
# Restk Collection
id: "600002"
name: Acme API
order: 10
description: ""
isHidden: false
isSharedWithAI: true
createdAt: "2026-01-15T10:00:00Z"   # used for stable ordering
defaultRunner: Smoke Test           # name of the default runner preset (optional)
auth:
    bearer: {token: '{{token}}'}
scripts:
    preRequest: |-
        nova.request.headers.set("X-Trace", nova.uuid())
variables:
    - {id: "v1", key: base_url, value: 'https://api.example.com', order: 10}
    - {id: "v2", key: api_token, order: 20, secret: true}
```

## Folder YAML Format

File: `{collection-slug}/{folder-slug}/folder.yaml`. The folder's parent is its
directory location — there are no parent-id fields.

```yaml
# Restk Folder
id: "800002"
name: Admin
order: 20
auth: {bearer: {token: '{{token}}'}}
scripts: {preRequest: |-
    nova.log("in admin folder")}
variables: []
```

## Environment YAML Format

File: `environments/{env-name}.yaml`.

```yaml
# Restk Environment
id: "200002"
name: Staging
description: Pre-prod environment
variables:
    - {id: "21", key: base_url, value: 'https://staging.example.com', order: 10}
    - {id: "22", key: api_token, order: 20, secret: true}   # value lives in .env
```

## Runner Preset YAML Format

File: `{collection-slug}/_runners/{preset-name}.runner.yaml` (folder-scoped
presets live under the folder's own `_runners/`).

```yaml
# Restk Runner
id: "700900"
name: Smoke Test
description: "Quick validation of critical endpoints"
order: 10
mode: sequential          # sequential | parallel
delayMs: 250              # delay between requests (sequential)
parallelLimit: 5          # max concurrent (parallel)
default: false            # is this the collection's default preset
selected:                 # request ids included in the run
    - "300001"
    - "300002"
requests:                 # execution order (sequential)
    - "300001"
    - "300002"
```

## Documentation pages (Markdown)

File: `{collection-slug}/_docs/{slug}.md`. Doc pages are plain Markdown with a
small YAML front-matter. The **page title is the first `#` heading** (single
source of truth — do not duplicate it in the front-matter).

```markdown
---
id: "900100"
order: 10
---
# Getting Started

Welcome to the Acme API. Authenticate with a bearer token…
```

(An older `_docs/{slug}.restk.yaml` doc form is still READ for backward
compatibility and converts to `.md` on the next export — never write it.)

## Removed / renamed fields (if you have seen older exports)

| Old (flat) | Now (v2) |
|---|---|
| `version: 1` | removed |
| `sortOrder` | `order` |
| `parameters` | `params` |
| `folderId` / `parentCollectionId` / `parentFolderId` | removed — folder = directory location |
| `bodyType` + `bodyContentType` + `bodyData` | `body: { json|text|xml|form|urlencoded|graphql|binary: … }` |
| `authType` + `isAuthOverridden` + `authConfigData` (JSON string) | `auth: { <type>: {…} }` (or `auth: none`, or omit to inherit) |
| `prescript` + `postscript` | `scripts: { preRequest, postResponse, onOpen, onMessage, onClose, onError }` |
| `isEnabled` / `isSecret` (rows) | `enabled` / `secret` |
| `executionMode` / `delayBetweenRequests` / `isDefault` / `selectedRequestIds` / `requestOrder` (runner) | `mode` / `delayMs` / `default` / `selected` / `requests` |
| `protocolType` / `websocketConfig` (JSON string) | `protocol` + `socket` / `sse` / `soap` blocks |
| doc pages as `.restk.yaml` | doc pages as `_docs/{slug}.md` |
