# H3 提示词技术规格（带出处）

这份文件只放**事实**，用来支撑 `SKILL.md` 里的规则。每条都标明本机出处，便于复核。

---

## 1. 提示词是裸文本，不做 chat 模板

出处：[`ComfyUI/comfy/text_encoders/minimax.py`](D:/ComfyUI-aki-v3/ComfyUI/comfy/text_encoders/minimax.py) 文件头注释（第 1–20 行）。

> The H3 presentation is **NOT chat-templated**: token ids are raw prompt/label text (no special tokens) with explicit vision blocks spliced in.

这个结论决定了三件事：

1. **不要写 `###`、`- `、`**加粗**` 这类 markdown 结构**给 H3 —— 它们会被原样编码成提示词内容。
2. 结构化字段名（`integrated_multimodal_description:` / `overall_soundscape:` / `non_diegetic_music:`）是**约定俗成的标签文本**，不是被解析的关键字；它们能被读懂，是因为模型在训练里见过这种排版，而不是因为代码在解析。
3. `<d>…</d>` 反过来**是真的特殊 token**（见下节），属于"必须写对"的那一类。

### 三种 conditioning 的实际拼装顺序

| 模式 | 拼装 |
| --- | --- |
| `t2va` | `<prompt>` |
| `fl2va` | `"<Picture 1>: "` + 视觉块 + `["<Picture 2>: "` + 视觉块]` + `<prompt>` |
| `ref2va` | 按请求顺序、分类型 1-based 编号：图像 → `"<Picture i>: "` + 视觉块；音频 → `"<Audio j>: "`（音频不进 Qwen）；视频 → `"<Video k>: "` + 每 2 帧一个时间块 `"<T.T seconds>"` + 视觉块；最后接 `<prompt>` |

**实践含义**：写提示词时不要自己写 `Picture 1` / `Picture 2` 的编号去"点"参考图 —— 编号是节点按接线顺序自动生成的。Herrgotts 的提示与节点 tooltip 都要求**引用当前的 `picture_map`**，而不是假定固定的 Picture 号。

### Conditioning 的取值位置

出处：同文件第 15–19 行。

> The conditioning is the unnormalized hidden state after LM layer 50 … Vision-pad positions carry adaLN token tag 0 (video modality) in the DiT; text positions carry tag 1.

即：文本编码器是 **Qwen3-VL-32B 截断到 50 层**，取第 50 层隐状态、**不做最终 norm**。

---

## 2. 特殊 token 表（`<d>` 是机器标签）

出处：[`minimax.py:32-34`](D:/ComfyUI-aki-v3/ComfyUI/comfy/text_encoders/minimax.py#L32-L34)：

```python
MINIMAX_EXTRA_TOKENS = {"<d>": 151669, "</d>": 151670, "<|cutoff|>": 151671,
                        "<|lyrics_start|>": 151672, "<|lyrics_end|>": 151673,
                        "<|caption_start|>": 151674, "<|caption_end|>": 151675}
