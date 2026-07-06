# SiYuan Inbox 参考

## 目录

- [状态与边界](#状态与边界)
- [前置条件](#前置条件)
- [CLI 工作流](#cli-工作流)
- [HTTP 只读回退](#http-只读回退)
- [整理工作流](#整理工作流)
- [破坏性端点](#破坏性端点)

## 状态与边界

- 最后校验的 CLI 版本：`3.7.1`（`2026-07-06`）
- HTTP 路由最后校验的内核版本：`3.7.0`（`2026-07-02`）
- CLI 提供 `inbox list`、`inbox get` 和 `inbox convert`。
- Inbox 存储云端速记（cloud shorthands）。本地内核用当前登录的 SiYuan 账号把请求代理到 SiYuan 云服务。
- Inbox 条目不是本地文档，不能通过工作区 SQL、文档列表或 `daily note` 路径发现。
- 这些路由存在于内核源码，但未在公开 `API.md` 中文档化。视为内部路由，升级后重新核对实时行为。

## 前置条件

1. 遵循 `SKILL.md` 和 [cli.md](cli.md) 中的 CLI 校验工作流。
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
