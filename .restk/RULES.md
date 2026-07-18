# Rules for Editing RESTk Workspace Files

## MUST Follow

1. **Never change any `id` field** — collection, folder, request, variable,
   header, param, runner, or doc page. An `id` is a Snowflake that links the
   file/row to the app's entity; changing it creates a DUPLICATE instead of
   updating the existing one.

2. **Keep IDs as quoted strings** — `id: "298356360100184083"`, not
   `id: 298356360100184083`. Unquoted large integers lose precision.

3. **Never edit `.restk-meta.json` or anything under `.restk/`** — meta tracks
   workspace ownership for cross-workspace import safety; `.restk/` is these docs.

4. **Emit valid YAML / Markdown.** An unparseable file is skipped on import — a
   silent no-op, not an error you will see.

5. **Auth is infer-from-key.** Put the credential under a single type key
   (`auth: { bearer: {…} }`). Do NOT reintroduce a `type:` discriminator or an
   `authConfigData` JSON string. Use `auth: none` to override to no-auth, or
   omit `auth:` entirely to inherit from the folder/collection.

## SHOULD Follow

6. **Use `{{variable}}` references** instead of hardcoding URLs, tokens, and
   credentials — this keeps secrets out of the files and enables env switching.

7. **Put secrets in `.env`**, never in YAML. Mark the variable/header
   `secret: true` and omit its `value`; the real value lives in git-ignored
   `.env` and is referenced by key name in `.env.sample`.

8. **Keep the file-naming convention** — `{METHOD}_{slug}.yaml` for requests
   (e.g. `POST_login.yaml`), `{name}.runner.yaml` under `_runners/`,
   `{slug}.md` under `_docs/`.

9. **To rename** a collection/folder, update its `name` in `collection.yaml` /
   `folder.yaml` — the app re-derives the directory slug on the next export.

10. **One entity per file** — one request per `.yaml`, one doc page per `.md`.

## Safe Operations

- ✅ Edit a request's `name`, `method`, `url`, `params`, `headers`, `body`,
  `auth`, `scripts`, `variables`
- ✅ Add or refine a request's `intent:` block (this is meant for you)
- ✅ Edit collection / folder `auth`, `scripts`, `variables`
- ✅ Edit environment variables (non-secret values)
- ✅ Edit runner presets and write `_docs/*.md` pages
- ✅ Add a new request file (follow the naming convention)
- ✅ Delete a request file (the entity is removed from the app)
- ✅ **Move** a request file into another folder's directory to re-parent it

## Unsafe Operations

- ❌ Change any `id` (creates duplicates)
- ❌ Reintroduce removed fields (`version`, `folderId`, `authConfigData`,
  `bodyType`, `prescript`/`postscript`, …) — see SCHEMA.md
- ❌ Edit `.restk-meta.json` or files under `.restk/`
- ❌ Hardcode secrets in YAML instead of `.env`
- ❌ Use unquoted large-integer IDs
- ❌ Write invalid YAML/Markdown
