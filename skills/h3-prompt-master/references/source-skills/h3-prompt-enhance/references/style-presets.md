# MiniMax-H3 风格模块库（style-presets）

本文件是一套可复用的「风格模块」。它的作用只有一个：把任意视觉风格，翻译成能塞进 H3 固定 11 槽位段语法的英文句子，并保证多段之间风格不漂移。

每个预设都是一组**可以直接复制进提示词的英文**，后面附中文说明，告诉你这句话在控制什么、什么时候该改、什么时候千万别改。

---

## 速查表

| 风格 | 适用场景 | 漂移禁令指向（A / B） | 画内文字可行性 |
| --- | --- | --- | --- |
| `2d-anime-cel` 2D 动漫赛璐璐 | 青春日常、热血动作、番剧式人物特写 | A = 3D ／ B = realistic | 高：粗黑描边 + 平涂色块的卡通海报字最稳；细衬线长正文会违和 |
| `3d-cgi-animation` 3D CG 动画 | 家庭向故事、拟人角色、儿童科普 | A = 2D flat illustration ／ B = photoreal live-action | 高：做成有厚度、被场景灯光照到的 3D 立体字；极细字重会碎 |
| `photoreal-live-action` 写实实拍电影感 | 剧情片、品牌故事、人物纪实 | A = cartoon/illustration ／ B = plastic CGI / game-engine look | 中：只能长在现实介质上（招牌、纸、屏幕）；漂浮纯屏字立刻假 |
| `ink-wash-guofeng` 国风水墨写意 | 山水意境、诗词叙事、东方品牌片 | A = Western oil / impasto ／ B = 3D render 或 live-action | 低但精：只出书法、印章、竖排题跋；硬边现代字体破坏质感 |
| `clay-stop-motion` 黏土定格动画 | 童趣短片、手作质感广告、角色小品 | A = smooth glossy CG ／ B = 2D hand-drawn | 高：黏土捏出或纸片剪出的立体字；锐利数字字体不搭 |
| `pixel-art` 像素／复古 8-16bit | 游戏改编、复古怀旧、UI 叙事 | A = smooth gradients ／ B = anti-aliased rendering | 高：必须位图像素字体并对齐像素网格；抗锯齿小字立刻穿帮 |
| `watercolor-storybook` 水彩绘本插画 | 儿童绘本、温情叙事、教育内容 | A = opaque oil / airbrush digital painting ／ B = 3D render | 高：手写体或毛笔字、边缘带洇开；精确数字排版不搭 |
| `cyberpunk-neon` 赛博朋克霓虹 | 未来都市、科技产品、夜间动作 | A = bright natural daylight realism ／ B = flat cartoon | 极高：霓虹招牌、全息广告、HUD 故障字都是风格本体 |
| `documentary-handheld` 纪录片手持纪实 | 真实访谈、幕后跟拍、社会题材 | A = cinematic graded drama ／ B = CG / cartoon | 中：实物标牌与简单字幕条；发光标题、品牌感排版会出戏 |
| `product-ad-studio` 产品广告棚拍 | 电商主图视频、新品发布、功能演示 | A = cluttered real-world location ／ B = cartoon / flat illustration | 极高：干净无衬线品牌字 + 大量留白，与棚拍打光一致 |

---

## 怎么用

### 每个预设给你 7 个字段

其中 **`style_anchor` / `drift_ban` / `world_anchor` / `graphic_anchor` / `texture` / `hygiene` / `exclusions` 这 7 个字段本身就是可直接粘进提示词的英文**，代码块里的句子照抄即可，不要翻译、不要改标点。代码块后面紧跟的中文说明，是给你判断「这句话在管什么、能不能动」用的。

另外每个预设还附两个执行备注字段 `typography`（画内文字）和 `audio_lean`（音乐/音效倾向），同样给出英文句子加中文说明，按需取用。

| 字段 | 作用 | 动它的后果 |
| --- | --- | --- |
| `style_anchor` | 钉死媒介与画法，一句话说明「这是什么做的」 | 改它 = 换风格，不要在同一个多段任务里改 |
| `drift_ban` | 点名最危险的两个漂移方向，第 2 段起全靠它 | 删它 = 后续段落必然向默认审美漂移 |
| `world_anchor` | 钉死空间质感与背景语言，防止背景换材质 | 改它 = 第 2 段开始世界会变成另一个世界 |
| `graphic_anchor` | 钉死画内文字/海报的统一画法 | 改它 = 同一支片里字体风格会前后跳 |
| `texture` | 钉死表面质感与噪点层级（有颗粒还是没有） | 改它 = 观感从「胶片」变「数码」而不自知 |
| `hygiene` | 该风格特有的动态模糊/帧间稳定问题 | 删它 = 运动镜头糊掉、闪色、抖动 |
| `exclusions` | 反向黑名单，把不该出现的东西提前赶走 | 删它 = 模型会自己补上违和元素 |
| `typography` | 该风格画内文字能否出字、怎么出才不违和 | 用错字体 = 一眼看出拼接感 |
| `audio_lean` | 该风格默认的音乐/音效倾向 | 音画不同源 = 观感廉价 |

### 组合公式

**第 1 段用完整锁**：7 个字段全上，把媒介、世界、图形语言、质感、防糊规则、黑名单一次性立住。

```
[style_anchor] [drift_ban] [world_anchor] [graphic_anchor] [texture] [hygiene] [exclusions]
```

