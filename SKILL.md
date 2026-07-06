---
name: siyuan-notes
description: 通过已安装的 SiYuan CLI 和本地内核 HTTP API，把 SiYuan（思源笔记）作为用户默认的个人笔记系统。用于搜索、回忆、交叉引用、总结、记录、更新和整理笔记；查询云端 Inbox/收集箱；操作笔记本、文档、块、属性、导出、模板、文件、SQL 以及结构化数据库/属性视图。当用户提到"我的笔记"、"笔记里"、"查下笔记"、"之前记过吗"、"记录到笔记"、"记到笔记"、"整理进笔记"、"收集箱"、Inbox、"知识库"、SiYuan/思源、SiYuan API、kernel API、database/属性视图，或显式调用 $siyuan-notes 时使用。每次工作流开始前，先校验已安装的 CLI 和运行中的内核，因为命令与 API 结构可能变化。除非用户明确要求把内容写入个人知识库，否则不要用于普通仓库文档。
---

# SiYuan Notes（思源笔记）

把已安装的 SiYuan CLI 作为处理日常笔记工作的默认入口，把本地内核 HTTP API 作为 CLI 未清晰暴露的已文档化能力的补充入口。

把对用户笔记或个人知识库的泛指请求都当作 SiYuan 请求处理。除非用户明确要求把内容复制进个人笔记，否则将仓库文档、源代码文件和生成的交付物与个人笔记分开。

> 为兼容 `npx skills@1.5.14 add <GitHub 仓库>` 的远程安装行为，本技能将运行所需参考内容内联在一个 `SKILL.md` 中。该安装路径只保留 `SKILL.md`，不要把下列参考章节拆成运行时必需的外部文件。

## 选择操作类型

- 对于概念性问题、组织结构设计，或跨多个 SiYuan 特性的工作流，先阅读 【参考：概念与操作原则】，再在文档、块引用、标签、属性、嵌入查询、数据库、模板、Inbox、历史、同步、备份之间做选择。
- 对于查找、比较或回忆类请求，执行只读搜索，返回相关笔记路径加上简洁的综合结论。
- 对于 Inbox 或收集箱请求，先阅读 【参考：Inbox】，优先使用实时 CLI 的 `inbox list|get|convert`；仅在 CLI 未暴露所需能力时使用内核 Inbox API。绝不要从名为 `Inbox`、`daily note`、`收件箱`、`收集箱` 的本地文档推断 Inbox 内容。
- 对于已文档化的内核 API 请求，先阅读 【参考：内核 API】，优先使用读端点；当 CLI 能以更安全的校验或 dry-run 提供同等操作时改用 CLI。
- 对于数据库或属性视图请求，同时阅读 【参考：数据库 API】，并保持 `avID`、`blockID`、`viewID`、`itemID`、`keyID` 各自区分。
- 对于记录、整理或更新请求，先搜索、读候选文档、追加到最合适的现有笔记，然后校验写入结果。
- 仅当没有合适的现有笔记且用户要求记录该内容时，才新建文档。
- 仅当用户明确请求某个精确变更时，才执行重命名、移动、删除或覆盖。

## 遵循工作流

1. 任何笔记操作前，检测并校验运行中的 CLI。
   - 运行 `command -v siyuan`，使用返回的可执行文件绝对路径。
   - 每次触发工作流（包括只读搜索）都运行 `<binary> --version`。
   - 构造命令前，运行 `<binary> --help`，再运行 `<binary> <command> --help` 以及必要的嵌套子命令帮助。
   - 把内置命令示例只当作历史参考。以已安装版本及其实时帮助为准。
   - 若版本与 【参考：CLI】 中最后校验的版本不同，未查帮助前不要复用旧语法。
   - 若实时帮助已不再暴露所需操作或参数，停止并说明兼容性差异，而不是猜测。
   - 不要安装或升级 SiYuan，也不要修改 shell 或全局包配置。
2. 定位工作区。
   - 工作区尚未确定时，用实时的 workspace-list 帮助来构造命令。
   - 把 【参考：CLI】 中的路径和笔记本 ID 视为默认值，失败或与当前上下文冲突时需重新校验。
3. 将已文档化的 HTTP API 请求路由到运行中的本地内核。
   - 构造内核请求前先阅读 【参考：内核 API】。
   - 确认已有监听器与 `/api/system/version`；桌面端已在监听时不要再启动第二个内核。
   - 版本不同或响应结构冲突时，比较实时内核版本与参考快照，并重新查阅官方 API 文档。
   - 从能回答请求的“最小变更”端点开始。任何 `set`、`update`、`move`、`remove`、`delete` 调用前先检查目标当前状态。
   - 同时检查 HTTP 结果和顶层 SiYuan `code`；`code` 非零时停止，不要把部分 `data` 当作成功。
4. 将 Inbox 请求路由到实时 CLI 或运行中的本地内核。
   - 操作 Inbox 前先阅读 【参考：Inbox】。
   - 把 SiYuan Inbox 视为由本地内核代理的云端速记（cloud shorthands），与工作区文档和本地 SQL 索引分开。
   - 先只读取全部列表分页，仅在分类或提取需要时再取单条详情。
   - 转换前先 dry-run，并显式传 `--remove-after=false`；SiYuan CLI 3.7.1 的该选项默认值为 `true`。
   - 把删除作为独立且需明确授权的动作。绝不要把导入或总结成功当作删除 Inbox 条目的授权。
