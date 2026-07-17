# 只读搜索与阅读工作流

用于查找、回忆、比较、总结本地 SiYuan 笔记。先读 `references/cli.md` 和当前平台 reference，并完成实时 CLI 校验。

## 检索策略

1. 从用户问题提炼 2 到 5 个聚焦关键词，包含产品名、技术名、中文近义词和可能的标题片段。
2. 已知路径或标题时，先搜索最具体片段；不要从 `Resources`、`AIGC` 这类宽泛词开始。
3. 优先查 `NodeDocument` 或文档分组结果，确认 `hPath`、`rootID`、标题和更新时间，再读正文。
4. 候选少时用 `block batch-kramdown`；目标明确时用 `block kramdown --id <rootID>`。
5. 只回答问题所需内容；私密笔记、长剪藏和无关命中不要展开。

## 输出裁剪

搜索 JSON 往往很宽。默认使用小结果页：

```text
<binary> search "keyword" --page-size 20 -w "<workspace>" --format json
```

按当前平台 reference 的 JSON 解析方式投影后再交给模型阅读。优先保留：

- 汇总：`matchedBlockCount`、`matchedRootCount`、`pageCount`
- 块：`id`、`rootID`、`parentID`、`hPath`、`type`、`subType`
- 内容：`markdown` 或必要的 `fcontent`
- 时间：`ial.updated`

标题明确时可先限制文档类型：

```text
<binary> search "Exact Title" --type document --page-size 20 -w "<workspace>" --format json
```

需要多关键词备选时使用正则，但保持结果页小：

```text
<binary> search "term1|term2|term3" --method 3 --page-size 20 -w "<workspace>" --format json
```

## 阅读与回答

- 读取候选文档前先说明检索路径；读取后只引用必要片段。
- 对概念判断类问题，区分用户原文、笔记来源内容、模型推断和建议改写。
- 对“之前记过吗”类问题，给出命中的笔记路径和一句结论；不要输出完整笔记。
- 若搜索结果不足，说明已查关键词和范围，再给出不确定性。
