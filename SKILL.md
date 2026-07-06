---
name: siyuan-notes
description: Use SiYuan as the user's default personal note system through the installed CLI and the local kernel HTTP API. Search, recall, cross-reference, summarize, record, update, and organize notes; query the cloud Inbox/收集箱; work with notebooks, documents, blocks, attributes, exports, templates, files, SQL, and structured databases/attribute views. Use whenever the user mentions “我的笔记”, “笔记里”, “查下笔记”, “之前记过吗”, “记录到笔记”, “记到笔记”, “整理进笔记”, “收集箱”, Inbox, “知识库”, SiYuan/思源, SiYuan API, kernel API, database/属性视图, or explicitly invokes $siyuan-notes. Before every workflow, validate the installed CLI and live kernel because commands and API schemas may change. Do not use for ordinary repository documentation unless the user asks to involve their personal knowledge base.
---

# SiYuan Notes

Use the installed SiYuan CLI as the default surface for ordinary note work and the local kernel HTTP API for documented capabilities that the CLI does not expose cleanly.

Treat generic references to the user's notes or personal knowledge base as SiYuan requests. Keep repository documentation, source files, and generated deliverables separate unless the user explicitly asks to copy their content into personal notes.

## Select the operation

- For conceptual questions, organization design, or workflows spanning several SiYuan features, read [references/concepts-and-operations.md](references/concepts-and-operations.md) before choosing between documents, block references, tags, attributes, embedded queries, databases, templates, Inbox, history, sync, or backup.
- For lookup, comparison, or recall requests, perform a read-only search and return the relevant note paths plus a concise synthesis.
- For Inbox or 收集箱 requests, use the dedicated kernel Inbox API. Never infer Inbox contents from a local document named `Inbox`, `daily note`, `收件箱`, or `收集箱`.
- For documented kernel API requests, read [references/kernel-api.md](references/kernel-api.md), prefer read endpoints first, and use the CLI instead when it provides the same operation with safer validation or dry-run support.
- For database or attribute-view requests, also read [references/database-api.md](references/database-api.md) and keep `avID`, `blockID`, `viewID`, `itemID`, and `keyID` distinct.
- For record, organize, or update requests, search first, read candidate documents, append to the best existing note, and verify the write.
- Create a new document only when no existing note is suitable and the user asked to record the content.
- Rename, move, delete, or overwrite only when the user explicitly requests that exact mutation.

## Follow the workflow

1. Detect and validate the live CLI before any note operation.
   - Run `command -v siyuan` and use the returned absolute executable path.
   - Run `<binary> --version` on every triggered workflow, including read-only searches.
   - Run `<binary> --help`, then `<binary> <command> --help` and any required nested-command help before constructing commands.
   - Treat bundled command examples as historical guidance only. The installed version and its live help are authoritative.
   - If the version differs from the last verified version in `references/cli.md`, do not reuse old syntax without checking help.
   - If live help no longer exposes the required operation or flags, stop and explain the compatibility gap instead of guessing.
   - Do not install or upgrade SiYuan and do not modify shell or global package configuration.
2. Locate the workspace.
   - Use the live workspace-list help to construct the command when the workspace is not already established.
   - Treat paths and notebook IDs in `references/cli.md` as defaults that require verification when they fail or conflict with current context.
3. Route documented HTTP API requests through the running local kernel.
   - Read [references/kernel-api.md](references/kernel-api.md) before constructing a kernel request.
   - Confirm the existing listener and `/api/system/version`; do not start a second kernel when the desktop app is already listening.
   - Compare the live kernel version with the reference snapshot and re-check the official API documentation when versions differ or a response shape conflicts.
   - Start with the least-mutating endpoint that can answer the request. Inspect the current target state before any `set`, `update`, `move`, `remove`, or `delete` call.
   - Check both the HTTP result and top-level SiYuan `code`; stop on non-zero `code` instead of interpreting partial `data` as success.
4. Route Inbox requests through the running local kernel.
   - Read [references/inbox.md](references/inbox.md) before calling an Inbox endpoint.
   - Treat SiYuan Inbox as cloud shorthands proxied by the local kernel, separate from workspace documents and the local SQL index.
   - Fetch all list pages read-only, then fetch individual details only when classification or extraction requires them.
   - Keep deletion as a separate, explicitly authorized action. Never treat a successful import or summary as authorization to remove Inbox entries.