5. 把数据库操作当作“读—检查—写—校验”事务处理。
   - 使用任何 `/api/av/*` 端点前先阅读 【参考：数据库 API】。
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

---

# 参考：概念与操作原则

## 来源与权威性

该工作模型于 `2026-07-06` 从 SiYuan `3.7.1` 中本地的 **SiYuan 用户指南** 笔记本提炼而来。其 75 篇文档大纲已清点，并跨内容块、编辑、搜索、数据库、模板、CLI/API、插件、AI、历史、同步、备份和安全等章节阅读了代表性内容。

把本章节视为持久的特性选择指南，而非不可变的命令或 API 规范。已安装 CLI 的帮助、实时内核响应和更新的官方文档始终具有权威性。请重新核对，因为指南和 CLI 都可能漂移。

## 心智模型

- 以**块优先**（block-first）而非页面优先思考。每个内容块有全局唯一 ID；文档是容器块，而非独立原语。
- 把规范知识保存在一个块中，通过块引用复用。正向链接展示依赖；反向链接和提及暴露复用与新出现的上下文。
- 把层级只视为一种组织层。SiYuan 融合笔记本/文档层级、引用、标签、书签、属性、嵌入查询和结构化数据库。
- 把工作区视为隔离与生命周期边界。它包含数据、配置、历史、快照、插件、模板和运行时文件，且同一时间只能有一个内核进程服务它。
- 假设本地笔记数据以明文存储。因此本地保密性取决于操作系统和设备安全。

## 选择正确的组织机制

| 需求 | 优先选择 | 原因 |
|---|---|---|
| 稳定导航与归属 | 笔记本与文档层级 | 提供清晰的人类可读归属 |
| 在多处复用同一事实 | 块引用 | 保留唯一规范来源并产生反向链接 |
| 展示现有块的动态集合 | 嵌入查询 | 渲染查询结果而不复制内容 |
| 轻量分类或收藏 | 层级标签与书签 | 无需 schema 的快速手动检索 |
| 机器可读元数据 | 内置或 `custom-*` 块属性 | 支持确定性查询与自动化 |
| 带类型字段和多视图的记录 | 数据库/属性视图 | 支持表格、看板、卡片、筛选、排序、分组、关联、汇总 |
| 重复的采集结构 | 模板与每日笔记 | 标准化重复笔记，同时保留普通块 |
| 临时外部采集 | 云端 Inbox | 让未处理素材与本地文档分离 |
| 长期记忆 | 闪卡 | 用块结构配合 FSRS 调度 |

对于数据库，记住多个视图可共享同一底层数据，而 filters、sorts、grouping 和布局是各视图独有的。自动化时保持 `avID`、`blockID`、`viewID`、`itemID`、`keyID` 各自区分。

## 操作原则

1. 写入前先搜索并阅读。解析出确切的笔记本、文档、块或数据库项，而非依赖记忆中的标题或 ID。
2. 优先选最具体的现有文档。追加结构化块并保留现有内容；仅当没有合适目标时才新建文档。
3. 信息应保持规范时用块引用。仅当新副本应刻意分叉时才复制文本。
4. 用标签做灵活分类，用属性做自动化，用嵌入查询做动态聚合，用数据库做类型化记录管理。不要把所有问题都硬塞进数据库。
5. 按此顺序优先选择语义化入口：带校验/dry-run 的实时 CLI、已文档化的本地内核 API，最后才在别处缺失所需能力时用内部路由。
6. 绝不直接编辑 `.sy` 文件。使用文档、块、数据库、导入/导出、历史或快照操作。
7. 构造命令前检查实时命令帮助。指南示例是历史证据，不是当前语法的证明。
8. 每次执行一个有界变更，并通过重读确切目标来校验。部分或含糊的成功后停止，防止重复。
9. 在用户请求整理或转换前，让云端 Inbox 条目与本地笔记分离。绝不把删除 Inbox 来源当作隐式清理步骤。
10. 区分插件与挂件：插件扩展应用行为；挂件扩展内容块。除非需要持久的应用内交互，Agent 自动化优先用 CLI 或内核 API。
11. 把内置或外部 AI 使用当作数据传输。仅在用户授权该范围时把笔记内容发送给外部模型，并保留提供方的成本和隐私边界。

## 数据安全模型

| 机制 | 用途 | 重要边界 |
|---|---|---|
| 文件历史 | 恢复近期更新、删除、同步冲突、格式化或替换 | 工作区 history 下的人类可读文件；保留期有限 |
| 数据快照 | 捕获仓库状态以回滚和对比 | 加密/压缩的仓库数据；回滚可能替换当前状态 |
| 同步 | 跨设备保持 `workspace/data/` 一致 | 不是备份；避免同时编辑，绝不与第三方同步盘混用 |
| 备份 | 保留选定快照用于灾难恢复 | 与同步分开，应包含异地副本 |

