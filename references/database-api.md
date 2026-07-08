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
