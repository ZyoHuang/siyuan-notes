# SiYuan Inbox API reference

## Status and boundary

- Last verified kernel version: `3.7.0`
- Verified on: `2026-07-02`
- The CLI has no `inbox` subcommand.
- Inbox stores cloud shorthands. The local kernel proxies requests to the SiYuan cloud service with the currently logged-in SiYuan account.
- Inbox entries are not local documents and are not discoverable through workspace SQL, document lists, or a `daily note` path.
- These routes exist in the kernel source but are not documented in the public `API.md`. Treat them as internal and re-check live behavior after upgrades.

## Preconditions

1. Follow the CLI validation workflow in `SKILL.md` and `references/cli.md`.
2. Detect the existing kernel listener. The desktop default is commonly `127.0.0.1:6806`, but do not assume the port when evidence shows another value.
3. Probe the kernel read-only:

```bash
curl --silent --show-error --max-time 10 \
  -X POST http://127.0.0.1:6806/api/system/version \
  -H 'Content-Type: application/json' \
  -d '{}'
```

4. If local access is blocked by the execution sandbox, request approval for read-only localhost access. Do not start a second kernel while SiYuan Desktop is already listening.
5. If kernel access authentication is enabled, add `Authorization: Token ${SIYUAN_TOKEN}`. Never print, persist, or include the token in command output or summaries.
6. Expect cloud requests to require a logged-in SiYuan account and working network access.

## Read endpoints

List one page:

```bash
curl --silent --show-error --max-time 30 \
  -X POST http://127.0.0.1:6806/api/inbox/getShorthands \
  -H 'Content-Type: application/json' \
  -d '{"page":1}'
```

Request body:

```json
{"page": 1}
```

The observed list response is nested because the local kernel returns the cloud result:

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "code": 0,
    "msg": "",
    "data": {
      "pagination": {
        "paginationPageCount": 2,
        "paginationPageNums": [1, 2],
        "paginationRecordCount": 24
      },
      "shorthands": []
    }
  }
}
```

Read `paginationPageCount` from page 1 and fetch every remaining page. Keep output compact because `shorthandContent` can contain a complete clipped article and large rendered HTML:

```bash
curl --silent --show-error --max-time 30 \
  -X POST http://127.0.0.1:6806/api/inbox/getShorthands \
  -H 'Content-Type: application/json' \
  -d '{"page":1}' |
  jq '.data.data | {
    pagination,
    items: [.shorthands[] | {
      oId,
      hCreated,
      shorthandTitle,
      shorthandURL,
      shorthandDesc,
      shorthandMdPreview: ((.shorthandMd // "")
        | gsub("[\\r\\n\\t]+"; " ")
        | .[0:500])
    }]
  }'
```

Read one entry by its `oId`:

```bash
curl --silent --show-error --max-time 30 \
  -X POST http://127.0.0.1:6806/api/inbox/getShorthand \
  -H 'Content-Type: application/json' \
  -d '{"id":"ENTRY_ID"}'
```

Request body:

```json
{"id": "ENTRY_ID"}
```

The detail object is returned directly in top-level `data`. Common fields are:

- `oId`: Inbox entry ID and creation-time source.
- `hCreated`: formatted creation time.
- `shorthandTitle`: captured title or date.
- `shorthandURL`: source URL, possibly empty.
- `shorthandDesc`: short description.
- `shorthandMd`: Markdown source; prefer this for analysis and import.
- `shorthandContent`: rendered HTML; use only when Markdown is insufficient.
- `shorthandFrom`: capture-source code.

Bare-link entries can have empty Markdown. Image-only entries require visual inspection before classification.

## Organization workflow

1. Fetch all pages and inventory metadata without dumping full article bodies.
2. Redact credential-like values such as subscription URLs, license keys, tokens, and private internal links.
3. Classify entries into merge-existing-note, create-new-note, link-only reference, sensitive migration, manual review, and deletion candidate.
4. Search local notes through the CLI and read candidate destinations before proposing any write.
5. Present a per-entry routing plan when destination choice is material.
6. Append or create notes only after the user requests the write operation; verify each write through the CLI.
7. Do not remove imported Inbox entries unless the user separately and explicitly requests deletion.

## Destructive endpoint

`POST /api/inbox/removeShorthands` accepts `{"ids":["ENTRY_ID"]}`. It permanently removes cloud Inbox entries. Do not call it for lookup, organization planning, deduplication, or as an automatic post-import cleanup step.