```

另有视觉块定界 token：`VISION_START = 151652`、`VISION_END = 151653`（同文件第 29–30 行）。

| Token | id | 用途 | 写错会怎样 |
| --- | --- | --- | --- |
| `<d>` | 151669 | 对话开始 | 语音不受控/不出现 |
| `</d>` | 151670 | 对话结束 | 后续文本被吞进对话段 |
| `<|lyrics_start|>` | 151672 | 歌词开始 | 唱词被念成旁白 |
| `<|lyrics_end|>` | 151673 | 歌词结束 | — |
| `<|caption_start|>` | 151674 | 字幕开始 | 字幕/画面文字混淆 |
| `<|caption_end|>` | 151675 | 字幕结束 | — |
| `<|cutoff|>` | 151671 | 截断标记 | — |

**必须是这一对尖括号形态**：`<d>` 而不是 `<D>`、`[d]`、`(d)`；`<|lyrics_start|>` 的竖线不能省。

**语言标签是文本而非 token**：`<d>[Chinese] …</d>` 里的 `[Chinese]` 是普通文本，但它决定了 TTS 发音语言，务必写。

---

## 3. 续接机制与 39 帧边界

出处：Herrgotts-H3-Infinite-Continuation-Suite `README.md` 第 166–199 行。

### 提示词侧的三条结论

1. 每个片段当作**一次短分镜转场**来写：描述这一段应该发生的动作/运动。
2. Last Frame 用来做强端点/画质重置；Qwen Reference 用来锁身份、服装、细节。
3. **被掩码的前序 AV 上下文不需要、也不应该在提示词里提** —— 它已经在目标 latent 里。

> The masked previous AV context does **not** need to be mentioned in the prompt. It is already present in the target latent.

### 边界数值

| 项 | 值 |
| --- | --- |
| 合法联合 AV 上下文长度 | `39, 90, 141, 192, …` 帧 |
| 默认 | **39 帧** |
| 39 帧对应时长 | 约 **1.625 秒** @ 24fps |
| 对应音频 latent | 恰好 **65 ticks** @ H3 的 40Hz 音频 latent 速率 |
| 视频交叉淡入 | 4 帧 |
| 音频去咔哒交叉淡入 | 15 ms |
| 安全尾桥 / 亮度匹配 | 均默认关闭（legacy 回退项） |

`duration_mode = "Net New Content"` 时：`5s` 会选 158 总帧 = 39 帧头 + **119 帧新生成（≈4.96s）**；旧的 `"Total Generation"` 行为是 124 总帧 = 85 新帧（≈3.54s）。

**实践含义（写台词时最关键）**：源片段里**从未生成过的词救不回来**。所以跨段台词的收口必须在源片段内部完成，且不要把关键词放在紧贴段尾的位置。

---

## 4. 模型能力与画布

出处：ComfyUI 官方模板 `video_minimax_h3_t2v.json` 的 `MarkdownNote`（`comfyui_workflow_templates_json` 包）。

| 项 | 值 |
| --- | --- |
| 模型 | MiniMax H3，omni-modal，**原生立体声**：人声/音效/音乐在同一次前向里联合建模，不是后期叠加 |
| 原生画布 | 短边 768px，上限 **768×1344**，尺寸四舍五入到 **32 的倍数** |
| 帧率 | **24fps** |
| 时长上限 | 约 15 秒 |
| 帧网格 | **17k+5**（时长按这个网格向上吸附） |
| 官方 768p | 1344×768 |

分辨率对照（官方模板原表，multiple=32）：

| megapixels | 16:9 输出 |
| --- | --- |
| 0.2 | 608×352 |
| 0.3 | 736×416 |
| 0.4 | 864×480 |
| 0.5 | 960×544 |
| 0.6 | 1056×608 |
| 0.7 | 1152×640 |
| 0.8 | 1216×672 |
| 0.9 | 1280×736 |
| 0.98 | 1344×768（官方 768p） |

> 源工作流用的是 **736×1280**（竖屏，短边 736 < 768，留显存余量给 8G 卡），不是上表的 16:9 档位。

### 提示词 embedding

官方提供 10 个风格 embedding，可写在提示词里：

```
embedding:minimaxh3_art_is_explosion
```

出处：官方模板 `MarkdownNote`，注明 "Prompt embeddings (supported since ComfyUI#15697)"。**可选**，不写也能跑。若使用，风格锚与 embedding 二选一或叠加需实测，不要盲目两者都上。

---

## 5. 源工作流的接线（十段示例）

文件：[`MiniMaxH3-小黑8G显存-多段无缝流.json`](D:/ComfyUI-aki-v3/ComfyUI/user/default/workflows/MiniMaxH3低显存/MiniMaxH3-小黑8G显存-多段无缝流.json)

```
Text Multiline(545..553,555)
   └─ STRING ─→ SetNode「Set_提示词1..10」
                    └─ GetNode「Get_提示词N」
                          └─ prompt ─→ 子图实例 (10 个)
                                         └─ H3ContinuousContinueV14「prompt」输入
