# SiYuan official kernel API reference

## Contents

- [Source and scope](#source-and-scope)
- [Protocol](#protocol)
- [Selection workflow](#selection-workflow)
- [Identifiers](#identifiers)
- [Risk tiers](#risk-tiers)
- [Endpoint catalog](#endpoint-catalog)
- [Verification patterns](#verification-patterns)

## Source and scope

- Official source: [siyuan-note/siyuan API.md](https://github.com/siyuan-note/siyuan/blob/master/API.md)
- Source reviewed: `2026-07-02`
- Snapshot SHA-256: `6413441a8357abe1ac8e1ac0fdd32a11a6a2255e90e78b12d8672d7d8efc4a99`
- Documented endpoints: `67`
- Live kernel verified during review: `3.7.0`

The official document tracks `master`, so treat this file as routing guidance rather than an immutable schema. Re-open the official source when the live kernel version differs, an endpoint returns a different shape, or a field is deprecated. Inbox routes are not in the public API document; use [inbox.md](inbox.md) separately.

## Protocol

- Default base URL: `http://127.0.0.1:6806`.
- Every documented route uses `POST`.
- Most requests use JSON with `Content-Type: application/json`.
- `/api/asset/upload` and `/api/file/putFile` use multipart form data.
- `/api/file/getFile` returns raw file bytes on HTTP `200`; errors use HTTP `202` with a JSON body.
- Normal JSON envelope: `{"code":0,"msg":"","data":...}`. Treat any non-zero `code` as failure.
- Authentication header when configured: `Authorization: Token ${SIYUAN_TOKEN}`. Keep the token in process environment or an approved secret source; never echo, log, or save it in the Skill or workspace.

Read-only probe:

```bash
curl --silent --show-error --max-time 10 \
  -X POST http://127.0.0.1:6806/api/system/version \
  -H 'Content-Type: application/json' \
  -d '{}'
```

Authenticated JSON pattern:

```bash
curl --silent --show-error --max-time 30 \
  -X POST "${SIYUAN_BASE_URL:-http://127.0.0.1:6806}/api/block/getBlockKramdown" \
  -H "Authorization: Token ${SIYUAN_TOKEN}" \
  -H 'Content-Type: application/json' \
  -d '{"id":"BLOCK_ID"}'
```

Only add the authorization header when access control requires it and the token is already available through an approved source.

## Selection workflow

1. Run the CLI detection, version, root help, and relevant command help required by `SKILL.md`.
2. Detect the existing kernel listener and probe `/api/system/version`. Do not start a second kernel beside SiYuan Desktop.
3. Prefer the CLI when it exposes the same operation, especially for ordinary note writes where dry-run is available.
4. Use this official HTTP API for capabilities the CLI lacks or models less directly, such as attribute views, raw exports, templates, and structured path conversion.
5. Read the target with the nearest read endpoint before any mutation.
6. Validate every notebook ID, document path, block ID, and database identifier against the live response.
7. Execute one bounded mutation at a time, inspect `code`, then verify by re-reading the target.

## Identifiers

- `notebook` or box ID: identifies a notebook.
- document internal `path`: storage path such as `/ID.sy`; it is not the readable title path.
- `hPath`: human-readable document path such as `/Unity/AssetBundle`.
- block/document `id`: SiYuan node ID.
- asset/workspace file `path`: rooted inside the workspace, commonly under `/data`, `/assets`, `/temp`, or `/conf`.
- database `avID`, embedding `blockID`, view `viewID`, row `itemID`, and field `keyID`: distinct identifiers; see [database-api.md](database-api.md).

Prefer ID-based document endpoints after resolving the target. Use path-conversion endpoints when only an `hPath` or storage path is available.

## Risk tiers

- `R` read-only: safe for lookup after authentication and target validation.
- `S` stateful/non-content side effect: changes runtime/UI state or creates temporary output.
- `W` write: creates or changes durable workspace data.
- `D` destructive: deletes, unbinds, or discards durable data; require explicit user authorization for that exact outcome.
- `H` high-impact external or executable action: external network forwarding or Pandoc execution; require explicit scope and inspect inputs.

HTTP APIs do not expose the CLI's general dry-run. For `W`, `D`, and `H`, read the current target first and verify afterward. Never use file APIs to edit `.sy` files directly.

## Endpoint catalog

### Notebooks

| Endpoint | Tier | Purpose and required input |
|---|---:|---|
| `/api/notebook/lsNotebooks` | R | List notebooks; `{}`. |
| `/api/notebook/getNotebookConf` | R | Read configuration; `notebook`. |
| `/api/notebook/openNotebook` | S | Open runtime notebook; `notebook`. |
| `/api/notebook/closeNotebook` | S | Close runtime notebook; `notebook`. |
| `/api/notebook/createNotebook` | W | Create by `name`; returns notebook object. |
| `/api/notebook/renameNotebook` | W | Rename; `notebook`, `name`. |
| `/api/notebook/setNotebookConf` | W | Replace configuration; read current config and merge fields before sending `notebook`, `conf`. |
| `/api/notebook/removeNotebook` | D | Permanently remove `notebook`. |

### Documents and paths

| Endpoint | Tier | Purpose and required input |
|---|---:|---|
| `/api/filetree/createDocWithMd` | W | Create document from GFM; `notebook`, readable `path`, `markdown`. Repeating a path does not overwrite the document. |
| `/api/filetree/renameDocByID` | W | Rename by `id`, `title`; prefer after target resolution. |
| `/api/filetree/renameDoc` | W | Rename by internal `notebook`, `path`, `title`. |
| `/api/filetree/moveDocsByID` | W | Move `fromIDs` to parent document/notebook `toID`. |
| `/api/filetree/moveDocs` | W | Move internal `fromPaths` to `toNotebook`, `toPath`. |
| `/api/filetree/removeDocByID` | D | Remove document `id`. |
| `/api/filetree/removeDoc` | D | Remove by internal `notebook`, `path`. |
| `/api/filetree/getHPathByID` | R | Resolve block/document `id` to readable path. |
| `/api/filetree/getHPathByPath` | R | Resolve internal `notebook`, `path` to readable path. |
| `/api/filetree/getPathByID` | R | Resolve `id` to `notebook` and internal path. |
| `/api/filetree/getIDsByHPath` | R | Resolve readable `path` plus `notebook` to matching IDs. |

### Assets and blocks

| Endpoint | Tier | Purpose and required input |
|---|---:|---|
| `/api/asset/upload` | W | Multipart upload with `assetsDirPath` and `file[]`; default to `/assets/`. |
| `/api/block/getBlockKramdown` | R | Read `id` as Kramdown. |
| `/api/block/getChildBlocks` | R | List direct children of `id`; heading content is included as children. |
| `/api/block/insertBlock` | W | Insert Markdown/DOM using at least one anchor; priority `nextID` > `previousID` > `parentID`. |
| `/api/block/prependBlock` | W | Prepend `dataType`, `data` to `parentID`. |
| `/api/block/appendBlock` | W | Append `dataType`, `data` to `parentID`. |
| `/api/block/updateBlock` | W | Replace target block `id` with `dataType`, `data`; read it first. |
| `/api/block/moveBlock` | W | Move `id` using `previousID` or `parentID`; `previousID` wins when both exist. |
| `/api/block/foldBlock` | S | Fold block `id`. |
| `/api/block/unfoldBlock` | S | Unfold block `id`. |
| `/api/block/transferBlockRef` | W | Transfer refs from `fromID` to `toID`; omitting `refIDs` transfers all refs. |
| `/api/block/deleteBlock` | D | Delete block `id`. |

### Attributes, SQL, and templates

| Endpoint | Tier | Purpose and required input |
|---|---:|---|
| `/api/attr/getBlockAttrs` | R | Read attributes for block `id`. |
| `/api/attr/setBlockAttrs` | W | Set `attrs` on `id`; custom keys require `custom-` prefix. |
| `/api/query/sql` | R | Execute read query `stmt`; restrict Skill use to `SELECT` or `WITH ... SELECT`. Not available in Publish Mode. |
| `/api/sqlite/flushTransaction` | S | Force transaction flush; avoid routine use. |
| `/api/template/render` | R | Render template file `path` in document context `id`; returns rendered DOM. |
| `/api/template/renderSprig` | R | Render Sprig `template`; useful for daily-note path preview. |

### Workspace files

| Endpoint | Tier | Purpose and required input |
|---|---:|---|
| `/api/file/getFile` | R | Read workspace `path`; raw HTTP `200`, JSON error on `202`. |
| `/api/file/readDir` | R | List directory `path`. |
| `/api/file/putFile` | W | Multipart create/write using `path`, `isDir`, `modTime`, `file`; never target `.sy`. |
| `/api/file/renameFile` | W | Rename workspace `path` to `newPath`; never use for note documents. |
| `/api/file/removeFile` | D | Remove workspace `path`; never use as a semantic note delete. |

### Export, conversion, notifications, and network

| Endpoint | Tier | Purpose and required input |
|---|---:|---|
| `/api/export/exportMdContent` | R | Export document `id`; returns readable `hPath` and Markdown `content`. |
| `/api/export/exportResources` | S | Create temporary zip from workspace `paths`; optional `name`. Same names inside the archive overwrite. |
| `/api/convert/pandoc` | H | Run Pandoc `args` inside `/temp/convert/pandoc/{dir}`. Validate filenames and arguments; do not pass shell syntax. |
| `/api/notification/pushMsg` | S | Show local message `msg`; optional `timeout`. |
| `/api/notification/pushErrMsg` | S | Show local error message `msg`; optional `timeout`. |
| `/api/network/forwardProxy` | H | Send an external request. Require explicit destination and payload authorization; never forward private note content implicitly. |

### System

| Endpoint | Tier | Purpose and required input |
|---|---:|---|
| `/api/system/bootProgress` | R | Get boot progress; `{}`. |
| `/api/system/version` | R | Get live kernel version; `{}`. |
| `/api/system/currentTime` | R | Get kernel time in milliseconds; `{}`. |

### Databases

The remaining 16 official `/api/av/*` routes are cataloged in [database-api.md](database-api.md). Read that file before any database request.

## Verification patterns

- Notebook config: call `getNotebookConf` and compare every preserved field.
- Document create/move/rename: resolve `getHPathByID`, then read root Kramdown.
- Block insert/prepend/append/update: capture operation IDs, then call `getBlockKramdown` or `getChildBlocks`.
- Attributes: call `getBlockAttrs` and compare only intended keys.
- SQL: enforce a bounded query and inspect row count; do not infer mutation from returned data.
- File write/rename: use `readDir` or `getFile`; do not inspect `.sy` through raw-file workflows.
- Export/Pandoc: verify the returned path stays under `/temp` and read only the expected output.
- Database: re-render the exact view and compare row, field, filter, sort, or layout state; see [database-api.md](database-api.md).
