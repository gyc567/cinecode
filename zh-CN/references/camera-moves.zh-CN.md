# 镜头运动与转场：命名词汇表

每个镜头的摄影机写成一行**镜头行**：**名称、秒数、速度缓动、目标**。

```text
dolly in, 3 s, slow ease-in-out, toward the figure
whip pan, 0.3 s, ease-in-out, from the window to the door
dolly zoom, 3 s, slow ease-in-out, on the title
```

这一行进 `shots.json`（`camera.move.line`，Film mode），或者进节拍表和场景函数的第一条注释（Quick mode）。两套引擎都实现了每一个名称：Quick mode 在 `film.html`（`CAM`、`retime`、`plane`、`TRANSITION`），Film mode 在 three.js 的 `starter/camera.mjs` 里。它们全都是时间的纯函数。

## 速度与缓动词

| 词 | 改变大小的运动（dolly、zoom、push） | 扫过的运动（pan、truck、orbit） |
|---|---|---|
| slow | 主体尺寸每秒变化 ≤ 10% | 一个物体 ≥ 6 s 横穿画面 |
| medium | 每秒 10–25% | 3–6 s 横穿 |
| fast | 每秒 > 25% | 1.5–3 s 横穿（更快就是甩镜） |

缓动：**linear**（机器、步行速度的跟拍）· **ease-in**（积蓄力量；结尾）· **ease-out**（在第 0 帧最快：钩子镜头）· **ease-in-out**（克制、自成一体）· **inertia**（35% linear + 65% smoothstep：第 0 帧已在动，切点时仍在动；段落内部的默认值）。第一个镜头绝不以 ease-in 开始（钩子规则）。

每个镜头一个主动作，最多再加一个次要动作（手持、一个小幅俯仰）。每个运动都有理由：跟随、揭示、施压（推）、释放（拉）。

## 摄影机运动

| 名称 | 它做什么 | 观众感受到什么 | 默认值 | 起始 → 结束构图 | 调用（Quick · three.js） |
|---|---|---|---|---|---|
| **pan** | 原地左右转；无视差 | 扫视、跟随、把两样东西连起来 | 3–6 s，slow–medium，inertia | 物 A 入画 → 物 B 入画 | `CAM.pan(c, e, 0, 0.4)` · `pan(pos, tgt, e, 0, 30)` |
| **tilt** | 原地上下转 | 上摇：敬畏、尺度；下摇：发现、重量 | 2–4 s，slow，ease-in-out | 底部 → 顶部（或反向） | `CAM.tilt` · `tilt()` |
| **dolly in** | 摄影机朝主体推进；带视差 | 亲近、张力、"这事重要" | 3–6 s，slow，inertia（钩子处用 ease-out） | 同一景别更近一档（WS → MS） | `CAM.dolly(c, e, 1.3, 0.8)` · `dolly(pos, tgt, e, 10, 6)` |
| **dolly out** | 摄影机拉远 | 孤立、释放、交代环境 | 3–6 s，slow，ease-in-out | 同一景别更远一档 | `CAM.dolly(c, e, 0.8, 1.3)` · `dolly()` |
| **zoom in** | 焦距变长；无视差 | 观察、监视、70 年代纪录片 | 1–3 s，medium，linear 或 ease-in | 宽 → 紧，平 | `CAM.zoom(c, e, 1, 1.6)` · `zoom(e, 40, 22)` |
| **zoom out** | 焦距变短 | 环境平铺地到来，抽离感 | 1–3 s，medium，ease-out | 紧 → 宽 | `CAM.zoom` · `zoom()` |
| **tracking / truck** | 横向移动，常与主体同行 | 行进、陪伴、视差带来的深度 | 3–8 s，步行速度（1–1.4 m/s），linear | 主体保持；背景流过 | `CAM.truck(c, e, -0.2, 0.2)` · `truck()` |
| **orbit / arc** | 绕着目标转并始终朝向它 | 重要、审视；快转即英雄感 | 4–8 s，5–30°，inertia | 侧面 → 四分之三面 | `CAM.orbit(c, e, -10, 15, 0)` · `orbit(tgt, e, r, -10, 15, h)` |
| **crane / pedestal up** | 升起，同时保持主体（下摇） | 希望、尺度、越过某物的揭示 | 3–6 s，slow，ease-in-out | 视平线 → 高处 | `CAM.crane(c, e, 0, 0.3)` · `crane(pos, tgt, e, 0, 3)` |
| **crane / pedestal down** | 下降 | 抵达、落定、重力 | 3–6 s，slow，ease-out | 高处 → 视平线 | `CAM.crane(c, e, 0.3, 0)` · `crane()` |
| **dolly zoom (vertigo)** | 距离和焦距反向变化；主体尺寸不变，背景扭曲 | 顿悟、不安、失真 | 2–4 s，slow，ease-in-out；每片一次 | 主体尺寸不变；背景变大或变小 | `CAM.dollyZoom(c, e, 0.8, 1.8, 0)` · `dollyZoom(e, 5, 16, 50)` + `dolly()` |
| **push-in on detail** | 朝偏心的细节推进，最后停在它上面 | "看这里"：线索、情绪 | 2–4 s，slow，ease-in-out | MS → 细节的 ECU | `CAM.pushTo(c, e, x, y, 1, 0.5)` · `pushTo(pos, tgt, detail, e, 0.8)` |
| **pull-back reveal** | 从细节出发，拉出到环境 | 惊讶、反讽、尺度 | 3–6 s，slow → medium，ease-in | ECU → WS | `CAM.dolly(c, e, 0.4, 1.6)` · `dolly()` |
| **FPV / fly-through** | 快速向前穿行于空间，在转弯处压弯 | 速度、刺激、沉浸 | 2–6 s，fast，linear 带压弯 | 穿过平面 / 沿一条路径 | `CAM.fpv(c, e, 2, -0.5)` · `fpv(points, e)` |
| **handheld**（轻微，带种子） | 低频晃动与漂移，绝不逐帧抖动 | 在场感、急迫、纪录片式的真实 | 整个镜头；幅度 0.5–1 为轻微，冲击时 3 以上 | 作为主动作 | `CAM.handheld(c, t, 1)` · `handheld(t, 1)` |
| **slow motion / speed ramp** | 时间变慢；速度坡度在镜头内部改速 | 重量、被拉长的瞬间；坡度用来断句 | 0.25–0.5×；坡度跨 0.3–0.6 s | — | `retime(t, [[1, 1], [1.4, 0.25]])`（两者通用） |
| **time-lapse** | 时间快得多 | 时间流逝、变化、自然的尺度 | 2–6 s，8–100× | — | `retime(t, [[0, 24]])` |
| **freeze frame** | 时间停住（摄影机仍可继续动） | 标点、"记住这个"、喜剧节拍 | 0.5–2 s | — | `retime(t, [[2, 1], [2, 0]])` |

