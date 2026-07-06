# siyuan-notes skill

一个 agent skill，通过 [SiYuan（思源笔记）](https://github.com/siyuan-note/siyuan) 的 CLI 和本地内核 HTTP API，把 SiYuan 作为个人笔记系统使用。覆盖搜索/回忆、记录与整理笔记、云端 Inbox（收集箱）、笔记本/文档/块、属性、导出、模板、SQL 以及数据库/属性视图。

本 skill 采用单文件发布：运行所需的工作流、CLI/API、Inbox 与概念参考全部内联在 `SKILL.md`。

> 原因：实测 `skills@1.5.14` 从本地路径安装时会复制完整目录，但通过 GitHub 仓库远程安装时只保留 `SKILL.md`。为了保证 `npx skills add ZyoHuang/siyuan-notes` 安装后能力完整，本仓库有意接受较长的单文件结构。

## 安装 Installation

> 安装方式参考自 [nuwa-skill](https://github.com/alchaincyf/nuwa-skill)。

### 方式一：一行命令（推荐，跨运行时）

直接告诉你的 AI agent：

```
帮我安装这个 skill：https://github.com/ZyoHuang/siyuan-notes
```

或使用第三方通用 CLI 安装器 [`skills`](https://www.npmjs.com/package/skills)（固定为本仓库已核对的版本）：

```bash
npx --yes skills@1.5.14 add ZyoHuang/siyuan-notes
```

- 加 `-g` 安装到全局（用户级）目录，例如 `~/.claude/skills/`。
- 加 `-a` 指定运行时，例如 `-a claude-code`、`-a codex`、`-a cursor`。
- `skills` 不是 OpenAI 官方工具。Codex 用户可直接输入：`使用 $skill-installer 从 ZyoHuang/siyuan-notes 仓库根目录（path .）安装，名称为 siyuan-notes`。

### 方式二：手动安装（git clone）

克隆到对应运行时的 skills 目录：

| 运行时 Runtime | 安装路径 Install Path |
|---|---|
| Claude Code | `~/.claude/skills/siyuan-notes/` |
| Codex CLI / App | `~/.agents/skills/siyuan-notes/` |
| Cursor | `~/.cursor/skills/siyuan-notes/` |
| 其他 Other | 克隆到对应运行时的 `skills/` 目录 |

```bash
git clone https://github.com/ZyoHuang/siyuan-notes <对应运行时的 skills 目录>
```

### 方式三：直接引用（不安装）

如果运行时不支持自动加载 skill，直接把 `SKILL.md` 提供给 Agent；它包含完整运行内容。

## 目录结构 Layout

- `SKILL.md` — 完整单文件 skill：入口、工作流、安全边界、CLI/API、数据库与 Inbox 参考。
- `agents/openai.yaml` — Codex 等运行时使用的 agent 元数据。
- `THIRD_PARTY_NOTICES.md` — 上游资料与许可证说明。

## 配置 Configuration

本 skill 按设备在运行时解析各项值，不做硬编码：

- `SIYUAN_WORKSPACE` — 你的 SiYuan 工作区目录路径。
- `SIYUAN_BASE_URL` — 内核基址（默认 `http://127.0.0.1:6806`）。
- `SIYUAN_TOKEN` — 内核 API 令牌，仅在开启访问控制时需要。

## License

本项目原创内容使用 MIT License。第三方来源及其适用许可证见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
