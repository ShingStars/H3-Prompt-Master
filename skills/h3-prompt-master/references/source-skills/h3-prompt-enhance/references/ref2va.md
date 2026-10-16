# Ref2VA 全参考模式

本文件覆盖 MiniMax H3 的 **Ref2VA（全参考）** 模式：六段式结构、四类 Reference Label（`<Subject N>` / `<Picture N>` / `<Video N>` / `<Audio N>`）、`subject_definitions`、`summary` 任务类型、`retention_analysis` 视觉/音频固定关系值、`detailed_description` 与参考关系冲突处理，并含官方完整 Ref2VA 案例。
何时用：当输入包含参考图/参考视频/参考音频，需要复用角色、场景、风格、动作、运镜、音色或音轨，或需要编辑/续写源视频时，使用 Ref2VA 而非 Base 三段式。
如何用：本文件是引用资料，不是摘要；写 Ref2VA 提示词时按 25.2 的固定顺序产出六段。
配合文件：流程与决策入口见 `SKILL.md`；Base 四种模式与模式选择见 `official-modes.md`；镜头、连续性、对白与声音的基础格式见 `shot-continuity-sound.md`（Ref2VA 沿用其格式，本文件只规定引用标签、分析段与全参考差异）。

## 13. Ref2VA：全参考模式

Ref2VA 不使用 Base 三段式，而使用六段式：

```text
subject_definitions:
...

summary:
...

retention_analysis:
...

detailed_description:
...

overall_soundscape:
...

non_diegetic_music:
...
```

所有重写字段使用英文；仅对白、歌词和真正可见的文字保留原语言。

## 13.1 四类标签

### `<Subject N>`

代表可复用的可见内容：

- 人物、动物、物体
- 场景和环境
- 服装、道具、界面、特效
- 风格、动作、表情或姿势

它描述“要追踪的内容单元”，不等同于源文件。

```text
<Subject 1> is the young woman in <Picture 1>, with long dark hair, a blue cardigan, and a thin silver necklace.
```

### `<Picture N>`

只有当图片本身承担具体帧锚点、构图锚点或分镜参考时，才单独定义 `<Picture N>`。

若图片只提供人物身份，不要多建 Picture 条目；在对应 `<Subject N>` 中引用图片即可。

### `<Video N>`

用于整个视频级关系：

- 编辑原视频
- 从原视频结尾继续
- 参考运镜、切点、节奏或时间结构

视频中的人物、道具和场景仍然应抽象为 `<Subject N>`，不能让 `<Video N>` 代替主体标签。

### `<Audio N>`

用于独立音频或启用的参考视频音轨：

- 完整或部分复制音频
- 参考音乐风格
- 参考音色和演绎
- 参考台词、歌词、音效
- 参考节拍、韵律或音频连续性

图片、视频和音频的编号分别独立，不表示彼此天然配对。

## 13.2 `subject_definitions`

每个需要追踪的主体单独一行，说明：

- 它是什么；
- 来自哪个素材；
- 该素材承担什么角色；
- 哪些特征需要保留。

不要在这里写剧情时间轴。

## 13.3 `summary`

用一个短英文段落总结目标视频和参考关系，并以任务类型开头：

```text
[reference generation] ...
[video continuation + keyframe completion] ...
[video editing + audio reuse] ...
```

官方任务关系包括：

| 类型 | 含义 |
|---|---|
| `keyframe completion` | 图片是首帧、关键帧、尾帧或编辑帧锚点 |
| `reference generation` | 素材只提供角色、场景、风格、动作、镜头等指导 |
| `video editing` | 直接编辑现有视频 |
| `video continuation` | 从现有视频继续、延长或衔接 |
| `audio reuse` | 直接复用全部或部分音频信号 |
| `audio reference` | 只参考音色、节奏、风格、内容或质感，不复制原信号 |

素材存在不等于自动产生任务类型。参考视频只提供运镜时，通常属于 `reference generation`，不是 `video editing`。

## 13.4 `retention_analysis`

### 视觉关系标记

