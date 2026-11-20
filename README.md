# Windows 本地视频制作 Skill

用 Claude Code 或 Codex 检查 Windows 视频工程、整理客户需求、预检口播脚本，并用现有引擎验收和交付本地视频。这个仓库提供流程、模板和演示数据。导出 MP4 需要另行准备可用的引擎及素材。

## 安装

1. 克隆或下载本仓库。
2. 将整个 `windows-video-production` 文件夹复制到目标项目的 Skill 目录，并随文件夹保留 [MIT 许可文本](LICENSE)。
3. 在目标项目目录打开新会话，按下表手动调用。

| 客户端 | 项目内安装路径 | 首次调用 |
| --- | --- | --- |
| Claude Code | `.claude/skills/windows-video-production/SKILL.md` | `/windows-video-production 无密钥验收` |
| Codex | `.agents/skills/windows-video-production/SKILL.md` | `$windows-video-production 帮我做无密钥验收` |

项目级安装与调用方式见 [Claude Code 文档](https://code.claude.com/docs/en/skills) 和 [Codex 文档](https://developers.openai.com/codex/skills)。

## 第一次使用

- **手头只有 Skill**。运行「无密钥验收」，检查当前工作目录、运行时和引擎缺项。可先填需求单，准备待审脚本。
- **已有兼容引擎**。在单独的干净副本中按引擎参考手册试跑 10 秒静音样片，记录实际分辨率、帧率、编码和帧数。
- **开始一条正式视频**。提供交付规格和获准使用的素材。先审稿与预检版式，再按本单授权配音、渲染和质检。
- **接入智心 AI MCP**。用目标会话可见的工具确认实际能力。只有发布能力时，制作仍由本地引擎完成。

## 仓库内容

- [Skill 入口](windows-video-production/SKILL.md)
- [Windows 引擎迁移与验收](windows-video-production/references/windows-engine.md)
- [智心 AI MCP 接入](windows-video-production/references/zhixin-mcp.md)
- [定制视频需求单](windows-video-production/templates/client-brief.md)
- [待审脚本示例](windows-video-production/examples/script.example.json)

[MIT 许可](LICENSE)适用于本仓库提供的 Skill、文档和示例。自行准备的引擎、字体、素材与外部服务按各自的许可及使用条款使用。
