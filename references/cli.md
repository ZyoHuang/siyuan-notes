# SiYuan CLI reference

## Version contract

- Last verified CLI version: `3.7.0`
- Verified on: `2026-07-02`
- Version command at verification time: `siyuan --version`

Always run the following before using any command in this file:

```bash
command -v siyuan
siyuan --version
siyuan --help
siyuan <command> --help
siyuan <command> <subcommand> --help
```

The executable returned by `command -v siyuan` is authoritative; `siyuan` above is shorthand. These examples describe version 3.7.0 and may become stale. If the installed version differs, reconstruct commands from live help. Never guess renamed commands or flags.

## Local defaults

Resolve these per device instead of hardcoding them:

- Binary: whatever `command -v siyuan` returns (commonly `/usr/local/bin/siyuan`).
- Workspace: set `SIYUAN_WORKSPACE` to the active workspace directory, or read it from `siyuan workspace list --format json`.
- Primary notebook and its ID: read from `siyuan notebook list -w "$SIYUAN_WORKSPACE" --format json`.

Treat these as defaults, not permanent facts. Verify them when a command fails or the user mentions another workspace. The examples below use `"$SIYUAN_WORKSPACE"` as the workspace placeholder.

## Version 3.7.0 read-command examples

The 3.7.0 CLI has no `inbox` subcommand. Inbox is a cloud-shorthand feature exposed through the running local kernel HTTP API, not through local document search or SQL. Read [inbox.md](inbox.md) before handling `Inbox` or `收集箱` requests.

For official kernel capabilities that are absent from the CLI or structurally richer over HTTP, read [kernel-api.md](kernel-api.md). For `/api/av/*` database operations, also read [database-api.md](database-api.md). Prefer the CLI when both surfaces support the same operation because live help and dry-run make target validation safer.

```bash
siyuan workspace list --format json
siyuan notebook list -w "$SIYUAN_WORKSPACE" --format json
siyuan search 'keyword' --method 0 --page-size 100 -w "$SIYUAN_WORKSPACE" --format json
siyuan search 'term1|term2' --method 3 --page-size 100 -w "$SIYUAN_WORKSPACE" --format json
siyuan block batch-kramdown --ids id1,id2 -w "$SIYUAN_WORKSPACE" --format json
siyuan block kramdown --id block-id -w "$SIYUAN_WORKSPACE" --format json
```

Search methods used most often:

- `--method 0`: keyword search.
- `--method 3`: regex search for several focused alternatives.
- Add `--group-by 1` when document-grouped results are easier to inspect.

## Version 3.7.0 safe-write examples

Append an existing document or block:

```bash
siyuan block append --parent block-id --file tmp/addition.md --dry-run -w "$SIYUAN_WORKSPACE" --format json
siyuan block append --parent block-id --file tmp/addition.md -w "$SIYUAN_WORKSPACE" --format json
```

Create a document only when needed:

```bash
siyuan document create --notebook notebook-id --title 'Title' --path /parent/path --markdown 'Initial content' --dry-run -w "$SIYUAN_WORKSPACE" --format json
siyuan document create --notebook notebook-id --title 'Title' --path /parent/path --markdown 'Initial content' -w "$SIYUAN_WORKSPACE" --format json
```

## Routing anchors

Do not hardcode note titles, paths, or block IDs. Always search first, read the candidate documents, then use the resolved `rootID`/`hPath` for that device's workspace.

## Verification pattern

After writing, search for a unique heading or phrase and confirm:

- `rootID` matches the intended note;
- `hPath` matches the intended path;
- the new block has the expected heading/paragraph type;
- `markdown` preserves code spans, links, tables, and internal block references.

The CLI may write its own log and workspace registration files under `~/.config/siyuan`. This is tool runtime metadata, not note content, and may require sandbox approval even for read operations.
