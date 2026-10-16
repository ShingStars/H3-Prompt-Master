---
name: ref2va-duel-prompt-generator
description: "参考图驱动的双角色多镜头对决提示词：锁定双方身份与招式体系，把 15 秒拆成切点递增的连续交手节拍，逐镜写清构图/动作/运镜/命中结果/声音，产出 MiniMax H3 Ref2VA 六段式提示词，并以远方觉醒之类的新钩子收尾。不适合单人展示、静态对峙、无参考图的纯文本生成或非战斗题材。"
whenToUse: 当用户给出两张（或更多）角色参考图并想要一场打斗、对决、切磋、Boss 战的短视频提示词；或要把已有对决提示词改写为 Ref2VA 六段式、调整镜头密度、补招式与音效时。
compatibility: Portable to any agent that can read local files — 只需本地文件读写能力；正文为中文，最终提示词正文为英文。
metadata:
  version: 1.0.0
  language: zh-CN 正文 + English 可粘贴提示词
  derived_from: 雷龙与牛来对决（15 秒 / 9 镜 / Ref2VA 六段式范本）
  upstream: 字段与标签规则以 h3-prompt-enhance（minimax-h3-prompt-suite）的 references/ref2va.md 为准；本 skill 只负责"对决"这一题材的编排方法
---

# Ref2VA 对决提示词生成器

把"两张角色参考图 + 一场打斗"编译成一份可直接生成的 **Ref2VA 六段式** 提示词：两名角色的身份由 `<Picture N>` 锁定，15 秒内用切点递增的微镜头串成连续交手，每一镜都有人吃亏、有物证、有声音，结尾留一个新钩子。

本文件是行动纲领；完整范本与逐镜拆解见 `references/duel-example.md`。

**三条铁律**：

1. **先锁身份，再编打斗**。身份锁写不实，镜头越多越糊。
2. **切点必须踩在物证上**。新镜的第一句是上一镜打出来的东西（扬尘、碎石、火花），不是新场地。
3. **每镜都要有一方吃亏**。没有身体位移和碎片反馈的镜头 = 废镜。

---

## 0. 交付物

| 交付 | 内容 | 何时给 |
| --- | --- | --- |
| 主交付 | 一份 Ref2VA 六段式提示词（英文正文，字段名与顺序固定） | 始终 |
| 次交付 | 对决节拍表（镜号 / 切点 / 主攻方 / 招式 / 命中结果 / 主运镜） | 用户要看编排、或镜头数 > 5 时 |
| 可选 | 稳妥版（5–7 镜）与快节奏版（9 镜）两套切点表 | 用户没定节奏、或明确担心角色一致性时 |

不要交付：剧情梗概式描述、分镜图、模型参数（除非用户明确要）。

---

## 1. 输入与默认值

| 需求项 | 缺省处理 |
| --- | --- |
| 参考图 | 必须有图才走 Ref2VA；确认每张图承担什么职责（角色身份 / 场景 / 风格）。只有一张图却要双角色 → 明确提示需要第二张，或降级为"无参考的第二方"并声明风险 |
| 时长 | 默认 15 秒（H3 支持 4–15 秒） |
| 身份 | 姓名（拼音）、种族/角色定位、体型、配色、标志部件、随身特效、阵营 |
| 招式 | 每方 2–4 个标志招式，类型要岔开（近战 / 远程 / 投技 / 位移） |
| 场景 | 默认"废墟战场 + 可数地标"（城门、三面断墙） |
| 结局钩子 | 默认"远方符文亮起、巨影苏醒" |
| 对白 | 默认无对白；用户给了台词就原样保留，不翻译不润色 |

**只问一个最关键的问题**：两张参考图分别是谁、各承担什么职责。其余自行推断并**简短声明**推断了什么。

---

## 2. 六段式骨架（不许发明）

```text
subject_definitions:
<Subject 1> ...
<Subject 2> ...

summary:
[reference generation] ...

retention_analysis:
<Subject 1> (appears in [Shot 1]–[Shot N]): fully_preserved - ...
<Subject 2> (appears in [Shot 1]–[Shot N]): fully_preserved - ...

detailed_description:
<1–2 句总体风格句>
[Shot 1] ...
...
[Shot N] ...

overall_soundscape: ...

non_diegetic_music: ...
```

