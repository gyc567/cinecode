# 排版

这部片子的字体如何被挑选、设定、驱动与检查，好让它读起来像设计师的作品。眼光来自三个地方：用户和研究员收集来的真实排版（§9）、从这些参考里量出来的数据（§9.2），以及跨片子记住的用户自己的喜好与否决（§9.3）。下面的规则是地板；参考和品味日志决定风格。

文件：`type-library.json`（精选字体与配对；给普通人的同一套库：`docs/type-library.md`）、`starter/type.mjs`（运行时，在 quick-mode 的 `film.html` 里内联）、`templates/type-specimen.html`（样张卡）、`scripts/type_deal.py`（通过 `deal.py … type` 调用）、`scripts/type_refs.py`、`scripts/type_extract.py`、`scripts/captions.py`。

## 1. 配对

- **一个声音，最多两个字族**，再加一个 mono 给数据、代码或时间戳。超级字族（同一设计下的衬线 + 无衬线 + 等宽，如 IBM Plex）算作一个声音。
- **结构上的对比，不是情绪上的对比**：衬线配无衬线，窄体配常规宽度，高对比度的标题字配低对比度的正文字。两个只差一点点的 grotesque 看着就像失误。
- **共享的比例**：相近的 x-height（库里是 `metrics.x`）和相近的字宽节奏，这样两者排在同一行上时不会有一个显得被压小。共用一行时，cap height 比例差距在约 10 % 以内。
- **角色全片固定**：`display` 给标题和大字，`text` 给字幕、标签、下三分之一条和法律声明行，`mono` 给数字和数据。一个角色绝不中途换字体（签名式的例外见 §11）。
- 库里每个配对都带一句理由（`why`）和一套测试过的 `setup`（字重与轴值）。

## 2. 各画幅的尺度与手机下限

以交付物的短边为基准的模数尺度（1080×1920、1080×1080 和 1920×1080 都是 1080 px）：

| 级别 | 用途 | 字号（占短边比例） |
|---|---|---|
| T1 | 标题卡，单词冲击 | 0.11–0.16（9:16、1:1）· 0.09–0.13（16:9） |
| T2 | 大标题、价格、CTA | 0.065–0.09 |
| T3 | 副标题、说明字幕、下三分之一条姓名 | 0.04–0.055 |
| T4 | 标签、职务行 | 0.034–0.04 |
| L | 法律声明行（静止，屏幕上 ≥ 3 s） | ≥ 下表的法定下限 |

相邻级别的比值在 1.25–1.6 之间（一部片子只用一个比值）。**手机下限（硬规则）**，按交付像素里的 cap height 计，由 `typeCheck()` 从声明的 `TYPE.check.sizes` 检查：

| 画幅 | 正文字 cap height | 法律声明 cap height | 原因 |
|---|---|---|---|
| 9:16 与 1:1（宽 1080） | ≥ 28 px（短边的 2.6 %） | ≥ 22 px | 1080 px 铺满手机约 390 pt 的宽度 |
| 16:9（1920×1080） | ≥ 40 px（3.7 %） | ≥ 32 px | 在手机上内嵌观看时高度只有约 0.56× |

字号 = cap height ÷ 该字体的 `metrics.cap`（约 0.65–0.72）。样张卡就是用这个方式给自己定号的。

## 3. 字距、行距、行长

- **字距按字号走**（`autoTrack()`，以 em 为单位份额）：小字放开（短边 2.5 % 以下 +0.015 em），标题收紧（6 % 以上 −0.01 到 −0.02 em）；**全大写永远放开**（小 +0.08、中 +0.05、大 +0.03）。小写标题的正字距绝不超过 +0.01 em，除非它是这部片子的签名（§11）。
- **行距**：正文 1.25–1.4；标题 0.95–1.05；全大写标题 1.0–1.1，**另外加上变音符号的余量**（§7）。
- **行长**：9:16 每行 ≤ 32 字符，16:9 ≤ 42；屏幕卡片 ≤ 6 个词。`wrap()` 负责断行；`balance` 让各行长度接近（用于标题）。

## 4. 视觉字号与可变轴

