# 参考：CLI

## 版本契约

- 最后校验的 CLI 版本：`3.7.2`
- 校验日期：`2026-07-17`
- 官方契约：CLI 直接访问工作区，不需要启动内核服务器。
- 官方工作区环境变量：`SIYUAN_WORKSPACE_PATH`

使用本章节任何命令前，先完整读取当前平台 reference：

- Windows：`references/platform-windows.md`
- macOS：`references/platform-macos.md`
- Linux：`references/platform-linux.md`

然后用平台 reference 解析出的绝对路径执行：

```text
<binary> --version
<binary> --help
<binary> <command> --help
<binary> <command> <subcommand> --help
```

`<binary>` 的绝对路径具有权威性。本章节示例描述 3.7.2 版本，可能随升级过时；若已安装版本不同，从实时帮助重建命令。绝不猜测被重命名的命令或参数。

## 本地默认值

按设备解析，不要硬编码：

- 二进制：按当前平台 reference 的发现顺序解析。
- 工作区：优先使用显式 `--workspace` / `-w`；也可使用官方环境变量 `SIYUAN_WORKSPACE_PATH`，或从 `workspace list --format json` 读取注册的工作区。
- 主笔记本及其 ID：从 `notebook list -w "<workspace>" --format json` 读取。

把这些当作默认值而非永久事实。命令失败或用户提到其它工作区时重新校验。以下 `<workspace>` 是当前平台的工作区绝对路径占位符；执行时按平台 reference 正确传递，不要把尖括号原样传给 CLI。

## 3.7.2 版本读命令示例

3.7.2 CLI 提供 `inbox list`、`inbox get` 和 `inbox convert`。处理 `Inbox` 或 `收集箱` 请求前先阅读 【参考：Inbox】。Inbox 是云端速记，不属于本地文档搜索或 SQL 索引。

对于 CLI 缺失或在 HTTP 上结构更丰富的官方内核能力，阅读 【参考：内核 API】。对于 `/api/av/*` 数据库操作，同时阅读 【参考：数据库 API】。当两个入口都支持同一操作时优先用 CLI，因为实时帮助和 dry-run 让目标校验更安全。

```text
<binary> workspace list --format json
<binary> notebook list -w "<workspace>" --format json
<binary> search "keyword" --method 0 --page-size 100 -w "<workspace>" --format json
<binary> search "term1|term2" --method 3 --page-size 100 -w "<workspace>" --format json
<binary> block batch-kramdown --ids id1,id2 -w "<workspace>" --format json
<binary> block kramdown --id block-id -w "<workspace>" --format json
<binary> inbox list --page 1 -w "<workspace>" --format json
<binary> inbox get --id shorthand-id -w "<workspace>" --format json
```

最常用的搜索方法：

- `--method 0`：关键词搜索。
- `--method 3`：对若干聚焦备选项做正则搜索。
- 需要按文档分组更易检查时加 `--group-by 1`。

## 3.7.2 版本安全写示例

追加到现有文档或块：

```text
<binary> block append --parent block-id --file "<temporary-markdown>" --dry-run -w "<workspace>" --format json
<binary> block append --parent block-id --file "<temporary-markdown>" -w "<workspace>" --format json
```

仅在需要时新建文档：

```text
<binary> document create --notebook notebook-id --title "Title" --path /parent/path --markdown "Initial content" --dry-run -w "<workspace>" --format json
<binary> document create --notebook notebook-id --title "Title" --path /parent/path --markdown "Initial content" -w "<workspace>" --format json
```

将 Inbox 条目转换为本地文档时，3.7.2 的 `--remove-after` 默认值为 `true`。除非用户明确要求同时删除云端来源，否则 dry-run 和实际调用都必须显式传 `--remove-after=false`：

```text
<binary> inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false --dry-run -w "<workspace>" --format json
<binary> inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false -w "<workspace>" --format json
```

## 路由锚点

不要硬编码笔记标题、路径或块 ID。始终先搜索、读候选文档，再用该设备工作区解析出的 `rootID` / `hPath`。

## 校验模式

写入后，搜索一个唯一标题或短语并确认：

- `rootID` 匹配目标笔记；
- `hPath` 匹配目标路径；
- 新块具有预期的标题/段落类型；
- `markdown` 保留了代码段、链接、表格和内部块引用。

CLI 可能在平台对应的用户配置目录写入日志和工作区注册文件。这是工具运行时元数据，不是笔记内容；即使只读操作也可能需要执行环境批准。