```

- 10 个 `Text Multiline` 节点（was-ns 包）→ 10 个 `Set_提示词N` → 10 个 `Get_提示词N` → 10 个子图。
- 段序与节点对应：`提示词1`→545、2→546、3→547、4→548、**5→553**、**6→549**、7→550、8→551、9→552、10→555。
  （节点 id 顺序与段序**不一致**，回填时按 `Set_提示词N` 的 N 走，不要按 id 排序。）
- 10 个子图靠 `previous_latent` + `handover` 首尾相接，最后经 9 个 `H3ContinuousSeamlessJoinV14` 拼成一条。
- 共享资源用 `SetNode`/`GetNode`（rgthree）广播：`Set_modle` / `Set_CLIP` / `Set_Video` / `Set_Audio` / `Set_采样器` / `Set_Sigma` / `Set_宽` / `Set_高` / `Set_秒` / `Set_Reference 1`。

**提示词回填要点**：只改 `Text Multiline` 的 `widgets_values[0]`，不要动 `SetNode`/`GetNode` 的 `title` 与 link，否则整条链断开。

---

## 6. 节点参数（widget 顺序已核对）

### `H3ContinuousContinueV14`

出处：`Herrgotts-H3-Infinite-Continuation-Suite/nodes.py:1520-1567`。

widget 顺序（源工作流实测值）：

| 位置 | 参数 | 源工作流值 | 说明 |
| --- | --- | --- | --- |
| 0 | `prompt` | `"Prompt — Clip 7"` | **被上游 link 覆盖**，链上仍有值属正常 |
| 1 | `width` | `736` | |
| 2 | `height` | `1280` | |
| 3 | `duration` | `10` | 由 `duration_mode` 解释 |
| 4 | `masked_context_frames` | `"39"` | 可选 `39/90/141/192` |
| 5 | `audio_feather_ticks` | `0` | 语音场景保持 0 |
| 6 | `ref_image_size` | `"match"` | |
| 7 | `duration_mode` | `"Net New Content"` | 或 `"Total Generation"` |
| 8 | `audio_tail_carryover` | `"Full Previous Tail"` | 对话场景推荐；或 `"Match Video Handover"` |

### `H3ContinuousAnalyzeHandoverV14`

widget 顺序（源工作流实测 18 值）：

```
["Balanced", 72, 8, 3, "39", 192, 0.012, 0.025, 3, 0.002, 0.01, 1, 70, 2, 15, 75, 3, "fixed"]
```

对应 `preset` / `analysis_window` / `freeze_hold` / `safety_margin` / `context_frames` / `analysis_size` / `final_mean_diff_threshold` / `final_active_pixel_threshold` / `max_final_active_area_percent` / `transition_mean_diff_threshold` / `transition_active_pixel_threshold` / `max_transition_active_area_percent` / `min_static_transition_percent` / `max_consecutive_motion_outliers` / `final_reference_frames` / `min_final_match_percent` / `max_consecutive_final_outliers` / `safety_mode`。

**这组参数只管接缝检测，与提示词内容无关** —— 写提示词时不需要动它，但知道 `context_frames` 被固定为 `"39"` 有助于理解规则 6。

---

## 7. 官方模板用的方言（storyboard 对照样本）

出处：官方模板 `video_minimax_h3_t2v.json` 中的提示词示例（实拍动作片预告片）。它的结构是：

```
{风格总述：媒介 + 色调 + 镜头语言 + 质感}
Scene overview: {情境}
Storyboard (each shot a separate scene, rapid cuts, all landing on the musical beats):
[0s-1.5s] Shot 1: {机位}: {动作}
[1s-2.5s] Shot 2: {动作}
[2.5s-4s] Shot 3: {动作}
[4s-5s] Shot 4: freeze: {定格}
Camera: {每镜一个机位，硬切，不要叠化}
Audio: {环境音 + 音效 + 音乐 + 节拍强调}
No text, subtitles, logos or watermarks of any kind, no {A}, no {B}, {质感保持句}.
```

与源十段的 field 方言**并存且互不冲突**，因为编码器只看到裸文本。`SKILL.md` 第 1 节的方言选择表就是基于这份对照样本定的。