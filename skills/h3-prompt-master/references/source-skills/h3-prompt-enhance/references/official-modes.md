# 官方四模式与三段式字段（T2VA / I2VA / FL2VA / L2VA）

本文件是 H3 提示词工程的 reference，覆盖官方能力边界速查、H3 三层系统、Base 模式官方三段式字段、四种 Base 模式（T2VA / I2VA / FL2VA / L2VA）的官方固定首行与状态结构，以及官方 Base Prompt Guide 的完整中文覆盖（含四个完整案例原文）。
什么时候读：需要在四种 Base 模式之间选型、写图像对齐固定首行、拼装 `integrated_multimodal_description` / `overall_soundscape` / `non_diegetic_music`，或核对官方字段定义、运镜词表、说话人规则与官方案例时读它。
配合文件：与 `SKILL.md` 配合做任务选型与总流程；镜头、切点与声音分层的展开细节见 `shot-continuity-sound.md`；参考素材（Ref2VA）用法见 `ref2va.md`。
本文件只保留事实、字段定义、模板、词表与官方案例原文，不做摘要式改写；区间外的小节不在此重复。

## 3. 官方能力边界速查

根据官方仓库当前说明：

| 项目 | 官方规格 |
|---|---|
| 单次输出时长 | 4–15 秒 |
| 帧率 | 24 FPS |
| 输出音频 | 32 kHz 立体声 |
| 常见画幅 | 21:9、16:9、4:3、1:1、3:4、9:16 等 |
| Base 输出 | 以 768p 为主要基准 |
| 2K | 官方完整系统通过 H3-Regenerate-2K 再生成，不等同于普通插值放大 |
| 稳定对白语言 | 中文、英语、日语、韩语、法语、德语、意大利语、西班牙语、葡萄牙语、俄语、阿拉伯语等 |
| FL2VA 条件 | 0、1 或 2 张图，分别对应 T2VA、I2VA/L2VA、FL2VA |
| Ref2VA 图片 | 最多 9 张 |
| Ref2VA 视频 | 最多 3 段，每段 2–15 秒，总时长不超过 15 秒 |
| Ref2VA 音频 | 最多 3 段，每段 2–15 秒，总时长不超过 15 秒 |
| Ref2VA 混合素材 | 所有输入文件合计最多 12 个 |

### 3.1 H3 三层系统

官方把完整系统描述为三部分：

1. **H3-Context-IR**：理解自由形式的文本、图片、视频和音频关系，把它们整理成 H3 更容易消费的上下文中间表示。
2. **H3-Base**：根据上下文联合生成视频 latent 与音频 latent，再分别解码为视频和立体声音频。
3. **H3-Regenerate-2K**：把 Base 视频与原始上下文再次交给 H3，以 in-context 方式重新生成高分辨率细节。

## 5. Base 模式的官方三段式

T2VA、I2VA、FL2VA、L2VA 的正文固定由以下三部分构成：

```text
integrated_multimodal_description: [Shot 1] ...

overall_soundscape: ...

non_diegetic_music: ...
```

### 5.1 `integrated_multimodal_description`

这是主时间轴，包含：

- 视觉风格
- 初始构图
- 人物和物体
- 场景与灯光
- 动作和反应
- 镜头切换与运镜
- 说话人、对白、歌唱
- 与画面时间严格同步的画内声音

原则：这里的每个细节都应该能被观众**看见或听见**。不要把项目管理说明、模型参数、验收口号和抽象愿望混入正文。

### 5.2 `overall_soundscape`

用 1–4 句英文连续描述全片的：

- 环境底噪
- 风雨、交通、机器、室内混响
- 脚步、衣料、碰撞、开门、物体操作
- 呼吸、笑声、喘息等非语言人声

不要在这里重复完整对白、歌词或画内音乐。

### 5.3 `non_diegetic_music`

描述只给观众听、角色听不到的配乐。应具体写：

- 乐器
- 速度
- 节奏
- 强弱变化
- 进入和退出时机

不要只写 `epic`、`emotional`、`beautiful`。如果没有画外配乐，写：

```text
non_diegetic_music: N/A
```

## 6. 四种 Base 模式

## 6.1 T2VA：文本生成完整音视频

### 官方定义

没有关键帧输入，由文本建立完整视听时间轴。

### 推荐状态结构