| 字段 | 写什么 | 硬约束 |
| --- | --- | --- |
| `subject_definitions` | 每个可追踪内容一行：标签指代谁、从哪张图来、必须遵循的特征、战力/阵营定位 | 一行一个 Subject；标签分配后跨六段含义不变 |
| `summary` | 任务类型 + 目标视频一句话 + 招式清单 + 结尾钩子 | 任务类型写在方括号里（本题材通常 `[reference generation]`） |
| `retention_analysis` | 每个 Subject 一行，说明保留方式 | 关系值只用官方四档；行内不写 `(Sx)` |
| `detailed_description` | Ref2VA 的主时间轴：风格句 + 逐镜 | 不要压缩成剧情梗概 |
| `overall_soundscape` | 1–4 句画内物理声音 | 不写对白、不写配乐 |
| `non_diegetic_music` | 只有观众听到的配乐 | 不写 `epic`/`emotional`/`beautiful` 了事；无配乐写 `N/A` |

> 有角色参考图却套 Base 三段式（`integrated_multimodal_description`）是最常见的结构性错误：那张图定义的是**身份**而不是第 0 秒画面，必须走 Ref2VA。

---

## 3. `subject_definitions`：身份锁模板

```text
<Subject 1> is the <角色定位> named <拼音名> whose appearance comes from <Picture 1>, with <特征 1>, <特征 2>, <特征 3>, <特征 4>, and <特征 5>, and <标志性动态>; his face is smooth and unmarked with no wounds. He is the <战力定位> of the fantasy battlefield.
<Subject 2> is the <角色定位> named <拼音名> whose appearance comes from <Picture 2>, with <特征 1>, <特征 2>, <特征 3>, and <特征 4>; his face is smooth and unmarked with no wounds, and aside from these features his body remains clean and unadorned. He is the <阵营定位> on the fantasy battlefield.
```

- 名字用**拼音**（如 `Lei Yuan Long Jun`、`Jin Jiao Niu Wei`）：专名比形容词更能拉住身份。
- 特征清单 5–7 条，**每条都必须看得见**：材质（dark-gold scales）、配色（silver-white mane）、部件（grey curved horns、hooked claws）、体型（massive build / short thick limbs）、随身特效（purple-blue lightning wrapped around his body）。
- 必写 **`his face is smooth and unmarked with no wounds`**：9 镜高速剪辑里最有效的防"越打越脏"锚点。
- 必写 **`aside from these features his body remains clean and unadorned`**：挡住模型自行加盔甲、纹身、武器。
- 第二方要写清**阵营对立**（`the enemy-faction character`），否则模型容易把两人当队友、不敢下狠手。

---

## 4. `summary`：一句话总述 + 钩子

```text
[reference generation] The target video is a <节奏> <类型> in which <Subject 1>, <一句话身份>, and <Subject 2>, <一句话身份>, trade successive <招式 1>, <招式 2>, <招式 3>, and <招式 4> across <场景>, before <结尾钩子>.
```

- 招式清单必须是后面镜头**真的会出现**的招式，顺序大致对应剧情推进。
- 结尾钩子在 `summary` 就要出现，并在最后一镜闭合（前后呼应，不是临时加戏）。

---

## 5. `retention_analysis`：保留声明 + 状态豁免

```text
<Subject 1> (appears in [Shot 1]–[Shot 9]): fully_preserved - <特征逐条复述>, with his face smooth and unmarked.
<Subject 2> (appears in [Shot 1]–[Shot 9]): fully_preserved - <特征逐条复述>; combat state changes such as <红眼/角上裂纹/擦伤> do not alter his defining appearance.
```

- 关系值只用官方四档：`fully_preserved` / `partially_preserved` / `attribute_transfer` / `weak_reference`。同一 Subject 全片出场就写 `fully_preserved`。
- 第二句的**战斗状态豁免**是关键：被打伤、眼睛发红、角上出现裂纹属于**状态**，不属于身份漂移。不写这句，模型会把"受伤"当成新特征，越打越走形。
- 镜号范围用 en dash：`[Shot 1]–[Shot 9]`。

---

## 6. `detailed_description`：风格句 + 逐镜六要素

风格句（1–2 句，定风格、光线、环境）：

```text
The target video is in a <风格，如 fast-paced 3D CG fantasy combat style> with <光线，如 dramatic high-contrast lighting>, <气氛，如 drifting cloud cover>, and <场景，如 a ruined battlefield of crumbling walls and scattered rubble>.
```

每个镜头按**固定顺序**写六件事：

1. **切点句**（第 2 镜起）：`[Shot 2] At 00:02.000, the camera cuts to <上一镜的物证>, <主运镜>.`
2. **镜别与构图**：`a medium side shot` / `a medium low-angle shot` / `a medium tracking shot`
3. **主攻方动作**：具体肢体轨迹与落点
4. **另一方应对**：闪、挡、硬吃、反制，都要有明确结果
5. **命中与代价**：被击退 / 撞墙 / 单膝跪地 / 地面环状开裂 / 滑出两道沟
6. **环境反馈**：扬尘、碎石、火花、冲击波、墙面坍塌