其中 `drift_ban` 不能省：第 1 段是唯一一次「定义规则」的机会，后续段只做引用，建立锁里少了它，第 2 段起的 `do not drift toward {A} or {B}` 就没有可引用的原点。

**第 2 段起用压缩引用锁**：不再重复长句，只留三句引用式锁定，把风格、世界、图形语言逐段续上。

```
Identical {style} to the previous clip, do not drift toward {A} or {B}. Same {world} as previous clips. Same {graphic} style as previous clips.
```

- `{style}` 取该风格的短名（例如 `2D anime cel style`）
- `{world}` 取 `world_anchor` 里的核心名词短语（例如 `bright flat-illustration cartoon world`）
- `{graphic}` 取 `graphic_anchor` 里的核心名词短语（例如 `bold cartoon poster`）
- `{A}` / `{B}` 取该风格小节里标注的**压缩锁变量**两个方向

实际粘出来长这样（以 2D 赛璐璐为例）：

```
Identical 2D anime cel style to the previous clip, do not drift toward 3D or realistic. Same bright flat-illustration cartoon world as previous clips. Same bold cartoon poster style as previous clips.
```

**为什么必须这样分工**：第 1 段负责「建立」，长句完整才不会一开始就糊成默认审美；第 2 段起负责「维持」，只需要否定式锚点，长句重复反而会稀释主体动作描述，还容易让每段各自重新诠释风格。

> **锁定级别的准确定义见 `SKILL.md` 第 3 节。** 这里给的是**完整引用锁**（推荐）。源工作流第 3–10 段实际用的是**简写变体**：`Identical {风格} to the previous clip. Same {世界}. Same {图文样式} style.` —— 省掉了 `do not drift toward {A} or {B}` 与 `as previous clips`。简写可用，但 `Identical … to the previous clip.` 这一句任何级别都不能省。

---

## 1. 2D 动漫赛璐璐（2d-anime-cel）

**style_anchor**

```
2D anime cel style, clean lineart, flat vibrant colors, subtle cel shading, anime key-visual quality.
```

中文说明：确立「线稿 + 平涂 + 轻微赛璐璐阴影」的三件套，并把完成度锚在动画主视觉插画级别。这是整套锁的根，不要动。

**drift_ban**

```
Keep 2D anime style locked in every clip; never drift to 3D or realistic.
```

中文说明：明确点名两个最危险的漂移方向。2D 素材在多段生成里最常见的失败就是被模型的默认三维光照「拉立体」。

**world_anchor**

```
The world is a bright flat-illustration cartoon space with a soft pastel background, consistent cartoon rendering throughout all clips; no live-action, no photoreal elements.
```

中文说明：把背景钉在「扁平插画卡通空间 + 柔和粉彩」，并顺手排除实拍与写实元素，防止背景先崩。

**graphic_anchor**

```
On-screen infographic text posters appear in a bold cartoon style with thick black outlines and bright solid color fills, identical across all clips.
```

中文说明：画内文字统一为粗描边、纯色填充的卡通海报。`identical across all clips` 是关键，它让所有段落的字体语言保持一致。

**texture**

```
Clean digital cel texture with even flat fills, crisp uniform line weight, no film grain, no paper texture, no painterly brush marks.
```

中文说明：赛璐璐的干净来自「没有颗粒、没有笔触」。一旦混入胶片颗粒或水彩笔痕，就会向厚涂滑落。

**hygiene**

```
Keep lineart crisp under motion: no heavy motion blur, no depth-of-field that softens the line layer, no frame-to-frame line-weight flicker or color shimmer; shading blocks stay fixed per shot.
```

中文说明：赛璐璐最怕动态模糊把线稿糊掉，也怕逐帧线宽抖动和色块闪色。这条专门保住「线」的存在感。

**exclusions**

```
no 3D rendering, no photoreal skin or fabric detail, no live-action footage, no oil or watercolor brushwork, no cinematic teal-orange grade, no lens flare.
```

中文说明：反向黑名单。写实肤色纹理和电影青橙调色是这一类素材最容易被偷偷加进去的两样东西。

**typography（画内文字）**

```
Bold cartoon lettering with thick black outlines and flat solid fills, short headline-length only.
```

中文说明：画内文字可行，且很好出效果——粗黑描边、纯色块、短标题长度最稳。禁止细衬线长正文、禁止带复杂渐变或投影的数字字，那会和赛璐璐的平面语言打架。

**audio_lean（音乐/音效倾向）**

```
Bright J-pop or upbeat instrumental, steady four-four rhythm, crisp cartoon whooshes and impacts.
```

中文说明：明快流行编制 + 清爽打击乐，动作段落用卡通化 whoosh 与打击拟音。避免低沉电影化 drone，会把番剧感拖成预告片感。

**压缩锁变量**：`{A}` = `3D`，`{B}` = `realistic`

---

## 2. 3D CG 动画（3d-cgi-animation）

**style_anchor**

```
Stylized 3D CG animation, Pixar-Disney family-film look, appealing rounded character shapes, soft global illumination, subsurface-lit skin, cinematic depth of field.
```

中文说明：锚定「风格化三维 + 家庭向圆润造型 + 柔和 GI」。注意是 stylized 而非 realistic，这一个词决定了角色不会被做成真人质感。

**drift_ban**

```
Keep stylized 3D CG locked in every clip; never drift to 2D flat illustration or to photoreal live-action.
```