```text
初始世界状态
→ 动作触发
→ 可见变化
→ 结果或反应
```

### 模板

```text
integrated_multimodal_description: [Shot 1] Live-action, cinematic, ...
[Shot 2] At 00:05.000, the camera cuts to ...

overall_soundscape: ...

non_diegetic_music: ...
```

### 适用技巧

- Shot 1 必须把人物、场景、光线和初始构图说清楚。
- 角色第一次出现时一次性建立稳定身份，后续用同一描述或固定称呼延续。
- 每个镜头应承担新信息；只有景别小变化时，优先运镜而不是切镜。
- 15 秒内不要同时要求多个地点、复杂打斗、长对白和连续变装。

## 6.2 I2VA：首帧向前发展

### 官方固定首行

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.
```

这句话必须位于 Prompt 第一行，其后空一行，再写三段式正文。

### 推荐状态结构

```text
首帧锚点
→ 动作开始
→ 连续发展
→ 结果或反应
```

### Prompt 要点

1. 先承认 `<Picture 1>` 已经给出了人物、构图和场景。
2. 描述从图片之后发生的变化，而不是用新形容词重新设计图片。
3. 锁定人物身份、服装、颜色、关键道具和空间关系。
4. 首帧中不存在的内容，必须说明它如何进入画面。

### 工程建议：真实尾帧链式接力

长视频可把上一段真实最后一帧作为下一段 `<Picture 1>`：

```text
Clip N 的真实最后帧
→ Clip N+1 的 I2VA 首帧
```

优点：

- 构图和人物位置连续；
- 不需要额外重绘桥接图；
- 转场关系最直观。

风险：

- 尾帧中的模糊、字幕、闭眼、穿模也会被继承；
- 片尾动作若仍高速运动，下一段容易抖动或突变；
- 视频编码最后帧不一定是最适合作为叙事锚点的帧。

生产上可提供两种策略：

- **真实尾帧模式**：严格使用最后一帧，连续性强，但会继承瑕疵。
- **稳定尾帧模式**：在最后若干帧中选择清晰、无遮挡、动作趋稳的帧，质量更稳，但不再是绝对最后帧。

应由项目需求明确选择，不要在没有说明时混用。

## 6.3 FL2VA：首尾帧之间建立可见路径

### 官方固定首行

```text
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot N) aligns with the S.SS-second mark of the target video.
```

`N` 是实际最后一个镜头编号；`S.SS` 是精确到两位小数的有效时长。

### 推荐状态结构

```text
首帧状态
→ 可观察中间变化
→ 差异逐步缩小
→ 命中尾帧状态
```

### 核心原则

- 不要只把两张图分别描述一遍；重点是“中间如何发生”。
- 官方倾向单镜头插值；只有用户明确需要时才使用多镜头。
- 尾帧不是灵感参考，而是最终必须到达的状态。
- 动作路径必须物理可行，例如手从桌面移动到门把手，而不是瞬间出现在终点。

### 适合 FL2VA 的动作

- 开门、合伞、转身、坐下、递物
- 表情从平静变为惊讶
- 灯光从白天过渡到夜晚
- 产品从闭合状态变为展开状态

### 不适合一段 FL2VA 强行完成的任务

- 两张图是完全不同地点、不同人物、不同镜头轴线；
- 首尾物体拓扑差异过大；
- 中间需要多次硬切和复杂叙事；
- 需要精准文字逐字变化。

## 6.4 L2VA：从合理前态收敛到尾帧

### 官方固定首行

```text
How the reference pictures align with the target video — <Picture 1> (from [Shot N]) aligns with the S.SS-second mark of the target video.
```

### 推荐状态结构

```text
合理前态
→ 明确动作路径
→ 最后镜头逐步收敛
→ 命中尾帧
```

### 常见用途

- 片尾定格
- 产品最终展示姿态
- 角色最后站位
- 预设 Ending Card 之前的动作收束

### 关键点

- `<Picture 1>` 属于最后的 `[Shot N]`，不是天然属于 Shot 1。
- 前态必须与尾帧在物理和叙事上兼容。
- 最后几秒要减少新信息，让模型有足够时间收敛到目标画面。

## 24. 官方 Base Prompt Guide 完整中文覆盖

## 24.1 四种任务

| 模式 | 官方任务定义 |
|---|---|
| T2VA | 从文本构建完整音视频时间线 |
| I2VA | T2VA 主体 + 首帧固定指令 + 从首帧向前发展的视觉路径 |
| FL2VA | T2VA 主体 + 首尾帧固定指令 + 从首帧连续过渡到尾帧的路径 |
| L2VA | T2VA 主体 + 尾帧固定指令 + 从合理前态收敛到尾帧的路径 |

## 24.2 最终 Prompt 结构

### T2VA

不写图像对齐指令，直接以三个核心字段开头。

### I2VA 固定首行

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.
```

