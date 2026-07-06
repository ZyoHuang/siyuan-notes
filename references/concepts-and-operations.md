# SiYuan concepts and operating principles

## Source and authority

This working model was distilled on `2026-07-06` from the local **SiYuan User Guide** notebook in SiYuan `3.7.1`. Its 75-document outline was inventoried and representative chapters were read across content blocks, editing, search, databases, templates, CLI/API, plugins, AI, history, sync, backup, and security.

Treat this file as durable feature-selection guidance, not an immutable command or API specification. Installed CLI help, live kernel responses, and newer official documentation remain authoritative. Re-check them because the guide and CLI can drift.

## Mental model

- Think **block-first**, not page-first. Every content block has a globally unique ID; a document is a container block rather than a separate primitive.
- Keep canonical knowledge in one block and reuse it through block references. Forward links show dependencies; backlinks and mentions expose reuse and emerging context.
- Treat hierarchy as only one organization layer. SiYuan combines notebook/document hierarchy, references, tags, bookmarks, attributes, embedded queries, and structured databases.
- Treat a workspace as an isolation and lifecycle boundary. It contains data, configuration, history, snapshots, plugins, templates, and runtime files, and only one kernel process may serve it at a time.
- Assume local note data is stored in plaintext. Local confidentiality therefore depends on the operating system and device security.

## Choose the right organization mechanism

| Need | Prefer | Reason |
|---|---|---|
| Stable navigation and ownership | Notebook and document hierarchy | Gives a clear human-readable home |
| Reuse one fact in many contexts | Block reference | Preserves one canonical source and creates backlinks |
| Show a dynamic collection of existing blocks | Embedded query | Renders query results without copying content |
| Lightweight classification or favorites | Hierarchical tags and bookmarks | Fast manual retrieval without a schema |
| Machine-readable metadata | Built-in or `custom-*` block attributes | Supports deterministic queries and automation |
| Records with typed fields and multiple views | Database/attribute view | Supports table, board, card, filtering, sorting, grouping, relations, and rollups |
| Repeated capture structure | Templates and daily notes | Standardizes recurring notes while preserving normal blocks |
| Temporary external capture | Cloud Inbox | Keeps unprocessed material separate from local documents |
| Long-term memorization | Flashcards | Uses block structure with FSRS scheduling |

For databases, remember that multiple views can share the same underlying data while filters, sorts, grouping, and layout remain view-specific. Keep `avID`, `blockID`, `viewID`, `itemID`, and `keyID` distinct during automation.

## Operating principles

1. Search and read before writing. Resolve the exact notebook, document, block, or database item instead of relying on a remembered title or ID.
2. Prefer the most specific existing document. Append structured blocks and preserve existing content; create a new document only when no suitable destination exists.
3. Use block references when information should remain canonical. Copy text only when the new copy should intentionally diverge.
4. Use tags for flexible classification, attributes for automation, embedded queries for dynamic aggregation, and databases for typed record management. Do not force every problem into a database.
5. Prefer semantic surfaces in this order: live CLI with validation/dry-run, documented local kernel API, then an internal route only when the required capability is absent elsewhere.
6. Never edit `.sy` files directly. Use document, block, database, import/export, history, or snapshot operations.
7. Inspect live command help before constructing a command. A guide example is historical evidence, not proof of current syntax.
8. Perform one bounded mutation at a time and verify by re-reading the exact target. Stop after partial or ambiguous success to prevent duplication.
9. Keep cloud Inbox entries separate from local notes until the user requests organization or conversion. Never delete an Inbox source as an implicit cleanup step.
10. Distinguish plugins from widgets: plugins extend application behavior; widgets extend content blocks. Use CLI or kernel API for Agent automation unless a persistent in-app interaction requires a plugin.
11. Treat built-in or external AI use as data transfer. Send note content to an external model only when the user authorizes that scope, and preserve the provider's cost and privacy boundary.

## Data-safety model

| Mechanism | Purpose | Important boundary |
|---|---|---|
| File history | Recover recent updates, deletes, sync conflicts, formatting, or replacements | Human-readable files under workspace history; retention is limited |
| Data snapshot | Capture a repository state for rollback and comparison | Encrypted/compressed repository data; rollback can replace current state |
| Sync | Keep `workspace/data/` consistent across devices | Not a backup; avoid simultaneous edits and never combine with third-party sync drives |
| Backup | Preserve selected snapshots for disaster recovery | Separate from synchronization and should include off-device copies |

- Before bulk formatting, replacement, import, conversion, move, or database restructuring, inspect history and create a meaningful snapshot when risk warrants it.
- Prefer alternating multi-device synchronization. Ensure participating devices use compatible time and the same repository key.
- Do not place an active SiYuan workspace in a third-party synchronization directory.
- Follow the 3-2-1 principle for important data: at least three copies, on two media types, with one off-site copy.
- Never store critical passwords, recovery secrets, or high-value keys as ordinary plaintext notes without a separate security decision.

## Interaction defaults

- Interpret “笔记里”, “我的笔记”, and “知识库” as local SiYuan note operations.
- Interpret “收集箱” or `Inbox` as the separate cloud shorthand store, not as a local document title.
- For explanations, return relevant human-readable note paths and a concise synthesis.
- For writes, report whether content was appended or created, the resulting note path, and the verification result.
- Require explicit authorization for delete, overwrite, rollback, source removal, sync-direction forcing, and external transmission.
