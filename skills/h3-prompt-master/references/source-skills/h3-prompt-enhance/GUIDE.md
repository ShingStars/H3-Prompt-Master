---
name: minimax-h3-prompt-suite
description: "撰写、升级、审校或批量回填 MiniMax H3 提示词：模式选择（T2VA/I2VA/FL2VA/L2VA/Ref2VA）、官方三字段与 Ref2VA 六段结构、镜头时间轴与运镜语法、连续性状态机、台词/画外音/声音分层、多段无缝长片，以及自带的校验、提取、插入工具链。"
whenToUse: 当需要为 MiniMax H3 写、改、审提示词；判定生成模式；把长剧本拆成多段无缝长片；锁住风格与跨段连续性；把提示词批量回填进 ComfyUI 工作流；或体检已有提示词时。
metadata:
  version: 3.0.0
  merged_from: h3-seamless-prompt · minimax-h3-prompt-engineering-cn · minimax-h3-prompt-standardizer
  language: zh-CN 正文 + English 可粘贴句
  verified_against: MiniMaxH3-小黑8G显存-30段无缝流.json（30 段）与 10 段无缝流
---

# MiniMax H3 提示词工程（整合版）

把任何输入——一句话点子、剧本、分镜表、首尾帧、多参考素材——编译成**可直接生成**的 H3 提示词，并把多段长片串成一条无缝长片。本文件是**行动纲领**；深度知识在 `references/`，工具在 `scripts/`。

**三条铁律**（违反任何一条，后面的工作基本白做）：

1. **先选模式，再写提示词**。模式决定固定首行与字段顺序，写错模式 = 结构错。
2. **先写时间轴，再写美术词**。H3 是时序生成器，时间与状态比形容词重要得多。
3. **字段名、标签、时间格式一律不自己发明**。官方结构是合同，不是建议。

---

## 0. 怎么用这份 skill

| 你的处境 | 走哪条路 |
| --- | --- |
| 只有一句话 / 模糊需求 | §2 输入策略 → §3 判模式 → §5–§9 写 → §12 输出 |
| 有剧本 / 长故事，超过一次生成 | §2 → §3 → **§10 多段无缝** → §12 输出多段版 |
| 有首帧 / 首尾帧 / 多参考素材 | §3 判模式（I2VA/FL2VA/L2VA/Ref2VA）→ §4 结构与固定首行 |
| 已有提示词，想知道哪里不合规 | **§14 工具链**：`validate_h3_prompts.py` 体检 |
| 想换风格（2D/实拍/像素/水墨…） | **§11 风格无关**：只换风格层，指向 `references/style-presets.md` |
| 提示词写好了，要填进 ComfyUI | **§14**：`insert_h3_prompts.py` 或双击启动器 |

---

## 1. 输入策略

接受一句话、长提示词、故事、剧本、对白、广告创意、分镜表、参考图，以及"缩短 / 扩写 / 拆分 / 续接 / 修复"的请求。**不要要求用户懂提示词术语。**

- 合理默认值自行推断，并**简短声明**你推断了什么。
- 只有"缺失的答案会实质改变结果"时，才问**最多一个**简短问题；否则直接做。
- **绝不要说你看得见并不存在的素材**（图/视频/音频），也不要发明不存在的参考标签。
- 用户给的对白**原文照用**，不擅自翻译、润色、补台词（只有明确要求改写时才改）。
- 字段正文用**英文**；对白、歌词、画面可见文字**保留原语言**。

---

## 2. 第一步：判定生成模式

```text
没有图片 / 视频 / 音频参考？            → T2VA（文本生成完整音视频）
只有一张图？
  ├─ 图是第 0 秒的真实首帧            → I2VA（首帧向前发展）
  └─ 图是视频结尾的最后状态            → L2VA（从合理前态收敛到尾帧）
两张图分别锁定开头与结尾？              → FL2VA（首尾帧之间建立可见路径）
需要多图角色 / 参考视频 / 音色 / 动作 / 场景 / 编辑关系？ → Ref2VA（全参考模式，六段式）
```