中文说明：三维动画的漂移是双向的——要么被压扁成二维插画，要么被拉成写实真人。两个方向都要点名。

**world_anchor**

```
The world is a fully modeled and lit 3D CG environment with consistent stylized proportions, unified light rig, and the same render look throughout all clips; no flat 2D backgrounds, no live-action plates.
```

中文说明：`unified light rig` 是这段的核心：多段之间灯光必须同源，否则同一场景会出现两种打光逻辑。

**graphic_anchor**

```
On-screen posters and labels are modeled as 3D props with a clean toon-clean sans-serif look, slightly rounded extruded lettering, shaded by the same light rig, identical across all clips.
```

中文说明：画内文字当成场景里的三维道具来做，有厚度、有倒角、和场景共用光源与反射。这样字才像「长在画面里」而不是贴上去。

**texture**

```
Smooth shaded surfaces with subtle micro-detail maps, soft ambient-occlusion contact shadows, no grain, no halftone dots, no hand-drawn line overlays.
```

中文说明：平滑着色 + 微弱细节贴图 + 柔和接触阴影。明确排除手绘描边叠加，那是二维化漂移的入口。

**hygiene**

```
Control temporal artifacts: no denoiser ghosting or smeared trailing edges, no hair or fur shimmer between frames, no material popping, and keep motion blur short and stable so silhouettes stay readable.
```

中文说明：三维渲染在动态里最典型的毛病是降噪鬼影、毛发闪烁、材质跳变。这条要求运动模糊短而稳定，轮廓必须读得出来。

**exclusions**

```
no 2D cel shading, no hand-drawn lineart, no photoreal human skin pores, no live-action footage, no flat vector graphics, no VHS or film grain.
```

中文说明：排除二维赛璐璐阴影、手绘线稿、真人毛孔级皮肤细节。这三样任意混进一个，风格立刻不再是家庭向三维。

**typography（画内文字）**

```
Extruded 3D lettering, slightly rounded edges, lit by the scene light rig, integrated as a physical prop.
```

中文说明：出字效果很好，但要当道具做。禁止极细字重（一渲染就碎）、禁止文字与毛发或粒子系统穿插，那会暴露合成的痕迹。

**audio_lean（音乐/音效倾向）**

```
Bright orchestral or warm pop arrangement, cartoon foley such as boing and pop, medium-to-large hall reverb.
```

中文说明：明亮管弦乐或温情流行编制，配卡通拟音。混响要带空间感，和三维场景的尺度匹配；避免 lo-fi 与重失真。

**压缩锁变量**：`{A}` = `2D flat illustration`，`{B}` = `photoreal live-action`

---

## 3. 写实实拍电影感（photoreal-live-action）

**style_anchor**

```
Photoreal live-action cinematography, natural human skin with visible pores, physically accurate materials, real lens rendering, 35mm filmic response.
```

中文说明：锚定物理正确的材质、真实的镜头与胶片响应。`real lens rendering` 这句在防止画面变成「干净到不像拍出来」的数码感。

**drift_ban**

```
Keep photoreal live-action locked in every clip; never drift to cartoon or illustration, and never drift to a plastic CGI or video-game look.
```

中文说明：实拍的漂移方向有两个，一个是卡通化，一个是游戏引擎化。第二个尤其常见——皮肤变塑料、材质失去微观层次。

**world_anchor**

```
The world is a real physical location with practical light, believable scale, and consistent weather, wardrobe, and set dressing across all clips; no stylized or illustrated environments.
```

中文说明：把天气、服装、陈设一起锁住。多段实拍任务里最容易穿帮的往往不是人，而是第二段突然换了天气或换了衣服。

**graphic_anchor**

```
On-screen text exists only on real media in the scene — signage, printed paper, monitors, phone screens — shot in perspective with matching lens and grain, identical across all clips.
```

中文说明：画内文字必须挂在现实介质上，并带上透视与相同的颗粒。这样字会随镜头自然运动，而不是悬在画面上。

**texture**

```
Fine filmic grain, true-to-life surface detail, subtle lens breathing and chromatic aberration, no painterly smoothing, no denoise plastic sheen.
```

中文说明：细颗粒 + 真实表面细节 + 轻微呼吸感与色散。明确排除降噪后的塑料光泽，那是最典型的数码假面。

**hygiene**

```
Manage shutter behavior: keep shutter smear and rolling-shutter skew minimal, avoid over-strong motion blur on pans, hold exposure and white balance steady, and keep grain consistent clip to clip.
```

中文说明：实拍特有的动态问题就是快门拖影与果冻效应。这条还额外要求曝光和白平衡稳定、段与段之间颗粒一致，否则拼接感会很重。

**exclusions**

```
no cartoon or anime rendering, no cel shading, no visible brush strokes, no game-engine materials, no impossible floating text, no oversaturated fantasy grade.
```

中文说明：排除卡通渲染、赛璐璐阴影、可见笔触、游戏引擎材质，以及最致命的「凭空漂浮的文字」。

**typography（画内文字）**

```
Text only on physical surfaces — signage, paper, screens — with correct perspective, depth of field, and matched grain.
```

中文说明：可以出字，但必须克制且依附现实介质。禁止漂浮的纯屏字、禁止华丽动画字体、禁止与画面颗粒不匹配的锐利矢量字。

**audio_lean（音乐/音效倾向）**

