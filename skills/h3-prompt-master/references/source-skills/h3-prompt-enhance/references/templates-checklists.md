# 工作流 · 生产模板 · 审计清单 · 官方来源索引

本文件是 MiniMax H3 提示词工程的**执行层参考资料**：什么时候读它——需要在「识别模式 → 建立素材角色表 → 写时间轴 → 编译官方结构 → 生成前审计 → 生成后验收」这条链路上直接照抄模板、逐条打勾、或核对官方规格与命令行时读它。它保留 §4 六步工作流、§15 各模式生产模板原文、§16 失败对照表、§17/§18 全部检查项、§19 Agent 执行规范与默认输出格式、§20 十条原则、§21–§23 官方覆盖与 §26 来源快照。配合关系：`SKILL.md` 负责总入口与模式路由，`official-modes.md` 负责官方字段与案例全文，`ref2va.md` 负责六段式标签细节，`shot-continuity-sound.md` 负责镜头、连续性与声音设计；本文件与它们冲突时，以本文件转录的官方固定格式为准。

## 本文件目录

- [1. Skill 的目标](#1-skill-的目标)
- [2. 适用场景](#2-适用场景)
- [4. 总体工作流](#4-总体工作流)
- [15. 生产模板](#15-生产模板)
- [15.1 T2VA 模板](#151-t2va-模板)
- [15.2 I2VA 模板](#152-i2va-模板)
- [15.3 FL2VA 模板](#153-fl2va-模板)
- [15.4 L2VA 模板](#154-l2va-模板)
- [15.5 Ref2VA 骨架](#155-ref2va-骨架)
- [15.6 中文对白示例](#156-中文对白示例)
- [16. 常见失败与修正](#16-常见失败与修正)
- [17. 生成前审计清单](#17-生成前审计清单)
- [18. 生成后验收清单](#18-生成后验收清单)
- [19. Agent 执行规范](#19-agent-执行规范)
- [20. 最重要的十条原则](#20-最重要的十条原则)
- [21. 官方文档完整覆盖说明](#21-官方文档完整覆盖说明)
- [22. 官方仓库 README 完整中文覆盖](#22-官方仓库-readme-完整中文覆盖)
- [22.1 官方提示词 Skill 的安装与兼容性](#221-官方提示词-skill-的安装与兼容性)
- [22.2 官方在线入口](#222-官方在线入口)
- [22.3 系统概览与官方规格](#223-系统概览与官方规格)
- [22.4 H3-Context-IR](#224-h3-context-ir)
- [22.5 H3-Base 架构](#225-h3-base-架构)
- [22.6 H3-Regenerate-2K 的官方定义](#226-h3-regenerate-2k-的官方定义)
- [22.7 推荐工作流](#227-推荐工作流)
- [22.8 本地 checkpoint 与目录结构](#228-本地-checkpoint-与目录结构)
- [22.9 官方 SGLang 启动命令](#229-官方-sglang-启动命令)
- [22.10 可复现的 768p 官方案例](#2210-可复现的-768p-官方案例)
- [22.11 完整 2K 工作流](#2211-完整-2k-工作流)
- [22.12 官方许可证与联系方式](#2212-官方许可证与联系方式)
- [23. 官方 `h3-prompt-writing` Skill 完整中文覆盖](#23-官方-h3-prompt-writing-skill-完整中文覆盖)
- [26. 官方参考资料与版本快照](#26-官方参考资料与版本快照)
- [来源](#来源)

---

## 1. Skill 的目标

本 Skill 不只是“把描述写得更华丽”，而是把用户意图编译成 H3 更容易执行的**多模态时序规格**。最终 Prompt 应同时回答以下问题：

1. 这是哪一种生成模式？
2. 哪些素材是角色参考，哪些是关键帧，哪些是运动或音色参考？
3. 视频从什么状态开始，经过什么可观察变化，最后停在什么状态？
4. 每个镜头何时开始，为什么切换？
5. 谁在说话，台词是什么语言，口型属于谁？
6. 环境声、动作声、对白、画内音乐和画外配乐分别属于哪一层？
7. 哪些外观、空间、物体状态和声音关系必须保持不变？
8. 视频时长能否容纳这些动作和台词？

高质量 H3 Prompt 的本质是：

```text
自然语言需求
→ 任务模式识别
→ 参考素材角色分配
→ 时间轴与状态机
→ 官方 Prompt 结构
→ 生成参数
→ 视听验收
```

---

## 2. 适用场景

在以下任务中启用本 Skill：

- 文生音视频：T2VA
- 首帧图生音视频：I2VA
- 首尾帧图生音视频：FL2VA
- 尾帧图生音视频：L2VA
- 多图片、多视频、多音频的全参考生成：Ref2VA
- 需要原生对白、旁白、环境声、动作音效或配乐的短片
- 需要固定角色、服装、场景、道具和镜头轴线的系列视频
- 需要把长剧本拆成多个 4–15 秒 H3 片段
- 需要设计 I2V 链式衔接、真实尾帧接力和发布前 QA

不应把本 Skill 当作：

- 精确排版、字幕制作或 Logo 设计工具
- 确定性的角色绑定系统
- 帧级视频编辑器
- 专业配音软件或声纹克隆工具
- 传统超分模型的替代品

H3 是生成模型。Prompt 能提高遵循率，但不能把概率生成变成绝对确定的渲染程序。

---

## 4. 总体工作流

### Step 1：识别输入模式

按以下决策树选择模式：

```text
没有图片、视频、音频参考？
└─ 是 → T2VA

只有一张图？
├─ 图是第 0 秒 → I2VA
└─ 图是视频结尾 → L2VA

两张图分别锁定开头和结尾？
└─ 是 → FL2VA

需要多图角色、参考视频、音色、动作、场景或编辑关系？
└─ 是 → Ref2VA
```

不要因为“有一张角色参考图”就机械地选择 I2VA。若图片只定义人物身份、并不是第 0 秒的具体画面，应使用 Ref2VA 的 `<Subject N>` 关系。

### Step 2：建立素材角色表

生成 Prompt 前，先在脑中或工作文件中建立素材台账：

| 素材 | 角色 | 允许继承 | 不应继承 |
|---|---|---|---|
| Picture 1 | 主角身份 | 脸、发型、服装、比例 | 背景文字、平台水印 |
| Picture 2 | 场景参考 | 建筑、布局、色彩 | 图片中的路人身份 |
| Video 1 | 运镜参考 | 镜头路径、剪辑节奏 | 原视频人物和品牌 |
| Audio 1 | 音色参考 | 音色、语速、情绪 | 原台词内容 |

如果一个素材同时承担多个职责，必须明确每个职责；如果职责冲突，应拆分或决定优先级。

### Step 3：先写时间轴，再写美术描述

推荐先形成最小时间轴：

```text
0.0–4.5 秒：建立人物和问题
4.5–10.0 秒：执行核心动作
10.0–15.0 秒：结果、反应和结尾锚点
```

再给每一段补充：

- 当前景别和构图
- 主体位置与朝向
- 动作起点和终点
- 摄影机运动
- 同步声音
- 台词和口型
- 需要保持的状态

### Step 4：编译为官方结构

Base 模式使用三段式；Ref2VA 使用六段式。字段名、字段顺序、标签和时间格式不要自行发明。

### Step 5：做静态检查

生成前检查：

- 所有时间点严格递增并落在总时长内；
- Shot 1 不写时间戳；
- 后续 Shot 使用 `[Shot N] At 00:SS.mmm, ...`；
- 所有说话人 ID 全程稳定；
- 台词只出现一次；
- 对白只放在 `<d>` 内；
- 环境声、动作声和配乐没有重复或冲突；
- 关键帧标签和素材数量一致；
- 动作数量符合 4–15 秒的物理承载能力。

### Step 6：生成后做视听 QA

Prompt 通过不代表结果通过。至少检查：

- 真实帧数、时长、帧率和音轨；
- 首帧/尾帧是否真正命中；
- 人脸、服装、道具数量是否漂移；
- 口型是否属于正确角色；
- 台词是否错读、重复、截断或串台；
- 是否意外生成字幕、Logo、乱码或多余人物；
- 是否有冻结帧、跳帧、黑帧、穿模或动作逆序；
- 剪辑点前后空间和声音是否连续。

---

## 15. 生产模板

## 15.1 T2VA 模板

```text
integrated_multimodal_description: [Shot 1] [Style], [initial composition]. [Subject identity, position, environment, lighting]. [Primary action]. [Speaker description] (S1) says: <d>[Language] Exact dialogue.</d> [Reaction and state change].
[Shot 2] At 00:SS.mmm, the camera cuts to [new information]. [Action, camera, synchronized sound, result].

overall_soundscape: [Ambient baseline]. [Physical and non-verbal sounds across the video].

non_diegetic_music: [Instrumentation, tempo, rhythm, dynamic development] / N/A
```

## 15.2 I2VA 模板

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.

integrated_multimodal_description: [Shot 1] [Style], the subject shown in <Picture 1> remains in the same opening composition, preserving [identity, clothing, pose, objects, spatial layout]. [Observable motion begins]. [Continuous development]. [Result or reaction].

overall_soundscape: ...

non_diegetic_music: ...
```

## 15.3 FL2VA 模板

```text
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot N) aligns with the SS.SS-second mark of the target video.

integrated_multimodal_description: [Shot 1] [Style], the shot begins in the exact state and composition established by Picture 1. [Step-by-step physical path]. [Intermediate visible changes]. The motion gradually settles into the pose, object state, lighting, spacing, and final composition established by Picture 2.

overall_soundscape: ...

non_diegetic_music: ...
```

## 15.4 L2VA 模板

```text
How the reference pictures align with the target video — <Picture 1> (from [Shot N]) aligns with the SS.SS-second mark of the target video.

integrated_multimodal_description: [Shot 1] [Style], [plausible earlier state compatible with Picture 1]. [Action path]. [Shot N] At 00:SS.mmm, [final convergence]. The subject and scene settle into the exact final pose, geometry, lighting, and composition established by <Picture 1>.

overall_soundscape: ...

non_diegetic_music: ...
```

## 15.5 Ref2VA 骨架

```text
subject_definitions:
<Subject 1> is ... from <Picture 1>, providing ...
<Subject 2> is ... from <Video 1>, providing ...
<Audio 1> is the voice-timbre reference for <Subject 1> (S1).

summary:
[reference generation + audio reference] The target video ...

retention_analysis:
<Subject 1> (appears in [Shot 1], [Shot 2]): fully_preserved - ...
<Subject 2> (appears in [Shot 1]): weak_reference - ...
<Audio 1>: reference - ...

detailed_description:
The target video uses ...
[Shot 1] ...
[Shot 2] At 00:SS.mmm, ...

overall_soundscape:
...

non_diegetic_music:
...
```

## 15.6 中文对白示例

```text
A calm adult Mandarin narrator with a warm, measured delivery (S1) says in an off-screen voiceover: <d>[Chinese] 今天，我们从这里出发。</d> while the on-screen character's lips remain completely closed.
```

---

## 16. 常见失败与修正

| 问题 | 常见原因 | 修正方向 |
|---|---|---|
| 人脸漂移 | 多个参考职责重叠；角色描述变化；镜头过多 | 缩窄参考职责；固定角色卡；减少切镜 |
| 服装变化 | 同一角色使用不同颜色或同义描述 | 保留同一服装短语；写清状态变化触发 |
| 首帧不像输入 | 未使用官方 I2VA 首行；重新设计了参考图 | 使用 `<Picture 1>` 固定首行；从图后动作写起 |
| 尾帧未命中 | 中间动作过多；最后才突然要求收敛 | 提前进入收敛阶段；减少末尾新动作 |
| 视频之间卡顿 | 片尾仍高速运动；编码拼接参数不一致；音频相位突变 | 片尾动作收束；统一 fps/编码；做音频淡化或连续参考 |
| 一帧一帧跳 | 镜头约束冲突；运镜过多；参考帧差异过大 | 每段一个主运镜；减少相互矛盾的构图要求 |
| 台词重复 | 同一句在多个区块或镜头重复 | 台词只在一个 `<d>` 中出现；跨切使用 `<scenetrans>` |
| 台词错语言 | 缺少 `[Language]`；文字在 `<d>` 外 | 使用标准 `<d>[Chinese] ...</d>` |
| 旁白绑定口型 | 没有使用官方 voiceover 句式 | 使用 `says in an off-screen voiceover` 并写嘴唇闭合 |
| 对白过快 | 台词量超过时长；同时动作过多 | 拆成多个片段；减少词数；给停顿时间 |
| 自动字幕 | 台词被当成可见文字；模型随机生成 | 不把台词写成可见文字；按官方结构重写；生成后 QA |
| 乱码招牌 | 让模型生成长而小的文字 | 文字后期完成；生成阶段预留安全区 |
| 手穿过物体 | 动作缺少接触与铰链路径 | 把动作拆为接近、接触、施力、运动、停止 |
| 多出人物/动物 | 群体描述模糊；参考图背景有额外主体 | 明确可见主体数量和进入路径；裁净参考素材 |
| 环境声混乱 | 声音在多个层重复；画内外音乐未区分 | 按 Shot、soundscape、music 三层重新分配 |
| 多图效果变差 | 每张图职责不清；身份、场景、动作互相竞争 | 建立素材角色表；一图一主职责；移除低价值参考 |

---

## 17. 生成前审计清单

### 模式

- [ ] 已明确 T2VA / I2VA / FL2VA / L2VA / Ref2VA
- [ ] 关键帧与普通身份参考没有混淆
- [ ] 没有把 Ref2VA 的六段式与 Base 三段式混用

### 结构

- [ ] 字段名和顺序符合官方格式
- [ ] I2VA/FL2VA/L2VA 固定首行正确
- [ ] 固定首行后有一个空行
- [ ] Shot 1 没有时间戳
- [ ] 后续 Shot 时间严格递增

### 时间

- [ ] 所有动作可在 4–15 秒完成
- [ ] 台词长度符合可用时间
- [ ] 末尾留有动作或情绪收束时间
- [ ] 尾帧任务给了足够的收敛时间

### 主体

- [ ] 角色身份、服装、发型和体型固定
- [ ] 主体数量明确
- [ ] 道具数量与状态明确
- [ ] 左右位置、视线和光线方向没有冲突

### 对白

- [ ] 每个真实发声源都有稳定 `(Sx)`
- [ ] ID 按首次实际发声顺序分配
- [ ] `<d>` 内只有语言标签和原台词
- [ ] 旁白使用官方句式
- [ ] 跨镜头台词使用 `<scenetrans>` 或明确连续关系
- [ ] 台词没有在 soundscape/music 中重复

### 音频

- [ ] 同步动作声写在对应 Shot
- [ ] 全局环境声写在 `overall_soundscape`
- [ ] 画外配乐写在 `non_diegetic_music`
- [ ] 没有配乐时使用 `N/A`

### 参考素材

- [ ] 每个参考素材职责明确且狭窄
- [ ] `<Subject N>` / `<Picture N>` / `<Video N>` / `<Audio N>` 含义稳定
- [ ] Ref2VA 每个标签都有 retention 关系
- [ ] 没有意外继承水印、字幕、背景人或错误文字

---

## 18. 生成后验收清单

### 文件层

- [ ] 文件可正常解码
- [ ] 时长符合请求
- [ ] 24 FPS 或项目指定帧率一致
- [ ] 分辨率和画幅正确
- [ ] 音轨存在且无明显爆音、静音或截断

### 画面层

- [ ] 首帧和尾帧锚点正确
- [ ] 角色脸、服装和身材未漂移
- [ ] 道具数量未变化
- [ ] 动作没有穿模、逆序或瞬移
- [ ] 没有冻结、黑帧和异常闪烁
- [ ] 没有多余字幕、乱码、Logo 或水印

### 音频层

- [ ] 说话人正确
- [ ] 台词完整且只读一次
- [ ] 语言、语速和情绪正确
- [ ] 口型与声音大致同步
- [ ] 环境声和配乐没有盖住对白
- [ ] 跨片段响度和底噪接近

### 连续性层

- [ ] 上一段结尾与下一段开头构图相容
- [ ] 镜头轴线没有无缘无故反转
- [ ] 动作状态能自然接续
- [ ] 情绪曲线连续
- [ ] 声音没有硬断或重复起句

---

## 19. Agent 执行规范

当本 Skill 被调用时，Agent 应按以下顺序工作：

1. 读取用户脚本和所有参考素材说明。
2. 判断生成模式；只有关键事实缺失且无法安全推断时才提问。
3. 建立素材角色表和说话人表。
4. 检查内容是否能放入 4–15 秒；必要时主动拆段。
5. 先写镜头时间轴，再编译成官方格式。
6. 保留用户对白原文，不擅自翻译、润色或补台词。
7. 输出字段正文使用英文；对白、歌词和可见文字保留原语言。
8. 不发明不存在的参考标签。
9. 不把环境声、对白和画外配乐混在同一区块。
10. 输出前执行审计清单。

### 默认输出格式

除非用户只要求 Prompt，否则按以下结构输出：

````text
推荐模式：I2VA
时长：15 秒
画幅：16:9
参考关系：Picture 1 = 首帧与人物/场景锚点

最终 H3 Prompt：
```text
...
```

检查结果：
- 时间轴通过
- 说话人 ID 通过
- 音频分层通过
- 关键帧标签通过
````

---

## 20. 最重要的十条原则

1. **先选模式，再写 Prompt。**
2. **先写时间轴，再写风格词。**
3. **Base 模式只用官方三段式。**
4. **Ref2VA 使用官方六段式和稳定标签。**
5. **Shot 1 无时间戳，后续 Cut 严格递增。**
6. **对白只在 `<d>[Language] ...</d>` 中出现一次。**
7. **旁白使用官方 off-screen voiceover 句式并锁住嘴唇。**
8. **关键帧要描述变化路径，而不是重复描述两张图片。**
9. **一个参考素材最好只有一个主要职责。**
10. **Prompt 是生成规格，最终质量必须靠生成后 QA 验证。**

## 21. 官方文档完整覆盖说明

本章到第 25 章用于保证本手册不只提炼“常用规则”，而是覆盖下列四份官方文档在 **2026-08-31** 版本中的全部有效信息：

| 官方文件 | 本文对应位置 | 覆盖内容 |
|---|---|---|
| `README.zh-CN.md` | 第 22 章 | 安装、在线入口、系统规格、模型架构、部署、768p 验证、2K 工作流、许可证与联系方式 |
| `skills/h3-prompt-writing/SKILL.md` | 第 23 章 | 官方 Skill 的工作流、模式路由、输出规则和效果建议 |
| `references/base-en.txt` / Hugging Face Base Guide | 第 24 章 | T2VA、I2VA、FL2VA、L2VA 的完整结构、规则、相机、对白、音频和四个案例 |
| `references/ref-en.txt` | 第 25 章 | Ref2VA 六段式、四类标签、任务类型、保留关系、说话人与音频规则和完整案例 |

### 21.1 “完整覆盖”的含义

- 所有官方章节、规则、限制、固定字符串、参数规格、部署命令和示例所表达的信息均写入本文。
- 固定字段、标签、时间格式、特殊 token、命令行参数和关键句式保留英文原形，避免中文翻译破坏可执行性。
- 官方长案例既给出可直接使用的英文结构，也给出中文解释。
- README 中纯装饰性的页眉 GIF、徽章和结果视频缩略图不复制进正文，但其指向的产品、仓库、脚本与结果资源均以链接或表格记录。
- 第 1–20 章包含生产经验；第 21–25 章是官方资料的中文化完整覆盖。两者冲突时，以第 21–25 章记录的官方固定格式为准。

### 21.2 官方内容索引

```text
官方 README
├── 提示词编写技能
├── 在线 API / 在线应用
├── 系统概览与输入输出规格
├── H3-Context-IR
├── H3-Base
│   ├── H3-Encoder
│   ├── H3-VisualVAE / H3-AudioVAE
│   └── H3-Omni-Transformer
├── H3-Regenerate-2K
├── H3-Base 本地部署
├── SGLang 部署
├── 可复现 768p 示例
├── 完整 2K 工作流
└── 许可证 / 联系方式

官方 Prompt Skill
├── 模式识别
├── Base Guide 路由
├── Ref Guide 路由
├── 输出规则
└── 效果建议

官方 Base Guide
├── 四种任务
├── 固定首行与三段式
├── 关键帧路径
├── 镜头、运镜、对白、文字与声音
└── 四个完整案例

官方 Ref Guide
├── 六段式
├── Subject / Picture / Video / Audio
├── summary 任务类型
├── retention_analysis
├── detailed_description
├── 音频与说话人
└── 完整案例
```

---

## 22. 官方仓库 README 完整中文覆盖

## 22.1 官方提示词 Skill 的安装与兼容性

官方仓库一共内置九个 Skill。通用的 H3 提示词 Skill 安装命令为：

```bash
npx skills add https://github.com/MiniMax-AI/MiniMax-H3 --skill h3-prompt-writing
```

该 Skill 在 `skills/h3-prompt-writing/references/` 中提供：

- `base-en.txt`：文本与关键帧模式，即 T2VA、I2VA、FL2VA、L2VA。
- `ref-en.txt`：全参考 Ref2VA 模式。

`h3-prompt-writing` 是纯 Markdown 与本地参考文件，不调用外部 API，因而可移植到 Claude Code、Claude Agent SDK、Cursor、Windsurf、OpenAI/Codex 类 Agent、LangChain，以及任何能读取 `SKILL.md` 和本地文件的环境。`agents/openai.yaml` 只提供可选的 ChatGPT/Codex UI 元数据，不会把 Skill 限定在 OpenAI 环境。

其余八个 Skill 面向 MiniMax Hub 画布中的 `hub_generate_video`、`hub_generate_image`、画布节点和选择卡片等专有工作流，官方明确说明它们不能直接移植到通用 Agent：

1. `minimalist-product-ad-generator`
2. `3d-animation-short-generator`
3. `papercraft-stop-motion-explainer`
4. `brand-promo-video-generator`
5. `music-video-subtitle-generator`
6. `co-op-game-intro-generator`
7. `paper-collage-explainer-generator`
8. `handdrawn-live-video-generator`

## 22.2 官方在线入口

### API

- Global：[MiniMax Global Video Generation V2 API](https://platform.minimax.io/docs/api-reference/video-generation-v2-create)
- 中国区：[MiniMax CN Video Generation V2 API](https://platform.minimaxi.com/docs/api-reference/video-generation-v2-create)

### 应用

- Web Global：[hailuoai.video](https://hailuoai.video/tools/minimax-h3)
- Web 中国区：[hailuoai.com](https://hailuoai.com/)
- Desktop / Hub Global：[hub.minimax.io](https://hub.minimax.io/)
- Desktop / Hub 中国区：[hub.minimaxi.com](https://hub.minimaxi.com/)

### 官方站点与社区

- MiniMax 官网：<https://www.minimax.io>
- GitHub：<https://github.com/MiniMax-AI/MiniMax-H3>
- Hugging Face：<https://huggingface.co/MiniMaxAI/MiniMax-H3>
- ModelScope：<https://modelscope.cn/organization/minimax>
- 中国区联系入口：<https://platform.minimaxi.com/docs/faq/contact-us>
- Discord：<https://discord.com/invite/dbMxutw7tP>

官方 README 同时提供英文、简体中文、韩文、日文版本；本文以官方简体中文 README 与两份英文 Prompt Guide 为覆盖基准。

## 22.3 系统概览与官方规格

MiniMax H3 是通用全模态生成系统：统一理解由文本、图像、视频和音频组成的上下文，并生成带原生立体声音频的视频。官方系统最高可到 2K、最长 15 秒，重点是复杂多模态指令遵循与任务泛化。

| 类别 | 官方规格 |
|---|---|
| 输出时长 | 4–15 秒 |
| 宽高比 | 支持多种比例，包括但不限于 21:9、16:9、4:3、1:1、3:4、9:16 |
| 分辨率 | 多种尺寸；默认短边 768 像素；经 H3-Regenerate-2K 可生成 2K |
| 帧率 | 24 FPS |
| 音频 | 32 kHz 立体声 |
| 稳定对白语言 | 阿拉伯语、中文、英语、法语、德语、意大利语、日语、韩语、葡萄牙语、俄语、西班牙语，共 11 种；其他语言也有不同程度支持 |

### 两个模型变体

| 变体 | 输入模式 | 官方输入限制 |
|---|---|---|
| `H3-Base-FL2VA` | T2VA / I2VA / FL2VA / L2VA | 0 张图=T2VA；1 张图=首帧或尾帧；2 张图=首尾帧 |
| `H3-Base-Ref2VA` | 全参考 Ref2VA | 图片 ≤9；视频 ≤3，每段 2–15 秒且总时长 ≤15 秒；音频 ≤3，每段 2–15 秒且总时长 ≤15 秒；音频不能作为唯一输入，必须和图片或视频一起使用；混合输入总文件数 ≤12 |

### 完整 H3 系统的三层结构

1. **H3-Context-IR**：理解、关联和细化自由形式的多模态输入，转换为 H3-Base 容易执行的上下文中间表示。
2. **H3-Base**：根据 Context-IR 输出联合生成 768p 视频与音频。
3. **H3-Regenerate-2K**：把 768p 结果连同原始上下文再次送入 H3，以 in-context 方式重新生成 2K。

官方特别强调：H3-Context-IR 对最终质量非常重要。生产流水线应接入该模块，或依据官方提示词指南自行构建具有相同职责的上下文处理系统。

## 22.4 H3-Context-IR

H3-Context-IR 是托管式预处理与编排系统，面向自由形式的多模态输入。它负责：

- 解析文本、图像、音频、参考视频之间的关系。
- 解析这些素材与目标结果之间的关系。
- 执行指令解析、跨模态关联、时间理解和复杂逻辑推理。
- 把理解结果序列化为 H3-Base 可接受的结构化表示。
- 在不偏离用户原意的前提下，适度补全缺失或描述不足的语义细节。

因为 Context-IR 依赖多阶段工作流与多个托管模型/服务，初始开源版本不包含它。MiniMax 提供 API 复现官方工作流，并提供 Prompt Guide 供开发者自行实现上下文处理。

### 官方安全防护说明

用户提交的文本、图像、视频以及增强后的 Prompt 会经过自动审核；涉嫌违法、色情或侵犯第三方权利的内容可能被拦截。官方说明过滤无法完全消除误判与漏判，且不会替代被许可方在 MiniMax H3 Community License 下承担的合法使用义务。

## 22.5 H3-Base 架构

### 22.5.1 总体数据流

```text
文本 ───────────────→ H3-Encoder ──────────────┐
图像/视频 ─→ H3-Encoder + H3-VisualVAE ───────┤
音频 ───────────────→ H3-AudioVAE ─────────────┤
                                                ↓
                                  统一打包多模态序列 + RoPE
                                                ↓
                                    H3-Omni-Transformer
                                                ↓
                              联合预测视频 latent + 音频 latent
                                                ↓
                                  视频解码 + 32kHz 立体声音频
```

- 文本由 H3-Encoder 编码。
- 视觉输入同时由 H3-Encoder 和 H3-VisualVAE 编码。
- 音频只由 H3-AudioVAE 编码。
- 编码后的不同模态被打包成统一序列，并用 RoPE 表示必要的时空关系。
- H3-Omni-Transformer 联合预测视频和音频 latent，随后分别解码。
- H3 原生支持稀疏注意力训练与推理；初始开源版本只提供全注意力推理，官方计划后续发布稀疏注意力实现。

### 22.5.2 H3-Encoder

- 使用 **Qwen3-VL-32B 的完整预训练权重**。
- 将第 **50 层 hidden states** 提供给 H3-Omni-Transformer。
- tokenizer 中加入了包括 `<d>` 在内的特殊 token。
- 推理时必须使用 H3 仓库提供的 tokenizer 与相关配置，不能随意换成同源模型的普通 tokenizer。

### 22.5.3 H3-VisualVAE

- 时间因果视频自编码器。
- 空间压缩因子 16×。
- 时间压缩因子 4×。
- latent 通道数 24。
- 官方记法：`f16t4d24`。
- Transformer 前，visual latent 在 `(time, height, width)` 上按 `1 × 2 × 2` 继续 patchify。
- 因此进入 Transformer 的视觉 token 有效空间下采样为 32×，时间下采样仍为 4×。
- latent 空间同时为重建质量与生成学习便利性优化。
- 编码器训练后，官方额外训练了 ViT-based decoder，用于降低解码成本并提高重建质量。

### 22.5.4 H3-AudioVAE

- 左右声道使用相同编码器和解码器，但两个声道独立处理。
- 解码后再重新合并为立体声。
- 每个声道把 32 kHz 音频压缩为时间率 40 Hz 的 latent token 序列。
- latent 空间参考 VA-VAE 的思想，在音频重建质量与生成学习便利性之间优化。

### 22.5.5 H3-Omni-Transformer

- 33B 参数、稠密、单流 Transformer。
- 约 13B 参数位于 AdaLN 相关分支。
- AdaLN 调制输出可以预计算并缓存，因此纯推理部署无需加载这部分参数；官方仍发布完整权重以支持微调等二次开发。
- Attention 和 FFN 都没有模态特定结构。
- 模态特定参数仅位于输入/输出层与 AdaLN 分支。
- 模态特定 AdaLN 以较低训练与推理开销提升生成质量。
- 使用三维多模态旋转位置编码 `MM-RoPE`，表达 `(t, h, w)` 三个维度。
- 训练末期引入原生稀疏注意力；初始开源版本暂不包含实现。

## 22.6 H3-Regenerate-2K 的官方定义

H3 的 2K 不是传统“单独训练一个超分模型，对 768p 像素做放大”。官方做法是：让 H3 Base 在原始多模态上下文和低分辨率结果的共同条件下，进行 **in-context 重新生成**。

官方列出的优势：

1. 最大限度复用 H3 Base 自身的生成能力。
2. 重新利用原始上下文，恢复传统超分只能猜测的信息，例如小文字和精细细节。
3. 这是 H3 任务泛化能力的一种体现。

该模块因系统复杂度尚未开源；官方提供 API 验证完整结果，并表示准备就绪后再发布。

## 22.7 推荐工作流

官方给出两种验证路径：

### 路径 A：本地 768p

```text
符合 Guide 的 Prompt
→ 本地 H3-Base
→ 768p 原生音视频
```

### 路径 B：完整 2K

```text
原始多模态输入
→ 官方 H3-Context-IR API
→ 本地 H3-Base
→ 官方 H3-Regenerate-2K API
→ 2K 原生音视频
```

第二条路径把托管 API 和本地 Base 结合起来，用于复现官方直接 API 的 2K 质量。

## 22.8 本地 checkpoint 与目录结构

官方发布两个任务专用 BF16 checkpoint。每个 checkpoint 都包含专用 Omni Transformer，以及 processor、tokenizer、text encoder、Visual VAE、Audio VAE。

| Checkpoint | 支持任务 | 条件 | 输出 | 精度 |
|---|---|---|---|---|
| MiniMax-H3 Base FL2VA | `t2va`、`fl2va`，包含首帧/尾帧/首尾帧 | 文本；可选首帧、尾帧或两者 | 视频+音频 | BF16 |
| MiniMax-H3 Base Ref2VA | `ref2va` | 文本和参考图像/视频/音频 | 视频+音频 | BF16 |

官方发布的是经过 **CFG distillation** 的 Omni Transformer 权重。

每个任务仓库是自包含 Hugging Face 风格目录：

```text
<TASK>/
├── model_index.json
├── processor/
├── tokenizer/
├── text_encoder/
├── transformer/
├── visual_vae/
└── audio_vae/
```

仓库同时托管原始 checkpoint 与 diffusers 格式。仓库级公开入口为根目录 `model_index.json`；任务族索引仍在 `FL2VA/model_index.json` 与 `Ref2VA/model_index.json`。

```bash
# 原始 checkpoint：下载两个任务族，供 SGLang / vLLM 等使用
hf download MiniMaxAI/MiniMax-H3 \
  --include "model_index.json" "FL2VA/*" "Ref2VA/*" \
  --local-dir MiniMax-H3

# 只下载 FL2VA 任务族
hf download MiniMaxAI/MiniMax-H3 \
  --include "model_index.json" "FL2VA/*" \
  --local-dir MiniMax-H3
```

Diffusers 用户不必手工下载全部文件：

```python
ModularPipeline.from_pretrained("MiniMaxAI/MiniMax-H3")
```

它会按需拉取组件。

官方推荐推理框架：

- [SGLang Cookbook](https://docs.sglang.io/cookbook/diffusion/MiniMax/MiniMax-H3)
- [vLLM Recipes](https://recipes.vllm.ai/MiniMaxAI/MiniMax-H3)
- [Diffusers MiniMax H3 文档](https://github.com/huggingface/diffusers/blob/minimax-h3/docs/source/en/api/pipelines/minimax_h3.md)
- [ComfyUI 教程](https://docs.comfy.org/tutorials/video/minimax/minimax-h3)
- [ComfyUI R2V 模板](https://github.com/Comfy-Org/workflow_templates/blob/main/templates/video_minimax_h3_r2v.json)
- [ComfyUI T2V 模板](https://github.com/Comfy-Org/workflow_templates/blob/main/templates/video_minimax_h3_t2v.json)

## 22.9 官方 SGLang 启动命令

### FL2VA

```bash
sglang serve \
  --model-path MiniMaxAI/MiniMax-H3 \
  --num-gpus 4 \
  --ulysses-degree 4 \
  --performance-mode speed \
  --host 0.0.0.0 \
  --port 30010 \
  --model-variant fl2va
```

### Ref2VA

```bash
sglang serve \
  --model-path MiniMaxAI/MiniMax-H3 \
  --num-gpus 4 \
  --ulysses-degree 4 \
  --performance-mode speed \
  --host 0.0.0.0 \
  --port 30011 \
  --model-variant ref2va
```

## 22.10 可复现的 768p 官方案例

官方仓库提供三个请求脚本与结果：

| 用例 | 请求脚本 | 官方结果 |
|---|---|---|
| T2VA | [`reproducible-768p-t2va-request.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/reproducible-768p-t2va-request.sh) | [`t2va.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/t2va.mp4) |
| FL2VA | [`reproducible-768p-fl2va-request.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/reproducible-768p-fl2va-request.sh) | [`fl2va.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/fl2va.mp4) |
| Ref2VA | [`reproducible-768p-ref2va-request.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/reproducible-768p-ref2va-request.sh) | [`ref2va.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/ref2va.mp4) |

## 22.11 完整 2K 工作流

先设置本地 SGLang 服务地址、API 区域和 Token：

```bash
SGLANG_DEPLOYMENT_URL="<sglang-deployment-url>"

# 中国区
MINIMAX_API_BASE="https://api.minimaxi.com"

# Global（二选一）
# MINIMAX_API_BASE="https://api.minimax.io"

TOKEN="<token>"
```

涉及三个官方 API：

1. 创建 H3-2K：`/video-generation-v2-create`
2. Context-IR：`/video-generation-v2-h3-context-ir`
3. Regenerate-2K：`/video-generation-v2-regeneration`

README 示例为了演示，会把本地 H3-Base 输出编码成 Base64 Data URL；官方建议生产环境将 base video 上传到可公开访问的 URL，再把 URL 作为 `base_video` 传入，避免巨大 Base64 请求。

### 官方 2K 请求脚本与结果文件

#### T2VA

| 阶段 | 请求脚本 | 结果 |
|---|---|---|
| H3-Context-IR | [`full-2k-t2va-h3-context-ir.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-t2va-h3-context-ir.sh) | 返回结构化 Prompt JSON |
| H3-Base | [`full-2k-t2va-h3-base.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-t2va-h3-base.sh) | [`t2va.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/t2va.mp4) |
| H3-Regenerate-2K | [`full-2k-t2va-h3-regenerate-2k.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-t2va-h3-regenerate-2k.sh) | [`t2va_2k.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/t2va_2k.mp4) |
| 开放平台直接 2K | [`full-2k-t2va-reference-2k-result-by-directly-calling-open-platform-api.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-t2va-reference-2k-result-by-directly-calling-open-platform-api.sh) | [`h3_direct_2k.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/h3_direct_2k.mp4) |
| 开放平台直接 768p | [`full-2k-t2va-reference-768p-result-by-directly-calling-open-platform-api.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-t2va-reference-768p-result-by-directly-calling-open-platform-api.sh) | [`h3_direct_768p.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/h3_direct_768p.mp4) |

#### I2VA

| 阶段 | 请求脚本 | 结果 |
|---|---|---|
| H3-Context-IR | [`full-2k-i2va-h3-context-ir.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-i2va-h3-context-ir.sh) | 返回结构化 Prompt JSON |
| H3-Base | [`full-2k-i2va-h3-base.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-i2va-h3-base.sh) | [`i2va.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/i2va.mp4) |
| H3-Regenerate-2K | [`full-2k-i2va-h3-regenerate-2k.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-i2va-h3-regenerate-2k.sh) | [`i2va_2k.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/i2va_2k.mp4) |
| 开放平台直接 2K | [`full-2k-i2va-reference-2k-result-by-directly-calling-open-platform-api.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-i2va-reference-2k-result-by-directly-calling-open-platform-api.sh) | [`i2va_direct_2k.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/i2va_direct_2k.mp4) |
| 开放平台直接 768p | [`full-2k-i2va-reference-768p-result-by-directly-calling-open-platform-api.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-i2va-reference-768p-result-by-directly-calling-open-platform-api.sh) | [`i2va_direct_768p.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/i2va_direct_768p.mp4) |

#### Ref2VA

| 阶段 | 请求脚本 | 结果 |
|---|---|---|
| H3-Context-IR | [`full-2k-ref2va-h3-context-ir.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-ref2va-h3-context-ir.sh) | 返回结构化 Prompt JSON |
| H3-Base | [`full-2k-ref2va-h3-base.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-ref2va-h3-base.sh) | [`r2va.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/r2va.mp4) |
| 开放平台直接 2K 参考结果 | [`full-2k-ref2va-reference-2k-result-by-directly-calling-open-platform-api.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-ref2va-reference-2k-result-by-directly-calling-open-platform-api.sh) | [`r2va_2k.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/r2va_2k.mp4) |
| 开放平台 H3 API 2K | [`full-2k-ref2va-h3-api-2k-in-open-platform-for-reference.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-ref2va-h3-api-2k-in-open-platform-for-reference.sh) | [`r2va_direct_2k.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/r2va_direct_2k.mp4) |
| 开放平台直接 768p | [`full-2k-ref2va-reference-768p-result-by-directly-calling-open-platform-api.sh`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/scripts/readme/full-2k-ref2va-reference-768p-result-by-directly-calling-open-platform-api.sh) | [`r2va_direct_768p.mp4`](https://github.com/MiniMax-AI/MiniMax-H3/blob/main/assets/r2va_direct_768p.mp4) |

### 2K 案例总表

| 用例 | 类型 | 时长 | 比例 | Context-IR 示例语义 | 示例 usage |
|---|---|---:|---|---|---|
| T2VA | 文生音视频 | 10 秒 | 16:9 | 星舰舰桥、女舰长、舰队跃迁；两镜头；环境机械声与太空歌剧配乐 | total 8565 / prompt 5650 / completion 2915 |
| I2VA | 首帧图生音视频 | 8 秒 | 自适应（示例返回 16:9） | 日式家庭餐桌，前景拉面；静态机位完成从拉面到家人的平滑 rack focus | total 22822 / prompt 12800 / completion 10022 |
| Ref2VA | 视频+音频全参考 | 5 秒 | 自适应（示例返回 16:9） | 粉色西装男子抱黑色小羊；保留原视频与 BGM，参考另一音频的男声音色生成新对白 | total 39299 / prompt 33323 / completion 5976 |

每个案例都包含 Context-IR 请求、本地 Base、Regenerate-2K，以及直接开放平台 2K/768p 参考结果；Ref2VA 还给出开放平台 H3 API 的 2K 参考结果。官方脚本目录：

- [T2VA 完整 2K 脚本](https://github.com/MiniMax-AI/MiniMax-H3/tree/main/scripts/readme)
- [I2VA 完整 2K 脚本](https://github.com/MiniMax-AI/MiniMax-H3/tree/main/scripts/readme)
- [Ref2VA 完整 2K 脚本](https://github.com/MiniMax-AI/MiniMax-H3/tree/main/scripts/readme)

### 三个 Context-IR 案例的关键信息

- **T2VA 星舰案例**：`Shot 1` 从舰桥中景缓慢推进，女舰长背对镜头观察暗灰舰队；推进器蓝光增强。`Shot 2` 在 `00:04.500` 切脸部特写并强烈震动，跃迁白光、舰体应力声、舰长踉跄与最后闭眼形成完整状态变化。
- **I2VA 拉面案例**：固定机位 8 秒，不靠移动摄影，而是用焦点转移制造叙事。开始拉面、叉烧、葱、海苔、筷子、茶杯清晰，七人家庭虚化；随后焦点平滑转到家人，蒸汽持续作为前景动态层。
- **Ref2VA 羊群案例**：使用 `subject_definitions → summary → retention_analysis → detailed_description → overall_soundscape → non_diegetic_music`。人物、原视频、同步 BGM、声音音色分别定义；对白结束时明确嘴唇闭合，并继续抚摸小羊。

## 22.12 官方许可证与联系方式

官方 README 的“提示词指导”说明：为了保持 GitHub README 的 Markdown 布局简洁，Hugging Face 发布中的完整提示词指南没有重复嵌入 README；本手册第 24、25 章已经把对应 Base 与 Ref Guide 纳入同一份中文文档。

- 许可证：[MiniMax H3 Community License Agreement](https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/LICENSE)
- 联系邮箱：[model@minimax.io](mailto:model@minimax.io)

---

## 23. 官方 `h3-prompt-writing` Skill 完整中文覆盖

官方 Skill 本身是一个路由与约束层，完整工作流如下：

1. 判断输入模式：T2VA、I2VA、FL2VA、L2VA 或 Ref2VA。
2. Base 文本/关键帧模式必须读取 `references/base-en.txt`，遵循其最终 Prompt 结构。
3. 全参考模式必须读取 `references/ref-en.txt`，遵循六段式。
4. 所选 Guide 中的字段名、章节顺序、标签和时间标记必须保持原样。

### 23.1 Base 模式职责

- T2VA：仅从文本构建完整视听时间线。
- I2VA：从首帧出发，向后连续发展。
- FL2VA：写清首帧与尾帧之间的连续可见路径。
- L2VA：推断合理的前置状态，并收敛到给定尾帧。

三个字段顺序固定：

```text
integrated_multimodal_description
→ overall_soundscape
→ non_diegetic_music
```

### 23.2 Ref2VA 职责

六个字段顺序固定：

```text
subject_definitions
→ summary
→ retention_analysis
→ detailed_description
→ overall_soundscape
→ non_diegetic_music
```

所有 Reference Label 必须跨六段保持一致。

### 23.3 官方输出规则

- Rewrite section 使用英文；只在 `<d>` 中的对白/歌词以及画面内可见文字保留原语言。
- 每个 shot 应描述构图、主体、环境、动作、相机、声音，以及参考内容实际出现或生效的准确位置。
- 避免只写剧情摘要。
- 避免未解析或前后不一致的 Reference Label。
- 时间必须与用户要求的 4–15 秒匹配。

### 23.4 官方效果建议

- 描述总时长必须与目标时长一致。
- `<Picture 1>`、`<Video 1>`、`<Audio 1>` 等标签必须在所有段落中保持一致。
- 比起抽象词 `cinematic`、`beautiful`，优先写具体可视、可听事实。
- I2VA / FL2VA / L2VA 必须明确首帧和/或尾帧如何连接到时间线。

---

## 26. 官方参考资料与版本快照

- MiniMax H3 官方仓库：<https://github.com/MiniMax-AI/MiniMax-H3>
- 官方中文 README：<https://github.com/MiniMax-AI/MiniMax-H3/blob/main/README.zh-CN.md>
- 官方 Base Prompt Guide：<https://huggingface.co/MiniMaxAI/MiniMax-H3/blob/main/docs/VIDEO_PROMPT_WRITING_GUIDE_base_en.md>
- 官方 H3 Prompt Writing Skill：<https://github.com/MiniMax-AI/MiniMax-H3/blob/main/skills/h3-prompt-writing/SKILL.md>
- 官方 Base Guide 源文件：<https://github.com/MiniMax-AI/MiniMax-H3/blob/main/skills/h3-prompt-writing/references/base-en.txt>
- 官方 Ref2VA Guide：<https://github.com/MiniMax-AI/MiniMax-H3/blob/main/skills/h3-prompt-writing/references/ref-en.txt>
- 官方 Skills 目录：<https://github.com/MiniMax-AI/MiniMax-H3/tree/main/skills>
- Hugging Face 模型页：<https://huggingface.co/MiniMaxAI/MiniMax-H3>
- ModelScope MiniMax 组织页：<https://modelscope.cn/organization/minimax>

### 26.1 维护规则

1. 本文官方覆盖快照日期为 `2026-08-31`。
2. 官方字段、标签、模式、模型输入限制、API 或部署方式更新时，以最新官方仓库与模型页为准。
3. 更新时先比较 README、`SKILL.md`、`base-en.txt`、`ref-en.txt` 四份文件，再同步修订第 21–25 章。
4. 固定语法变更必须同时更新第 1–20 章中的生产模板，避免“官方附录”和“工程模板”不一致。

---

## 来源

整理自《提示词工程技巧.md》的以下区间（源文件为 MiniMax H3 中文提示词工程 Skill，版本 2.0.0，updated 2026-08-31）：

| 本文件小节 | 源文件区间 |
|---|---|
| §1、§2 | 第 16–72 行 |
| §4 | 第 114–202 行 |
| §15、§16 | 第 1028–1135 行 |
| §17 | 第 1136–1190 行 |
| §18 | 第 1191–1228 行 |
| §19、§20 | 第 1229–1280 行 |
| §21 | 第 1281–1344 行 |
| §22 | 第 1345–1717 行 |
| §23 | 第 1718–1773 行 |
| §26 | 第 2378 行至文件末尾（第 2395 行） |

本文件不含 §6、§7–§12、§13、§14、§24、§25 的内容，那些小节由 `official-modes.md`、`ref2va.md`、`shot-continuity-sound.md` 负责。