**最容易判错的一步**：不要因为"有一张角色参考图"就选 I2VA。若那张图只定义**人物身份**、并不是第 0 秒的具体画面，应走 Ref2VA 的 `<Subject N>` 关系。判据是**这张图在时间轴上扮演什么角色**，不是"有没有图"。

判完模式，先建**素材角色表**（写进工作文件或脑子里，别写进提示词正文）：

| 素材 | 角色 | 允许继承 | 不应继承 |
| --- | --- | --- | --- |
| Picture 1 | 主角身份 | 脸、发型、服装、比例 | 背景文字、平台水印 |
| Picture 2 | 场景参考 | 建筑、布局、色彩 | 图里的路人身份 |
| Video 1 | 运镜参考 | 镜头路径、剪辑节奏 | 原视频人物与品牌 |
| Audio 1 | 音色参考 | 音色、语速、情绪 | 原台词内容 |

一个素材尽量只承担**一个主要职责**；职责冲突就拆开或定优先级。上传顺序与职责要**在提示词之前列清**：

```text
图片上传顺序：
@图片1：人物主参考，锁定面容、发型、服装和体型
@图片2：场景参考，锁定空间、色彩和光线方向
```

> 深层依据与四模式完整定义见 `references/official-modes.md`（§3、§5、§6）。

---

## 3. 字段与结构（不许发明）

### Base 模式（T2VA / I2VA / FL2VA / L2VA）——官方三段式

```text
integrated_multimodal_description: [Shot 1] ...

overall_soundscape: ...

non_diegetic_music: ...
```

| 字段 | 写什么 | 硬约束 |
| --- | --- | --- |
| `integrated_multimodal_description` | 主时间轴：视觉风格、初始构图、人物物体、场景灯光、动作反应、镜头切换与运镜、说话人对白歌唱、与画面同步的画内声音 | 每个细节都要**看得见或听得见**；不要塞项目管理说明、模型参数、验收口号、抽象愿望 |
| `overall_soundscape` | 1–4 句英文：环境底噪、风雨交通机器混响、脚步衣料碰撞、呼吸笑声等非语言人声 | **不要**重复完整对白、歌词或画内音乐 |
| `non_diegetic_music` | 只给观众听、角色听不到的配乐：乐器、速度、节奏、强弱变化、进出时机 | 不许只写 `epic`/`emotional`/`beautiful`；没有配乐就写 `N/A` |

### 模式固定首行（I2VA / FL2VA / L2VA）

官方要求这句**位于最终 Prompt 第一行**，其后空一行再写三段式；`N` 是实际最后一个 shot 编号，`S.SS` 是有效时长（严格两位小数）。

```text
# I2VA
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.

# FL2VA
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot N) aligns with the S.SS-second mark of the target video.

# L2VA
How the reference pictures align with the target video — <Picture 1> (from [Shot N]) aligns with the S.SS-second mark of the target video.
```

> ⚠️ **方言冲突（本机实测）**：官方 API / SGLang 路径**必须**写上面这句对齐句。但**本机 ComfyUI + Herrgotts 续接套件**那条路（`MiniMaxH3-小黑8G显存-10/30段无缝流`）首尾帧是由**节点注入**的，实测可用写法是**不写**对齐句，直接 `integrated_multimodal_description: [Shot 1] <风格锁> …`。**同一批提示词不要一半写、一半不写**——按你的执行路径二选一。ComfyUI 侧的 token 与节点事实见 `references/h3-prompt-spec.md`。

### Ref2VA——官方六段式

结构、四类标签（`<Subject N>` / `<Picture N>` / `<Video N>` / `<Audio N>`）、`subject_definitions` / `summary` / `retention_analysis` / `detailed_description` 的格式与固定关系值、Speaker 分配与官方完整案例，全部见 `references/ref2va.md`（§13、§25）。**Ref2VA 不要套用 Base 三段式。**

---

## 4. 第二步：先写时间轴，再写美术描述

先产出一条最小时间轴，再给每段补细节：

```text
0.0–4.5 秒：建立人物和问题
4.5–10.0 秒：执行核心动作
10.0–15.0 秒：结果、反应和结尾锚点
```

每段补：当前景别与构图、主体位置与朝向、动作起点与终点、摄影机运动、同步声音、台词与口型、需要保持的状态。