- 在批量格式化、替换、导入、转换、移动或数据库重构前，检查历史，风险高时创建有意义的快照。
- 优先交替式多设备同步。确保参与设备时间兼容且使用相同的仓库密钥。
- 不要把活动中的 SiYuan 工作区放进第三方同步目录。
- 对重要数据遵循 3-2-1 原则：至少三份副本，两种介质，一份异地。
- 绝不把关键密码、恢复密钥或高价值密钥当作普通明文笔记存储，除非有单独的安全决策。

## 交互默认约定

- 把"笔记里"、"我的笔记"、"知识库"理解为本地 SiYuan 笔记操作。
- 把"收集箱"或 `Inbox` 理解为独立的云端速记存储，而非本地文档标题。
- 对于解释类请求，返回相关的人类可读笔记路径和简洁综合结论。
- 对于写入，报告内容是追加还是新建、结果笔记路径和校验结果。
- 对删除、覆盖、回滚、来源移除、强制同步方向和外部传输，需要明确授权。

---

# 参考：CLI

## 版本契约

- 最后校验的 CLI 版本：`3.7.1`
- 校验日期：`2026-07-06`
- 校验时的版本命令：`siyuan --version`

使用本章节任何命令前，始终先运行：

```bash
command -v siyuan
siyuan --version
siyuan --help
siyuan <command> --help
siyuan <command> <subcommand> --help
```

`command -v siyuan` 返回的可执行文件具有权威性；上文的 `siyuan` 是简写。这些示例描述 3.7.1 版本，可能过时。若已安装版本不同，从实时帮助重建命令。绝不猜测被重命名的命令或参数。

## 本地默认值

按设备解析，不要硬编码：

- 二进制：以 `command -v siyuan` 返回为准（常见为 `/usr/local/bin/siyuan`）。
- 工作区：把 `SIYUAN_WORKSPACE` 设为当前工作区目录，或从 `siyuan workspace list --format json` 读取。
- 主笔记本及其 ID：从 `siyuan notebook list -w "$SIYUAN_WORKSPACE" --format json` 读取。

把这些当作默认值而非永久事实。命令失败或用户提到其它工作区时重新校验。下面的示例用 `"$SIYUAN_WORKSPACE"` 作为工作区占位符。

## 3.7.1 版本读命令示例

3.7.1 CLI 已提供 `inbox list`、`inbox get` 和 `inbox convert`。处理 `Inbox` 或 `收集箱` 请求前先阅读 【参考：Inbox】。Inbox 仍是云端速记，不属于本地文档搜索或 SQL 索引。

对于 CLI 缺失或在 HTTP 上结构更丰富的官方内核能力，阅读 【参考：内核 API】。对于 `/api/av/*` 数据库操作，同时阅读 【参考：数据库 API】。当两个入口都支持同一操作时优先用 CLI，因为实时帮助和 dry-run 让目标校验更安全。

```bash
siyuan workspace list --format json
siyuan notebook list -w "$SIYUAN_WORKSPACE" --format json
siyuan search 'keyword' --method 0 --page-size 100 -w "$SIYUAN_WORKSPACE" --format json
siyuan search 'term1|term2' --method 3 --page-size 100 -w "$SIYUAN_WORKSPACE" --format json
siyuan block batch-kramdown --ids id1,id2 -w "$SIYUAN_WORKSPACE" --format json
siyuan block kramdown --id block-id -w "$SIYUAN_WORKSPACE" --format json
siyuan inbox list --page 1 -w "$SIYUAN_WORKSPACE" --format json
siyuan inbox get --id shorthand-id -w "$SIYUAN_WORKSPACE" --format json
```

最常用的搜索方法：

- `--method 0`：关键词搜索。
- `--method 3`：对若干聚焦备选项做正则搜索。
- 需要按文档分组更易检查时加 `--group-by 1`。

## 3.7.1 版本安全写示例

追加到现有文档或块：

```bash
siyuan block append --parent block-id --file tmp/addition.md --dry-run -w "$SIYUAN_WORKSPACE" --format json
siyuan block append --parent block-id --file tmp/addition.md -w "$SIYUAN_WORKSPACE" --format json
```

仅在需要时新建文档：

```bash
siyuan document create --notebook notebook-id --title 'Title' --path /parent/path --markdown 'Initial content' --dry-run -w "$SIYUAN_WORKSPACE" --format json
siyuan document create --notebook notebook-id --title 'Title' --path /parent/path --markdown 'Initial content' -w "$SIYUAN_WORKSPACE" --format json
```

将 Inbox 条目转换为本地文档时，3.7.1 的 `--remove-after` 默认值为 `true`。除非用户明确要求同时删除云端来源，否则两次调用都必须显式传 `--remove-after=false`：

```bash
siyuan inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false --dry-run -w "$SIYUAN_WORKSPACE" --format json
siyuan inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false -w "$SIYUAN_WORKSPACE" --format json
```

## 路由锚点

不要硬编码笔记标题、路径或块 ID。始终先搜索、读候选文档，再用该设备工作区解析出的 `rootID`/`hPath`。

## 校验模式

写入后，搜索一个唯一标题或短语并确认：

- `rootID` 匹配目标笔记；
- `hPath` 匹配目标路径；
- 新块具有预期的标题/段落类型；
- `markdown` 保留了代码段、链接、表格和内部块引用。