```
Location sound first, room tone and footsteps preserved, minimal score such as solo piano or low strings, or no score at all.
```

中文说明：现场同期声优先，保留房间底噪和脚步。配乐要么极简要么没有。禁止卡通拟音，那会立刻把纪实感拆掉。

**压缩锁变量**：`{A}` = `cartoon`，`{B}` = `CGI look`

---

## 4. 国风水墨写意（ink-wash-guofeng）

**style_anchor**

```
Chinese ink-wash painting style, xieyi freehand brushwork, flowing ink gradients, generous negative space, rice-paper absorbency, restrained monochrome with sparse accent color.
```

中文说明：锚定写意笔法、墨色流动、大量留白与宣纸吸墨。`xieyi`（写意）这个词很关键，它把风格推向概括而非工笔描摹。

**drift_ban**

```
Keep Chinese ink-wash style locked in every clip; never drift to Western oil or impasto painting, and never drift to 3D render or live-action.
```

中文说明：水墨最常被拉去两个地方——西式厚涂油画，和三维立体渲染。都要点名排除。

**world_anchor**

```
The world is a misty ink-painted landscape on rice paper, mountains and water implied by brush strokes and empty space, consistent ink density and paper tone across all clips; no photographic depth, no modeled geometry.
```

中文说明：山与水由笔触和空白「暗示」出来，而不是靠体积塑造。`ink density` 与 `paper tone` 统一，是段与段之间不跳色的关键。

**graphic_anchor**

```
On-screen text appears only as calligraphy, seal stamps, or vertical inscription in the same ink and paper, identical across all clips.
```

中文说明：画内文字只允许书法、印章、竖排题跋三种形态，并且用同一套墨色与纸色。

**texture**

```
Visible rice-paper fiber, ink bleed and feathering edges, dry-brush streaks, faint wash blooms; no digital gradient mesh, no glossy highlights.
```

中文说明：纸纤维、洇墨毛边、飞白、淡淡的水痕。明确排除数字渐变网格与高光，这两样会让水墨瞬间变成数码贴图。

**hygiene**

```
Protect brush structure in motion: keep strokes legible instead of smearing into a single grey mass, limit ink diffusion drift between frames, and avoid heavy motion blur that dissolves brushwork.
```

中文说明：水墨在动态里最大的敌人是「笔触糊成一片」——一运动就退化成一团灰。这条要求笔触结构可读，并限制帧间洇墨失控。

**exclusions**

```
no oil paint texture, no thick impasto, no 3D shading, no photoreal detail, no neon color, no hard-edged vector shapes.
```

中文说明：排除油画质感、厚涂、三维明暗、写实细节与霓虹色。硬边矢量形状也会立刻破坏水墨的柔性边缘。

**typography（画内文字）**

```
Calligraphy and seal stamps only, vertical inscription, same ink and paper as the painting.
```

中文说明：出字能力是「低但精」。竖排题跋、印章、匾额都可以。禁止现代无衬线长句、禁止硬边数字字体——它们与纸张的吸墨边缘完全冲突。

**audio_lean（音乐/音效倾向）**

```
Guqin, xiao, or guzheng solo, sparse ambient textures such as wind, water, and paper friction, wide negative space in the mix.
```

中文说明：古琴、箫、古筝独奏，环境音要稀疏留白。混音上也要留白。避免电子节拍与鼓组。

**压缩锁变量**：`{A}` = `Western oil painting`，`{B}` = `3D render`

---

## 5. 黏土定格动画（clay-stop-motion）

**style_anchor**

```
Clay stop-motion animation, handmade plasticine characters, visible fingerprint and thumbprint detail, matte clay surfaces, miniature practical set, 12fps stepped motion feel.
```

中文说明：锚定手作黏土、指纹痕迹、哑光表面、微缩实景与 12fps 顿挫感。`12fps stepped` 是风格的灵魂，不要删。

**drift_ban**

```
Keep handmade clay stop-motion locked in every clip; never drift to smooth glossy CG, and never drift to 2D hand-drawn animation.
```

中文说明：定格动画最容易被「修」成光滑 CG——补帧、加反射、去指纹，三件事做完风格就死了。另一端是滑向二维手绘。

**world_anchor**

```
The world is a tabletop miniature set built from clay, felt, cardboard, and fabric, lit with practical small-scale lights, consistent handmade props and scale across all clips; no CG environment, no full-size real location.
```

中文说明：世界是桌面微缩布景，材料限定在黏土、毛毡、纸板、布料。`consistent scale` 保证段与段之间道具比例不漂。

**graphic_anchor**

```
On-screen posters and signs are clay-cut or paper-cut props with slightly uneven hand-trimmed edges, identical across all clips.
```

中文说明：画内文字/海报做成黏土片或剪纸片道具，边缘略有不齐的手裁感。

**texture**

```
Matte clay with smudges and tool marks, felt fuzz, tiny dust and set-wear, gentle stop-motion stutter; no specular plastic shine, no digital noise.
```

中文说明：哑光黏土、涂抹痕、工具痕、毛毡绒毛、细微灰尘与使用感。排除塑料高光与数字噪点。

**hygiene**

```
Keep the stop-motion cadence intentional: preserve the stepped 12fps beat, avoid interpolation that smooths it away, and prevent fingerprint and surface-detail shimmer between frames; use little to no motion blur.
```

