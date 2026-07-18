# siyuan-notes skill

一个 agent skill，通过 [SiYuan（思源笔记）](https://github.com/siyuan-note/siyuan) 的 CLI 和本地内核 HTTP API，把 SiYuan 作为个人笔记系统使用。覆盖搜索/回忆、记录与整理笔记、云端 Inbox（收集箱）、笔记本/文档/块、属性、导出、模板、SQL 以及数据库/属性视图。

当前兼容性目标是 Windows、macOS 和 Linux。仓库采用单一 Skill、共享业务 reference、平台专属执行 reference 的结构，不复制多套容易漂移的业务规则：

- `references/platform-windows.md`：PowerShell、普通安装包与 Microsoft Store 版的 CLI 发现、JSON 和 HTTP 调用。
- `references/platform-macos.md`：POSIX shell、应用包内 CLI、官方符号链接、JSON 和 HTTP 调用。
- `references/platform-linux.md`：POSIX shell、CLI 发现、官方符号链接、JSON 和 HTTP 调用。
- 其它 reference：平台无关的 SiYuan 命令语义、工作流与安全边界。

> 本 skill 采用**轻量入口 + 按需 reference** 结构：`SKILL.md` 只保留平台分流、任务路由、安全红线和最小工作流，具体执行方式按需读取，降低每次调用的 token 成本。

## 安装 Installation

> 当前版本依赖 `references/` 目录做按需读取。不要使用只安装单个 `SKILL.md` 的安装器；如果安装器不会复制整个目录，运行时将无法读取参考文件。

### Codex CLI / App

Codex 当前内置 Skill 安装器默认使用 `$CODEX_HOME/skills`；未设置 `CODEX_HOME` 时回退到 `~/.codex/skills`。Windows PowerShell：

```powershell
$codexHome = if ($env:CODEX_HOME) {
    $env:CODEX_HOME
} else {
    Join-Path $HOME '.codex'
}
$skillsDir = Join-Path $codexHome 'skills'
New-Item -ItemType Directory -Force -Path $skillsDir | Out-Null
git clone https://github.com/ZyoHuang/siyuan-notes (Join-Path $skillsDir 'siyuan-notes')
```

macOS / Linux：

```bash
skills_dir="${CODEX_HOME:-$HOME/.codex}/skills"
mkdir -p "$skills_dir"
git clone https://github.com/ZyoHuang/siyuan-notes "$skills_dir/siyuan-notes"
```

若目标目录已存在，先按你的运行时更新/卸载流程处理，不要直接覆盖一个正在使用的 Skill。

Codex 也会发现用户级 `$HOME/.agents/skills`。已有 Skill 若安装在那里可以继续使用；不要同时安装两个同名副本。

### 其它支持 Skill 的运行时

| 运行时 Runtime | 安装路径 Install Path |
|---|---|
| Claude Code | `~/.claude/skills/siyuan-notes/` |
| Cursor | `~/.cursor/skills/siyuan-notes/` |
| 其他 Other | 复制或克隆到该运行时约定的 `skills/siyuan-notes/` |

手动复制时必须包含：

- `SKILL.md`
- `references/`
- `agents/openai.yaml`

## SiYuan CLI 前置条件

- Windows 官方安装包会把内核目录加入 `PATH` 并提供 `siyuan.exe`。Microsoft Store 版受 MSIX 限制，不能修改 `PATH`；官方 CLI 指南提供了稳定的 `siyuan.cmd` shim 方案。
- macOS 安装包内的二进制位于 `/Applications/SiYuan.app/Contents/Resources/kernel/SiYuan-Kernel`；官方 CLI 指南建议创建 `/usr/local/bin/siyuan` 符号链接。
- Linux 安装目录因分发方式而异；官方 CLI 指南建议把 `<install-dir>/resources/kernel/SiYuan-Kernel` 符号链接到 `/usr/local/bin/siyuan`。
- Skill 自身只发现和诊断 CLI，不会安装/升级 SiYuan，也不会擅自创建 shim、符号链接或修改 `PATH`。
- SiYuan CLI 直接访问工作区，CLI-only 工作流不要求桌面端或 6806 内核运行。

完整命令见对应的 `references/platform-*.md`。

## 目录结构 Layout

- `SKILL.md` — 轻量入口：平台分流、触发范围、按需路由、最小工作流、安全红线。
- `references/platform-windows.md` — Windows/PowerShell CLI 发现、调用、JSON 与 HTTP 模式。
- `references/platform-macos.md` — macOS/POSIX shell CLI 发现、调用、JSON 与 HTTP 模式。
- `references/platform-linux.md` — Linux/POSIX shell CLI 发现、调用、JSON 与 HTTP 模式。
- `references/cli.md` — 平台无关的 CLI 版本契约、常用命令和运行时元数据边界。
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

- `SIYUAN_WORKSPACE_PATH` — SiYuan 官方 CLI 工作区环境变量；也可以每次显式传 `--workspace` / `-w`。
- `SIYUAN_BASE_URL` — 仅用于内核 HTTP API，默认 `http://127.0.0.1:6806`。
- `SIYUAN_TOKEN` — 仅在内核 API 开启访问控制时需要；不要写入仓库或日志。

## License

本项目原创内容使用 MIT License。第三方来源及其适用许可证见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。