- 存在 `opsz` 就用它（Fraunces、Bodoni Moda、Newsreader、Imbue、Piazzolla、Bricolage、Big Shoulders）：标题用高值（更细的极细笔画、更紧的间距），小字行用低值（更结实）。Chrome 会自动按像素大小套用 `opsz`；当 look 需要别的效果时，用一个停靠点把它钉住。
- **把轴的变化绑在音乐上，而不是绑在计时器上**：字重呼吸（`breathe(t, BPM, lo, hi)`）在拍点上到峰值；宽度变化（Anybody、Archivo、Mona Sans）落在击打上；SOFT/WONK（Fraunces）、ELSH（Danfo）、YEAR（Climate Crisis，绑到日期计数器）、XROT/YROT（Tilt Warp）、MONO/CASL（Recursive）、BNCE（Shantell Sans）在某个 cue 上或跨一个段落里变化。一次只动一根轴；在这一行必须被读完之前先稳定下来。
- Canvas 2D 的事实（在无头 Chrome 里实测）：字重轴在 `font` 字符串里是连续的（`650` 可以用）；宽度会吸附到简写里的 stretch 关键字；**其他任何轴和每一种 OpenType 特性只能通过 `FontFace` 描述符**（`variationSettings`、`featureSettings`）在分离的 canvas 上生效，所以非字重轴要通过**停靠点**来动画（每个停靠点一个字体实例，`TYPE.roles.x.stops`），每帧取最近的停靠点。

## 5. OpenType 特性

在 TYPE 块里按角色设置（`features: '"tnum", "ss01"'`），以 `FontFace featureSettings` 方式应用：

- 计数器、价格和时间码用 `tnum`（数字变化时不能抖动）；老式衬线里的价格用 `lnum`；全大写配 `case`（会抬高短横线和括号）；`ss0x` 风格集作为一种 look 决定（Le Murmure ss01–ss10、Uncut 风格的替换字形），在字体规格里写明；食谱和规格用 `frac`。
- Fontsource 的网络文件保留的特性很少（库里的 `features` 就是实际能渲染的）。对于只有完整字体才有的特性（`features_full`），在本地加载完整文件（`load.full`，放进 `fonts/`）。
- `letterSpacing` 和 `fontKerning` 由 `setType()` 设置；字距对（kerning）保持开启。

## 6. 排印运动

配合工具包的 MOTION 助手和 `typeReveal()`：

- **单位**：≤ 2 个词的标题按字符，整行文字按词；错开幅度是入场时长的 0.35–0.6，顺序用 `seq`（阅读顺序），除非创意另有说法。
- **遮罩与基线位移**：单位在一个遮罩内上升行高的 0.6–1.0，遮罩顶部包含变音符号余量，所以 İ 的点和 Ğ 的 breve 永远不会被切掉。入场缓出（`EASE.out`，轻微过冲用 `MOTION.back`），退场缓入；没有任何东西是线性移动的。
- **可读性优先**：一行落位后至少静止 `0.4 s + 0.06 s × words`；一次只动一个文字层；在眼睛需要读之前，运动必须停下。
- **字重与轴的运动**遵循 §4。字距动画（从宽到正常）只允许用在标题上。
- 广告、解释视频和短片用的**动感手法**（每片一到两个，每个都得有活儿）：带错开落下、其中一个字母被改成强调色的字母；大标题以一个彩色句点收尾；一个单词砸下、上下带轮廓残影；数字一位一位滚动或翻动、滚动下方带一道刻度；标签（chip）擦入并排成一行；一条斜向字带滚过章节回顾；边角的小字框住镜头。被选中的东西只用一种强调色。
- **绝不为填满屏幕编造真实内容**（假邮件、假名字、假数据）：改成画圆角块或模糊的线条。
- **在全尺寸静帧上做版式检查**：两个统计数据绝不占据同一水平带上的同一高度；当底部 12 % 留给字幕或进度条时，内容不要进那部分。

## 7. 网格、孤字、土耳其语