中文说明：定格动画的卫生问题和别人相反——要**保住**顿挫，而不是消除它。同时防止指纹与表面细节在帧间闪烁。运动模糊要极少或没有。

**exclusions**

```
no glossy smooth CG surfaces, no ray-traced reflections, no 2D cel lineart, no photoreal humans, no digital glow.
```

中文说明：排除光滑 CG 表面、光线追踪反射、二维赛璐璐线稿、写实人物、数码辉光。每一条都是在守住「手工」二字。

**typography（画内文字）**

```
Letters formed from clay or cut paper, slightly uneven hand-trimmed edges, visible handmade thickness.
```

中文说明：出字可行，但必须是捏出来或剪出来的立体字，边缘略糙、有手作厚度。避免锐利数字字体与完美对齐，那与手作质感直接矛盾。

**audio_lean（音乐/音效倾向）**

```
Playful xylophone and plucked strings, handmade foley such as clay squish, fabric friction, and light tap footsteps, slight room ambience.
```

中文说明：木琴、拨弦、轻快小型编制，配手作拟音。保留一点房间感，让观众相信这是在一个真实的小空间里拍出来的。

**压缩锁变量**：`{A}` = `smooth glossy CG`，`{B}` = `2D hand-drawn`

---

## 6. 像素／复古 8-16bit（pixel-art）

**style_anchor**

```
Retro pixel art, 16-bit era sprite aesthetic, strict low-resolution pixel grid, limited indexed palette, crisp hard pixel edges, no anti-aliasing.
```

中文说明：锚定 16 位时代精灵美学、严格低分辨率像素网格、受限索引调色板、硬边像素。`no anti-aliasing` 必须留着。

**drift_ban**

```
Keep retro pixel art locked in every clip; never drift to smooth gradients, and never drift to anti-aliased high-resolution rendering.
```

中文说明：像素风的漂移方向就是「变光滑」——渐变和抗锯齿是两条最典型的路径，两个都要点名。

**world_anchor**

```
The world is a tile-based pixel-art scene with a single fixed pixel scale, consistent sprite resolution and palette across all clips; no smooth vector backgrounds, no photographic plates.
```

中文说明：`single fixed pixel scale` 是这段的命门。多段任务里最常见的崩坏就是第二段像素变大了，看起来像换了分辨率。

**graphic_anchor**

```
On-screen text is drawn in a bitmap pixel font snapped to the same pixel grid, limited to the scene palette, identical across all clips.
```

中文说明：画内文字用位图像素字体，并且「吸附」到同一像素网格、只用场景调色板里的颜色。

**texture**

```
Hard-edged pixels, dithering used for shading, flat color blocks, clean scanline-free rendering; no blur, no soft shadow, no gradient ramp.
```

中文说明：硬边像素、用抖动（dithering）做明暗过渡、纯色块。排除模糊、软阴影与渐变。

**hygiene**

```
Avoid sub-pixel jitter: lock the sprite to the pixel grid so pixels do not crawl or wobble, snap all motion and camera moves to whole-pixel steps, and never resample or rotate sprites with smoothing.
```

中文说明：像素风特有的动态问题是**子像素抖动**。相机或角色做亚像素移动时，整个像素网格会像在呼吸一样游动。必须把运动吸附到整像素步进。

**exclusions**

```
no anti-aliasing, no smooth gradients, no photographic textures, no 3D shading, no motion blur, no vector shapes.
```

中文说明：排除抗锯齿、平滑渐变、照片纹理、三维明暗、运动模糊、矢量形状。像素风的画面里不应该出现任何「曲线」。

**typography（画内文字）**

```
Bitmap pixel font snapped to the pixel grid, palette-limited colors, integer scale sizes only.
```

中文说明：出字可行，规则很硬：位图像素字体、对齐像素网格、颜色取自受限调色板、字号只能整数倍缩放。极小号抗锯齿文字会立刻穿帮。

**audio_lean（音乐/音效倾向）**

```
Chiptune with square and triangle wave leads, 8-bit sound effects such as coin, jump, blip, and explosion.
```

中文说明：chiptune 方波/三角波旋律加 8-bit 音效。不要真实乐器录音，音画同源比音质重要。

**压缩锁变量**：`{A}` = `smooth gradients`，`{B}` = `anti-aliased`

---

## 7. 水彩绘本插画（watercolor-storybook）

**style_anchor**

```
Watercolor storybook illustration, soft transparent washes, visible paper texture, gentle pencil under-drawing, pastel palette, children's-book warmth.
```

中文说明：锚定透明水彩、可见纸纹、温和铅笔底稿、粉彩调子与绘本温度。`pencil under-drawing` 是廉价但极有效的风格识别点。

**drift_ban**

```
Keep watercolor storybook style locked in every clip; never drift to opaque oil or airbrushed digital painting, and never drift to 3D render.
```

中文说明：水彩的两条漂移路径是「变厚重」（油画、喷绘）和「变立体」（三维渲染）。一个词就能同时防住：transparent。

**world_anchor**

```
The world is a hand-painted watercolor page with soft wash horizons and unpainted paper edges, consistent paper tone and pigment behavior across all clips; no modeled 3D space, no photographic background.
```

中文说明：世界始终是「一页手绘纸」，包括未上色的纸边。纸色与颜料行为统一，段落之间才不会出现两种纸。

**graphic_anchor**

```
On-screen text is hand-lettered in watercolor or ink with slightly irregular strokes and bleed at the edges, identical across all clips.
```

