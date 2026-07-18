# 平台执行参考：Windows

本文件只适用于 Windows。使用 PowerShell 调用 SiYuan CLI；不要把 macOS 的 shell 语法混入命令。

## CLI 发现顺序

SiYuan 官方安装包会把内核目录加入 `PATH` 并创建 `siyuan.exe`。Microsoft Store 版受 MSIX 限制，不能修改 `PATH`；可以动态解析当前包位置。

```powershell
$SiYuan = $null
$command = Get-Command siyuan -ErrorAction SilentlyContinue
if ($command) {
    $SiYuan = $command.Path
}

if (-not $SiYuan) {
    $candidate = Join-Path $env:LOCALAPPDATA 'Programs\SiYuan\resources\kernel\siyuan.exe'
    if (Test-Path -LiteralPath $candidate -PathType Leaf) {
        $SiYuan = (Resolve-Path -LiteralPath $candidate).Path
    }
}

if (-not $SiYuan) {
    $package = Get-AppxPackage '*SiYuan*' -ErrorAction SilentlyContinue |
        Select-Object -First 1
    if ($package) {
        $candidate = Join-Path $package.InstallLocation 'app\resources\kernel\SiYuan-Kernel.exe'
        if (Test-Path -LiteralPath $candidate -PathType Leaf) {
            $SiYuan = (Resolve-Path -LiteralPath $candidate).Path
        }
    }
}

if (-not $SiYuan) {
    throw '未找到 SiYuan CLI。请确认已安装桌面版，并参考下方官方 PATH/shim 处理方式。'
}

& $SiYuan --version
```

始终用调用运算符 `&` 和解析出的绝对路径执行后续命令：

```powershell
& $SiYuan --help
& $SiYuan workspace --help
& $SiYuan workspace list --format json
```

不要把可执行文件路径拼进字符串后交给 `Invoke-Expression`。

## 工作区

官方环境变量名是 `SIYUAN_WORKSPACE_PATH`：

```powershell
$env:SIYUAN_WORKSPACE_PATH = 'D:\Path\To\Workspace'
& $SiYuan notebook list --format json
```

也可显式传参，显式参数优先且更容易审计：

```powershell
$workspace = 'D:\Path\To\Workspace'
& $SiYuan notebook list -w $workspace --format json
```

路径作为单独参数传给 `&`，不要手工增加嵌套引号。工作区未知时先执行 `workspace list --format json`，不要猜默认目录。

## JSON 裁剪

PowerShell 原生使用 `ConvertFrom-Json`，不要求安装 `jq`：

```powershell
$result = & $SiYuan search 'keyword' --page-size 20 -w $workspace --format json |
    ConvertFrom-Json

$summary = [pscustomobject]@{
    matchedBlockCount = $result.matchedBlockCount
    matchedRootCount  = $result.matchedRootCount
    pageCount         = $result.pageCount
    blocks            = @($result.blocks | Select-Object -First 20 `
        id, rootID, parentID, hPath, type, subType, markdown, fcontent,
        @{Name = 'updated'; Expression = { $_.ial.updated }})
}
```

模型只读取 `$summary` 所需字段；不要输出原始大 JSON。

## 按需调用内核 HTTP API

CLI-only 工作流跳过本节。只有共享 reference 明确要求 HTTP API 时，才探测已运行的内核；不要为了探测而启动第二个 SiYuan 进程。

```powershell
$baseUrl = if ($env:SIYUAN_BASE_URL) {
    $env:SIYUAN_BASE_URL.TrimEnd('/')
} else {
    'http://127.0.0.1:6806'
}

$headers = @{}
if ($env:SIYUAN_TOKEN) {
    $headers.Authorization = "Token $($env:SIYUAN_TOKEN)"
}

$response = Invoke-RestMethod -Method Post `
    -Uri "$baseUrl/api/system/version" `
    -Headers $headers `
    -ContentType 'application/json' `
    -Body '{}' `
    -TimeoutSec 10

if ($response.code -ne 0) {
    throw "SiYuan API 失败：code=$($response.code)，msg=$($response.msg)"
}
```

其它 JSON POST 复用同一模式，把 PowerShell 对象通过 `ConvertTo-Json -Depth 20 -Compress` 生成请求体。不要打印 `$headers`、`SIYUAN_TOKEN` 或带令牌的命令行。

## 官方 PATH 与 Microsoft Store 修复方式

只负责诊断；未经用户明确授权，不执行本节变更。

- 官方安装包：安装器会把 `<安装目录>\resources\kernel` 加入用户或系统 `PATH`，并创建 `siyuan.exe`。新终端中 `Get-Command siyuan` 应直接成功。
- Microsoft Store：官方说明 Store 版不能修改 `PATH`，建议把稳定的 `siyuan.cmd` shim 放到默认已在 `PATH` 中的 `%LOCALAPPDATA%\Microsoft\WindowsApps`。shim 每次调用都通过 `Get-AppxPackage` 解析更新后的包目录。

官方一次性 shim：

```powershell
$shimDir = "$env:LOCALAPPDATA\Microsoft\WindowsApps"
@(
    '@echo off'
    'setlocal'
    'set "ROOT="'
    'for /f "delims=" %%i in (''powershell -NoProfile -Command "(Get-AppxPackage *SiYuan*).InstallLocation"'') do set "ROOT=%%i"'
    'if not defined ROOT goto :noshim'
    '"%ROOT%\app\resources\kernel\SiYuan-Kernel.exe" %*'
    'exit /b %ERRORLEVEL%'
    ':noshim'
    '1>&2 echo siyuan: Microsoft Store edition not found'
    'exit /b 1'
) | Set-Content "$shimDir\siyuan.cmd"
```

卸载 Store 版后，shim 不会自动删除；官方清理命令是：

```powershell
Remove-Item "$env:LOCALAPPDATA\Microsoft\WindowsApps\siyuan.cmd"
```

来源：[SiYuan v3.7.2 内置 Command Line Interface 指南](https://github.com/siyuan-note/siyuan/blob/v3.7.2/app/guide/20210808180117-6v0mkxr/20200923234011-ieuun1p/20210808180303-xaduj2o/20260612192252-sgw2paw.sy)。
