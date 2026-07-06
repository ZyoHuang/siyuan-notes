---
name: siyuan-notes
description: 通过已安装的 SiYuan CLI 和本地内核 HTTP API，把 SiYuan（思源笔记）作为用户默认的个人笔记系统。用于搜索、回忆、交叉引用、总结、记录、更新和整理笔记；查询云端 Inbox/收集箱；操作笔记本、文档、块、属性、导出、模板、文件、SQL 以及结构化数据库/属性视图。当用户提到"我的笔记"、"笔记里"、"查下笔记"、"之前记过吗"、"记录到笔记"、"记到笔记"、"整理进笔记"、"收集箱"、Inbox、"知识库"、SiYuan/思源、SiYuan API、kernel API、database/属性视图，或显式调用 $siyuan-notes 时使用。每次工作流开始前，先校验已安装的 CLI 和运行中的内核，因为命令与 API 结构可能变化。除非用户明确要求把内容写入个人知识库，否则不要用于普通仓库文档。
---

# SiYuan Notes（思源笔记）

把已安装的 SiYuan CLI 作为处理日常笔记工作的默认入口，把本地内核 HTTP API 作为 CLI 未清晰暴露的已文档化能力的补充入口。

把对用户笔记或个人知识库的泛指请求都当作 SiYuan 请求处理。除非用户明确要求把内容复制进个人笔记，否则将仓库文档、源代码文件和生成的交付物与个人笔记分开。

## 选择操作类型

- 对于概念性问题、组织结构设计，或跨多个 SiYuan 特性的工作流，先阅读 [references/concepts-and-operations.md](references/concepts-and-operations.md)，再在文档、块引用、标签、属性、嵌入查询、数据库、模板、Inbox、历史、同步、备份之间做选择。
- 对于查找、比较或回忆类请求，执行只读搜索，返回相关笔记路径加上简洁的综合结论。
- 对于 Inbox 或收集箱请求，先阅读 [references/inbox.md](references/inbox.md)，优先使用实时 CLI 的 `inbox list|get|convert`；仅在 CLI 未暴露所需能力时使用内核 Inbox API。绝不要从名为 `Inbox`、`daily note`、`收件箱`、`收集箱` 的本地文档推断 Inbox 内容。
- 对于已文档化的内核 API 请求，先阅读 [references/kernel-api.md](references/kernel-api.md)，优先使用读端点；当 CLI 能以更安全的校验或 dry-run 提供同等操作时改用 CLI。
- 对于数据库或属性视图请求，同时阅读 [references/database-api.md](references/database-api.md)，并保持 `avID`、`blockID`、`viewID`、`itemID`、`keyID` 各自区分。
- 对于记录、整理或更新请求，先搜索、读候选文档、追加到最合适的现有笔记，然后校验写入结果。
- 仅当没有合适的现有笔记且用户要求记录该内容时，才新建文档。
- 仅当用户明确请求某个精确变更时，才执行重命名、移动、删除或覆盖。

## 遵循工作流

1. 任何笔记操作前，检测并校验运行中的 CLI。
   - 运行 `command -v siyuan`，使用返回的可执行文件绝对路径。
   - 每次触发工作流（包括只读搜索）都运行 `<binary> --version`。
   - 构造命令前，运行 `<binary> --help`，再运行 `<binary> <command> --help` 以及必要的嵌套子命令帮助。
   - 把内置命令示例只当作历史参考。以已安装版本及其实时帮助为准。
   - 若版本与 [references/cli.md](references/cli.md) 中最后校验的版本不同，未查帮助前不要复用旧语法。
   - 若实时帮助已不再暴露所需操作或参数，停止并说明兼容性差异，而不是猜测。
   - 不要安装或升级 SiYuan，也不要修改 shell 或全局包配置。
2. 定位工作区。
   - 工作区尚未确定时，用实时的 workspace-list 帮助来构造命令。
   - 把 [references/cli.md](references/cli.md) 中的路径和笔记本 ID 视为默认值，失败或与当前上下文冲突时需重新校验。
3. 将已文档化的 HTTP API 请求路由到运行中的本地内核。
   - 构造内核请求前先阅读 [references/kernel-api.md](references/kernel-api.md)。
   - 确认已有监听器与 `/api/system/version`；桌面端已在监听时不要再启动第二个内核。
   - 版本不同或响应结构冲突时，比较实时内核版本与参考快照，并重新查阅官方 API 文档。
   - 从能回答请求的“最小变更”端点开始。任何 `set`、`update`、`move`、`remove`、`delete` 调用前先检查目标当前状态。
   - 同时检查 HTTP 结果和顶层 SiYuan `code`；`code` 非零时停止，不要把部分 `data` 当作成功。
4. 将 Inbox 请求路由到实时 CLI 或运行中的本地内核。
   - 操作 Inbox 前先阅读 [references/inbox.md](references/inbox.md)。
   - 把 SiYuan Inbox 视为由本地内核代理的云端速记（cloud shorthands），与工作区文档和本地 SQL 索引分开。
   - 先只读取全部列表分页，仅在分类或提取需要时再取单条详情。
   - 转换前先 dry-run，并显式传 `--remove-after=false`；SiYuan CLI 3.7.1 的该选项默认值为 `true`。
   - 把删除作为独立且需明确授权的动作。绝不要把导入或总结成功当作删除 Inbox 条目的授权。
