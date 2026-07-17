# 写入、追加与校验工作流

用于把内容记录、整理或追加到 SiYuan 本地笔记。先读 `references/cli.md`、`references/search-read.md` 和当前平台 reference。

## 目标选择

1. 写入前必须搜索并阅读候选文档。
2. 优先追加到最具体的现有笔记，而不是泛化父级文档。
3. 仅当没有合适目标且用户确实要求记录内容时才新建文档。
4. 若多个目标同样合理且会影响长期组织结构，先问一个简洁确认问题。

## 内容组织

- 保留现有文本，追加结构化 Markdown。
- 去重已存在的内容。
- 区分已验证事实、用户经验、推断和建议。
- 保留对话中有用的来源链接。
- 有用时用 SiYuan 块引用关联现有笔记：`((block-id "锚文本"))`。

## 安全写入

用实时帮助确认当前 CLI 参数。若支持 dry-run，先 dry-run：

```text
<binary> block append --parent block-id --file "<temporary-markdown>" --dry-run -w "<workspace>" --format json
<binary> block append --parent block-id --file "<temporary-markdown>" -w "<workspace>" --format json
```

仅在需要时新建文档：

```text
<binary> document create --notebook notebook-id --title "Title" --path /parent/path --markdown "Initial content" --dry-run -w "<workspace>" --format json
<binary> document create --notebook notebook-id --title "Title" --path /parent/path --markdown "Initial content" -w "<workspace>" --format json
```

通过当前平台 reference 的变量和参数传递方式替换占位符，不要把 `<binary>`、`<workspace>` 或 `<temporary-markdown>` 原样传给 CLI。当工作区或 SiYuan 运行时元数据在当前可写项目之外时，请求环境所需批准。绝不要直接编辑 `.sy` 文件。

## 校验与清理

写入后搜索一个唯一标题或短语并确认：

- `rootID` 匹配目标笔记；
- `hPath` 匹配目标路径；
- 新块具有预期的标题或段落类型；
- `markdown` 保留代码段、链接、表格和内部块引用。

只删除本次操作创建的临时文件。部分成功或返回含糊时停止，不要盲目重试。
