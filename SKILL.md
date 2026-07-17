---
name: siyuan-notes
description: 通过 Windows 或 macOS 上已安装的 SiYuan CLI 和按需使用的本地内核 HTTP API，把 SiYuan（思源笔记）作为用户默认的个人笔记系统。用于搜索、回忆、交叉引用、总结、记录、更新和整理笔记；查询云端 Inbox/收集箱；操作笔记本、文档、块、属性、导出、模板、文件、SQL 以及结构化数据库/属性视图。当用户提到"我的笔记"、"笔记里"、"查下笔记"、"之前记过吗"、"记录到笔记"、"记到笔记"、"整理进笔记"、"收集箱"、Inbox、"知识库"、SiYuan/思源、SiYuan API、kernel API、database/属性视图，或显式调用 $siyuan-notes 时使用。每次工作流开始前先识别平台并校验 CLI；只有任务确实需要 HTTP API 时才校验运行中的内核。除非用户明确要求把内容写入个人知识库，否则不要用于普通仓库文档。
---

# SiYuan Notes（思源笔记）

把已安装的 SiYuan CLI 作为处理日常笔记工作的默认入口，把本地内核 HTTP API 作为 CLI 未清晰暴露的已文档化能力的补充入口。CLI 直接访问工作区，不要求 SiYuan 桌面端或 6806 内核处于运行状态。

本文件是轻量入口。每次先选择一个平台 reference，再根据任务读取必要的其它 `references/*.md`；不要默认读取全部参考文件。除非用户明确要求把内容复制进个人笔记，否则将仓库文档、源代码文件和生成的交付物与个人笔记分开。

## 平台分流

- Windows：完整读取 `references/platform-windows.md`，用 PowerShell 语法执行命令。
- macOS：完整读取 `references/platform-macos.md`，用 POSIX shell 语法执行命令。
- 每次工作流只读取与当前主机匹配的一个平台 reference。不要在 Windows 上运行 macOS 示例，也不要在 macOS 上运行 PowerShell 示例。
- 其它平台不在当前兼容性承诺内；停止并说明未验证平台，不要猜测安装路径或 shell 语法。

## 按需读取路由

- 查找、回忆、比较、总结本地笔记：读 `references/cli.md` 和 `references/search-read.md`。
- 记录、整理、追加到现有笔记或必要时新建文档：读 `references/cli.md`、`references/search-read.md` 和 `references/write-append.md`。
- Inbox / 收集箱 / 云端速记：读 `references/cli.md` 和 `references/inbox.md`；若要导入本地笔记，再读 `references/write-append.md`。
- 已文档化内核 API、导出、模板、属性、路径转换、文件读等：读 `references/cli.md` 和 `references/kernel-api.md`。
- 数据库 / 属性视图 / attribute view / `/api/av/*`：读 `references/kernel-api.md` 和 `references/database-api.md`。
- 概念性组织设计、笔记结构、块引用、标签、同步、备份或安全边界：读 `references/concepts-and-operations.md`。

如果任务跨多个类别，只读实际需要的 reference。被选中的 reference 必须完整阅读；未选中的 reference 不要为“以防万一”读取。

## 启动检查

1. 识别当前平台并完整读取对应平台 reference；按其中顺序解析 CLI 可执行文件绝对路径。
2. 每次触发工作流都运行 `<binary> --version`。
3. 构造命令前运行 `<binary> --help`，再运行实际会用到的 `<binary> <command> --help` 和必要的嵌套子命令帮助。
4. 工作区优先使用 CLI 参数 `--workspace` / `-w`；需要环境变量时只使用官方名称 `SIYUAN_WORKSPACE_PATH`。
5. CLI-only 任务不要探测 6806，也不要为了使用 CLI 启动 SiYuan。只有确定需要 HTTP API 时，才按平台 reference 探测已有内核并调用 `/api/system/version`。
6. 把 reference 中的命令示例视为历史参考。已安装 CLI 帮助、实时内核响应和官方文档具有更高优先级。
7. 若 CLI、内核版本或响应结构和 reference 不一致，先按实时帮助重建最小命令；仍不确定时停止并说明差异。
8. 不要安装或升级 SiYuan，不要创建 shim/符号链接，也不要修改 `PATH`、shell 配置或全局包配置。只诊断问题；如需修复，向用户说明对应平台 reference 中的官方方式并等待明确授权。

## 默认工作流

1. 定位工作区和笔记本；工作区未知时通过实时 CLI 的 workspace 命令解析。
2. 写入前先搜索并阅读候选目标，优先选择最具体的现有文档。
3. 只读任务输出相关路径、块 ID 和简洁结论；避免倾倒完整搜索 JSON 或长篇正文。
4. 写入任务先 dry-run（若 CLI 支持），再执行一个有界变更，最后重新搜索或读取目标校验。
5. HTTP API 仅在 CLI 缺失或结构不够直接时使用；任何写入或高影响调用前先读当前目标。
6. 同时检查 HTTP 状态和 SiYuan 顶层 `code`；`code` 非零时停止。

## 输出与 token 纪律

- 搜索默认使用小页数和聚焦关键词；先找 `NodeDocument` 或最可能的根文档，再读取全文。
- CLI/HTTP 返回 JSON 时，使用当前平台 reference 的 JSON 解析方式，优先投影必要字段，例如 `id`、`rootID`、`hPath`、`type`、`subType`、`markdown`、`ial.updated`。
- 不要把大段剪藏正文、完整 Inbox HTML、数据库全量视图或无关搜索命中发回模型。
- 需要引用 memory 或日志时，只读取命中行附近的小范围内容，不要整文件编号输出。

## 安全红线

- 未经用户明确请求，绝不调用 `block delete`、`document remove`、`notebook remove`、`/api/block/deleteBlock`、`/api/filetree/removeDoc*` 或等价删除端点。
- 未经明确指名删除 Inbox，绝不调用 `/api/inbox/removeShorthands`，也不要把导入成功当作删除来源授权。
- 未经明确指名破坏性后果，绝不删除文档、笔记本、块、文件、数据库行或数据库字段。
- 绝不为了新增内容覆盖整篇文档；追加优先。
- 绝不用 `/api/file/putFile` 编辑 `.sy` 数据文件。
- 绝不用 `/api/query/sql` 做变更；只允许只读 `SELECT` 或 `WITH ... SELECT`。
- 除非用户明确授权外部传输，绝不用 `/api/network/forwardProxy` 传输笔记内容、凭证或私有 URL。
- 绝不在总结中暴露访问令牌、订阅 URL、激活码、私有工作区链接或其它凭证类内容；先脱敏。
- 不要把 SiYuan 内容或个人笔记上传到外部服务。
- 写入部分成功时停止后续变更，检查受影响文档，避免重复写入。

## 汇报约定

- 只读任务：报告相关人类可读路径、必要块 ID、检索范围和简洁综合结论。
- 写入任务：报告内容是追加还是新建、目标笔记路径、校验方式和结果。
- 删除、覆盖、回滚、来源移除、强制同步方向和外部传输：先请求明确授权。