| 标记 | 含义 |
|---|---|
| `fully_preserved` | 定义的参考角色完全保留 |
| `partially_preserved` | 仍使用参考，但部分特征改变或仅部分保留 |
| `attribute_transfer` | 把参考属性转移给另一个可识别主体 |
| `weak_reference` | 只保留宽泛风格、类别、构图或气氛 |

### 音频关系标记

| 标记 | 含义 |
|---|---|
| `fully_copy` | 源音频完整成为目标最终音轨 |
| `partially_copy` | 只复制部分时间或部分声音层，或复制后发生增删改 |
| `reference` | 只参考音色、节奏、音乐风格、台词内容或声音质感 |
| `weak_reference` | 只保留宽泛类别或气氛 |

每个标签一行，不要改变其在 `subject_definitions` 中已经定义的含义。

## 13.5 `detailed_description`

这是 Ref2VA 的主时间轴。与 Base 的差异：

- Base 主字段是 `integrated_multimodal_description`；Ref2VA 是 `detailed_description`。
- Ref2VA 先用 1–2 句英文定义总体风格，再进入 `[Shot 1]`。
- 每个参考主体首次清晰出现时，说明其位置、外观和当前动作。
- 在参考关系实际生效的时间段自然引用 `<Picture N>`、`<Video N>`、`<Audio N>`。

生成类任务通常可写约 350–500 个英文单词；对白密集任务优先满足完整时间轴，不必机械凑字数。这个范围来自官方 Ref Guide，不是所有简单编辑任务的硬要求。

## 13.6 参考关系冲突处理

当多张图或视频相互冲突时，先指定职责：

```text
<Picture 1> defines face and hairstyle.
<Picture 2> defines wardrobe and body proportions.
<Video 1> provides walking motion and camera rhythm only.
```

不要写“综合所有参考”后让模型自行决定，因为模型可能平均化脸型、混合服装或复制错误背景。

## 25. 官方 Ref2VA Guide 完整中文覆盖

## 25.1 总则

- 六个 rewrite section 全部使用英文。
- 只有 `<d>` 内的对白、歌词以及场景中真实可见文字保留原语言。
- `detailed_description` 要尽可能具体；每个 shot 明确当前构图、主体外观与位置、环境与光线、动作与状态变化、相机运动、当前声音，以及参考内容真正出现或生效的时点。
- 不要把主描述压缩成剧情梗概或参考关系列表。
- Shot、运镜、speaker、dialogue 与 ordinary sound 的基础格式沿用 Base Guide；Ref Guide 重点规定引用标签、分析段与全参考差异。

## 25.2 六段式及固定顺序

| 顺序 | 字段 | 作用 |
|---:|---|---|
| 1 | `subject_definitions` | 定义参考内容及标签 |
| 2 | `summary` | 概述任务类型、目标视频和主要参考关系 |
| 3 | `retention_analysis` | 说明参考内容如何被保留、迁移或复用 |
| 4 | `detailed_description` | 按播放顺序写视觉、动作、镜头、声音和对白 |
| 5 | `overall_soundscape` | 总结环境与物理声音 |
| 6 | `non_diegetic_music` | 描述只有观众听到的配乐 |

## 25.3 四类 Reference Label

| 标签 | 官方含义 |
|---|---|
| `<Subject N>` | 从参考素材抽象出的、可在目标视频复用或修改的可见内容 |
| `<Picture N>` | 作为具体目标帧或镜头规划锚点的参考图 |
| `<Video N>` | 提供直接编辑源、续写起点或全视频时间结构的参考视频 |
| `<Audio N>` | 被复制或被参考的音频信号 |

标签一旦分配，跨六段必须保持相同含义。

`subject_definitions` 中每个需要单独追踪的参考内容各写一行，说明：标签指代什么、承担什么参考职责、哪些主要特征必须遵循，以及必要时的来源素材。如果某个 Picture/Video 只用来定义另一个 Subject，后续不会单独分析或使用，就只在 Subject 定义中引用来源，不单列 Picture/Video。

### 25.3.1 `<Subject N>`

可用于：

- 人、动物、物体。
- 场景、背景、环境。
- 服装、道具、界面、视觉特效。
- 风格、动作、表情、姿势。