## 焦点

| 名称 | 它做什么 | 观众感受到什么 | 默认值 | 调用 |
|---|---|---|---|---|
| **rack focus** | 焦点从一个平面跳到另一个平面 | 注意力移动；两样东西被连起来 | 0.5–1.5 s，medium，ease-in-out | Quick：`CAM.rackFocus(c, e, 0, 1.2, 1)`（平面按深度虚化）· three.js：`dof.cocMaterial.worldFocusDistance = rackFocus(e, 2.5, 9)` |
| **focus pull** | 焦点跟着在纵深里移动的主体 | 注意力始终留在移动上 | 与运动等长，linear | 同上，但焦点距离跟着主体走 |

## 转场

| 名称 | 它做什么 | 观众感受到什么 | 默认值 | 调用（Quick：入场镜头的 `in`） |
|---|---|---|---|---|
| **hard cut** | 瞬间改变 | 清晰、动量 | 0 帧 | 不写 `in`（或 `{ type: 'cut' }`） |
| **match cut** | 一个形状、运动或颜色跨过切点延续 | 关联、诗意、时间跳跃 | 0 帧，然后 0.3–0.6 s 稳定 | `{ type: 'match' }` + 在 B 中 `CAM.matchFrom(c, e, dx, dy)` |
| **whip pan** | 极快的甩镜，用拖影藏住变化 | 能量、原地或时间的跳跃、喜剧 | 0.2–0.35 s，ease-in-out | `{ type: 'whip', dur: 0.3, dir: 1 }` |
| **cross dissolve** | A 溶入 B | 时间流逝、记忆、柔和 | 0.4–1.5 s | `{ type: 'dissolve', dur: 0.6 }` |
| **iris** | B 在某一点上从一个长大的圆形（或形状）里打开 | 进入主体、故事书或默片感 | 0.4–0.7 s，ease-in-out | `{ type: 'iris', dur: 0.5, at: [0.5, 0.4] }` |
| **wipe** | 一条边缘把 B 扫过 A | 翻页；图形化、俏皮（只在风格主导时用） | 0.3–0.8 s，ease-in-out | `{ type: 'wipe', dur: 0.5, dir: 1 }` |
| **smash cut** | 在对立的能量之间硬切（大 → 静，平静 → 混乱） | 震惊、喜剧 | 0 帧；声音在同一帧改变 | `{ type: 'smash' }` + `score()` 里在同一提示点上的声音改变 |
| **J-cut** | B 的声音早于 B 的画面出现 | 预期感，被向前拉 | 声音提前 0.3–1.5 s | 在 `score()` 里，于 `cue − lead` 启动 B 的声音 |
| **L-cut** | A 的声音延续在 B 的画面上 | 连贯、回味 | 尾巴 0.5–2 s | 在 `score()` 里，让 A 的声音越过提示点继续 |