中文说明：画内文字用手写体或钢笔/毛笔字，笔画略不规整，边缘带洇开。

**texture**

```
Cold-press paper grain, pigment granulation, wet-on-wet blooms, dry-brush edges, faint pencil lines; no digital gradient glow, no airbrush smoothness.
```

中文说明：冷压纸纹、颜料颗粒沉积、湿画法水痕、飞白、淡淡铅笔线。排除数字渐变辉光与喷绘般的顺滑。

**hygiene**

```
Keep washes clean: stop colors from muddying into grey, hold pigment edges instead of smearing them across frames, and prevent paper grain from swimming or boiling between frames.
```

中文说明：水彩的动态问题是两层——颜色在运动中被搅浑发灰，以及纸纹在帧间「游动沸腾」。两条都要处理。

**exclusions**

```
no oil impasto, no thick opaque paint, no 3D shading, no neon digital glow, no lens flare, no photoreal detail.
```

中文说明：排除厚涂、不透明颜料、三维明暗、霓虹辉光、镜头光晕与写实细节。水彩的柔和来自「薄」，不是来自模糊。

**typography（画内文字）**

```
Hand-lettered watercolor or ink lettering, slightly irregular strokes, visible ink bleed at the edges.
```

中文说明：出字可行，走手写路线。避免数字精确排版与硬边无衬线，那会与纸张吸墨的边缘语言冲突。

**audio_lean（音乐/音效倾向）**

```
Piano and string chamber music, soft natural sounds such as birdsong, wind, and page turns.
```

中文说明：钢琴与弦乐室内乐，柔和的自然声。避免重低音与电子鼓，会把绘本的轻盈感压掉。

**压缩锁变量**：`{A}` = `opaque oil painting`，`{B}` = `3D render`

---

## 8. 赛博朋克霓虹（cyberpunk-neon）

**style_anchor**

```
Cyberpunk neon aesthetic, rain-slick night city, dense holographic signage, high-contrast magenta-cyan neon, wet reflective streets, anamorphic flare.
```

中文说明：锚定雨夜都市、密集全息招牌、品红-青的高对比霓虹、湿滑反光地面与变形宽银幕光晕。这几项加起来基本无法误认。

**drift_ban**

```
Keep cyberpunk neon night style locked in every clip; never drift to bright natural daylight realism, and never drift to flat cartoon illustration.
```

中文说明：赛博朋克最容易被「照亮」——一旦变成白天自然光，风格就不存在了。另一端是被压成扁平卡通，霓虹的辉光层次全丢。

**world_anchor**

```
The world is a dense futuristic night city of wet streets, layered signage, and steam, with the same neon palette and weather across all clips; no daytime natural-light location, no clean rural setting.
```

中文说明：天气、霓虹配色一起锁。`same weather` 很重要——雨夜不能上一段下雨、下一段干了。

**graphic_anchor**

```
On-screen text appears as glowing neon signs, holographic billboards, and HUD overlays in the scene's magenta-cyan palette, identical across all clips.
```

中文说明：画内文字是这套风格的组成部分而非附属品。霓虹招牌、全息广告、HUD 面板全部统一在品红-青体系里。

**texture**

```
Wet asphalt reflections, condensation on glass, atmospheric haze, fine rain streaks, subtle digital scan and glitch artifacts; no clean dry daylight surfaces.
```

中文说明：湿沥青反射、玻璃凝露、空气雾、细密雨丝、轻微扫描与故障感。排除干净的干燥日光表面。

**hygiene**

```
Control glow spill: keep neon bloom from washing out faces and text, avoid frame-to-frame flare popping, and limit motion blur so wet reflections stay structured rather than smeared.
```

中文说明：霓虹的动态问题是泛光失控——过强的 bloom 会吃掉人脸和文字，光晕还会在帧间跳变。运动模糊要限制住，否则湿地反射会糊成一坨。

**exclusions**

```
no bright daylight, no natural green landscape, no pastel cartoon palette, no clean studio white, no anime cel shading.
```

中文说明：排除明亮日光、自然绿景、粉彩卡通配色、干净影棚白，以及动漫赛璐璐阴影。

**typography（画内文字）**

```
Glowing neon signs, holographic billboards, and HUD overlays with perspective and consistent bloom strength.
```

中文说明：出字效果极好，几乎是风格本体。字要带辉光和透视，且辉光强度与全画面一致。避免柔和衬线正文，会显得与城市语言无关。

**audio_lean（音乐/音效倾向）**

```
Synthwave or dark synth, industrial percussion, low-frequency drone, neon electrical hum and rain.
```

中文说明：synthwave / dark synth 打底，工业打击与低频 drone，叠加霓虹电流嗡鸣与环境雨声。避免原声木吉他这类暖质乐器。

**压缩锁变量**：`{A}` = `bright daylight realism`，`{B}` = `flat cartoon`

---

## 9. 纪录片手持纪实（documentary-handheld）

**style_anchor**

```
Documentary handheld realism, available-light shooting, natural color, imperfect framing, real people in real locations, unpolished observational camera.
```

中文说明：锚定现场光、自然色、不完美构图与观察式机位。`imperfect framing` 是纪实感的来源，不要优化掉。

**drift_ban**

```
Keep documentary handheld realism locked in every clip; never drift to cinematic graded drama, and never drift to CG or cartoon rendering.
```