- **网格**：一个边距（≥ 短边的 7 %），每张卡一个对齐方式（左对齐或居中，绝不两者都用），基线落在共享的节奏上（就是正文行距）。信息类用左对齐，仪式类用居中（标题卡、片尾卡）。
- **孤字与孤行**：末行不能只有一个词（`wrap()` 会把词挪下去），行尾不能落在一个单字母的词上，数字和它的单位要待在一起，用不换行空格（`24.990 ₺`）。
- **土耳其语大小写**：一律用 `up(s, 'tr')` / `low(s, 'tr')`（`istanbul → İSTANBUL`，`IŞIK → ışık`）；土耳其语片子里出现的英文词用 `'en'` 处理大小写。`TYPE.lang` 设置默认值。
- **变音符号余量**：文本框、遮罩和行距都从 `headroom()` 量起（İ、Ğ、Ü 在 cap height 之上；Ş、Ç 在基线之下），绝不能只按 cap height 量。每个文本框都要用 `İĞÜŞÖÇ ığüşöç` 测试。
- **覆盖度**：库里每个字体都验证过 İıĞğŞşÇçÖöÜü；`typeCheck()` 依然会在加载时测量每个角色的字形（用户自己的字体不在库里），遇到缺字就停下渲染。

## 8. 电影感标题、广告排印、出镜口播的字幕策略

**电影感**（Art of the Title 是参考书架）：一张标题卡，居中或靠在强边的左缘，大量留白，在一个乐句上慢慢揭示（1–2 s），克制的配色（墨色落在片子最暗或最亮的明度上），署名用 text 角色排在 T4、用放开的大写字距。能进世界的标题就进世界（在某个表面上、在某个前景元素后面），但阅读规则照旧成立。

**广告**（Fonts In Use 的 `format:advertising`、`posters-flyers`、`social-media`）：
- **下三分之一条**：姓名用 text 角色（T3，字重 600）压在实色块或压暗层上，职务行在下面（T4）；左缘对齐到边距；9:16 里放在底部 UI 区（下方 20 %）之上。
- **价格和 CTA**：价格用 display 或 mono 加 `tnum`（T2），CTA 是全大写的动词，放在对比色的胶囊里，每个变体位置相同；绝不超过一个 CTA。
- **法律声明行**：text 角色，静止，至少达到法定下限（§2），≥ 3 s，对比度 ≥ 4.5:1。

**出镜口播启发的字幕**（给有旁白的片子与广告的可选样式；绝不作默认）：
- **词级锁定的动感字幕**（可选；需要 Whisper，只有用户同意后才安装；普通整行字幕什么都不需要，见 `craft.md` §5），从旁白的词时间轴来：用 Whisper 的词级时间戳转录我们自己的 TTS 台词（`captions.py cmd`），组成 1–3 个词一组（`captions.py build`，≤ 18 字符，在 > 0.25 s 的停顿和标点处断开），把 `CAPTIONS` 贴进 film.html；每个词在自己被念到的时刻弹入。
- **强调词**：每组最多一个词，用对比字体（比如斜体衬线对 grotesque，或一个 `accent` 角色）或强调色：那个承载意义的词（价格、益处、动词）。
- **强调处推近**：在 `--accent` 里用 `!` 标记的词，会在 0.08 s 内把画面推近 5–8 %，再用 0.5 s 稳住（`captionPunch()`）；一句最多一个，1.5 s 内绝不超过两个。
- **快节奏**：组的切换跟着说话，而不是跟着计时器；剪切和击打可以落在字幕切换上。
- 字幕位于安全区内（9:16 时在 UI 区之上的中央带），用 display 或 text 角色，T2–T3，画面背景杂乱时加阴影或底框；排印 QC（§13）适用于它们出现的每一帧。

## 9. 设计师的眼光

### 9.1 排版参考

按片子收集（只留情绪：一个链接和一张小缩略图；绝不交付）：

- `asset_search.py search --type typography`（与 `type_refs.py` 相同）：**Fonts In Use** 按 `format:` 分类的真实用例（film-video、advertising、posters-flyers、branding-identity、social-media、packaging …）、按 `topic:` 分类（film-and-television、music、food-beverage、technology、fashion-apparel、sports …）或一个字体页面 URL；它的 robots.txt 要求 10 s 的爬取延迟并且禁止搜索，所以工具从不搜索；**Art of the Title**（`title:<slug>` 或关键词，用于电影感用途）；**Typewolf**（一个字体家族的页面：样张、建议配对、使用它的站点；`sotd`）；**Are.na** 公共 API（排版频道、图像块；它的 Content-Signal 允许参考用途）；**Typographic Posters** 只收链接（页面在浏览器里渲染）。字厂样张从库里链过去。
- **用户自己的参考放在 `refs/type/`**：截图、照片、扫描件。**Instagram 工作流**：用户把喜欢的帖子存进一个 Instagram 收藏（比如 "film type"），再把它们的截图丢进 `refs/type/`（AirDrop、照片导出或一个截图文件夹）；工具包从不登录或抓取 Instagram。要 3–8 个参考；用户自己的参考高于其他一切。
- 一块**类型板**（`asset_search.py board --dirs research/type-refs --refs refs/type --out research/type-board`）：每个参考要取什么（大小写、字重对比、版式、颜色、运动）以及要避开什么。看 contact sheet，不要看单张图。

