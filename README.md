# siyuan-notes skill

一个 agent skill，通过 [SiYuan（思源笔记）](https://github.com/siyuan-note/siyuan) 的 CLI 和本地内核 HTTP API，把 SiYuan 作为个人笔记系统使用。覆盖搜索/回忆、记录与整理笔记、云端 Inbox（收集箱）、笔记本/文档/块、属性、导出、模板、SQL 以及数据库/属性视图。

> 本 skill 采用**轻量入口 + 按需 reference** 结构：`SKILL.md` 只保留路由、安全红线和最小工作流，具体 CLI / 搜索阅读 / 写入 / 内核 API / 数据库 API / Inbox / 概念参考拆在 `references/` 下，降低每次调用的 token 成本。

## 安装 Installation

> 当前版本依赖 `references/` 目录做按需读取。不要使用只安装单个 `SKILL.md` 的安装器；如果安装器不会复制整个目录，运行时将无法读取参考文件。

### 方式一：手动安装（git clone，推荐）

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

### 方式二：手动复制整个目录

如果已经下载本仓库，也可以把整个 `siyuan-notes/` 目录复制到对应运行时的 skills 目录。必须包含：

- `SKILL.md`
- `references/`
- `agents/openai.yaml`

### 方式三：直接引用（仅用于临时对话）

如果你的运行时不支持自动加载 skill，可以把 `SKILL.md` 作为入口粘贴进对话；复杂任务还需要按其中路由继续提供对应 `references/*.md` 文件内容。

## 目录结构 Layout

- `SKILL.md` — 轻量入口：触发范围、按需路由、最小工作流、安全红线。
- `references/cli.md` — CLI 版本契约、常用命令、运行时元数据边界。
- `references/search-read.md` — 只读搜索、字段裁剪、候选文档阅读流程。
- `references/write-append.md` — 追加/新建笔记、dry-run、校验流程。
- `references/inbox.md` — 云端 Inbox/收集箱读取、整理和删除边界。
- `references/kernel-api.md` — 官方内核 API 路由、协议、端点风险层级。
- `references/database-api.md` — 属性视图/数据库 API 标识符与读写事务。
- `references/concepts-and-operations.md` — SiYuan 概念、组织机制、同步和备份边界。
- `agents/openai.yaml` — Codex 等运行时使用的 agent 元数据。
- `THIRD_PARTY_NOTICES.md` — 上游资料与许可证说明。

## 配置 Configuration

本 skill 按设备在运行时解析各项值，不做硬编码：

- `SIYUAN_WORKSPACE` — 你的 SiYuan 工作区目录路径。
- `SIYUAN_BASE_URL` — 内核基址（默认 `http://127.0.0.1:6806`）。
- `SIYUAN_TOKEN` — 内核 API 令牌，仅在开启访问控制时需要。

## License

本项目原创内容使用 MIT License。第三方来源及其适用许可证见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