它表示目标视频实际使用的“内容单元”，而不是文件本身。一个 Subject 可由多个素材共同定义，一个素材也可提供多个 Subject。

```text
<Subject 1> is the young woman in <Picture 1>, with long dark hair, a blue cardigan, and a thin silver necklace.
```

多源定义时要写清每个来源提供什么：

```text
<Subject 1> is the woman whose appearance comes from <Picture 1> and whose walking motion comes from <Video 1>.
```

### 25.3.2 `<Picture N>`

只有当图片本身承担以下职责时才单列：

- 首帧。
- 关键帧。
- 尾帧。
- 被编辑的关键帧。
- 构图锚点。

```text
<Picture 2> is the first frame of [Shot 1], showing a woman seated beside a café window.
```

若图片只定义角色、场景、服装或风格，不单列 Picture，而是在 Subject 中注明来源。

若图片充当 storyboard / shot-planning reference，要写明映射哪些 shot，以及提供了视角、主体位置、镜头顺序中的哪些信息：

```text
<Picture 3> is a storyboard reference for [Shot 1] and [Shot 2], defining their viewpoint, subject placement, and shot order.
```

### 25.3.3 `<Video N>`

只保留给整段视频关系：

- 直接编辑原视频。
- 从原视频末尾续写。
- 参考原视频的运镜、切镜、节奏或时间结构。

```text
<Video 1> is the source video for the target video edit.
```

从视频中复用的人、物、场景、动作或效果仍应定义为 Subject。Video 只表示素材/结构来源，不能代替 Subject。

### 25.3.4 `<Audio N>`

表示独立音频或从参考视频中启用的同步音轨，常见职责：

- 完整或部分复制音频信号。
- 参考 BGM 风格。
- 参考说话人音色与表达。
- 使用原音频对白、歌词或音效。
- 参考节拍、韵律或音频连续性。

若 Audio 明确对应目标 speaker，沿用目标视频的全局 speaker ID：对应已定义主体时写 `<Subject N> (Sx)`；否则写稳定声音描述再接 `(Sx)`。Audio 定义不独立分配新编号。

```text
<Audio 1> is the voice-timbre reference for <Subject 1> (S1).
```

一个音频承担多个职责时，在同一句自然语言里写清，不创建额外子段。

### 25.3.5 同一参考视频的视觉轨与音轨

- `<Video N>` 与 `<Audio N>` 独立编号。
- 索引只代表各自类别中的出现顺序，不编码配对。
- 因而同一个源文件完全可以是 `<Video 1>` 与 `<Audio 2>`。
- 普通参考视频即使文件带声音，也不会自动产生 `<Audio N>`；只有启用其音轨作为参考或复用时才定义 Audio。
- Audio 定义主要说明音频职责，不必总写来自哪个 Video；只有存在来源歧义时才说明。

```text
<Video 1> is the source video for the target video edit.
<Audio 2> is the synchronized audio track of <Video 1> and is reused in the target video.
```

## 25.4 `summary`

使用一个简短英文段落，并以方括号任务类型开头：

```text
[reference generation] ...
[video editing + reference generation + audio reuse] ...
```

### 六种任务类型

| 类型 | 使用条件 |
|---|---|
| `keyframe completion` | 图片是目标视频首帧、关键帧、尾帧、被编辑帧或其他具体帧锚点 |
| `reference generation` | 图片/视频/音频只提供角色、场景、风格、动作、运镜、storyboard 等生成指导，不是具体帧，也不是被编辑/续写源视频 |
| `video editing` | 直接修改已有源视频；编辑图片或静态关键帧之间生成不属于此类 |
| `video continuation` | 从现有源视频继续、扩展、恢复或过渡 |
| `audio reuse` | 完整或部分复用同一音频信号 |
| `audio reference` | 不直接复制信号，只参考音乐风格、音色、对白/歌词内容、音效质感、节拍或连续性 |

多种关系用 ` + ` 合并，不能重复：

```text
[video continuation + keyframe completion]
[video editing + audio reuse]
```

重要边界：

