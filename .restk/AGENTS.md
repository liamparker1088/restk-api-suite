# Working with a RESTk Workspace (for AI agents)

You are editing a **RESTk API workspace** — a git-synced folder of API
collections, folders, requests, environments, runner presets, and documentation.

**The files ARE the source of truth.** RESTk watches this folder: when you
create, edit, move, or delete a file, the app imports the change into its
database automatically. You never call an API to make changes — you edit files.

## Your role

You may, by editing files directly:

- Add / edit / delete **requests** (one `.yaml` file per request)
- Add / edit / delete **folders** and **collections** (directories + their
  `folder.yaml` / `collection.yaml`)
- Edit **environments** (`environments/*.yaml`) — non-secret values only
- Edit **runner presets** (`_runners/*.runner.yaml`)
- Write **documentation** pages (`_docs/*.md`)
- Annotate a request with an `intent:` block — this metadata is written FOR
  you, to record when/how an endpoint should be used (see SCHEMA.md).

Read **SCHEMA.md** for the exact format, then **RULES.md** for the rules.

## At a glance: what you MAY vs MUST NOT modify

| ✅ You MAY modify | ❌ You MUST NOT touch |
|---|---|
| `name`, `method`, `url`, `description`, `order` | the `id` field (links file → app entity) |
| `params`, `headers`, `body`, `auth`, `scripts`, `variables` | `.restk-meta.json` (workspace ownership) |
| `intent` (endpoint annotation, for agents) | any file under `.restk/` (these docs) |
| doc pages (`_docs/*.md`) and runner presets (`_runners/*.runner.yaml`) | secret **values** — they are never written to these files |
| adding / deleting / **moving** request files (move = change folder) | the `id` of a variable / header / param row |

**Golden rules** (full list in RULES.md):

1. Never change any `id`. Changing it creates a duplicate instead of an update.
2. Keep IDs quoted: `id: "298356360100184083"` (unquoted large ints lose precision).
3. **Never write a secret value into these files.** Mark the variable/header
   `secret: true` and omit its `value` — the real value is held by the app
   and never exported. Reference it as `{{VAR}}`.
4. **Folder membership is the directory** — to move a request to another folder,
   move the FILE into that folder's directory. There is no parent-id field.
5. Emit valid YAML. An invalid file is skipped on import (silent no-op).