中文说明：纪实的漂移方向是「变得太好看」——电影级调色、戏剧化打光一上来，可信度就没了。另一端是 CG 化。

**world_anchor**

```
The world is a real working environment with practical light and ordinary detail, same location logic and time of day across clips; no stylized set, no fantasy environment.
```

中文说明：真实工作环境、现场光、普通细节，且地点逻辑与时段一致。`ordinary detail` 是刻意的——过于干净的场景会立刻假。

**graphic_anchor**

```
On-screen text appears only as real signage, labels, or printed material caught by the camera in perspective, identical across all clips.
```

中文说明：画内文字只来自被镜头顺手拍到的真实标牌、标签、印刷品，并且带透视。不做设计过的字幕排版。

**texture**

```
Light sensor noise, slightly soft practical focus, mild lens imperfection, honest skin and fabric texture; no glossy grade, no filmic teal-orange, no beauty smoothing.
```

中文说明：轻微传感器噪点、略软的对焦、镜头小瑕疵、诚实的皮肤与布料质感。排除磨皮般的平滑和电影青橙调色。

**hygiene**

```
Balance handheld motion: keep shake organic but small, avoid stacking shake with strong shutter smear, prevent rolling-shutter wobble on fast pan, and stop auto-exposure from pumping between frames.
```

中文说明：手持的动态卫生有两个陷阱：抖动叠加快门拖影、以及自动曝光在帧间来回抽动。抖动要「有机但小」，不能大到让人晕。

**exclusions**

```
no cinematic color grade, no dramatic rim lighting, no CGI elements, no cartoon rendering, no slow-motion hero shots, no anamorphic flare.
```

中文说明：排除电影调色、戏剧轮廓光、CG 元素、卡通渲染、慢动作英雄镜头与变形镜头光晕。这些全是「表演感」，与纪实相反。

**typography（画内文字）**

```
Real signage, printed labels, and simple lower-third style captions only, no animated typography.
```

中文说明：出字能力中等且需克制。实物标牌、被拍到的印刷品、简朴字幕条都可以。禁止发光标题、华丽动画字体、品牌感排版。

**audio_lean（音乐/音效倾向）**

```
Location sync sound first, ambient room tone preserved, spare or absent score, no cartoon foley.
```

中文说明：现场同期声优先，环境底噪保留。配乐要么克制要么干脆不用。禁止卡通拟音与重型鼓组。

**压缩锁变量**：`{A}` = `cinematic graded drama`，`{B}` = `CG or cartoon`

---

## 10. 产品广告棚拍（product-ad-studio）

**style_anchor**

```
High-end studio product commercial, seamless controlled backdrop, softbox key and gradient falloff, macro-clean product surface, precise specular highlights.
```

中文说明：锚定无缝背景、柔光箱主光与渐变衰减、微距级干净表面、精准高光。`gradient falloff` 是高级感的来源。

**drift_ban**

```
Keep studio product commercial style locked in every clip; never drift to cluttered real-world location shooting, and never drift to cartoon or flat illustration.
```

中文说明：棚拍的漂移方向是「跑出影棚」和「被画成插画」。前者表现为背景突然出现生活杂物，后者是产品被扁平化。

**world_anchor**

```
The world is a controlled studio set with a seamless backdrop, fixed light rig, and identical product staging across all clips; no messy environment, no outdoor location, no props that compete with the product.
```

中文说明：世界只有背景、灯光、产品三样。`no props that compete` 明确要求任何道具都不能抢产品注意。

**graphic_anchor**

```
On-screen text is clean flat sans-serif brand typography laid over the studio background with generous spacing, identical across all clips.
```

中文说明：画内文字用干净扁平的无衬线品牌字体，大量留白，直接压在影棚背景上。

**texture**

```
Immaculate product surfaces, fine material micro-texture, controlled specular sheen, dust-free background; no film grain, no sensor noise, no fingerprints on product.
```

中文说明：产品表面无瑕、材质微观纹理保留、高光受控、背景无尘。排除颗粒与噪点——棚拍追求的是「干净」，不是「胶片感」。

**hygiene**

```
Protect edge and highlight fidelity: keep turntable and push-in moves from smearing the product silhouette, cap specular bloom so highlights do not swallow material detail, and eliminate handheld jitter.
```

中文说明：棚拍的动态风险是转盘与推镜把产品边缘拖糊，以及高光溢出吃掉材质细节。手持抖动必须彻底消除。

**exclusions**

```
no cluttered background, no natural window light, no cartoon rendering, no rough handheld camera, no heavy film grain, no dramatic colored gels.
```

中文说明：排除杂乱背景、自然窗光、卡通渲染、粗糙手持、重颗粒与戏剧化彩色滤色片。彩色灯光会干扰产品本色的准确性。

**typography（画内文字）**

```
Clean flat sans-serif brand typography, generous spacing, subtle shadow or reflection, no exaggerated effects.
```

中文说明：出字能力极强，这是它的主场。无衬线品牌字体 + 大量留白，可加极轻的阴影或倒影，但不要夸张特效。

**audio_lean（音乐/音效倾向）**

```
Clean minimal electronic beat or ambient pad, material foley such as clicks, fabric, liquid, and paper, no ambient noise.
```

中文说明：干净极简的电子节拍或氛围 pad，配材质拟音（咔哒、布料、液体、纸张）。不要环境底噪，也不要卡通音效。

