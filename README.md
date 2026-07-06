# siyuan-notes skill

一个 agent skill，通过 [SiYuan（思源笔记）](https://github.com/siyuan-note/siyuan) 的 CLI 和本地内核 HTTP API，把 SiYuan 作为个人笔记系统使用。覆盖搜索/回忆、记录与整理笔记、云端 Inbox（收集箱）、笔记本/文档/块、属性、导出、模板、SQL 以及数据库/属性视图。

> 本 skill 为**单文件**设计：所有内容（工作流 + CLI / 内核 API / 数据库 API / Inbox / 概念参考）都内联在 `SKILL.md` 中，方便 `npx skills add` 一键完整安装。

## 安装 Installation

> 安装方式参考自 [nuwa-skill](https://github.com/alchaincyf/nuwa-skill)。

### 方式一：一行命令（推荐，跨运行时）

直接告诉你的 AI agent：

```
帮我安装这个 skill：https://github.com/ZyoHuang/siyuan-notes
```

或使用通用 CLI 安装器（自动识别当前运行时）：

```bash
npx skills add ZyoHuang/siyuan-notes
```

- 加 `-g` 安装到全局（用户级）目录，例如 `~/.claude/skills/`。
- 加 `-a` 指定运行时，例如 `-a claude-code`、`-a codex`、`-a cursor`。

### 方式二：手动安装（git clone）

克隆到对应运行时的 skills 目录：

| 运行时 Runtime | 安装路径 Install Path |
|---|---|
| Claude Code | `~/.claude/skills/siyuan-notes/` |
| Codex CLI | `~/.codex/skills/siyuan-notes/` |
| Cursor | `~/.cursor/skills/siyuan-notes/` |
| 其他 Other | 克隆到对应运行时的 `skills/` 目录 |

```bash
git clone https://github.com/ZyoHuang/siyuan-notes ~/.claude/skills/siyuan-notes
```

### 方式三：直接引用

如果你的运行时不支持自动加载 skill，把 `SKILL.md` 的内容直接粘贴进对话即可 —— 它本质就是一份 markdown + YAML frontmatter。

## 目录结构 Layout

- `SKILL.md` — 单文件 skill：入口、工作流，以及内联的 CLI / 内核 API / 数据库 API / Inbox / 概念参考。
- `agents/openai.yaml` — Codex 等运行时使用的 agent 元数据。

## 配置 Configuration

本 skill 按设备在运行时解析各项值，不做硬编码：

- `SIYUAN_WORKSPACE` — 你的 SiYuan 工作区目录路径。
- `SIYUAN_BASE_URL` — 内核基址（默认 `http://127.0.0.1:6806`）。
- `SIYUAN_TOKEN` — 内核 API 令牌，仅在开启访问控制时需要。

## License

MIT