### 9.2 从参考里测量

`type_extract.py measure refs/type/* research/type-refs/thumbs/0N.img --out type/direction-draft.json` 为每张图估算：尺寸层级（最大/中位行、尺寸级数）、全大写还是混合大小写（`--case 1=caps` 可覆盖）、字距代理指标、对齐、文字区占比、文字与纸面的颜色，以及一个置信度；随后给出一份方向草案（大小写、对齐、`title_share`、尺度比、字距、颜色）。它是启发式的：把你自己对缩小参考的读法写进 `visual_read`（这些字做了什么、为什么适合这部片子），两份都拿给用户看，只有被确认的草案才能成为这部片子的字体规格。样张卡用 `dir=direction-draft.json` 接收它（路径相对 `type/specimen.html`）。

### 9.3 品味日志

每一个排印决定都是一条品味记录：`deal.py pick type`（喜欢；其他被发到的配对：pass）、`redeal type --reason`（否决，附用户给的理由）、`taste type <pairing or font> --verdict like|reject --reason "…"` 用来记录对某个细节的反应。记录保存字族、标签和用户给的一句话理由，放在项目的 `history.json` 里，并且（加 `--user-taste` 时，或 `~/.cinecode/taste.json` 存在后；`CINECODE_TASTE` 可覆盖路径）放在一部片子一个用户级文件里共享。下一次发牌时的权重（新的在前，每个事件衰减 0.85）：喜欢的配对 ×(1+0.5d)，共享字族 ×(1+0.25d)，共享标签 ×(1+0.12d)；被否决的配对 ×(1−0.6d)，字族 ×(1−0.35d)，标签 ×(1−0.15d)；pass 掉的标签 ×(1−0.03d)；look 匹配 ×1.6；钳制在 0.05–4 之间，所以没有东西会变得不可能，而用户自己的答案（`pick type own --families "…"`）永远是开放的。`deal` 会打印出日志当前偏向谁、避开谁；出牌时引用它。

## 10. TYPE DIRECTION 步骤（两种流程都在 look 之后）

1. **参考**，按用户对参考的回答来："search" = 用户的 `refs/type/` + 为这个用途研究的 6–12 个参考（§9.1）；"only my own" = 只有 `refs/type/`；"skip" = 不要（直接去第 3 步）。出一张 contact sheet。
2. **测量**：对最强的 3–5 个跑 `type_extract.py measure`；加上你的视觉判断；写出一段字的排版方向。
3. **发牌**（精选配对优先）：`deal.py deal type --purpose <cinematic|ad> --look <look or recipe id> --mood "<the film's tone in a few words>" --title "<the headline as it stands on screen>"`（再加 `--sub`、`--cta`、`--still`：卡片只显示这部片子自己的词，配它的画面）（基调，比如 "komik çizgi film"（喜剧动画短片）、"korku"（恐怖）、"belgesel"（纪录片）、"lüks"（奢华），只会挑出承载这种基调的配对；look 再进一步收窄）→ 按用途、土耳其语覆盖和品味过滤出 3 个配对；`type/specimens.json`、`type/specimen.html` 和 `type/type.mjs` 会被写出。
   **只有当三个里没有一个承载这部片子的声音**（或用户要求别的）时：`python3 tools/fonts.py search "<the film's tone>" [--role text]` 按基调给 2000+ 款开放授权、带土耳其语字母的字体排序（用 Google Fonts 的表现力评分：playful、calm、rugged、vintage、sophisticated、futuristic…），并且排除 AI 设计用滥了的字体；`fonts.py role <id>` 打印出钉住的 TYPE 角色。