5. 把数据库操作当作“读—检查—写—校验”事务处理。
   - 使用任何 `/api/av/*` 端点前先阅读 [references/database-api.md](references/database-api.md)。
   - 先 render 或 get 数据库，从响应中获取 ID。绝不要从单元格值猜测行/项 ID，也不要在需要 item ID 的地方使用数据库块 ID。
   - 变更前先快照 filters、sorts、grouping、fields 和目标行，因为若干 setter 会替换整个数组或定义。
   - 写入后重新 render 或重新读取确切的视图，并比较归一化后的返回值或结构。
6. 写入前先搜索。
   - 从内容中提炼聚焦关键词，包括产品名、技术名和中文近义词。
   - 优先选用 `NodeDocument` 结果，比较其 `hPath`、标题和近期内容。
   - 避免像 `Resources` 这类宽泛的纯英文查询导致无关结果；组合词或使用正则。
7. 阅读候选文档。
   - 候选集较小时用 `block batch-kramdown`。
   - 检查现有完整内容，关注重复、术语、结构、事实边界和内部链接。
8. 选择目标位置。
   - 优先选最具体的现有笔记，而非泛化的父级笔记。
   - 仅当每部分内容有明显不同的长期检索目的时，才把内容拆分到两个或多个现有笔记。
   - 若多个候选同样合理且选择会实质影响组织结构，提一个简洁的问题确认。
9. 组织要追加的内容。
   - 保留现有文本，追加结构良好的 Markdown 段落。
   - 去重已存在的内容。
   - 区分已验证事实、用户经验、推断和建议。
   - 保留对话中有用的来源链接。
   - 有用时用 SiYuan 块引用关联现有笔记：`((block-id "锚文本"))`。
10. 变更前校验。
    - 遵循当前工作区 `AGENTS.md` 及环境对临时文件的约束。优先用其指定的临时目录；否则用当前可写工作区内的临时路径。
    - 用实时帮助确认是否支持 dry-run，若当前 CLI 支持则先跑 dry-run。
    - 检查 dry-run 目标，指向非预期块时不要继续。
11. 通过最安全的受支持入口写入。
    - 现有笔记优先用 CLI `block append`，存在等价功能时优先用其它 CLI 命令。
    - 仅当所需能力必须用官方 HTTP 变更、且目标已先读取并校验时才使用。
    - 绝不要直接编辑 `.sy` 文件。
    - 当工作区或 SiYuan 运行时元数据在当前可写项目之外时，请求环境所需的批准。
12. 校验并清理。
    - 搜索一个或多个唯一的新标题或短语。
    - 确认预期的根文档、路径、块类型和 Markdown。
    - 只删除本次操作创建的临时文件。
    - 报告改动的笔记路径、内容是追加还是新建、以及校验如何通过。

## 执行安全规则

- 未经用户明确请求，绝不调用 `block delete`、`document remove`、`notebook remove` 或等价的破坏性命令。
- 未经明确指名删除 Inbox 的请求，绝不调用 `/api/inbox/removeShorthands`。
- 未经明确指名删除 Inbox 来源的请求，绝不让 `siyuan inbox convert` 使用默认删除行为；必须显式传 `--remove-after=false`。
- 未经明确指名该破坏性后果的请求，绝不调用文档、笔记本、块、文件、数据库行或数据库字段的删除端点。
- 绝不为了新增一个段落而覆盖整篇文档。
- 绝不用 `/api/file/putFile` 编辑 `.sy` 数据文件；通过语义化的文档或块 API 写入笔记内容。
- 除非用户明确请求该外部传输，绝不用 `/api/network/forwardProxy` 传输笔记内容、凭证或私有 URL。
- 绝不用 `/api/query/sql` 做变更；把 Skill 工作流限制为只读 `SELECT` 或 `WITH ... SELECT` 语句。
- 绝不把 `/api/sqlite/flushTransaction` 当作常规清理；除非用户明确要求诊断性控制，交由正常的内核/CLI 生命周期管理事务。
- 绝不在总结中暴露访问令牌、订阅 URL、激活码、私有工作区链接或其它类凭证的 Inbox 内容；脱敏并标记为敏感。
- 绝不把 Office 锁文件、生成的搜索高亮或无关关键词命中当作素材。
- 不要把 SiYuan 内容或个人笔记上传到外部服务。
- 不要安装全局依赖。使用 `command -v siyuan` 返回的可执行文件；不要假设它永远位于 `/usr/local/bin/siyuan`。
- 写入部分成功时，停止后续变更，检查受影响文档，避免重复已成功的部分。

## 按需读取参考资料

只读取当前请求所需的参考文件；实时 CLI 帮助、内核响应和更新的官方文档始终优先于仓库快照。

- 概念选择、内容块、组织方式、历史、同步与备份：[references/concepts-and-operations.md](references/concepts-and-operations.md)
- CLI 版本、命令示例和校验模式：[references/cli.md](references/cli.md)
- 官方内核 HTTP API、端点风险与校验：[references/kernel-api.md](references/kernel-api.md)
- 数据库/属性视图的 ID、值形状和变更规则：[references/database-api.md](references/database-api.md)
- 云端 Inbox/收集箱读取、整理、转换和删除边界：[references/inbox.md](references/inbox.md)