CLI 可能在 `~/.config/siyuan` 下写自己的日志和工作区注册文件。这是工具运行时元数据，不是笔记内容，即使只读操作也可能需要沙箱批准。

---

# 参考：内核 API

## 来源与范围

- 官方来源：[siyuan-note/siyuan API.md](https://github.com/siyuan-note/siyuan/blob/master/API.md)
- 审阅日期：`2026-07-02`
- 快照 SHA-256：`6413441a8357abe1ac8e1ac0fdd32a11a6a2255e90e78b12d8672d7d8efc4a99`
- 已文档化端点数：`67`
- 审阅期间校验的实时内核：`3.7.0`

官方文档跟踪 `master`，因此把本章节视为路由指南而非不可变 schema。当实时内核版本不同、端点返回不同结构或字段被弃用时，重新打开官方来源。Inbox 路由不在公开 API 文档中；单独使用 【参考：Inbox】。

## 协议

- 默认基址：`http://127.0.0.1:6806`。
- 每个已文档化路由都用 `POST`。
- 多数请求用 JSON 加 `Content-Type: application/json`。
- `/api/asset/upload` 和 `/api/file/putFile` 用 multipart form data。
- `/api/file/getFile` 在 HTTP `200` 时返回原始文件字节；错误用 HTTP `202` 加 JSON body。
- 正常 JSON 信封：`{"code":0,"msg":"","data":...}`。任何非零 `code` 视为失败。
- 配置访问控制时的认证头：`Authorization: Token ${SIYUAN_TOKEN}`。把令牌保存在进程环境或经批准的密钥源中；绝不在 Skill 或工作区中回显、记录或保存它。

只读探测：

```bash
curl --silent --show-error --max-time 10 \
  -X POST http://127.0.0.1:6806/api/system/version \
  -H 'Content-Type: application/json' \
  -d '{}'
```

带认证的 JSON 模式：

```bash
curl --silent --show-error --max-time 30 \
  -X POST "${SIYUAN_BASE_URL:-http://127.0.0.1:6806}/api/block/getBlockKramdown" \
  -H "Authorization: Token ${SIYUAN_TOKEN}" \
  -H 'Content-Type: application/json' \
  -d '{"id":"BLOCK_ID"}'
```

仅当访问控制需要且令牌已通过经批准来源可用时，才添加认证头。

## 选择工作流

1. 运行 `SKILL.md` 工作流要求的 CLI 检测、版本、根帮助和相关命令帮助。
2. 检测已有内核监听器并探测 `/api/system/version`。不要在 SiYuan 桌面端旁边启动第二个内核。
3. 当 CLI 暴露同一操作时优先用 CLI，尤其是可用 dry-run 的普通笔记写入。
4. 对 CLI 缺失或建模不够直接的能力用官方 HTTP API，如属性视图、原始导出、模板和结构化路径转换。
5. 任何变更前用最近的读端点读取目标。
6. 对照实时响应校验每个笔记本 ID、文档路径、块 ID 和数据库标识符。
7. 每次执行一个有界变更，检查 `code`，然后通过重读目标校验。

## 标识符

- `notebook` 或 box ID：标识一个笔记本。
- 文档内部 `path`：存储路径如 `/ID.sy`；不是可读标题路径。
- `hPath`：人类可读文档路径如 `/Project/Architecture`。
- 块/文档 `id`：SiYuan 节点 ID。
- 资源/工作区文件 `path`：根植于工作区内，常见于 `/data`、`/assets`、`/temp` 或 `/conf` 下。
- 数据库 `avID`、嵌入 `blockID`、视图 `viewID`、行 `itemID`、字段 `keyID`：各自不同；见 【参考：数据库 API】。

解析出目标后优先用基于 ID 的文档端点。仅有 `hPath` 或存储路径时用路径转换端点。

## 风险层级

- `R` 只读：认证与目标校验后可安全用于查找。
- `S` 有状态/非内容副作用：改变运行时/UI 状态或创建临时输出。
- `W` 写：创建或改变持久工作区数据。
- `D` 破坏性：删除、解绑或丢弃持久数据；需要用户明确授权该确切后果。
- `H` 高影响外部或可执行动作：外部网络转发或 Pandoc 执行；需要明确范围并检查输入。

HTTP API 不暴露 CLI 的通用 dry-run。对 `W`、`D`、`H`，先读当前目标，事后校验。绝不用文件 API 直接编辑 `.sy` 文件。

## 端点目录

### 笔记本

| 端点 | 层级 | 用途与必需输入 |
|---|---:|---|
| `/api/notebook/lsNotebooks` | R | 列出笔记本；`{}`。 |
| `/api/notebook/getNotebookConf` | R | 读配置；`notebook`。 |
| `/api/notebook/openNotebook` | S | 打开运行时笔记本；`notebook`。 |
| `/api/notebook/closeNotebook` | S | 关闭运行时笔记本；`notebook`。 |
| `/api/notebook/createNotebook` | W | 按 `name` 创建；返回笔记本对象。 |
| `/api/notebook/renameNotebook` | W | 重命名；`notebook`、`name`。 |
| `/api/notebook/setNotebookConf` | W | 替换配置；发送 `notebook`、`conf` 前先读当前配置并合并字段。 |
| `/api/notebook/removeNotebook` | D | 永久移除 `notebook`。 |

### 文档与路径

| 端点 | 层级 | 用途与必需输入 |
|---|---:|---|
| `/api/filetree/createDocWithMd` | W | 从 GFM 创建文档；`notebook`、可读 `path`、`markdown`。重复路径不会覆盖文档。 |
| `/api/filetree/renameDocByID` | W | 按 `id`、`title` 重命名；优先在解析目标后使用。 |
| `/api/filetree/renameDoc` | W | 按内部 `notebook`、`path`、`title` 重命名。 |
| `/api/filetree/moveDocsByID` | W | 把 `fromIDs` 移动到父文档/笔记本 `toID`。 |
| `/api/filetree/moveDocs` | W | 把内部 `fromPaths` 移动到 `toNotebook`、`toPath`。 |
| `/api/filetree/removeDocByID` | D | 移除文档 `id`。 |
| `/api/filetree/removeDoc` | D | 按内部 `notebook`、`path` 移除。 |
| `/api/filetree/getHPathByID` | R | 把块/文档 `id` 解析为可读路径。 |
| `/api/filetree/getHPathByPath` | R | 把内部 `notebook`、`path` 解析为可读路径。 |
| `/api/filetree/getPathByID` | R | 把 `id` 解析为 `notebook` 和内部路径。 |
| `/api/filetree/getIDsByHPath` | R | 把可读 `path` 加 `notebook` 解析为匹配的 ID。 |

### 资源与块

| 端点 | 层级 | 用途与必需输入 |
|---|---:|---|
| `/api/asset/upload` | W | multipart 上传，带 `assetsDirPath` 和 `file[]`；默认 `/assets/`。 |
| `/api/block/getBlockKramdown` | R | 以 Kramdown 读取 `id`。 |
| `/api/block/getChildBlocks` | R | 列出 `id` 的直接子块；标题内容作为子块包含。 |
| `/api/block/insertBlock` | W | 用至少一个锚点插入 Markdown/DOM；优先级 `nextID` > `previousID` > `parentID`。 |
| `/api/block/prependBlock` | W | 向 `parentID` 前置 `dataType`、`data`。 |
| `/api/block/appendBlock` | W | 向 `parentID` 追加 `dataType`、`data`。 |
| `/api/block/updateBlock` | W | 用 `dataType`、`data` 替换目标块 `id`；先读取。 |
| `/api/block/moveBlock` | W | 用 `previousID` 或 `parentID` 移动 `id`；两者都有时 `previousID` 胜出。 |
| `/api/block/foldBlock` | S | 折叠块 `id`。 |
| `/api/block/unfoldBlock` | S | 展开块 `id`。 |
| `/api/block/transferBlockRef` | W | 把引用从 `fromID` 转移到 `toID`；省略 `refIDs` 转移全部。 |
| `/api/block/deleteBlock` | D | 删除块 `id`。 |

### 属性、SQL 与模板

| 端点 | 层级 | 用途与必需输入 |
|---|---:|---|
| `/api/attr/getBlockAttrs` | R | 读块 `id` 的属性。 |
| `/api/attr/setBlockAttrs` | W | 在 `id` 上设置 `attrs`；自定义键需 `custom-` 前缀。 |
| `/api/query/sql` | R | 执行只读查询 `stmt`；把 Skill 用途限制为 `SELECT` 或 `WITH ... SELECT`。发布模式下不可用。 |
| `/api/sqlite/flushTransaction` | S | 强制刷新事务；避免常规使用。 |
| `/api/template/render` | R | 在文档上下文 `id` 中渲染模板文件 `path`；返回渲染后的 DOM。 |
| `/api/template/renderSprig` | R | 渲染 Sprig `template`；用于每日笔记路径预览。 |

### 工作区文件

| 端点 | 层级 | 用途与必需输入 |
|---|---:|---|
| `/api/file/getFile` | R | 读工作区 `path`；HTTP `200` 原始字节，`202` 返回 JSON 错误。 |
| `/api/file/readDir` | R | 列出目录 `path`。 |
| `/api/file/putFile` | W | multipart 创建/写入，用 `path`、`isDir`、`modTime`、`file`；绝不指向 `.sy`。 |
| `/api/file/renameFile` | W | 把工作区 `path` 重命名为 `newPath`；绝不用于笔记文档。 |
| `/api/file/removeFile` | D | 移除工作区 `path`；绝不当作语义化笔记删除。 |

### 导出、转换、通知与网络

| 端点 | 层级 | 用途与必需输入 |
|---|---:|---|
| `/api/export/exportMdContent` | R | 导出文档 `id`；返回可读 `hPath` 和 Markdown `content`。 |
| `/api/export/exportResources` | S | 从工作区 `paths` 创建临时 zip；可选 `name`。归档内同名会覆盖。 |
| `/api/convert/pandoc` | H | 在 `/temp/convert/pandoc/{dir}` 内运行 Pandoc `args`。校验文件名和参数；不要传 shell 语法。 |
| `/api/notification/pushMsg` | S | 显示本地消息 `msg`；可选 `timeout`。 |
| `/api/notification/pushErrMsg` | S | 显示本地错误消息 `msg`；可选 `timeout`。 |
| `/api/network/forwardProxy` | H | 发送外部请求。需明确目标和载荷授权；绝不隐式转发私有笔记内容。 |

### 系统

| 端点 | 层级 | 用途与必需输入 |
|---|---:|---|
| `/api/system/bootProgress` | R | 获取启动进度；`{}`。 |
| `/api/system/version` | R | 获取实时内核版本；`{}`。 |
| `/api/system/currentTime` | R | 获取内核时间（毫秒）；`{}`。 |

### 数据库

其余 16 个官方 `/api/av/*` 路由编录于 【参考：数据库 API】。任何数据库请求前先阅读该文件。

## 校验模式

- 笔记本配置：调用 `getNotebookConf` 并比较每个保留字段。
- 文档创建/移动/重命名：解析 `getHPathByID`，然后读根 Kramdown。
- 块插入/前置/追加/更新：捕获操作 ID，然后调用 `getBlockKramdown` 或 `getChildBlocks`。
- 属性：调用 `getBlockAttrs` 并只比较目标键。
- SQL：强制有界查询并检查行数；不要从返回数据推断变更。
- 文件写/重命名：用 `readDir` 或 `getFile`；不要通过原始文件工作流检查 `.sy`。
- 导出/Pandoc：校验返回路径仍在 `/temp` 下，只读预期输出。
- 数据库：重新渲染确切视图并比较行、字段、filter、sort 或布局状态；见 【参考：数据库 API】。

---

# 参考：数据库 API

## 模型与标识符

SiYuan 数据库即属性视图（attribute view）：

- `avID`：数据库定义 ID。
- `blockID`：嵌入该数据库的文档块；一个数据库可有多个块/镜像。
- `viewID`：表格、画廊或看板视图 ID。
- `itemID`：行 ID。绑定行通常等于绑定块 ID；游离行是独立生成的 ID。
- `keyID`：字段/列 ID。

绝不把一种 ID 类型替换成另一种。特别是 `/api/av/setAttributeViewBlockAttr` 需要来自 `data.view.rows[].id` 的 `itemID`，而非单元格值 ID。遗留的 `rowID` 参数已弃用，审阅的官方文档称将在 `2026-12-01` 后移除。

字段类型有 `block`、`text`、`number`、`date`、`select`、`mSelect`、`url`、`email`、`phone`、`mAsset`、`template`、`created`、`updated`、`checkbox`、`relation`、`rollup`、`lineNumber`。`block` 主键不能用 `addAttributeViewKey` 添加。

## 读工作流

1. `avID` 未知时用 `/api/av/searchAttributeView` 按名称查找数据库。
2. 用 `/api/av/renderAttributeView` 并设 `createIfNotExist:false` 渲染目标，作为真正只读调用。
3. 从响应捕获 `avID`、`blockID`、`viewID`、`rows[].id` 和 `columns[].id`。
4. 仅当需要原始字段值和所有视图定义时用 `/api/av/getAttributeView`；它不返回计算后的渲染行。
5. 用 `/api/av/getAttributeViewPrimaryKeyValues` 做分页主键查找。
6. 修改 filter 或 sort 配置前用 `/api/av/getAttributeViewFilterSort`。

读示例：

```bash
curl --silent --show-error --max-time 30 \
  -X POST "${SIYUAN_BASE_URL:-http://127.0.0.1:6806}/api/av/renderAttributeView" \
  -H "Authorization: Token ${SIYUAN_TOKEN}" \
  -H 'Content-Type: application/json' \
  -d '{
    "id":"AV_ID",
    "blockID":"BLOCK_ID",
    "viewID":"",
    "page":1,
    "pageSize":50,
    "query":"",
    "groupPaging":{},
    "createIfNotExist":false
  }'
```

对游离数据库，省略 `blockID`。镜像报告 `data.isMirror:true`，应视为只读。

## 写工作流

1. 渲染确切目标视图，快照目标行、字段、filters、sorts、group、布局和顺序。
2. 从该响应解析所有标识符；不要生成或猜测行 ID。
3. 构造改变一个有界属性的最小请求。
4. 调用一个变更端点并要求顶层 `code:0`。
5. 重新渲染或调用匹配的读端点。
6. 把归一化后的返回单元格值或完整配置与预期结果比较。
7. 部分成功时停止；不要盲目重试，因为行/字段创建可能已经成功。

## 单元格值形状

`/api/av/setAttributeViewBlockAttr` 接受形状随字段类型而定的部分 `value` 对象：

| 类型 | `value` 载荷 |
|---|---|
| `block` | `{"block":{"content":"First row","id":"BOUND_BLOCK_ID"},"isDetached":false}` |
| `text` | `{"text":{"content":"Some text"}}` |
| `number` | `{"number":{"content":42,"isNotEmpty":true}}`；清空用 `isNotEmpty:false` |
| `date` | `{"date":{"content":1676042451000,"isNotEmpty":true}}`（毫秒） |
| `select` | `{"mSelect":[{"content":"Done","color":"1"}]}`，至多一个选项 |
| `mSelect` | `{"mSelect":[{"content":"A","color":"1"},{"content":"B","color":"2"}]}` |
| `url` | `{"url":{"content":"https://example.com"}}` |
| `email` | `{"email":{"content":"a@example.com"}}` |
| `phone` | `{"phone":{"content":"1234567890"}}` |
| `checkbox` | `{"checkbox":{"checked":true}}` |

在请求 `value` 中包含外层 `type`，例如 `{"type":"number","number":...}`。响应返回完全归一化的 `data.value`；用它做校验。

## 端点目录

### 读

| 端点 | 用途与关键输入 |
|---|---|
| `/api/av/searchAttributeView` | 搜索数据库名；`keyword`，可选 `excludes`。结果分组数据库与子视图。 |
| `/api/av/renderAttributeView` | 渲染计算视图；`id`，可选 `blockID`、`viewID`、分页/查询。只读设 `createIfNotExist:false`。 |
| `/api/av/getAttributeView` | 按 `id` 获取原始完整数据库定义；计算行用 render。 |
| `/api/av/getAttributeViewPrimaryKeyValues` | 按 `id`、`keyword`、`page`、`pageSize` 分页主键值。 |
| `/api/av/getAttributeViewFilterSort` | 用数据库 `id` 和嵌入 `blockID` 读当前 `filters` 和 `sorts`。 |

### 写

| 端点 | 层级 | 用途与关键输入 |
|---|---:|---|
| `/api/av/setAttributeViewBlockAttr` | W | 用 `avID`、`keyID`、行 `itemID`、类型化 `value` 设置一个单元格。 |
| `/api/av/addAttributeViewBlocks` | W | 用 `avID`、所属 `blockID`、可选 view/group/position 和 `srcs` 添加绑定或游离行。重新渲染以获取新行 ID。 |
| `/api/av/removeAttributeViewBlocks` | D | 移除 `srcIDs` 中的行 ID；游离行被删除，绑定块只是被解绑。 |
| `/api/av/changeAttrViewLayout` | W | 把当前布局改为 `table`、`gallery` 或 `kanban`；`avID`、`blockID`、`layoutType`。 |
| `/api/av/setAttrViewGroup` | W | 用 `avID`、`blockID`、`group` 替换/清除看板分组。空 `group.field` 清除分组。 |
| `/api/av/setAttrViewFilters` | W | 用 `avID`、`blockID`、`data` 替换完整 filter 数组；`[]` 清除全部。 |
| `/api/av/setAttrViewSorts` | W | 用 `avID`、`blockID`、`data` 替换完整 sort 数组；`[]` 清除全部。 |
| `/api/av/addAttributeViewKey` | W | 用有效生成的 `keyID`、`keyName`、`keyType`、图标和 `previousKeyID` 添加字段。 |
| `/api/av/removeAttributeViewKey` | D | 删除 `keyID` 及所有存储值；`removeRelationDest:true` 同时移除目标反向关联。 |
| `/api/av/sortAttributeViewKey` | W | 用 `keyID`、`previousKeyID` 在所有视图全局重排字段。 |
| `/api/av/sortAttributeViewViewKey` | W | 仅在 `viewID` 内重排字段；不改变全局字段顺序。 |

## 关键变更规则

- `setAttrViewFilters` 和 `setAttrViewSorts` 替换完整数组。始终先读取并保留无关规则。
- filter 组可用 `combination: "and"|"or"` 递归嵌套；叶节点用 `column`、`operator`、`value` 和可选 `relativeDate`。
- 支持的 filter 运算符包括 `=`、`!=`、比较、`Contains`、`Does not contains`、`Is empty`、`Is not empty`、`Starts with`、`Ends with`、`Is between`、`Is true`、`Is false`。
- 移除数据库项不等于删除绑定的文档块；报告该行是游离还是绑定。
- 移除字段会删除其值。即使数据库仍在，也视为破坏性。
- `addAttributeViewKey.keyID` 必须是由 `Lute.NewNodeID()` 或等价的经批准运行时生成器生成的有效 SiYuan 节点 ID；不要手动编造时间戳字符串。
- `addAttributeViewBlocks` 返回 `data:null`；设置单元格前重新渲染以发现新项 ID。
- 活动 filter 或 group 可能在 `rowCount` 非零时产生空渲染行。不要在未检查视图规则时断定数据库为空。
- 对布局/分组变更，服务器返回重新渲染的视图；第二次变更前仍需重读。

---

# 参考：Inbox

## 状态与边界

- 最后校验的 CLI 版本：`3.7.1`（`2026-07-06`）
- HTTP 路由最后校验的内核版本：`3.7.0`（`2026-07-02`）
- CLI 提供 `inbox list`、`inbox get` 和 `inbox convert`。
- Inbox 存储云端速记（cloud shorthands）。本地内核用当前登录的 SiYuan 账号把请求代理到 SiYuan 云服务。
- Inbox 条目不是本地文档，不能通过工作区 SQL、文档列表或 `daily note` 路径发现。
- 这些路由存在于内核源码，但未在公开 `API.md` 中文档化。视为内部路由，升级后重新核对实时行为。

## 前置条件

1. 遵循 `SKILL.md` 和 【参考：CLI】 中的 CLI 校验工作流。
2. 检测已有内核监听器。桌面默认常为 `127.0.0.1:6806`，但有证据显示其它值时不要假设端口。
3. 只读探测内核：

```bash
curl --silent --show-error --max-time 10 \
  -X POST http://127.0.0.1:6806/api/system/version \
  -H 'Content-Type: application/json' \
  -d '{}'
```

4. 若本地访问被执行沙箱阻止，请求只读 localhost 访问的批准。SiYuan 桌面端已在监听时不要启动第二个内核。
5. 若启用了内核访问认证，添加 `Authorization: Token ${SIYUAN_TOKEN}`。绝不在命令输出或总结中打印、持久化或包含令牌。
6. 预期云请求需要已登录的 SiYuan 账号和可用网络。

## CLI 工作流

读取列表与单条详情：

```bash
siyuan inbox list --page 1 -w "$SIYUAN_WORKSPACE" --format json
siyuan inbox get --id shorthand-id -w "$SIYUAN_WORKSPACE" --format json
```

转换条目会写入本地笔记。先 dry-run，并在未获得删除授权时显式关闭来源删除：

```bash
siyuan inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false --dry-run -w "$SIYUAN_WORKSPACE" --format json
siyuan inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false -w "$SIYUAN_WORKSPACE" --format json
```

3.7.1 中 `--remove-after` 的默认值是 `true`。未经用户明确授权，不要省略该选项，也不要传 `true`。

## HTTP 只读回退

当实时 CLI 不可用、缺少所需字段或响应不足以完成只读整理时，再使用以下内核路由。

列出一页：

```bash
curl --silent --show-error --max-time 30 \
  -X POST http://127.0.0.1:6806/api/inbox/getShorthands \
  -H 'Content-Type: application/json' \
  -d '{"page":1}'
```

请求体：

```json
{"page": 1}
```

观察到的列表响应是嵌套的，因为本地内核返回云端结果：

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "code": 0,
    "msg": "",
    "data": {
      "pagination": {
        "paginationPageCount": 2,
        "paginationPageNums": [1, 2],
        "paginationRecordCount": 24
      },
      "shorthands": []
    }
  }
}
```

从第 1 页读 `paginationPageCount` 并取回所有剩余页。保持输出紧凑，因为 `shorthandContent` 可能包含完整剪藏文章和大段渲染 HTML：

```bash
curl --silent --show-error --max-time 30 \
  -X POST http://127.0.0.1:6806/api/inbox/getShorthands \
  -H 'Content-Type: application/json' \
  -d '{"page":1}' |
  jq '.data.data | {
    pagination,
    items: [.shorthands[] | {
      oId,
      hCreated,
      shorthandTitle,
      shorthandURL,
      shorthandDesc,
      shorthandMdPreview: ((.shorthandMd // "")
        | gsub("[\\r\\n\\t]+"; " ")
        | .[0:500])
    }]
  }'