4. **在片子的 look 下做样张卡**，按交付画幅，每个配对一张（标题、副标题、标签、数字、一行土耳其语文字；广告还要下三分之一条 + 价格 + CTA + 法律声明行；电影感的则是一张由方向塑形的标题卡）：`node tools/capture.mjs --url type/specimen.html --frames 0 --format jpeg --out
   review/type-<n> --params "i=<n-1>&w=…&h=…&purpose=…&title=…&bg=…&ink=…&accent=…&still=../review/<frame>.jpg&dir=direction-draft.json"`。
   用本地文件的配对需要先把它的文件放进 `fonts/`（由用户下载；GitLab 和 Fontshare 的文件绝不自动抓取）。
   对于 2–4 张标题卡、章节卡、CTA、封面或片尾卡，并且风格混合的，用 `--mixed` 再发一次牌（主导 + 强调配对，§12）；它的卡片来自 `type/specimens-mixed.json`（`--params "spec=specimens-mixed.json&i=…&acc=<accent word>"`）。
5. **询问**：有交互式提问工具时用它（选项 = 三个配对 + "my own"），并把三张卡片并排展示；否则用编号提问。把这次挑选连同用户理由记录下来；选 "none" 就是带着理由的 `redeal`。
6. **字体规格**：把 `type/specimens.json` 里的角色对象复制进 TYPE 块（film.html 或场景页）和 `STATE.md` 的 Type 一行，加上 `check.strings`（标题、CTA、一行土耳其语文字）和 `check.sizes`；每个字族在 LICENSES.csv 里一行（角色 `font`，授权与 URL 取自库里）。

## 11. 规则层级：硬规则与可破的风格规则

**硬规则：绝不打破**，由审阅者检查，部分也由代码检查：
可读性（手机 cap height 下限、正文与法律声明行对比度 ≥ 4.5:1、大标题 3:1、保持时长）；没有切掉的变音符号或缺失字形；授权（库里或已验证的授权、LICENSES.csv 一行、品牌与 Fontshare 的文件绝不外传也绝不提交）；响度（−14 LUFS、≤ −1 dBTP）；开场钩子（第 0 帧就有动作，不要从黑场淡入）。

**可破的风格规则：一部片子可以有意打破的默认值**：最多两个字族；手写体或装饰体只用于 ≤ 3 个词；效果（描边、阴影、纹理、3D）只加在一层上；每张卡一个对齐方式；字距按字号走；一次只动一根轴；电影感标题居中、广告标题左对齐。

**有意打破规则**允许成立的前提是以下三条都满足：
1. **在计划里被点名**并给出理由（STATE.md / ROADMAP.md："rule break: <the rule>: <how>, because <a reason from this story>"）；
2. **作为这部片子的签名被一致地执行**（每一处都遵循同样的逻辑；不是一次性的）；
3. **由审阅者**确认是有意为之：审阅者在计划里找到这次被点名的破例，确认它在全片一致，且没有触碰任何硬规则。没有被点名的破例就是失误，得修掉。

## 12. Stil karıştırma / 混搭风格

混搭卡片在同一张卡上放**一个主导声音和一个强调**（粗 grotesque + 手写体，窄无衬线 + 斜体衬线）。这里的每条规则都是**可破的风格规则**（§11）：只有在它是被点名、一致、并且经审阅者确认的签名时才打破。用 `deal.py deal type --mixed` 发牌（`type-library.json` 里标了 `mixed` 的配对）；强调是第四个 TYPE 角色，叫 `accent`。