**压缩锁变量**：`{A}` = `cluttered real-world location`，`{B}` = `cartoon`

---

## 11. 自定义风格槽（custom）

当目标风格不在上面 10 种里，用这一个槽位现场拼。你只需要填 4 个空。

### 四个填空

| 填空 | 问自己什么 | 写出来的形式 |
| --- | --- | --- |
| **媒介 medium** | 这东西是画出来的、捏出来的、拍出来的，还是算出来的？用什么材料？ | 一个名词短语，例如 `hand-painted gouache illustration` |
| **线条与边缘 line & edge** | 边缘是硬是软？有没有描边？粗细是否均匀？ | 形容词 + 名词，例如 `soft irregular edges with visible brush contours` |
| **色彩与光影 color & lighting** | 色域受限还是连续？光从哪来、有几个光源、对比多强？ | 例如 `muted earth-tone palette with a single warm key light` |
| **质感与噪点 texture & grain** | 表面有什么颗粒？纸纹、胶片颗粒、像素块、还是完全干净？ | 例如 `cold-press paper grain with slight pigment speckle` |

### 拼成 `style_anchor` 的句式模板

```
{medium}, {line & edge}, {color & lighting}, {texture & grain}.
```

按顺序填完就是一个合格的 `style_anchor`：

```
Hand-painted gouache illustration, soft irregular edges with visible brush contours, muted earth-tone palette with a single warm key light, cold-press paper grain with slight pigment speckle.
```

配套的 `drift_ban` 用同一个模板反向写：

```
Keep {medium short name} locked in every clip; never drift toward {A} or {B}.
```

```
Keep hand-painted gouache illustration locked in every clip; never drift toward glossy digital vector art or toward photoreal 3D rendering.
```

`world_anchor` 与 `graphic_anchor` 参照前面 10 个预设的写法，把「空间是什么做的」和「画内文字是什么做的」各写一句，并都带上 `identical across all clips` 或 `consistent ... across all clips`。`texture` 直接复用第四个填空并加上排除项，`hygiene` 按该媒介最容易糊掉的东西来写（见下面的推导法）。

### 未知风格的三步推导法

**第一步：先定媒介。**
媒介是根，定错后面全错。问一个最朴素的问题：这个画面**如果是实物，它是用什么做的**？水彩纸、黏土、像素屏幕、实拍镜头、还是三维工程文件？把答案写成一个名词短语，这就是 `style_anchor` 的第一段。不要在这一步写形容词，形容词会掩盖媒介判断。

**第二步：再定边缘与色域。**
媒介定了，接着定两件事——**边缘硬度**和**色彩自由度**。
- 边缘：有描边还是没有？粗细均匀还是随压力变化？硬切还是晕开？
- 色域：是受限调色板（像素、版画、绘本）还是连续真实色彩（实拍、三维）？光是一个光源还是多个？对比强还是柔？

这两项一写，风格的可识别度基本就到位了，`texture` 与 `hygiene` 也有了依据：受限色域 + 硬边，通常不需要颗粒卫生规则，但需要防止被重采样；连续色域 + 软边，通常需要控制运动模糊与帧间纹理稳定。

**第三步：最后定漂移禁令指向哪两个方向。**
找这个媒介**最近的两个错误邻域**。通用做法是问两个问题：
1. 如果模型把它「做得更光滑、更立体、更高级」，会变成什么？（这通常是 `{A}`）
2. 如果模型把它「做得更扁平、更手绘、更卡通」，会变成什么？（这通常是 `{B}`）

把这两个答案填进 `drift_ban` 和压缩引用锁的 `{A}` / `{B}`。

**一个完整示例——纸雕剪影（paper-cut shadow）**

- 媒介：`Hand-cut paper diorama`
- 边缘与色域：`hard scissor-cut edges with slight fiber fraying`；`backlit monochrome layers with a single warm rim light`
- 质感与噪点：`visible paper fiber and soft cast shadow between layers`

拼出的 `style_anchor`：

```
Hand-cut paper diorama, hard scissor-cut edges with slight fiber fraying, backlit monochrome layers with a single warm rim light, visible paper fiber and soft cast shadow between layers.
```

对应的 `drift_ban`：

```
Keep hand-cut paper diorama locked in every clip; never drift toward full 3D volumetric lighting or toward flat 2D vector illustration.
```

对应的 `hygiene`：

```
Keep the layered cut edges crisp in motion: no heavy motion blur that merges the layers, no frame-to-frame fiber shimmer, and no depth-of-field that softens the paper silhouette.
```

这里的两个漂移方向分别是「更立体」（三维体积光）和「更扁平」（二维矢量插画）——正是第三步问出来的两个错误邻域。

---

## 附：字段速用提醒

1. **第 1 段永远用完整锁。** 压缩引用锁只负责延续，不负责建立。
2. **`drift_ban` 里的两个方向必须具体。** 写 `never drift` 而不写漂去哪，等于没写。
3. **`hygiene` 按风格定制。** 像素风防子像素抖动，水墨防笔触糊成一片，实拍防快门拖影，三维防降噪鬼影，棚拍防边缘拖糊。
4. **画内文字先问「它长在什么上」。** 长在现实介质上就写介质，长在画面里就写道具和光照，这是不违和的唯一办法。
5. **音画同源。** 卡通的画面配卡通的拟音，纪实的画面保留现场声。音效不对，风格会被观众重新判定。