```

按 `oId` 读单条：

```bash
curl --silent --show-error --max-time 30 \
  -X POST http://127.0.0.1:6806/api/inbox/getShorthand \
  -H 'Content-Type: application/json' \
  -d '{"id":"ENTRY_ID"}'
```

请求体：

```json
{"id": "ENTRY_ID"}
```

详情对象直接返回在顶层 `data`。常见字段：

- `oId`：Inbox 条目 ID 和创建时间来源。
- `hCreated`：格式化的创建时间。
- `shorthandTitle`：捕获的标题或日期。
- `shorthandURL`：来源 URL，可能为空。
- `shorthandDesc`：简短描述。
- `shorthandMd`：Markdown 源；分析和导入优先用它。
- `shorthandContent`：渲染 HTML；仅当 Markdown 不足时用。
- `shorthandFrom`：捕获来源代码。

纯链接条目可能 Markdown 为空。仅图片条目分类前需要视觉检查。

## 整理工作流

1. 取回所有页并清点元数据，不倾倒完整文章正文。
2. 脱敏类凭证值，如订阅 URL、许可证密钥、令牌和私有内部链接。
3. 把条目分类为：合并到现有笔记、新建笔记、纯链接引用、敏感迁移、人工复核、删除候选。
4. 通过 CLI 搜索本地笔记，提出任何写入前先读候选目标。
5. 当目标选择重要时呈现逐条路由计划。
6. 仅在用户请求写入操作后才追加或新建笔记；通过 CLI 校验每次写入。
7. 除非用户单独且明确请求删除，不要移除已导入的 Inbox 条目。

## 破坏性端点

`POST /api/inbox/removeShorthands` 接受 `{"ids":["ENTRY_ID"]}`。它永久移除云端 Inbox 条目。`siyuan inbox convert` 的 `--remove-after=true` 具有同类删除后果。不要为查找、整理规划、去重或作为导入后自动清理步骤使用它们。

---

# 许可证与第三方来源

本技能的原创工作流与说明按 MIT License 发布，完整条款见 <https://github.com/ZyoHuang/siyuan-notes/blob/main/LICENSE>。

本技能基于公开接口文档、本地 CLI 帮助、运行时响应和用户指南整理 SiYuan 兼容性信息。SiYuan 上游采用 AGPL-3.0；若任何片段依法构成对上游材料的复制或改编，该部分仍受上游许可证约束。详情见 <https://github.com/ZyoHuang/siyuan-notes/blob/main/THIRD_PARTY_NOTICES.md>。本项目不是 SiYuan 官方项目。