### FL2VA 固定首行

```text
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot N) aligns with the S.SS-second mark of the target video.
```

### L2VA 固定首行

```text
How the reference pictures align with the target video — <Picture 1> (from [Shot N]) aligns with the S.SS-second mark of the target video.
```

其中：

- `N` 是实际最后一个 shot 的编号。
- `S.SS` 是有效视频时长，严格保留两位小数。
- 指令必须位于最终 Prompt 第一行。
- 固定指令后空一行，再写核心字段。

### 三个核心字段

```text
integrated_multimodal_description: [Shot 1] ...

overall_soundscape: ...

non_diegetic_music: ...
```

- `integrated_multimodal_description`：按时间写视觉、动作、镜头、说话人、对白、歌唱和画内音频。
- `overall_soundscape`：总结环境、物理动作和非语言人声。
- `non_diegetic_music`：描述角色听不到、只有观众听到的配乐。

## 24.3 关键帧融入方式

### I2VA

`<Picture 1>` 是目标视频 0.00 秒真实首帧，属于 `[Shot 1]`。先建立图中的风格、主体、构图、场景锚点，再向后描述动作。人物身份、服装、颜色、关键物体、空间关系应保持一致。

```text
首帧锚点 → 动作启动 → 连续发展 → 结果或反应
```

### FL2VA

Picture 1 是开头，Picture 2 是结尾。重点写：主体怎么移动、姿势怎么变化、物体如何被操作、构图如何演化、场景或光线如何过渡。

官方通常更偏好单镜头插值；只有用户明确要求时才使用多镜头。结尾必须在视频末尾由最后 `[Shot N]` 到达。

```text
首帧状态 → 可观察的中间变化 → 差异逐渐缩小 → 尾帧状态
```

### L2VA

`<Picture 1>` 是目标视频最后一帧，属于最后 `[Shot N]`，并不天然属于 Shot 1。根据用户意图与尾帧推断前态，再让人物、物体、相机和场景逐步接近参考图。

```text
合理前态 → 明确动作/过渡路径 → 最后 Shot 中逐渐收敛 → 准确落到尾帧
```

## 24.4 主描述必须沿时间线展开

每个细节都应对应可见或可听事实：

- 视觉风格。
- 起始构图。
- 主体外观与位置。
- 场景和关键道具。
- 动作与反应。
- 镜头变化。
- 语言与对白。
- 同步画内声。

`[Shot 1]` 开始处应说明整体风格和初始构图。官方列举的风格包括：`Cinematic`、`live-action`、`2D-animated`、`3D CG`、`claymation`、`watercolor`、`vintage film`。关键帧任务从参考图推断风格；T2VA 从用户文本选择。

```text
[Shot 1] Live-action, cinematic, a medium-wide shot frames...
```

## 24.5 Shot 与 Cut

- Shot 1 不加时间戳。
- 后续 shot 编号连续。
- 每个后续 shot 必须用严格递增且位于视频时长内的切点开始。

```text
[Shot 2] At 00:03.500, the camera cuts to...
```

普通切镜可以使用：

- `the camera cuts to`
- `the shot cuts to`
- `the shot transitions to`
- `the shot changes to`
- `the shot switches to`

只有用户明确要求时才使用 cross-dissolve、fade 或 wipe。切镜应带来主体、空间、状态、视角或时间的新信息；如果只是轻微角度或景别变化，优先使用相机运动而不是切镜。

## 24.6 运镜词表：类型 + 幅度 + 速度

