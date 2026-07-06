# siyuan-notes skill

An agent skill for using [SiYuan](https://github.com/siyuan-note/siyuan) as a personal note system through its CLI and local kernel HTTP API. It covers search/recall, recording and organizing notes, the cloud Inbox (收集箱), notebooks/documents/blocks, attributes, exports, templates, SQL, and databases/attribute views.

## 安装 Installation

### 方式一：一行命令（推荐，跨运行时）

直接告诉你的 AI agent：

```
帮我安装这个 skill：https://github.com/ZyoHuang/siyuan-notes
```

或使用通用 CLI 安装器（自动识别当前运行时）：

```bash
npx skills add ZyoHuang/siyuan-notes
```

可用 `-a` 指定运行时，例如 `-a claude-code`、`-a codex`、`-a cursor`。

### 方式二：手动安装（git clone）

克隆到对应运行时的 skills 目录：

| 运行时 Runtime | 安装路径 Install Path |
|---|---|
| Claude Code | `~/.claude/skills/siyuan-notes/` |
| Codex CLI | `~/.codex/skills/siyuan-notes/` |
| Cursor | `~/.cursor/skills/siyuan-notes/` |
| 其他 Other | 克隆到对应运行时的 `skills/` 目录 |

```bash
git clone https://github.com/ZyoHuang/siyuan-notes <对应路径>
```

例如 Codex CLI：

```bash
git clone https://github.com/ZyoHuang/siyuan-notes ~/.codex/skills/siyuan-notes
```

### 方式三：直接引用

如果你的运行时不支持自动加载 skill，把 `SKILL.md` 的内容直接粘贴进对话即可 —— 它本质就是一份 markdown + YAML frontmatter。

## 目录结构 Layout

- `SKILL.md` — 入口与工作流 entry point and workflow.
- `references/` — CLI、kernel API、database API、Inbox、概念参考 references.
- `agents/openai.yaml` — agent 元数据 metadata.

## 配置 Configuration

The skill resolves per-device values at runtime instead of hardcoding them:

- `SIYUAN_WORKSPACE` — path to your SiYuan workspace directory.
- `SIYUAN_BASE_URL` — kernel base URL (default `http://127.0.0.1:6806`).
- `SIYUAN_TOKEN` — kernel API token, only if access control is enabled.

## License

MIT