**要让它们有变化。** 同一个设计过的转场绝不连续出现两次（普通硬切是默认的关节，可以接连出现）；图形化转场（划像、推移、条幅、穿越缩放）持续 0.45–0.6 s。让画面自己选：穿过眼睛已经盯着的东西（一个方块、一扇窗、一个标签）**推**进下一个镜头；让下一个镜头从刚亮起的东西里**长出来**（一个点、一盏灯、一个节点）；同一个层次的话题切换用**侧向推**；画面上已经在扫过扫描线或划像时用**扫**；在节拍上用**条幅**揭示。在节拍表或镜头表里写明每个镜头怎么离开。

**切点落在哪里（cut seams）。** 当运动跨过切点延续时，切在运动最快的那一刻，而不是它收住时；两侧保持同一方向和同一速度（400 px/s 的右摇接上大约 400 px/s 的右摇）；缩放跨切点保持符号（推近就一直是推近）；主体位置和视线落点在画幅的十分之一之内对齐。绝不要把硬切放进带运动模糊的帧里：模糊留在切点的一侧（Quick starter 会把子帧钳制在场景内）。

Film mode（three.js 的镜头分别编码）：hard、match 和 smash cut 在拼接里就是普通硬切；dissolve 和 wipe 用 ffmpeg `xfade`（`transition=fade`、`wipeleft`、`wiperight`）；甩镜是在 A 镜头最后 4–6 帧和 B 镜头最前 4–6 帧上同方向的一次快速 `pan()`，运动模糊来自 look 流水线；J-cut 和 L-cut 是混音里的偏移量。

## 一个连续镜头

整部片子是摄影机在一个世界里的一次不间断运动。当片子是一个地点、一个主体，而它的戏剧性在于尺度变化、追逐或揭示时，选它。`python3 tools/moves.py form "<the brief>" --seconds N` 会从简报里读出这种形式（一次旅程、"follows"、"grows into"、"close pass"、"drone"、"orbit"、"tek plan"、"takip"）并说明形式的理由；由卡片来定。

- **把世界建一次**，用一台摄影机画出它的每一帧：一个场景从 0 到结尾，不切、不转场。沿着主体或绕着它移动的摄影机以透视站在世界里（`VIEW.of(CAM.path(keys, t), S)`，keys 为 `{ t, x, y, z, tx, ty, tz, fov }`；手绘的东西用 `view.sprite` 摆放，地面和物体用 `view.poly`，远处的东西用 `view.haze` 淡出）；正面运动可以保持在平面深度上（`plane()`）。
- **运动就是故事：** 把它写成运动内部的 3–4 个节拍（定场大景 · 跟随 · 近距离掠过或推向细节 · 拉远或揭示），每个都是 `CAM.path(keys, t)` 里的一个 key；摄影机在 key 之间流动，绝不停在某一个上。移动的主体用 `CAM.follow` 追（摄影机像摄影师那样落后它零几秒）。
- **什么让它成立：** 真实的尺度变化（主体从几个像素大到充满画面）、视差（近处平面飞快掠过，远处几乎不动）、前景物体在最接近的时刻擦过镜头、进入掠过处的速度坡度（`retime`）、主体穿过对焦平面时的焦点转移、一点经过低通滤波的漂移（`CAM.handheld`，幅度 0.2–0.4）用来表现无人机或手持。
- **构图也在动：** 主体始终在画面里有一个有力的位置（三分线、中心、留出前方的空间牵引运动方向），同时它的尺寸在变；地平线随摄影机转弯而倾斜，绝不偶然歪。
- **超过大约一分钟时**，单一运动很少撑得住：把接缝藏在里面（一次甩镜、摄影机从前景物体后面或某个表面穿过、推进黑场），并始终守住同一个世界。
- **在广告里：** 绕产品的一次环绕，同时一道光扫过它；一次穿过产品世界并抵达产品的运动；一次从细节拉开到全景。尾卡可以单独跟上来，作为唯一的硬切。

## 选择

- **先说理由：** 推 = 张力或重要，拉 = 释放或揭示，跟 = 行进，升降 = 尺度，环绕 = 审视，手持 = 在场，锁定 = 让主体或光去动。
- **转场服从想法：** 默认是硬切；溶解意味着时间，甩镜意味着能量，match cut 意味着关联。为装饰而用的通用转场会削弱片子。
- **大动作只花一次：** 推拉变焦、定格、撞切和速度坡度正因为罕见才立得住。
- **相邻镜头要不同**，在运动、景别或方向上变化，同时影片保持一种视觉语言（统一中的多样：`quick-mode.md` §7、`craft.md`）。