**Shot 语法**（细节见 `references/shot-continuity-sound.md` §7、§8）：

- **`[Shot 1]` 不写时间戳**；后续 Cut 严格递增：`[Shot N] At 00:SS.mmm, ...`。
- 所有时间点必须严格递增并落在总时长内。
- **值得切镜**：景别/视点/地点/时间/主体/已完成动作发生可见变化。**只有景别小变化时优先运镜，而不是切镜**。
- 官方倾向单镜头；15 秒内不要同时要求多个地点、复杂打斗、长对白和连续变装。
- 运镜写**类型 + 幅度 + 速度**的自然动作句；避免互相冲突的运镜（如同时推近又拉远）。

---

## 5. 第三步：写可观察指令（七步顺序）

按这个顺序写正文，前一步没写清不要跳到下一步：

1. 主体 + 一个核心事件
2. 场景与连续性状态
3. 时间线 / 镜头流：具体动作、反应、终点
4. 服务于故事的摄影机行为
5. 灯光、色彩、质感、视觉风格
6. 说话人、口型、环境声、动作声、配乐
7. 稳定性与排除项

**把形容词换成可生成信息**：`震撼`、`高级`、`电影感` 这类词一律替换为具体构图、运动、色彩、光线、节奏、质感。动作要写**方向、身体重量、因果、反应、与环境互动、收尾姿态**；对白要写**视线、呼吸、停顿、微表情、语气、逐字台词、口型**；转场要指出**可见的匹配元素**，而不是只写"丝滑转场"。

---

## 6. 第四步：锁住连续性

- **角色锁定卡**：第一次出现就一次性建立稳定身份（面容、发型、服装、体型、关键道具），后续用**同一描述或固定称呼**延续，不要每段换形容词。
- **把连续性写成状态，不是口号**：不写"保持一致"，写"雨仍在下、湿发贴在额前、刀尖未动"。
- **对象状态机**：物体在每个镜头里的位置/姿态/是否损坏要能接上，不许瞬移。
- **屏幕轴线**：对话双方的左右关系、运动方向不要越过轴线；轴反转是多段长片最常见的崩坏之一。

细节（含状态机写法、轴线判据）见 `references/shot-continuity-sound.md` §9。

---

## 7. 第五步：对白、旁白与声音分层

### 对白：`<d>` 只放语言与台词

```text
The calm young man (S1) says: <d>[Chinese] 我们现在出发。</d>
```

- **`<d>` 外**写：谁说、音色、年龄感、语速、情绪、动作与表情。
- **`<d>` 内只写**：`[Language]` + 用户要求的完整台词。**不要把表演说明塞进 `<d>`**。
- 对白**只在 `<d>` 中出现一次**；同一句跨 Cut 用 `<scenetrans>` 并声明声音无缝延续，**不要两个镜头各抄一遍完整台词**（会读两遍）。
- 台词被片尾故意截断用 `<cutoff>`，不要让长台词自然溢出时长。
- 说话人 ID 全程稳定：`(S1)`、`(S2)`… 同一角色从头到尾同一编号。

> token 记法差异：官方 Guide 写 `<cutoff>` / `<scenetrans>`；本机 ComfyUI tokenizer 注册的是带竖线的 `<|cutoff|>`（见 `references/h3-prompt-spec.md` 的 token 表）。落地到 ComfyUI 时以本机 token 表为准。

### 旁白：官方固定写法

```text
The man (S1) says in an off-screen voiceover: <d>[Chinese] 我仍然记得那条路。</d> while his lips remain completely closed.
```

非画面角色的旁白要定义成**稳定的旁白声源**（如 `A calm adult female narrator with a warm, measured Mandarin delivery (S1)`）。**必须说明画面人物嘴唇闭合**，否则旁白会被错误绑定到人物口型上。

### 声音分三层，别混

| 层 | 归属 | 写在哪 |
| --- | --- | --- |
| 环境声 / 动作声 / 非语言人声 | 画内 | `overall_soundscape` |
| 对白 / 歌唱 / 画内音源音乐 | 画内、与画面同步 | `integrated_multimodal_description`（`<d>` 内） |
| 画外配乐 | 只有观众听见 | `non_diegetic_music` |