| 维度 | 官方表达 | 含义 |
|---|---|---|
| 类型 | `Zoom In / Zoom Out` | 机位不动，改变焦距 |
| 类型 | `Push In / Pull Out` | 相机前移 / 后移 |
| 类型 | `Pan Left / Pan Right` | 机位不动，镜头水平转动 |
| 类型 | `Truck Left / Truck Right` | 相机横向平移 |
| 类型 | `Tilt Up / Tilt Down` | 机位不动，镜头垂直转动 |
| 类型 | `Pedestal Up / Pedestal Down` | 整台相机上移 / 下移 |
| 类型 | `Arc Shot` | 围绕主体做弧线运动 |
| 类型 | `Tracking Shot` | 跟随移动主体 |
| 类型 | `Static Shot` | 机位与镜头保持静止 |
| 类型 | `Shake Slightly / Shake Strongly` | 轻微 / 强烈震动 |
| 类型 | `POV` | 主体主观视角 |
| 类型 | `Roll Clockwise / Roll Counterclockwise` | 围绕镜头轴顺 / 逆时针滚转 |
| 幅度 | `with small amplitude` | 小范围构图变化 |
| 幅度 | `with large amplitude` | 大范围构图变化 |
| 速度 | `at slow speed` | 慢速 |
| 速度 | `at fast speed` | 快速 |

中等幅度和正常速度通常省略。运镜要写成 shot 内自然英文动作，不要在句尾堆标签：

```text
The camera pushes in with small amplitude at slow speed toward the folded letter in her hands.
The camera pans right with large amplitude at fast speed, revealing the open doorway.
The camera holds a static shot as the runner exits the frame.
```

## 24.7 说话人、对白与歌唱

- 发声的人物、歌者或画外人声使用稳定 `(S1)`、`(S2)`。
- 多名已编号说话人同时发声使用 `(S1,S2)`。
- 同一发声者跨 shot 保持相同 ID。
- 从不发声的人不分配 speaker ID。
- 首次出现要用人物类型、年龄、性别、是否在画面、音高、音色、语速或口音等信息建立身份。
- 说话人身份、ID、动作、语气写在 `<d>` 外。
- `<d>` 内只放语言标签和用户原始台词。
- 用户原词与标点必须逐字保留，不翻译、不改写。

```text
The young woman with a quiet, breathy voice (S1) says: <d>[English] I get off at the next station.</d>
The two children (S1,S2) shout together, <d>[English] Wait for us!</d>
```

### 画外旁白固定写法

必须使用短语 `says in an off-screen voiceover`。每个旁白 `<d>` 后立刻说明对应画面人物嘴唇完全闭合：

```text
The man (S1) says in an off-screen voiceover: <d>[English] I still remember that road.</d> while his lips remain completely closed.
```

### 跨切镜对白

同一句对白或歌词跨 cut 时：

1. 两部分连接点都用 `<scenetrans>`。
2. 明确音频跨切镜连续。
3. 可用 `continues seamlessly across the cut`、`continues uninterrupted into the next shot`、`carries over from the previous shot`、`remains audible across the transition`。
4. 视频结束前台词被截断时用 `<cutoff>`。

## 24.8 画面文字

Banner、sign、label、subtitle、neon text 等实际可见文字必须放入英文双引号，原文和标点逐字保留，不翻译：

```text
A red neon sign reading "营业中" glows above the doorway.
```

## 24.9 `overall_soundscape`

- 1–4 个英文句子。
- 单段连续文字。
- 总结全片环境声、物理动作声和非语言人声，例如风雨、交通、脚步、衣物、碰撞、呼吸、笑声、喘息。
- 对白、歌唱、画内音乐已经属于主描述，不在这里重复。
- 只有用户明确要求全片完全静音时才写 `N/A`。

```text
overall_soundscape: Steady rain taps against the café windows while low room ambience continues underneath. The entrance bell rings once, followed by wet footsteps and the soft scrape of a chair.
```

## 24.10 `non_diegetic_music`

- 1–3 个英文句子。
- 只写角色听不到、观众听到的画外配乐。
- 聚焦乐器、速度、节奏与动态变化。
- 不用抽象 mood words，也不解释配乐的情绪功能。
- 角色能听到的歌唱、演奏、收音机、电视或手机音乐属于 diegetic event，应写进主描述。
- 没有画外配乐时写 `N/A`。

```text
non_diegetic_music: Sparse piano notes at a slow tempo, joined by sustained low strings that gradually increase in volume before fading out.
```

## 24.11 官方四个完整案例

以下保留官方 Prompt 的英文可执行形式。