**规则**
- **一个主导 + 一个强调**：每张卡最多 2 个字族。一个小 mono 标签（1080 下 ≤ 11 px 当量，比如日期或代码）不算。
- **最多在 2 个轴上做对比**：宽度 + 字重（Archivo Expanded Black 配一支刷子手写体），或结构 + 斜度（Bricolage 窄体配 Instrument Serif Italic）。绝不要分类、宽度、字重、年代四个一起上。
- **强调只占 1–3 个词**，承载情绪而不是信息（感觉、动词、转折）；绝不做正文、字幕、整句、价格或事实。
- **尺寸**：主导的 x-height 约为强调的 2–3 倍；强调的 cap height 是主导的 40–60 %。
- **把手写体收进去**其自身高度的 15–30 %，压进粗体的基线（与主导行的底部重叠）。
- **只旋转强调**，最多 6°；主导保持在网格上。
- **效果只加在一层上**：描边、阴影、纹理或 3D 加在主导*或*强调上，绝不两者都加。
- **色彩层级**：主导用中性色或品牌色，强调用一个高亮色；绝不让两者都被高亮。
- **文字颜色按角色变化，但绝不随机：** 一部长片不会把每个字都设成白色。给正文三到四个角色，用片子的配色（正文墨色、给关键数字或姓名的强调色、给次要行和日期的更安静的色调，以及章节卡的颜色），保持每个角色一致，并让章节卡或各幕带上自己的色调，这样十分钟不会读起来像同一张卡重复十次。对比度在最忙的那一帧上检查（`type_extract.py contrast`）。
- **动画顺序**：主导先落位；强调在 150–300 ms 之后书写或画入（笔画揭示，或用 `typeReveal` 按字符），跟着音乐。
- **用在哪**：标题、章节、CTA、封面和片尾卡。**不用于**字幕、法律声明行、数据、价格或长引文。**用多频繁**：每片最多 2–4 张混搭卡；其余部分用普通的 TYPE 角色。
- **土耳其语渲染测试（强制）**：选强调之前，先用 `İğneli Şükrü Çığlık Öğün` 渲染每一个强调（样张卡会做，`typeCheck()` 也会检查强调角色的字形）；字体的 `latin-ext` 子集并不能证明手写体能把 İ、ğ、ş、ı 正确画出来。

**过头了：审阅时该修的信号**
- 一张卡上超过 2 个字族（不算那个小 mono）；
- 手写体超过 4 个词，或整句话都用手写体；
- 两层都加了效果；
- 每张卡都混搭；
- 主导和强调同样大，或同样颜色；
- 强调旋转超过 10°，或排成好几行；
- 字体逐词变化。

**库里现成的混搭配对**（授权都干净，并且已从字体文件验证过土耳其语）：Archivo Expanded Black + Caveat Brush + JetBrains Mono · Unbounded Black + Sedgwick Ave Display + Space Mono · Anton + Yellowtail · Big Shoulders Display Black + Marck Script + IBM Plex Mono · Bricolage Grotesque ExtraBold (wdth 75) + Instrument Serif Italic + Geist Mono · Anybody (wdth 150, Black) + Kalam Bold · Dela Gothic One + Mynerve · Syne ExtraBold + Square Peg（字幕用 Figtree；Inter 只作为用户自己的选择）。

## 13. 排印 QC（审阅标准）

自动化：加载时跑 `typeCheck()`（缺字形和对比度 < 3:1 会停下渲染；cap height 低于手机下限、对比度 < 4.5:1、行过长都会以 `qc:` 记进抓帧日志）；`type_extract.py contrast <frame> --box … --ink …` 检查压在画面上的文字（只框住被检查的那部分文字）；样张卡的页脚。审阅时人工做：

1. **手机可读性**：在手机尺寸下看 9:16 导出；每一行都在它的保持时长内可读。
2. **与背景的对比度**，在每个文字时刻最忙的那一帧上查（`type_extract.py contrast`），而不只是看配色。
3. **行长与孤字**：≤ 32/42 字符，没有单词的末行，单位与数字不分离。
4. **变音符号没被切掉**：İ 的点、Ğ 的 breve、Ş/Ç 的尾钩在遮罩、文本框和行距里都完整（逐帧看 strip）。
5. **字族一致**：角色在全片不换字体（除非要点名的签名，§11）；混搭卡遵循 §12（每片 ≤ 2–4 张，且没有任何过头的信号）。
6. **大小写**：土耳其语大小写正确（`İ`、`ı`），大写字距放开。
7. **运动**：入场缓出，一次只动一个文字层，行保持静止以便被读完。
8. **授权**：每个字族在 LICENSES.csv 一行；git 里没有品牌或 Fontshare 的文件。
9. **破例**：每一次都在计划里被点名、一致，且没有触碰硬规则。

## 14. 加载

TYPE 块（film.html、场景页）通过 `FontFace` 从钉住版本的 Fontsource CDN（`@fontsource/<pkg>@<version>` 静态切分，`@fontsource-variable/<pkg>@<version>` 可变文件的 `full` / `standard` / `wght`；latin + latin-ext 范围）或从本地 `fonts/` 加载每一个角色（gitignored：品牌字体、Fontshare 字体、在钉住 commit 上下载的 Velvetyne 文件）。加载失败会停下渲染（`READY` reject）。把 woff2 文件拷进 `fonts/` 或 `engine/vendor/fonts/` 以便离线渲染。