三层之间**不要重复**：`overall_soundscape` 里不再抄对白与画内音乐。口型稳定技巧：一个镜头只让一个主体承担主要台词；台词期间不要同时高速跑动、遮脸、转身、大幅运镜；台词结束写 `closes his mouth naturally`。

---

## 8. 第六步：画面文字与排除项

- **默认不生成字幕、说明文字、Logo、水印**（默认排除项见 §12）。
- 需要画面文字时，必须指定**逐字内容、位置、出现时机、消失时机**；同一文案在不同段必须逐字一致。
- 海报 / 招牌的字写在**画面描述**里（`a poster reading "…"`）；真正的字幕才用 `<|caption_start|>`/`<|caption_end|>` 一类 token。
- 生成文字的风险（乱码、错字、风格违和）与官方规则见 `references/shot-continuity-sound.md` §11。

---

## 9. 多段无缝长片（本工具包的核心能力）

一次生成只有 **4–15 秒**（官方规格），更长的片子必须拆段——而拆段不是"切成几块"，是**把状态接起来**。

### 拆段判据

- 每段定义为一个**完整的微事件或情绪节拍**。
- **绝不**把同一个未完成的构图、姿态、动作阶段、情绪状态拆到两次独立生成里。
- 切点放在**可见变化处**：景别、视点、地点、时间、主体、或一个动作**已完成**。
- 复杂度可控时**优先用更长的连贯段**，而不是更多短段。

### 11 槽段语法（每段一条，顺序即权重）

| # | 槽位 | 内容 |
| --- | --- | --- |
| 1 | `[Shot N]` 标记 | 段落序号或时间码 |
| 2 | **风格锚** | 风格名 + 3–6 个具体属性 |
| 3 | **漂移禁令** | 该风格最易漂去的 2 个方向 |
| 4 | **世界锚** | 场景/背景/渲染一致性 + 排除项 |
| 5 | **图文锚** | 画内文字/海报样式与逐字字符串 |
| 6 | **主体动作节拍** | 谁 + 做什么 + 道具 |
| 7 | **景别与运镜** | `Medium shot` / `low angle` / 推拉摇移 |
| 8 | **结尾定格** | `holds the … pose at the end` |
| 9 | **渲染卫生** | `all sharp, no motion blur` |
| 10 | **台词** | `(S1) says: <d>[Chinese] …</d>` |
| 11 | **声音** | `overall_soundscape:` + `non_diegetic_music:` |

### 三级锁（源十段逐字核对得出）

| 级别 | 用在 | 写法 |
| --- | --- | --- |
| **建立锁**（4 句） | 每条链第 1 段 | 风格锚 + `Keep {风格} locked in every clip; never drift to {A} or {B}.` + `The world is {世界} …` + `On-screen {文字载体} … identical across all clips.` |
| **完整引用锁**（3 句，推荐） | 第 2 段起 | `Identical {风格} to the previous clip, do not drift toward {A} or {B}.` + `Same {世界} as previous clips.` + `Same {图文样式} as previous clips.` |
| **简写引用锁**（可用不推荐） | 第 2 段起 | `Identical {风格} to the previous clip.` + `Same {世界}.` + `Same {图文样式} style.` |

`Identical … to the previous clip.` 是**地基句**，任何级别的引用锁都不能省。一个文件可以装多条独立链：**自带建立锁且不含 `previous clip` 的段会被判为新链起点**（校验脚本给 `R16` 提示）。

### 7 条连贯硬规则

1. **每段自包含**：被掩码的前序 AV 内容**已经在 latent 里**，不要描写、不要写"承接上文"。推论：单独重出第 5 段不需要动前四段。
2. **第 2 段起三句承接缺一不可**（风格 / 世界 / 图文）。缺风格承接 → 从该段起换画风；缺世界承接 → 背景重画。
3. **音乐是同一条**：`Same {曲风} as previous clip, low volume, continuous.`；音量档位词全片固定；**只有末段**允许 `resolves to …`。
4. **环境音不重启**：写延续/叠加（`room tone continuing`），不写"新的环境音开始"。
5. **定格–起手配对**：上段结尾定格姿势必须是下段起手能延续的姿势。
6. **台词不越 39 帧边界**（≈1.625s @24fps，`audio_tail_carryover="Full Previous Tail"`）：跨段台词必须在**源片段内部**说完，**每段台词以句末标点收口**，不要把半句话留到下一段。
7. **画内字符串逐段一致**：同一张海报每次出现逐字相同；海报文案 ≤ 12 字/条、单段 ≤ 3 条。

