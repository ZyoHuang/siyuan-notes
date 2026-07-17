# 参考：Inbox

## 状态与边界

- 最后校验的 CLI 版本：`3.7.2`（`2026-07-17`）
- HTTP 路由最后校验的实时内核版本：`3.7.2`（`2026-07-17`）
- CLI 提供 `inbox list`、`inbox get` 和 `inbox convert`。
- Inbox 存储云端速记（cloud shorthands）。本地内核用当前登录的 SiYuan 账号把请求代理到 SiYuan 云服务。
- Inbox 条目不是本地文档，不能通过工作区 SQL、文档列表或 `daily note` 路径发现。
- 这些 HTTP 路由存在于内核源码，但未在公开 `API.md` 中文档化。视为内部路由，升级后重新核对实时行为。

## 前置条件

1. 遵循 `SKILL.md` 和 【参考：CLI】 的平台分流与 CLI 校验工作流。
2. CLI 直接访问工作区；使用 `inbox list/get/convert` 时不要求 6806 内核运行。
3. 只有 CLI 不可用、缺少所需字段或响应不足以完成只读整理时，才按当前平台 reference 探测已有内核并调用 HTTP 回退。
4. 本地访问若被执行沙箱阻止，请求只读 localhost 访问批准。SiYuan 桌面端已监听时不要启动第二个内核。
5. 内核启用访问认证时添加 `Authorization: Token <SIYUAN_TOKEN>`。绝不在命令输出或总结中打印、持久化或包含令牌。
6. 云请求需要已登录的 SiYuan 账号和可用网络。

## CLI 工作流

读取列表与单条详情：

```text
<binary> inbox list --page 1 -w "<workspace>" --format json
<binary> inbox get --id shorthand-id -w "<workspace>" --format json
```

转换条目会写入本地笔记。先 dry-run，并在未获得删除授权时显式关闭来源删除：

```text
<binary> inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false --dry-run -w "<workspace>" --format json
<binary> inbox convert --ids shorthand-id --notebook notebook-id --path /Inbox --remove-after=false -w "<workspace>" --format json
```

3.7.2 中 `--remove-after` 的默认值是 `true`。未经用户明确授权，不要省略该选项，也不要传 `true`。

## HTTP 只读回退

按照当前平台 reference 的 HTTP 模式发送请求，同时检查 HTTP 状态和 SiYuan 顶层 `code`。

列出一页：

```http
POST <base-url>/api/inbox/getShorthands
Content-Type: application/json

{"page":1}
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

同时检查本地信封 `code` 和嵌套云端信封 `data.code`。从第 1 页读 `data.data.pagination.paginationPageCount`，再取回所有剩余页。按当前平台 reference 的 JSON 方式把每条记录投影为：

- `oId`
- `hCreated`
- `shorthandTitle`
- `shorthandURL`
- `shorthandDesc`
- `shorthandMdPreview`：把 `shorthandMd` 换行/制表符压成空格后截取前 500 字符

不要把 `shorthandContent` 或完整剪藏文章发回模型。

按 `oId` 读单条：

```http
POST <base-url>/api/inbox/getShorthand
Content-Type: application/json

{"id":"ENTRY_ID"}
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