5. Handle database operations as a read-inspect-write-verify transaction.
   - Read [references/database-api.md](references/database-api.md) before using an `/api/av/*` endpoint.
   - Render or get the database first and capture IDs from the response. Never guess a row/item ID from a cell value or use a database block ID where an item ID is required.
   - Snapshot filters, sorts, grouping, fields, and the target row before mutation because several setters replace entire arrays or definitions.
   - Re-render or re-read the exact view after a write and compare the normalized returned value or structure.
6. Search before writing.
   - Derive focused keywords from the content, including product names, technologies, and Chinese synonyms.
   - Prefer `NodeDocument` results and compare their `hPath`, title, and recent content.
   - Avoid broad English-only queries such as `Resources` when they produce unrelated notes; combine terms or use regex.
7. Read candidate documents.
   - Use `block batch-kramdown` for a small candidate set.
   - Check the complete existing content for duplication, terminology, structure, factual boundaries, and internal links.
8. Choose the destination.
   - Prefer the most specific existing note over a generic parent note.
   - Split content across two or more existing notes only when each section has a clearly different long-term retrieval purpose.
   - If multiple candidates remain equally plausible and the choice would materially affect organization, ask one concise question.
9. Compose the addition.
   - Preserve existing text and append a well-structured Markdown section.
   - Deduplicate content already present.
   - Separate verified facts, user experience, inference, and advice.
   - Preserve useful source links from the conversation.
   - Use SiYuan block references for related existing notes when useful: `((block-id "Anchor"))`.
10. Validate before mutation.
   - Follow the current workspace's `AGENTS.md` and environment constraints for temporary files. Prefer its designated temporary directory; otherwise use a temporary path inside the current writable workspace.
   - Use the live help to find dry-run support and run a dry-run first when the current CLI provides it.
   - Inspect the dry-run target and do not proceed when it points to an unexpected block.
11. Write through the safest supported surface.
   - Prefer CLI `block append` for existing notes and other CLI commands when equivalent functionality exists.
   - Use an official HTTP mutation only when it is required for the requested capability and the target has been read and validated first.
   - Never edit `.sy` files directly.
   - Request the environment's required approval when the workspace or SiYuan runtime metadata is outside the current writable project.
12. Verify and clean up.
   - Search for one or more unique new headings or phrases.
   - Confirm the expected root document, path, block type, and Markdown.
   - Remove only the temporary files created for the operation.
   - Report the note paths changed, whether content was appended or created, and how verification succeeded.

## Enforce safety rules

- Never call `block delete`, `document remove`, `notebook remove`, or equivalent destructive commands without an explicit user request.
- Never call `/api/inbox/removeShorthands` without an explicit request naming Inbox deletion.
- Never call document, notebook, block, file, database-row, or database-field removal endpoints without an explicit request naming that destructive outcome.
- Never overwrite a complete document merely to add a section.
- Never use `/api/file/putFile` to edit `.sy` data files; write note content through semantic document or block APIs.
- Never use `/api/network/forwardProxy` to transmit note content, credentials, or private URLs unless the user explicitly requests that external transfer.
- Never use `/api/query/sql` for mutation; restrict Skill workflows to read-only `SELECT` or `WITH ... SELECT` statements.
- Never call `/api/sqlite/flushTransaction` as routine cleanup; let normal kernel/CLI lifecycle manage transactions unless the user explicitly asks for diagnostic control.
- Never expose access tokens, subscription URLs, activation codes, private workspace links, or other credential-like Inbox content in summaries; redact and classify them as sensitive.
- Never treat Office lock files, generated search highlights, or irrelevant keyword matches as source material.
- Do not upload SiYuan content or personal notes to external services.
- Do not install global dependencies. Use the executable returned by `command -v siyuan`; do not assume it permanently remains at `/usr/local/bin/siyuan`.
- When a write partially succeeds, stop further mutations, inspect the affected documents, and avoid duplicating the successful portion.

## Load the CLI reference

Read [references/concepts-and-operations.md](references/concepts-and-operations.md) for the durable mental model, feature-selection guidance, and data-safety distinctions learned from the SiYuan User Guide. Read [references/cli.md](references/cli.md) for the last verified CLI version, examples, workspace defaults, and routing anchors. Read [references/kernel-api.md](references/kernel-api.md) for the official HTTP contract, endpoint catalog, risk tiers, and verification patterns. Read [references/database-api.md](references/database-api.md) for structured database IDs, value shapes, and mutation rules. Read [references/inbox.md](references/inbox.md) for the undocumented cloud Inbox routes. Never let a bundled reference override the installed CLI, live help, live kernel response, or newer official documentation.
