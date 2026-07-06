# SiYuan database API reference

## Contents

- [Model and identifiers](#model-and-identifiers)
- [Read workflow](#read-workflow)
- [Write workflow](#write-workflow)
- [Cell value shapes](#cell-value-shapes)
- [Endpoint catalog](#endpoint-catalog)
- [Critical mutation rules](#critical-mutation-rules)

## Model and identifiers

SiYuan databases are attribute views:

- `avID`: database definition ID.
- `blockID`: document block embedding the database; one database can have multiple blocks/mirrors.
- `viewID`: table, gallery, or kanban view ID.
- `itemID`: row ID. For a bound row it usually equals the bound block ID; for a detached row it is an independent generated ID.
- `keyID`: field/column ID.

Never substitute one ID type for another. In particular, `/api/av/setAttributeViewBlockAttr` requires `itemID` from `data.view.rows[].id`, not a cell-value ID. The legacy `rowID` parameter is deprecated and the reviewed official document says it will be removed after `2026-12-01`.

Field types are `block`, `text`, `number`, `date`, `select`, `mSelect`, `url`, `email`, `phone`, `mAsset`, `template`, `created`, `updated`, `checkbox`, `relation`, `rollup`, and `lineNumber`. The `block` primary key cannot be added with `addAttributeViewKey`.

## Read workflow

1. Find a database by name with `/api/av/searchAttributeView` when `avID` is unknown.
2. Render the target using `/api/av/renderAttributeView` with `createIfNotExist:false` for a genuinely read-only call.
3. Capture `avID`, `blockID`, `viewID`, `rows[].id`, and `columns[].id` from the response.
4. Use `/api/av/getAttributeView` only when raw field values and all view definitions are needed; it does not return computed rendered rows.
5. Use `/api/av/getAttributeViewPrimaryKeyValues` for paginated primary-key lookup.
6. Use `/api/av/getAttributeViewFilterSort` before changing filter or sort configuration.

Read example:

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

For a detached database, omit `blockID`. Mirrors report `data.isMirror:true` and should be treated as read-only.

## Write workflow

1. Render the exact target view and snapshot the target row, fields, filters, sorts, group, layout, and ordering.
2. Resolve all identifiers from that response; do not generate or guess row IDs.
3. Construct the smallest request that changes one bounded property.
4. Call one mutation endpoint and require top-level `code:0`.
5. Re-render or call the matching read endpoint.
6. Compare the normalized returned cell value or complete configuration with the intended result.
7. Stop on partial success; do not retry blindly because row/field creation may already have succeeded.

## Cell value shapes

`/api/av/setAttributeViewBlockAttr` accepts a partial `value` object whose shape follows the field type:

| Type | `value` payload |
|---|---|
| `block` | `{"block":{"content":"First row","id":"BOUND_BLOCK_ID"},"isDetached":false}` |
| `text` | `{"text":{"content":"Some text"}}` |
| `number` | `{"number":{"content":42,"isNotEmpty":true}}`; clear with `isNotEmpty:false` |
| `date` | `{"date":{"content":1676042451000,"isNotEmpty":true}}` in milliseconds |
| `select` | `{"mSelect":[{"content":"Done","color":"1"}]}` with at most one option |
| `mSelect` | `{"mSelect":[{"content":"A","color":"1"},{"content":"B","color":"2"}]}` |
| `url` | `{"url":{"content":"https://example.com"}}` |
| `email` | `{"email":{"content":"a@example.com"}}` |
| `phone` | `{"phone":{"content":"1234567890"}}` |
| `checkbox` | `{"checkbox":{"checked":true}}` |

Include the outer `type` in the request `value`, for example `{"type":"number","number":...}`. The response returns a fully normalized `data.value`; use it for verification.

## Endpoint catalog

### Reads

| Endpoint | Purpose and key input |
|---|---|
| `/api/av/searchAttributeView` | Search database names; `keyword`, optional `excludes`. Results group database and child views. |
| `/api/av/renderAttributeView` | Render computed view; `id`, optional `blockID`, `viewID`, pagination/query. Set `createIfNotExist:false` for read-only. |
| `/api/av/getAttributeView` | Get raw full database definition by `id`; use render for computed rows. |
| `/api/av/getAttributeViewPrimaryKeyValues` | Paginate primary-key values by `id`, `keyword`, `page`, `pageSize`. |
| `/api/av/getAttributeViewFilterSort` | Read current `filters` and `sorts` using database `id` and embedding `blockID`. |

### Writes

| Endpoint | Tier | Purpose and key input |
|---|---:|---|
| `/api/av/setAttributeViewBlockAttr` | W | Set one cell using `avID`, `keyID`, row `itemID`, typed `value`. |
| `/api/av/addAttributeViewBlocks` | W | Add bound or detached rows using `avID`, owning `blockID`, optional view/group/position, and `srcs`. Re-render to get new row IDs. |
| `/api/av/removeAttributeViewBlocks` | D | Remove row IDs in `srcIDs`; detached rows are deleted, bound blocks are only unbound. |
| `/api/av/changeAttrViewLayout` | W | Change current layout to `table`, `gallery`, or `kanban`; `avID`, `blockID`, `layoutType`. |
| `/api/av/setAttrViewGroup` | W | Replace/clear kanban grouping with `avID`, `blockID`, `group`. Empty `group.field` clears grouping. |
| `/api/av/setAttrViewFilters` | W | Replace the complete filter array using `avID`, `blockID`, `data`; `[]` clears all. |
| `/api/av/setAttrViewSorts` | W | Replace the complete sort array using `avID`, `blockID`, `data`; `[]` clears all. |
| `/api/av/addAttributeViewKey` | W | Add field with valid generated `keyID`, `keyName`, `keyType`, icon, and `previousKeyID`. |
| `/api/av/removeAttributeViewKey` | D | Delete `keyID` and every stored value; `removeRelationDest:true` also removes the destination back-relation. |
| `/api/av/sortAttributeViewKey` | W | Reorder a field globally across every view using `keyID`, `previousKeyID`. |
| `/api/av/sortAttributeViewViewKey` | W | Reorder a field only inside `viewID`; does not change global field order. |

## Critical mutation rules

- `setAttrViewFilters` and `setAttrViewSorts` replace complete arrays. Always read and preserve unrelated rules first.
- Filter groups can be recursively nested with `combination: "and"|"or"`; leaf nodes use `column`, `operator`, `value`, and optional `relativeDate`.
- Supported filter operators include `=`, `!=`, comparisons, `Contains`, `Does not contains`, `Is empty`, `Is not empty`, `Starts with`, `Ends with`, `Is between`, `Is true`, and `Is false`.
- Removing a database item is not equivalent to deleting a bound document block; report whether the row was detached or bound.
- Removing a field deletes its values. Treat it as destructive even if the database remains.
- `addAttributeViewKey.keyID` must be a valid SiYuan node ID generated by `Lute.NewNodeID()` or an equivalent approved runtime generator; do not invent a timestamp string manually.
- `addAttributeViewBlocks` returns `data:null`; re-render to discover new item IDs before setting cells.
- Active filters or groups can produce empty rendered rows even when `rowCount` is non-zero. Do not conclude the database is empty without inspecting view rules.
- For layout/group mutations, the server returns a re-rendered view; still re-read before a second mutation.