- 有视频/音频输入不等于自动产生对应 task type。
- 视频若只提供运镜、切镜或节奏，一般属于 `reference generation`。
- 只有直接修改或续写源视频，才用 `video editing` / `video continuation`。
- 编辑源视频且原声仍可听见时，同时写 `audio reuse`。
- 续写视频但不直接复制音频信号，仅延续其听觉特征时，用 `audio reference`。
- Summary 只能使用前面已经定义的标签，不得引入新标签。
- 视频编辑任务在前缀后固定以这句开头：

```text
The target video is an edited version of <Video 1>.
```

## 25.5 `retention_analysis`

每个 reference label 一行，说明它在目标视频中如何被保留、迁移、复制或参考。不得改变 `subject_definitions` 中已经确立的职责。

### 可见内容的固定关系值

| Marker | 含义 |
|---|---|
| `fully_preserved` | 所定义职责全部保留 |
| `partially_preserved` | 内容仍被使用，但部分特征改变或只保留一部分 |
| `attribute_transfer` | 把参考特征迁移到另一个可识别目标主体 |
| `weak_reference` | 只保留大类风格、类别、构图或氛围相似性 |

```text
<Subject 1> (appears in [Shot 1], [Shot 3]): fully_preserved - ...
<Picture 2> ([Shot 1] first frame): fully_preserved - ...
<Video 1> (cut and pacing structure): weak_reference - ...
```

### 音频的固定关系值

| Marker | 含义 |
|---|---|
| `fully_copy` | 完整源音频成为目标视频完整最终音轨 |
| `partially_copy` | 只复制部分时间/层，或复制后增删替换其他声音 |
| `reference` | 不复制信号，只参考音色、节奏、音乐风格、对白内容或声音质感 |
| `weak_reference` | 只保留宽泛类别或氛围相似性 |

```text
<Audio 1>: fully_copy - <Audio 1> is reused 1:1 as the target video's complete final audio track.
<Audio 2>: reference - the target speaker follows <Audio 2>'s voice timbre and measured delivery without copying the original signal.
```

Marker 只能在该标签已定义的职责内选择。目标视频新增动作、背景或剧情，并不自动意味着参考保真损失。

## 25.6 `detailed_description`

### 基础格式

- 正文英文，对白/歌词/可见文字保留原语言。
- `[Shot 1]` 无时间戳。
- 后续格式为 `[Shot N] At MM:SS.mmm, ...`。
- 相机运动写成自然英文；需要时包含类型、幅度、速度。
- 发声源使用稳定 `(S1)`、`(S2)`。
- 对白/歌词使用 `<d>[Language] ...</d>`。
- 跨 cut、结尾截断、连续音频使用 `<scenetrans>`、`<cutoff>` 及明确连续性描述。

### 与 T2VA 的差异

| 维度 | T2VA | Ref2VA |
|---|---|---|
| 主字段 | `integrated_multimodal_description` | `detailed_description` |
| 风格开头 | 写在 `[Shot 1]` 后 | 在 `[Shot 1]` 前用 1–2 句英文建立 |
| Reference Label | 不使用全参考标签 | 在首次出现及职责生效处插入 Subject/Picture/Video/Audio |
| 音频关系 | 写目标视频自身声音 | 在对应 shot 或音频阶段说明 Audio 被复制还是参考 |

```text
The target video is in a cinematic, literary music-video style with soft lighting and a slightly desaturated color palette.
[Shot 1] The scene opens in a crowded urban street...
[Shot 2] At 00:09.000, the shot cuts to an extreme close-up...
```

全参考生成任务的 `detailed_description` 通常为 **350–500 个英文单词**。对白密集任务优先保证完整口播时间线，不机械追字数；视频编辑描述按源视频复杂度伸缩。单镜头不等于描述可以很短，应按信息负载分配细节。

## 25.7 在 Shot 中使用 Reference Label

重要 Subject 第一次清晰出现时，要结合实际可见内容写：参考特征、画面位置和当前动作。后续继续使用同一标签，不重新定义。

具体关键帧锚点使用自然固定短语：

```text
the shot begins from <Picture 1>
the shot's keyframe corresponds to <Picture 2>
the shot ends on <Picture 3>
```