---

## 7. 对决节拍引擎（9 拍）

| 拍 | 功能 | 主攻 | 招式类型 | 命中结果 | 参考切点（15 秒） |
| --- | --- | --- | --- | --- | --- |
| 1 | 建立 + 首次接触 | 双方 | 高空俯冲 + 角冲 | 爪角相擦、火花成串 | 00:00 |
| 2 | 反制 | 双方 | 侧身滑步 + 上挑角击 | 被削肩、连退数步 | 00:02.000 |
| 3 | 显能 | 甲 | 空中翻身 + 闪电柱 | 击空、石柱炸裂 | 00:03.700 |
| 4 | 压制 | 甲 | 双爪连击 + 突进 | 撞墙、扬尘 | 00:05.400 |
| 5 | 对冲 | 双方 | 交叉对冲 | 双方各吃一招、同时被撞开 | 00:07.100 |
| 6 | 投技 | 甲 | 抓角根 + 借腰力抛投 | 撞碎第三面墙 | 00:08.800 |
| 7 | 蓄力 | 甲 | 升空蓄能 + 尾部抽击 | 侧向趔趄 | 00:10.400 |
| 8 | 终结技 | 甲 | 锥形闪电柱贯穿 | 冲击波环、整体崩塌 | 00:12.000 |
| 9 | 重整 + 钩子 | 乙 | 再冲锋 + 爪击击退 | 双沟、远方符文亮起 | 00:13.500 |

- 每镜**只让一方占上风**，另一方必须给出可见反应；两人都稳住不动 = 废镜。
- **招式轮替**：相邻两镜不要同类型招式（近战 / 远程 / 投技 / 位移岔开）。
- 弱势一方必须至少赢下 1–2 拍（第 2、5 拍），否则节奏塌成单方面挨打。
- 最后一镜留钩子（符文亮起、巨影苏醒、第三者登场），不要"打完收工"。

---

## 8. 微镜头链式规则（把冲击当切点）

```text
[Shot 2] At 00:02.000, the camera cuts to the dust rising inside the ruined wall, a medium tracking shot hugging the wall and trucking laterally in sync with the two fighters' exchanges, keeping both in frame.
[Shot 3] At 00:03.700, the camera cuts to the sparks on the dragon's scales, arcing halfway around the dragon's body while rising in sync with his ascent into a medium low-angle shot.
```

- 新镜第一句 = **上一镜制造出的具体物证**（扬起的尘、碎裂的石柱、龙鳞上的火花）。硬切处最容易丢空间和身份，物证把切点变成因果。
- 每镜**只给一个主运镜**，写法是"运动类型 + 幅度 + 速度"：`a medium side shot tracking low along the ground ... whipping a half-pan with large amplitude`。
- 每镜都要写 **`keeping both in frame`**：这是防双人身份漂移最省字的一句。
- 运镜方向尽量延续上一镜（`in sync with`、`continues from camera left`）；硬切后不要反向拉。
- **场景锚点链**：用可辨识、可数的地标当坐标（`before the northern gate` → `a second ruined wall` → `a third ruined wall`），让观众确认两人始终在同一场地。

---

## 9. 时间轴与镜头预算

15 秒 / 9 镜实测切点：

```text
00:00 → 00:02.000 → 00:03.700 → 00:05.400 → 00:07.100 → 00:08.800 → 00:10.400 → 00:12.000 → 00:13.500
```

- 第 1 镜 2.0 秒（建立 + 首次接触）；中间 5 镜 1.6–1.7 秒；最后 2 镜 1.5 秒。
- 切点**严格递增、三位小数、全部小于总时长**；格式固定为 `At 00:0X.XXX,`。

| 方案 | 镜头数 | 切点 | 适用 |
| --- | --- | --- | --- |
| 稳妥 | 5 | 00:00 / 00:03.000 / 00:06.000 / 00:09.000 / 00:12.000 | 角色设计复杂、用户最在意一致性 |
| 中等 | 7 | 约每 1.9–2.2 秒一切 | 通用默认升级版 |
| 快节奏 | 9 | 见上表 | 用户明确要"高强度快打" |

官方经验：15 秒超过 4 镜就有身份漂移、空间断裂、声音拥挤风险。9 镜是靠**物证切点 + 双人同框 + 每镜身份复述**三件套压住的，缺一件就降镜头数。

---

## 10. 招式与命中写法

