# 平台执行参考：Linux

本文件只适用于 Linux。使用 POSIX shell 语法调用 SiYuan CLI；不要混入 PowerShell 或 macOS 应用包路径。

## CLI 发现顺序

先查 `PATH`：

```bash
SIYUAN_BIN="$(command -v siyuan 2>/dev/null || true)"

if [ -z "$SIYUAN_BIN" ]; then
  printf '%s\n' '未找到 SiYuan CLI。请确认已安装桌面版，并参考下方官方符号链接方式。' >&2
  exit 1
fi

"$SIYUAN_BIN" --version
```

始终引用解析出的路径：

```bash
"$SIYUAN_BIN" --help
"$SIYUAN_BIN" workspace --help
"$SIYUAN_BIN" workspace list --format json
```

Linux 安装目录取决于发行包或用户选择。不要猜测 AppImage、deb、rpm 或其它分发方式的安装路径。

## 工作区

官方环境变量名是 `SIYUAN_WORKSPACE_PATH`：

```bash
export SIYUAN_WORKSPACE_PATH="/home/you/SiYuan"
"$SIYUAN_BIN" notebook list --format json
```

也可显式传参，显式参数优先且更容易审计：

```bash
workspace="/home/you/SiYuan"
"$SIYUAN_BIN" notebook list -w "$workspace" --format json
```

工作区未知时先执行 `workspace list --format json`，不要猜默认目录。

## JSON 裁剪

若 `jq` 已存在，用它投影必要字段；不要为了本 Skill 安装 `jq`：

```bash
"$SIYUAN_BIN" search "keyword" --page-size 20 -w "$workspace" --format json |
  jq '{
    matchedBlockCount,
    matchedRootCount,
    pageCount,
    blocks: [.blocks[:20][] | {
      id, rootID, parentID, hPath, type, subType,
      markdown, fcontent,
      updated: .ial.updated
    }]
  }'
```

若 `jq` 不可用，保留小页数并由运行时解析 JSON，只投影同一组字段；不要输出原始大 JSON，也不要擅自安装全局工具。

## 按需调用内核 HTTP API

CLI-only 工作流跳过本节。只有共享 reference 明确要求 HTTP API 时，才探测已运行的内核；不要为了探测而启动第二个 SiYuan 进程。

```bash
base_url="${SIYUAN_BASE_URL:-http://127.0.0.1:6806}"
base_url="${base_url%/}"

if [ -n "${SIYUAN_TOKEN:-}" ]; then
  response="$(
    curl --fail --silent --show-error --max-time 10 \
      -X POST "$base_url/api/system/version" \
      -H "Authorization: Token $SIYUAN_TOKEN" \
      -H 'Content-Type: application/json' \
      -d '{}'
  )"
else
  response="$(
    curl --fail --silent --show-error --max-time 10 \
      -X POST "$base_url/api/system/version" \
      -H 'Content-Type: application/json' \
      -d '{}'
  )"
fi
```

检查响应顶层 `code`。若 `jq` 可用：

```bash
printf '%s' "$response" | jq -e '.code == 0' >/dev/null
```

`jq` 不可用时由运行时解析响应并执行同一检查。不要使用 `set -x`，不要回显 `SIYUAN_TOKEN`，也不要把带认证头的完整命令写入日志。

## 官方符号链接方式

只负责诊断；未经用户明确授权，不执行本节变更。

SiYuan 官方 CLI 指南要求 Linux 安装后创建符号链接：

```bash
ln -s <install-dir>/resources/kernel/SiYuan-Kernel /usr/local/bin/siyuan
```

创建后在新 shell 中用 `command -v siyuan` 和 `siyuan --version` 验证。若目标已存在，先检查它指向何处，不要覆盖未知文件或链接。

来源：[SiYuan v3.7.2 内置 Command Line Interface 指南](https://github.com/siyuan-note/siyuan/blob/v3.7.2/app/guide/20210808180117-6v0mkxr/20200923234011-ieuun1p/20210808180303-xaduj2o/20260612192252-sgw2paw.sy)。