编辑或续写原视频时，在源状态、结构或续写关系真正生效处自然引用 `<Video N>`。Audio 在对应 shot 或语义阶段真正起作用时引用。

## 25.8 Speaker、Audio Source 与 Dialogue

参考主体物理发声时，必须同时保留 Subject 和 speaker ID：

```text
<Subject 2> (S1) turns toward the woman and says, <d>[English] Last summer, I went to my grandfather's house. He talked about you.</d>
```

- `<Subject N>` 识别参考主体。
- `(Sx)` 识别实际说话人。
- 同一主体画外发声仍保持相同形式，并标记 `off-screen`。
- 没有对应 Subject 的 speaker 使用稳定声音描述 + `(Sx)`。

### BGM 内歌词不一定是新 Speaker

若语言内容只是直接复用 BGM 或完整 soundtrack 中的 cue，并没有独立人物/旁白真正发声，则用 `<Audio N>` 作为声音来源，不虚构 `(Sx)`：

```text
When <Audio 1> reaches the phrase <d>[English] I'm lonely lonely lonely lonely lonely I'm lonely</d>, <Subject 1> performs the corresponding hand gesture without becoming a separate speaker source.
```

若由具体人物、角色、旁白或其他独立声音源产生，则必须分配并复用 `(Sx)`。

### 复用参考音频中的对白、旁白或歌词

- 直接复用，或用户明确要求重新表演时，保留源音频原词和原语言。
- 听不清处写 `[unclear]`，不得猜测。
- 标点标准化为表达句意所需的 `, . ? !`。
- 删除重复波浪线、emoji、项目符号和装饰性重复标点。
- 完整陈述句、疑问句、感叹句在 `</d>` 前分别以 `. ? !` 结束。
- 若只参考音色、节奏、情绪或 delivery，不要把参考音频原台词带入目标视频。

### Speaker ID 分配规则

- 按目标视频中“实际首次发声事件”的先后，只分配一次。
- `detailed_description` 中每次真实发声都复用同一 ID。
- `subject_definitions` 中绑定目标 speaker 的 Audio 复用同一 ID，但不能独立创造新 ID。
- `retention_analysis` 不写 `(Sx)`。
- 仅存在于复用 BGM/完整 soundtrack 中的语言 cue 使用 `<Audio N>`；由具体独立声音源产生的声音使用 `(Sx)`。

## 25.9 Ref2VA 的两个声音段

`overall_soundscape` 与 `non_diegetic_music` 定义仍与 Base Guide 相同。

- 环境与物理声音写 `overall_soundscape`。
- 对白、歌唱和与具体 shot 同步的声音事件留在 `detailed_description`。
- 观众可听、角色不可听的配乐写 `non_diegetic_music`。
- 引用 Audio 时，只在与听觉层匹配的 section 说明 copy/reference 关系。
- 同一 Audio 同时提供环境/音效和画外配乐时，在两个对应 section 分别说明。
- 完整对白与歌词只写在 `detailed_description` 的 `<d>` 中，不在两个声音段重复。

```text
overall_soundscape: The copied ambience layer from <Audio 1> continues throughout the target video.
non_diegetic_music: <Audio 2> is directly reused as the complete audience-only score.
```

## 25.10 官方完整 Ref2VA 案例

该案例包含：咖啡店环境、白色萨摩耶、金发女子、灰色连帽衫男子，以及女子音色参考。以下保留官方六段式英文结构。