| 招式 | 写法要点 |
| --- | --- |
| 爪击 / 抓取 | 写轨迹与落点：`whips his claw around in a rising slash across <Subject 2>'s shoulder` |
| 角冲 | 低头、蹄下发力、超音速冲刺、角尖擦过反光部件溅火花 |
| 尾扫 | 宽弧轨迹、命中腰腹、把人抽得横飞撞墙 |
| 闪电 / 远程 | 从喉部或口中喷出**锥形**柱体；击空时明确打在石柱/墙上并炸碎 |
| 投技 | 抓住 horns 根部、借腰力甩出、身体水平旋转、撞碎墙体 |
| 砸地 / 践踏 | 从空中砸向脊背，把对方压到单膝/双膝着地，地面环状开裂 |

- 每镜动作三拍：**A 出手 → B 应对（闪 / 挡 / 硬吃）→ 追加一拍收束**。
- 命中必须写**身体位移量**：被击退、斜向飞出、撞穿墙、蹄子刨出两道沟。
- 不要写"两人激烈战斗"这类无信息量句；每一击都要能画出来。

---

## 11. 声音分层

`overall_soundscape`（1–4 句，只写画内物理声音）：

```text
Ground ruptures with deep booming cracks as stone debris and gravel burst outward. Lightning crackles and roars around the dragon with electric hissing, while thunderous shockwaves from the lightning pillars explode against the walls. The bull's hooves scrape and stomp against rubble with heavy thuds, and the air howls with each supersonic charge and tail-sweep.
```

写法：地面/碎石层 + 双方**各自的标志音**（甲=电流噼啪与雷鸣，乙=蹄声与冲刺破空）+ 每次冲击的回声。不写对白，不写配乐。

`non_diegetic_music`（只有观众听到）：

```text
Epic orchestral combat score at a fast tempo with pounding taiko drums, aggressive brass stabs, and soaring strings, building in intensity and ending with a sudden cutoff on the final freeze-frame.
```

写法：乐器 + 速度 + 强弱走向 + 进出时机。禁止只写 `epic` 了事。收尾用 `a sudden cutoff` 呼应最后一镜的钩子。

---

## 12. 自检清单

1. 六段齐全、顺序正确、字段名未自创
2. `summary` 的任务类型方括号值与参考图实际职责一致
3. 每个 Subject 一行；名称、特征清单、来源 `<Picture N>` 齐全
4. 两行 `retention_analysis` 都复述了特征，且都有 `fully_preserved` 或明确降级理由
5. 战斗状态豁免句存在（红眼 / 角裂 / 擦伤不算身份变化）
6. 所有 `<Subject N>` 在六段中含义一致，无未定义标签
7. 切点严格递增、三位小数、都小于总时长
8. 每镜六要素齐全（切点 / 构图 / A 动作 / B 应对 / 命中结果 / 环境反馈）
9. 第 2 镜起的首句都是上一镜的物证
10. 每镜只有一个主运镜 + 幅度 + 速度，且写了 `keeping both in frame`
11. 招式无相邻重复；每镜都有一方吃亏并有可视化代价；弱势方至少赢 1–2 拍
12. 两段声音分层正确，配乐写明收尾方式；结尾钩子与 `summary` 呼应

---

## 13. 常见错误

| 错误 | 后果 | 改法 |
| --- | --- | --- |
| 有角色参考图却写成 Base 三段式 | 结构不合规，参考关系丢失 | 改走 Ref2VA 六段式 |
| 9 镜但没有物证切点 | 硬切处空间与身份断裂 | 每镜首句接上一镜的碎片/扬尘/火花 |
| 每镜换一片新场地 | 观众失去空间坐标，像剪贴画 | 用地标锚点链，全场只在 1 个场景内 |
| 一方永远挨打、无反制 | 节奏塌、看点消失 | 让弱势方在第 2、5 拍反制得手 |
| 一秒内堆推拉摇环绕 + 高速跑动 + 转身 | 运镜冲突，画面随机 | 每个时间段只留一个主运镜 |
| 用 `cinematic`/`epic` 代替可见细节 | 模型自由发挥，一致性崩 | 换成可见或可听的具体描述 |
| 把受伤写进外观定义 | 越打越脏、越打越不像 | 伤写在动作/结果里，外观定义保持干净 |

---

## 14. 范本

`references/duel-example.md` 收录 15 秒 / 9 镜的完整 Ref2VA 范本（东方雷龙 vs 双足牛战士），附逐镜拆解表（切点 / 主攻方 / 招式 / 结果 / 主运镜 / 下一镜入口物证），可直接对照改写。