> 机制依据（39 帧合法边界、掩码前序不需提及、节点参数）见 `references/h3-prompt-spec.md` 与 `references/params-baseline.md`。

---

## 10. 风格无关：只换风格层

槽 2/3/4/5 是**唯一**需要随风格替换的部分；结构层（§3、§9）与连贯层（§9 三级锁、7 规则）一字不动。

- 10 种风格预设（2D 动漫 cel / 3D CG / 写实实拍 / 国风水墨 / 黏土定格 / 像素 / 水彩绘本 / 赛博霓虹 / 纪录片手持 / 产品广告棚拍）+ 自定义槽，每种给**可直接粘贴的英文**风格锚、漂移禁令（点名该风格最易漂去的两个方向）、世界锚、图文锚、质感、卫生、排除项：`references/style-presets.md`。
- 换风格实证（同一套结构层与连贯层换成实拍与像素风）：`examples/demo-other-styles.md`。

---

## 11. 输出契约

### 单次生成（默认）

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

### 多段长片

````text
【整体设定】角色、场景、画面、声音与跨段连续性锚点

【第1段｜10s｜完整事件名称】
integrated_multimodal_description: [Shot 1] <建立锁 4 句> <主体动作> <景别> <结尾定格>, all sharp, no motion blur. (S1) says: <d>[Chinese] …</d>

overall_soundscape: …
non_diegetic_music: …

【第2段｜10s｜…】
<完整引用锁 3 句> …
````

**每段必须自包含**：不写"同上"、不引用其它段。正文里不要出现 markdown 标题、列表符号、加粗——它们会被当成提示词内容送进编码器。**用户只要最终 Prompt 时，省略所有解释**。

---

## 12. 审计与验收

**生成前静态检查**（7 大类：模式 / 结构 / 时间 / 主体 / 对白 / 音频 / 参考素材）与**生成后视听验收**（4 大类：文件层 / 画面层 / 音频层 / 连续性层）逐条清单见 `references/templates-checklists.md`（§17、§18）。常用排除项：

```text
no subtitles, no captions, no logos, no watermarks, no on-screen text, no extra limbs, no malformed hands, no identity drift, no style drift
```

**常见失败与修正**（身份漂移、口型串台、动作瞬移、轴反转、文字乱码、风格漂移…）见同文件 §16。**Prompt 通过 ≠ 结果通过**：真实帧率、首尾帧是否命中、台词是否截断、有无字幕/乱码/黑帧，都要看成片。

---

## 13. 最重要的十条原则

1. 先选模式，再写 Prompt。
2. 先写时间轴，再写风格词。
3. Base 模式只用官方三段式。
4. Ref2VA 使用官方六段式和稳定标签。
5. Shot 1 无时间戳，后续 Cut 严格递增。
6. 对白只在 `<d>[Language] ...</d>` 中出现一次。
7. 旁白使用官方 off-screen voiceover 句式并锁住嘴唇。
8. 关键帧要描述**变化路径**，而不是重复描述两张图片。
9. 一个参考素材最好只有一个主要职责。
10. Prompt 是生成规格，最终质量必须靠生成后 QA 验证。

---

## 14. 工具链（可执行验证，不是靠眼睛看）

三个脚本都在 `scripts/`，构成一条流水线：**导出 → 体检 → 回填 → 复验**。