```text
subject_definitions:
<Subject 1> is the coffee-shop environment in <Picture 1>, featuring an exposed brick wall, an orange tufted sofa with patterned pillows, a neon sign, and a wooden coffee table.
<Subject 2> is the fluffy white Samoyed in <Picture 2>, <Picture 3>, and <Picture 4>, with thick white fur, pointed ears, a dark nose, and a curved tail.
<Subject 3> is the young blonde woman in <Video 1>, with long blonde hair and a light-pink button-down shirt with rolled-up sleeves.
<Subject 4> is the young man in <Video 2>, with short wavy brown hair and a dark-grey hoodie with drawstrings.
<Audio 1> is the voice-timbre reference for <Subject 3> (S1), containing a spoken English vocal layer.

summary:
[reference generation + audio reference] The target video shows <Subject 3> eating a cookie in <Subject 1>. <Subject 4> enters with <Subject 2>, which lunges toward the cookie. The three-shot exchange uses <Audio 1> as the voice-timbre reference for <Subject 3> and ends with a canned audience laugh.

retention_analysis:
<Subject 1> (appears in [Shot 1], [Shot 2], [Shot 3]): fully_preserved - the exposed brick wall, orange tufted sofa, patterned pillows, neon sign, and wooden coffee table are retained.
<Subject 2> (appears in [Shot 1], [Shot 2]): fully_preserved - the Samoyed's thick white fur, pointed ears, dark nose, and curved tail are retained.
<Subject 3> (appears in [Shot 1], [Shot 2], [Shot 3]): fully_preserved - the blonde woman's identity, long hair, and light-pink shirt are retained.
<Subject 4> (appears in [Shot 1], [Shot 2]): fully_preserved - the young man's short wavy brown hair and dark-grey hoodie are retained.
<Audio 1>: reference - its vocal timbre guides the dialogue delivery of <Subject 3> without copying the original signal.

detailed_description:
The target video uses a realistic multi-camera sitcom style with warm indoor lighting.
[Shot 1] A medium shot establishes <Subject 1>, the coffee shop with its exposed brick wall, orange tufted sofa, patterned pillows, neon sign, and wooden coffee table. <Subject 3> (S1), the young woman with long blonde hair and a light-pink button-down shirt with rolled-up sleeves, sits on the sofa holding a chocolate-chip cookie. From the left, <Subject 4>, the young man with short wavy brown hair and a dark-grey hoodie with drawstrings, enters holding the leash of <Subject 2>, the thick-furred white Samoyed with pointed ears, a dark nose, and a curved tail. The dog lunges toward the cookie and pulls the leash taut. <Subject 3> (S1) jerks her hand back and, using the clear youthful voice timbre referenced from <Audio 1>, exclaims with light annoyance, <d>[English] Hey! Watch your dog!</d> She closes her lips and guards the cookie while <Subject 4> pulls the dog back.
[Shot 2] At 00:03.000, the shot cuts to a close-up of <Subject 4> (S2), the young man in the dark-grey hoodie from Shot 1, sitting beside <Subject 3> on the sofa and holding <Subject 2> securely in his arms. <Subject 4> (S2) says in a casual young male voice with a playful tone and an easy conversational pace, <d>[English] He just likes cookies more than me.</d> He closes his mouth into an apologetic smile and strokes the dog's thick white fur.
[Shot 3] At 00:05.000, the shot cuts to a close-up of <Subject 3> (S1), the blonde woman in the light-pink shirt from Shot 1. Her annoyance softens as she looks toward the Samoyed. <Subject 3> (S1) replies in the same clear youthful voice referenced from <Audio 1> with an amused cadence, <d>[English] Well, he has good taste at least.</d> She smiles and raises the cookie in a small toast-like gesture. A classic canned audience laugh begins immediately after the line and continues through the final frame.

overall_soundscape:
Soft indoor coffee-shop room tone continues throughout the scene.

non_diegetic_music:
N/A
```

## 来源

整理自《提示词工程技巧.md》：

- §13 Ref2VA：全参考模式（第 773–924 行）：13. Ref2VA 六段式、13.1 四类标签、13.2 `subject_definitions`、13.3 `summary`、13.4 `retention_analysis`、13.5 `detailed_description`、13.6 参考关系冲突处理。
- §25 官方 Ref2VA Guide 完整中文覆盖（第 2044–2377 行）：25.1 总则、25.2 六段式及固定顺序、25.3 四类 Reference Label、25.4 `summary`、25.5 `retention_analysis`、25.6 `detailed_description`、25.7 在 Shot 中使用 Reference Label、25.8 Speaker、Audio Source 与 Dialogue、25.9 Ref2VA 的两个声音段、25.10 官方完整 Ref2VA 案例。

未收录 §6（四种 Base 模式）、§7–§12（镜头/连续性/对白/声音）、§15–§23（模板与清单）。