### Case 1：T2VA 面包店

```text
integrated_multimodal_description: [Shot 1] Live-action, cinematic, a medium-wide shot frames a baker opening the shutters of a small street bakery before sunrise. The camera pushes in with small amplitude at slow speed as the middle-aged baker with a calm, slightly raspy voice (S1) places a fresh loaf on the wooden counter and says: <d>[English] First batch of the morning.</d> [Shot 2] At 00:05.000, the camera cuts to a close-up of steam rising from the sliced bread while the baker's final words carry over from the previous shot.

overall_soundscape: Wooden shutters scrape open over a quiet street as trays clink softly inside the bakery. The doorbell rings once, followed by light footsteps and the crisp sound of bread being sliced.

non_diegetic_music: A soft acoustic-guitar pattern at a moderate tempo, joined by sparse upright-bass notes and a gentle fade at the end.
```

### Case 2：I2VA 雨夜列车

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is fully referenced.

integrated_multimodal_description: [Shot 1] Live-action, cinematic, the young woman shown in <Picture 1> remains beside the rain-covered train window, preserving her appearance, clothing, seat position, and the carriage layout. The camera trucks right with small amplitude at slow speed as she lifts her gaze from the folded letter toward the passing city lights. Her reflection moves across the glass while the quiet, breathy young woman (S1) says: <d>[English] I get off at the next station.</d> She folds the letter along its existing crease.

overall_soundscape: The train wheels produce a steady metallic rhythm beneath a low ventilation hum. Rain ticks against the window while paper rustles softly in her hands.

non_diegetic_music: Sustained cello notes at a slow tempo with widely spaced piano tones, gradually decreasing in volume.
```

### Case 3：FL2VA 雨中自行车与雨伞

```text
How the reference pictures align with the target video — Picture 1 (from Shot 1) aligns with the 0.00-second mark of the target video; Picture 2 (from Shot 1) aligns with the 8.00-second mark of the target video.

integrated_multimodal_description: [Shot 1] Live-action, cinematic, a rain-soaked cyclist begins in the position and framing established by Picture 1, holding a closed black umbrella beside a silver bicycle. The camera pulls out with small amplitude at slow speed as she releases the bicycle handle, raises the umbrella above her shoulder, and presses the runner upward until the canopy opens. Water rolls from the expanding fabric while she steps beneath it, rotates the handle into the final angle, and settles into the pose, spacing, and composition established by Picture 2 at the end of the shot.

overall_soundscape: Rain falls steadily on the pavement, followed by the metallic click of the umbrella runner and the soft snap of the canopy opening. Water drips from the bicycle frame as distant traffic passes.

non_diegetic_music: N/A
```

### Case 4：L2VA 玻璃杯破碎并落到尾帧

```text
How the reference pictures align with the target video — <Picture 1> (from [Shot 1]) aligns with the 6.00-second mark of the target video.

integrated_multimodal_description: [Shot 1] Live-action, cinematic, a close shot begins with an intact drinking glass near the edge of a dark wooden table, while the same hand and sleeve visible in <Picture 1> approach from the right. The camera pushes in with small amplitude at slow speed as the fingertips strike the rim. The glass tips, falls, and hits the floor with a sharp impact; cracks spread through it as fragments slide outward. Toward the end, the moving pieces lose momentum and settle into the exact broken arrangement, hand position, camera angle, lighting, and final composition established by <Picture 1>.

overall_soundscape: Fingertips tap the glass before it scrapes across the tabletop, falls, and breaks with a sharp crash. Small fragments scatter and gradually stop sliding across the floor.

non_diegetic_music: A low electronic pulse at a slow tempo, ending immediately after the glass breaks.
```

## 来源

整理自《提示词工程技巧.md》§3.1、§5、§6、§24：

- §3 官方能力边界速查（含 §3.1 H3 三层系统）：源文件第 73–99 行
- §5 Base 模式的官方三段式：源文件第 203–257 行
- §6 四种 Base 模式（§6.1 T2VA、§6.2 I2VA、§6.3 FL2VA、§6.4 L2VA）：源文件第 259–418 行
- §24 官方 Base Prompt Guide 完整中文覆盖（含 §24.11 官方四个完整案例）：源文件第 1774–2041 行

源资料：`提示词工程技巧.md`（本地来源路径已省略）
