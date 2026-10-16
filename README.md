# H3-Prompt-Master

MiniMax H3 提示词与分镜工作流，提供 **独立 Skill** 和 **GPT/Codex Plugin** 两种安装形式，共用一套内容。

## 形式与下载

| 形式 | 安装内容 | 下载 |
|---|---|---|
| Skill | `skills/h3-prompt-master/` | [独立 Skill ZIP](https://github.com/ShingStars/H3-Prompt-Master/raw/refs/heads/main/packages/H3-Prompt-Master-Skill-v1.0.0.zip) |
| GPT/Codex Plugin | 根目录 `plugin.json` 与 `skills/` | [插件 ZIP](https://github.com/ShingStars/H3-Prompt-Master/raw/refs/heads/main/packages/H3-Prompt-Master-GPT-Codex-Plugin-v1.0.0.zip) |

## 安装 Skill

解压 Skill ZIP，把 `h3-prompt-master` 文件夹放入支持 Agent Skills 的客户端技能目录。Codex 可使用 `~/.agents/skills/`。

也可以在 Codex 中让 Skill Installer 安装本仓库的 `skills/h3-prompt-master` 路径。新建对话后使用 `$h3-prompt-master`，或直接描述 H3 提示词需求。

## 安装 GPT/Codex Plugin

本仓库包含插件清单与 marketplace，可通过支持本地/仓库插件市场的客户端安装：

```powershell
codex plugin marketplace add ShingStars/H3-Prompt-Master
codex plugin add h3-prompt-master@h3-prompt-master
```

安装后重新启动客户端或新建对话，在插件列表中选择 **H3-Prompt-Master**。支持插件提及的 ChatGPT Work 界面可用 `@H3-Prompt-Master`。

这是 GitHub 分发包；是否能从某个 ChatGPT 界面直接安装，取决于该界面对仓库或本地 marketplace 的支持。发布到 GitHub 不会自动上架 OpenAI 公共插件目录。

## 能力

- H3 模式选择：T2VA、I2VA、FL2VA、L2VA、Ref2VA。
- 提示词撰写、改写、审校、镜头时间轴、跨片段连续性与声音设计。
- 3D 动画短片、品牌宣传、极简产品广告、双人游戏开场。
- 手绘与实拍融合、高密度打斗、参考图双角色对决。
- 图像提示词、歌词字幕 MV、纸质拼贴与纸艺定格解说。
- 剧本分镜拆解、中文导演稿、英文提示词与 HTML 制作表。

## 使用示例

> 使用 H3-Prompt-Master，根据这两张角色参考图，写一段 15 秒连续攻防的 H3 Ref2VA 提示词，给出中文导演稿和英文六段稿。

> 使用 H3-Prompt-Master，把这份剧本拆成连续短片单元，保留角色、空间和声音的连续性。

## 结构

```text
plugin.json                       GPT/Codex 插件入口
.agents/plugins/marketplace.json   仓库插件市场
skills/h3-prompt-master/SKILL.md    唯一 Skill 入口
skills/h3-prompt-master/references/ 内部工作流、参考资料和模板
packages/                         两种独立下载包
```

插件采用官方支持的 skills-only 结构，无需另行运行 MCP 服务。提示词与规划可直接使用；实际图片、视频和音频生成取决于调用环境可用的工具。

## 来源

本项目整合用户提供的 14 套工作流，详见 [SOURCES.md](SOURCES.md)。内部入口命名为 `GUIDE.md`，避免被重复识别为独立 Skill。

