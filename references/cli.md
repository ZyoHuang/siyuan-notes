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
