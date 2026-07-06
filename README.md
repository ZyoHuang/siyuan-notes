# siyuan-notes skill

An agent skill for using [SiYuan](https://github.com/siyuan-note/siyuan) as a personal note system through its CLI and local kernel HTTP API. It covers search/recall, recording and organizing notes, the cloud Inbox (收集箱), notebooks/documents/blocks, attributes, exports, templates, SQL, and databases/attribute views.

## Install

Clone into your agent's skills directory, e.g.:

```bash
git clone https://github.com/<you>/siyuan-notes.git ~/.agents/skills/siyuan-notes
```

## Layout

- `SKILL.md` — entry point and workflow.
- `references/` — CLI, kernel API, database API, Inbox, and concepts references.
- `agents/openai.yaml` — agent metadata.

## Configuration

The skill resolves per-device values at runtime instead of hardcoding them:

- `SIYUAN_WORKSPACE` — path to your SiYuan workspace directory.
- `SIYUAN_BASE_URL` — kernel base URL (default `http://127.0.0.1:6806`).
- `SIYUAN_TOKEN` — kernel API token, only if access control is enabled.