| 脚本 | 一句话 | 最常用命令 |
| --- | --- | --- |
| `extract_h3_prompts.py` | 把工作流里的提示词**导出来**（只读，段号按 `Set_提示词N` 排） | `& $py "$kit\scripts\extract_h3_prompts.py" $wf -o "基准.md"` |
| `validate_h3_prompts.py` | **体检**提示词或工作流：段语法 + 跨段连贯 + 链边界，只报问题不改东西 | `& $py "$kit\scripts\validate_h3_prompts.py" $wf` |
| `insert_h3_prompts.py` | 按段号**回填**进 `Text Multiline`（自动备份 + 回读校验 + 最小改动校验） | `& $py "$kit\scripts\insert_h3_prompts.py" --workflow $wf --prompts "稿子.md"` |

**双击即用**：`scripts\插入30段提示词.cmd` —— 零参数，自动找「无缝流」工作流（段位最多者，排除 `backup\`）与脚本目录里的 `.md`，原地插入并备份，窗口不会一闪而过。

```powershell
$py  = "D:\ComfyUI-aki-v3\python\python.exe"
$kit = "<本工具包目录>"
$wf  = "<工作流>.json"

& $py "$kit\scripts\validate_h3_prompts.py" $wf          # 体检工作流
& $py "$kit\scripts\insert_h3_prompts.py" --workflow $wf --prompts "稿子.md" --dry-run   # 预览
& $py "$kit\scripts\insert_h3_prompts.py" --workflow $wf --prompts "稿子.md"             # 原地写入（自动备份）
```

- **退出码**：`0` 正常（含"只有 WARN"）；`1` 有 FAIL 或参数错误。可直接当门禁。
- 校验规则号速查、报告读法、`--min-dialogue` 等参数、常见问题：`USAGE.md`。
- ⚠️ 写入前**先在 ComfyUI 里关掉该工作流**（或用 `--out` 写新文件）；从界面保存会用界面里的旧内容覆盖写入。

---

## 15. 文件地图

| 文件 | 覆盖什么 |
| --- | --- |
| `references/official-modes.md` | 官方能力边界与三层系统、Base 三段式字段定义、**四种 Base 模式**（固定首行 / 状态结构 / 模板 / 适用与不适用）、官方 Base Prompt Guide 与四个完整案例 |
| `references/ref2va.md` | **Ref2VA 全参考模式**：六段式、四类标签、`subject_definitions`/`summary`/`retention_analysis`/`detailed_description`、参考冲突处理、Speaker 与声音段、官方完整案例 |
| `references/shot-continuity-sound.md` | Shot 与时间轴语法、15 秒信息预算、摄影机语言与运镜词表、连续性状态机与屏幕轴线、说话人与口型、画面文字、声音分层、**Prompt 底层原理六条** |
| `references/templates-checklists.md` | 六步工作流、各模式生产模板与中文对白示例、常见失败与修正、**生成前审计清单 / 生成后验收清单**、Agent 执行规范、官方来源索引与版本快照 |
| `references/style-presets.md` | 10 种风格预设 + 自定义槽（英文可粘贴句） |
| `references/h3-prompt-spec.md` | **本机实测事实**：token 表、三种 conditioning 结构、39 帧边界、分辨率与帧网格、工作流接线与节点参数 |
| `references/params-baseline.md` | 段数 × 时长 × 画幅建议值、续接节点默认值、时长↔帧数换算 |
| `templates/segment-template.md` | 逐槽填空模板（建立锁 / 完整引用锁 / 简写变体 / storyboard 方言） |
| `examples/source-ten-prompts.md` | 源十段原文（回归基准与风格锁范例） |
| `examples/demo-other-styles.md` | 换风格实证（实拍 + 像素，附校验结果） |
| `USAGE.md` | 三个脚本的参数、报告读法、规则号速查、常见问题 |
| `scripts/` | `validate` / `extract` / `insert` 三脚本 + 双击启动器 |

> 来源与版本：本文件整合自 ① 本工具包原 `h3-seamless-prompt`（多段无缝规范与工具链）② 《提示词工程技巧.md》（官方 H3 提示词工程完整中文覆盖，含 T2VA/I2VA/FL2VA/L2VA/Ref2VA、官方 Base Prompt Guide 与 Ref2VA Guide，版本快照 2026-08-31）③ `minimax-h3-prompt-standardizer`（新手工作流与输出契约）。官方结构相关内容以官方文档为准；标为"工程建议/本机实测"的内容是在官方结构之上总结的生产经验与本地验证结果。