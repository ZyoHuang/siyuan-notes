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
