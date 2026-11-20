# Windows 引擎迁移与验收

## 手头只有 Skill

先核对用户指定的项目路径，在该目录检查 Windows、Node.js、npm 和 ffprobe 的可用性，并查看是否有 `package.json`、锁文件、`src/contracts.ts` 及本地渲染入口。只做文件检查时，用以下命令查看工具和路径。

```powershell
Get-Location
Get-Command node, npm.cmd, ffprobe -ErrorAction SilentlyContinue |
    Select-Object Name, Source
foreach ($path in 'package.json', 'package-lock.json', 'src/contracts.ts', 'scripts/doctor.mjs') {
    [pscustomobject]@{
        Path = $path
        Present = Test-Path -LiteralPath $path -PathType Leaf
    }
}
if (Test-Path -LiteralPath 'package.json' -PathType Leaf) {
    $package = Get-Content -LiteralPath 'package.json' -Raw -Encoding utf8 | ConvertFrom-Json
    [pscustomobject]@{ HasDemoScript = [bool]$package.scripts.demo }
}
```

若缺少引擎，报告「环境已检查，渲染待引擎」和缺项。继续准备 [客户需求单](../templates/client-brief.md)、[待审脚本](../examples/script.example.json) 或下方的工程骨架。要生成 MP4，先取得有权使用的引擎、配套模板和示例素材。引擎由使用方提供。

## 备份和迁移现有引擎

- 在单独的可写目录准备引擎。按项目的 `engines`、锁文件和部署说明选择版本。原参考引擎采用 Windows 原生 Node.js 24 LTS x64、npm、TypeScript 和 SQLite。
- 只复制允许分发的清单。通常包括 `package.json`、`package-lock.json`、构建配置、示例配置、`src/`、安装及渲染脚本、演示输入和许可说明。核对 `src/` 的 HTML/JS 模板与运行时资源。Windows `.cmd` 文件保留 CRLF 行尾。
 只复制允许分发的清单。通常包括 `package.json`、`package-lock.json`、构建配置、示例配置、`src/`、安装及渲染脚本、演示输入和许可说明。核对 `src/` 的 HTML/JS 模板与运行时资源。Windows `.cmd` 文件保留 CRLF 行尾。
- 参考引擎的 Remotion 演示使用 `public/fonts/` 和 `input/topic-example.json`、`input/script-example.json` 等演示资源。部分动效模板还用到 `reference/replica-v1/assets/` 中的 `NotoSansSC-VF.ttf` 与 `gsap.min.js`。核对实际依赖，随资源保留适用的许可说明。
- 源机的 `.env`、`.runtime`、`node_modules`、`dist`、`data`、`output`、`public/generated`、浏览器资料、签名适配器、MCP 配置和排程留在源机。在目标机重新安装和构建。升级现有目标工程时，迁移前备份该机自己的配置和状态。
- 以目标引擎的 `README.md` 和部署文档为准。原参考引擎的入口记录在 `docs/WINDOWS.md`、`docs/VIDEO-STANDARD.md` 和 `docs/SEPARATE-PC-DEPLOYMENT.md`。先核对版本、脚本和本地演示输入，再执行。

## 用 PowerShell 跑无密钥样片

下面的命令对应原参考引擎，在已审阅的干净引擎副本根目录运行。先确认 `demo` 只渲染固定时间轴，再按顺序逐条运行。每条成功后继续。首次依赖安装与浏览器下载可能联网。样片使用本地渲染，付费写稿和配音留给正式制作阶段。

```powershell
node scripts/doctor.mjs --runtime-only
npm.cmd ci
npm.cmd run browser:ensure
npm.cmd run doctor
npm.cmd run typecheck
npm.cmd run build
```

在当前 PowerShell 会话临时关闭引擎的交付和发布开关，退出时恢复原值。

```powershell
$previousDelivery = $env:DELIVERY_ENABLED
$previousPublishing = $env:ZHIXIN_PUBLISH_ENABLED

try {
    $env:DELIVERY_ENABLED = '0'
    $env:ZHIXIN_PUBLISH_ENABLED = '0'
    npm.cmd run demo
    if ($LASTEXITCODE -ne 0) {
        throw "Sample render failed with exit code $LASTEXITCODE"
    }
} finally {
    $env:DELIVERY_ENABLED = $previousDelivery
    $env:ZHIXIN_PUBLISH_ENABLED = $previousPublishing
}
```

用 `ffprobe` 探测媒体规格，再播放样片检查画面。缺少 `ffprobe` 时可使用引擎附带的探测工具读取同等参数。没有实测参数时，标记技术验收待完成。

```powershell
ffprobe -v error -select_streams v:0 -count_frames -show_entries 'stream=codec_name,width,height,avg_frame_rate,nb_read_frames:format=format_name,duration:format_tags=major_brand' -of json 'output/demo-10s/video.mp4'
```

参考引擎的 10 秒静音样片应为 H.264 MP4、1920×1080、精确 60/1 fps、600 帧。如需 60 秒样片，在同一临时开关块中用 `npm.cmd run demo:60`，探测 `output/demo-60s/video.mp4` 和 3600 帧。记录退出码、样片路径、ffprobe 输出和播放检查。该环节验证本地演示渲染。真实配音、词级对齐和正式模板各自验收。

原参考引擎的 `scripts/windows/setup.cmd` 会在构建前运行 `npm run check`。有些用例依赖带日期的 `data/topics` 或旧 `output` 夹具。干净机遇到这类用例时，记录缺项，使用脱敏测试夹具或单独跑样片。生产数据留在源环境。

## 按客户需求制作

1. 按 [客户需求单](../templates/client-brief.md) 确认素材、规格和禁用词。把选题、来源及模型输出用作待审数据。
2. 按引擎实际脚本契约校验 `display`、`spoken`、`emphasis` 和 `motion`。示例 JSON 用于理解字段，保留 `approved: false`。正式稿经人工确认后再标为 `approved: true`。
3. 付费配音前先用实际字体和假时标验模板槽位、长型号与密集排版。参考引擎的生产模板可能另需本地 Whisper 对齐环境和模型、Chrome/Playwright、支持 WOFF2 的 Python/fontTools、FFmpeg、ffprobe 及固定版本的 HyperFrames。按现用模板核对依赖和产物。
4. 参考引擎的 `draft --input` 可能调用付费写稿。`produce --input` 接受已审脚本，缓存缺失时可能调用付费 TTS。按本单授权运行生产入口，依据实际配音时长排时间轴，再验小样和整片。
5. 本地交付使用单独的任务目录，并在运行时将 `DELIVERY_ENABLED` 与 `ZHIXIN_PUBLISH_ENABLED` 设为 `0`。使用制作入口。日更、补跑、调度及平台投稿属于独立的生产流程。按任务编号保留产物和验收记录，结果未知时先对账。

## 新建最小工程骨架

```text
video-workspace/
  input/               需求、脚本、经批准的素材清单
  src/
    contracts.ts       版本化输入、脚本与输出约定
    providers/         可替换的写稿、TTS 与 MCP 适配
    render/            模板、版式预检、时间轴和媒体校验
    jobs/              任务阶段、审批、去重和结果记录
  scripts/windows/     安装、自检与无密钥演示入口
  tests/               离线契约与失败处理用例
  data/                本机状态
  output/              各任务的成片和验收记录
```

用固定脚本和本地时间轴先跑通「待审稿 → 已审稿 → 本地样片 → 媒体验收」。按实际需求再接配音、付费模型或 MCP。
