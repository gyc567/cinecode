# Looks（视觉风格）

一个 look 是**参数预设**，不是手写效果：一套配色、一套布光、一套材质风格，外加一条为 **three.js**（MIT）上的 **pmndrs/postprocessing**（Zlib）准备的收尾链路。预设放在 `starter/looks.mjs`；`starter/scene.html` 用 `?look=<id>` 渲染其中任意一个。每个预设都是**起点**：每部片子都会按自己的创意和参考去改。

## 1. 八个家族

| 预设 id | 家族 | 一句话说明 | 建议画幅 |
|---|---|---|---|
| `cinematic-realistic` | cinematic realistic（电影写实） | 低位暖调主光、电影感色调曲线（AgX）、轻微辉光、颗粒、暗角 | 16:9, 2.39:1, 9:16 |
| `anime-cel` | anime / cel（动画／赛璐璐） | 三阶卡通着色、深色描边、饱和的日光天空 | 16:9, 9:16 |
| `pastel-storybook` | pastel storybook（粉彩绘本） | 柔和正面光、四阶卡通、提亮的粉彩、较低的对比 | 16:9, 1:1, 9:16 |
| `claymation` | claymation（黏土定格） | 粗糙哑光材质、移轴微缩景深、角色按二帧画 | 16:9, 1:1 |
| `papercut-collage` | papercut / collage（剪纸／拼贴） | 二阶卡通、细边、静态纸纹、12 fps | 16:9, 1:1, 9:16 |
| `retro-synthwave-vhs` | retro / synthwave / VHS（复古／合成波／VHS） | 无光照霓虹形状、重辉光、色散边缘、扫描线、30 fps | 16:9, 9:16, 4:3 |
| `glossy-commercial` | glossy commercial（高光商业） | 明亮扫光、低粗糙度、锐利高光、干净的颜色 | 16:9, 1:1, 9:16 |
| `handmade-ink` | hand-made traditions（手作传统） | 纸色调、近乎单色、墨线、半调网点、12 fps | 16:9, 9:16 |

画幅永远跟着交付物走（16:9、9:16、1:1、4:5、2.39:1）。没有哪个预设会加黑边。

## 2. 每个参数做什么

| 组 | 参数 | 作用 |
|---|---|---|
| `palette` | `[background, ground, main, secondary, accent]` | 测试场景用的颜色；片子自己的配色会把它们换掉 |
| `materials` | `shading: pbr / toon / unlit`、`steps`（卡通）、`roughness`（pbr） | 表面如何响应光；卡通的 steps = 平涂色阶的数量 |
| `light` | `azimuthDeg`（0 = 逆光，±90 侧光，180 来自相机方向）、`elevationDeg`、`intensity`、`ambient`、`fog`、`exposure` | 主光方向与强度、天空补光、纵深雾；`exposure` 缩放所有灯 |
| `post.tone` | `AGX`、`ACES_FILMIC`、`NEUTRAL` | 色调曲线：AgX 给自然的高光，ACES 给更有劲的对比，NEUTRAL 让平涂颜色保持准确（卡通、平面设计） |
| `post.saturation`、`hue`、`brightness`、`contrast` | −1…1（hue 用弧度） | 色调映射之后的分级 |
| `post.bloom` | `threshold`、`smoothing`、`intensity`、`radius` | 亮部辉光；低 threshold = 霓虹，高 = 只有太阳发光 |
| `post.chromatic` | 在 1080p 下的像素数 | 边缘色散（镜头、VHS） |
| `post.outline` | `color`、`strength` | 给每个网格描墨线，带 `userData.noOutline` 的除外（地面） |
| `post.tiltShift` | `focusArea`、`feather` | 微缩景深带（黏土定格、微缩景观） |
| `post.dotScreen` | `scale`、`angle`、`opacity` | 印刷半调网点 |
| `post.scanline` | `density`、`opacity` | 视频扫描线 |
| `post.pixelation` | 以像素为单位的颗粒度 | 粗颗粒像素 |
| `post.vignette` | `offset`、`darkness` | 四角压暗 |
| `post.grain` | `intensity`、`size`、`chroma`、`hold` | 胶片颗粒（工具包自带效果）；`hold: 1000` = 静态纸纤维 |
| `render` | `fps`、`hold`（角色按二帧 = 2）、`aspects` | 帧率与步进；相机保持逐帧（looks.mjs 里的 `heldTime()`） |

## 3. 如何选择和修改一个 look

1. 从风格问题里定下的方向所属的基础家族出发（或者从最接近参考的那个预设出发）。
2. 先以小尺寸渲染测试场景看看这个预设（`--params "look=<id>&w=960&h=540"`），再用它渲染片子自己的第一个镜头。
3. **一次只改一组**，按这个顺序：光 → 配色 → 材质 → 收尾。每改一次重渲染一帧。
4. 把这部片子的 look 写成 `engine/looks.mjs` 里它自己的对象（复制一个预设，给它新 id，改数值），这样工具包的预设保持原样。
5. 已确认的风格帧是之后每一帧的标尺。

## 4. 从参考定制 look（"从我的参考"这条路）

当用户把图片丢进 `refs/`：

1. **小看看，同时一步量完：**
   ```bash
   python3 tools/asset_search.py board --refs refs --out research/refs-board      # one numbered sheet of refs/ (any image type)
   python3 tools/direction.py measure refs/*.* --out research/direction          # palette, key, contrast, saturation, warmth, a look patch
   ```
2. **用上测量结果：** `research/direction/direction.json` 里的配色（从暗到亮，强调色在最后）和 look patch 就是预设的起始值（`direction.py` 就是方向板用的同一个工具，见 §10）。
3. **读光：** 主光从哪来（方位角、仰角），影子有多硬，最暗区域有多暗，高光会不会发光。
4. **读收尾：** 有颗粒还是干净，有边缘还是没有，饱和度高还是低调，对比高还是被提亮，有没有印刷或视频纹理。
5. **照着这些读数写出预设**（从最接近的家族出发改数值）。
6. **并排比较**（风格帧在前，参考图同高），一次改一组：
   ```bash
   ffmpeg -v error -i review/style-v1.png -i refs/a.jpg -i refs/b.jpg -filter_complex "[0]scale=-2:540[s];[1]scale=-2:540[a];[2]scale=-2:540[b];[s][a][b]hstack=3" -frames:v 1 review/style-v1-vs-refs.jpg
   ```
受版权保护的影视作品和图片**只作为情绪参考**：可以研究和比较，绝不放进画面里。

## 5. 代码适配：东西怎么被呈现（以及怎么处理人）

这一节决定一件事怎么被呈现，绝不决定片子讲什么：任何主体、地点、时刻或光线都可以是这部片子。

**什么读起来像老 CG。** 一个写实的人物、建筑、船只、载具、房间或产品，如果整体完整地、中距离地、在干净均匀的光里出现，无论打磨多久都读起来像旧渲染。同样一个东西，换这些方式就很好看：作为一个局部或一个微距、运动中、带着真实材质、有方向的光、在一种有意为之的媒介里被画出来（油毡版画、墨、纸、卡通）、作为真实材质（照片、档案、授权允许的素材），或者部分处在阴影里。这张卡给每样这样的东西点名并给出路线。低多边形配平面无纹理的脸不是一种媒介：它本身就是老 CG 的样子，不管配色多时髦。三维世界从没有「关掉纹理」这个选项：每个表面都带着材质（边缘、磨损、印刷或笔刷纹理），光有方向且会衰减，纵深把近和远分开。

**写实渲染的标准。** 当 look 是写实的或材质的（`shading: pbr`；`cinematic-realistic` 或 `glossy-commercial` 家族；一个本就该读起来像真的世界），每个场景在给出之前都要达到这条线。风格化的 look（卡通、赛璐璐、纸、墨、像素、黏土）则按它自己那种风格的工艺来要求，下面这些一条都不套用到它身上——PBR、颗粒和照片级后期会毁掉它。

- **真实材质，来自真实来源。** 任何被光照到的表面都不是平的、没有纹理的：albedo + roughness + 法线／微观细节 + 菲涅尔，有动因的主光，真实阴影 + 环境光遮蔽 + 反弹。先找真实扫描贴图或 HDRI，再考虑手做的（`asset_search.py search --type texture` / `--type hdri`，Poly Haven CC0；`--type image` 用于真实贴图或底板，那比着色器更合适的场合——行星表面、望远镜天空、海底）；只有在没有扫描件合用时才手做。**只用在会被看到的地方：** 在黑暗、雾里或剪影中，表面是读不出来的，所以那里的杠杆是光和空气，不是纹理，也就不必为它去抓扫描件。
- **每一帧都要有纵深。** 近、中、远三层，远的靠明度或颜色分开；哪怕在最亮的光里，每个受光表面上都有微观变化。一个能看到远处的户外视野（风景、城市、地平线）要带上真实空气永远带来的那层薄空气透视，正午也一样：`fog` 压低并用天空的颜色，让远处发白发冷；这是物理，不是氛围，而少了它的远景会读起来像 CG。霾、雾或天气只有在故事里本来就有时才作为氛围出现。
- **电影级收尾。** ≥1.5× 超采样到半浮点缓冲以获得干净边缘（设 `post.msaa: 0`，免得它和默认 MSAA 叠加），一次合并的后期 pass（ACES 或 AgX 色调、分级、经过测量的辉光、细颗粒）。真实的快门运动模糊留给用户决定，默认关闭（它在快速镜头上的渲染代价是 ×6–8，`purpose-cinematic.md` §2）：只在卡或风格帧打开它时才用，而且只在真的有东西在动的地方用。
- **一种材质语言；针对每一个场景，不是只针对测试场景。** 全片用同一套光和材质逻辑；每个场景都为自己而建，绝不是对所有场景做一遍自动处理；这条线要由**每一个**场景在片子被叫做完成之前都达到——前面的世界精致、后面扁平，不算完成。

**快不等于质量的代价；浪费才是。** 材质和光是便宜的——贴图和 HDRI 就是查表，所以放心用。真正的代价是逐帧的乘数：每帧重渲染整个场景的反射（把环境预烘一次，或者用屏幕空间）、大量快门子帧（要少，只用在运动处）、高采样的环境光遮蔽（屏幕空间、低采样、烘进静态贴图）、每个物体一盏投影灯（一两盏，一张阴影图）。建一张最重场景的全尺寸测试帧，把它的毫秒数打出来，把全尺寸下每帧的时间压在 1 秒以内；超了就点名那个昂贵的效果并砍掉它，不要降低 look。已确认的那一帧和它的耗时是其余部分的标尺。

更难的：近距离的写实人脸和手。当故事需要一个人时：风格化角色（卡通、黏土、纸）、剪影、远处的人影、拍手和物件而不是脸，或者和用户提供的实拍／生成素材混合。脸是允许的；要做特写就建议风格化或剪影。

**选风格之前的代码适配检查。** 对着创意问：观众必须近看什么？写实的人体或脸在运动中（走路、跳舞、演奏运动、发丝和手）是代码唯一画不好的东西，无论它磨多久。于是 ★ 保留创意和它的主体，并用这个媒介擅长的方式呈现人物（风格化的演员阵容，`characters.md`）。写实选项仍然留在名单上，配一句诚实的话："insan figürleri kodla kaba görünür"（用代码画人，人物看起来会很糙）。

**硬需求就给它可行的那条路线。** 当创意要求的是代码不擅长的东西（写实人物、人群、体育、舞蹈、动物特写）时，★ 就是最接近且能做好的那种风格，用一句话连同理由说明；three.js 能深入地构建物体、建筑和景观，按「什么读起来像老 CG」那节所说去布光和摆放；写实角色不是它的强项，但想要的用户会得到它们，代价只说一次，然后全力帮忙。工具包绝不会在一张不管用的画上花掉一小时再拿出来给人看：前两个场景就是测试（`quick-mode.md` §3），测试失败就用一句话切到 ★ 路线。除非用户主动要求，接入的图像或视频生成器不在这些路线之列（`SKILL.md`：Generated pictures only when the user asks）。

**风格为主体服务。** 最重的风格化（半调、故障、笔触、重颗粒、海报化、色彩偏移）放在背景和次要层上。主体保持干净的轮廓、它的关键特征（地标的形状、产品 logo 那一面、动物的头和耳朵），以及与背景足够的对比。对主体做那一秒的辨认测试：在一个 96 px 缩略图上（`qc.py thumb … 96`），观众仍然说得出它是什么。当 look 和主体互相抢戏时，先降低主体那一层的效果，而不是改 look。

## 6. 库、版本与离线使用

| 库 | 授权 | 固定版本 | 作用 |
|---|---|---|---|
| [three.js](https://github.com/mrdoob/three.js) | MIT | 0.186.1 | 渲染器、场景、材质 |
| [pmndrs/postprocessing](https://github.com/pmndrs/postprocessing) | Zlib | 6.39.5 | 效果链路 |
| `starter/looks.mjs` FilmGrain 效果 | MIT（本工具包） | — | 带尺寸、保持和亮度响应的颗粒 |
| [Fontsource](https://fontsource.org) 字体文件 | 每个字族各自的授权（多数是 SIL OFL 1.1；查这个包的 `LICENSE`） | 包版本，例如 `@fontsource/inter@5.2.8` | 片子的字体，每台机器上都一样 |

- 工具包不安装任何东西。`scene.html` 用 import map 从 jsDelivr CDN 加载固定版本（第一次渲染需要网络）。
- **离线渲染：** 把两个模块文件一次性下载到 `engine/vendor/`，并把 import map 指过去：
  ```bash
  mkdir -p engine/vendor
  curl -L -o engine/vendor/three.module.js https://cdn.jsdelivr.net/npm/three@0.186.1/build/three.module.js
  curl -L -o engine/vendor/postprocessing.js https://cdn.jsdelivr.net/npm/postprocessing@6.39.5/build/index.js
  ```
  （starter 不需要 `three/addons/`。）下载前先问用户。
- `starter/index.html` 完全不需要任何库（单一片元着色器）：没有网络时，或者世界只需要着色器时，用它。
- 两个库的授权已经是项目 `LICENSES.csv` 里的行。
- **字体：** 系统字体在不同机器上不一样，而且常常缺土耳其语或其他拉丁变音符号。挑一个符合年代和基调的开放授权字族（最多两个：display 和 text），从固定版本的 CDN 上的 Fontsource 加载，`https://cdn.jsdelivr.net/npm/@fontsource/<pkg>@<version>/files/<pkg>-<subset>-<weight>-normal.woff2`，把 `latin` 和 `latin-ext` 两个子集都作为带各自 unicode range 的 `FontFace` 载入（Quick 模式：在 `film.html` 里设 `TYPE`；Film 模式：在 `engine/scene.html` 里设 `TYPE`，它已经从 `./type.mjs` 导入了 `loadType()`；完整方法见 `typography.md`），并在 `READY` 之前等它们加载完。每个字族在 `LICENSES.csv` 里写一行（名称、版本、授权取自包的 `LICENSE`）。每个文本框都用 `İĞÜŞÖÇ ÉÅ Ñ` 测试。离线：把 woff2 文件一次性下载到 `engine/vendor/fonts/` 并把 `base` 指过去（先问）。

## 7. 确定性

- 时间只来自帧序号（`t = frame / fps`）；`buildLook` 设置每个效果的时间，并以 delta 0 渲染。
- 所有随机都带种子（颗粒对 `(frame, seed)` 做哈希）；抓帧里没有 `Math.random`、`Date`、`performance.now` 或 `Clock`。
- 只建一次的带种子数据（星场、散布的道具）要惰性创建，在种子从 URL 读出之后：一个小的缓存，例如 `const lazy = (make) => { let v; return () => (v ??= make()); };`，在 `renderFrame` 内部首次使用时填充，绝不在 `SEED` 存在之前的模块顶层。
- 在 `READY` resolve 之前加载所有资源（贴图、字体：`await document.fonts.ready`）。
- `node tools/capture.mjs --url engine/scene.html --verify` 必须说 IDENTICAL。

## 8. 视觉陈词滥调（截至 2026 年 10 月）

**画廊平均值，实测（2026 年 10 月，703 部用 Opus 5.5 写代码做的公开影片中的 300 帧，以及 948 个标题）：** 56 % 的画面里，超过 3 % 的面积没有饱和色：一片米白或炭灰的底，配一个很小的强调色；有强调色时最常见的是橙红（16 %），其次是琥珀（9 %）；紫蓝底很少（3 %），奶油加陶土的「助手」look 约 2 %。被要求最多的片子是产品或应用发布，配浮动 UI 卡片、计数器、动感字体，其次是游戏、解释视频、对比和历史时间线。当一部片子坐在中性底上、配一个小强调色和浮动卡片时，它就读起来像那个平均值：至少要有一个观众看得见的颜色决定（在关键时刻，让配色的某种颜色占住画面的某一部分），除非 look 是有意选择单色并且卡上写明了。`make.py` 会为几乎不用颜色的片子打印一行 `colour:`。

`make.py` 在每次渲染时都会读片子自己的代码，并为粒子爆发、RGB 分离、镜头光晕、霓虹辉光、故障或弹性缓动打印一行 `cliché:`，如果导演卡没有连同理由点名它。

**先说被 AI 用滥的选择。** 工具包没有默认字体或默认配色，但生成的设计总漂向同样那几个选择：look 家族原封不动的配色；暖奶油配陶土、靛蓝或紫罗兰渐变、石板海军蓝、明亮的 Tailwind 蓝；Inter、Roboto、Montserrat、Poppins、Space Grotesk、Instrument、Playfair、DM、Geist。一部片子的配色来自它自己的参考（板上量出的颜色、地点、材质、年代），字体来自它的基调（`fonts.py search "<tone>"` 在 2000+ 款开放字体里搜，或用精选配对）。`make.py` 在每次静帧和草稿时都会打印一行 **taste**，只要这些还在；它要么在导演卡上给出理由，要么被改掉，绝不无视。

代码和 AI 做的视频共享着一批观众如今一眼就能认出的视觉习惯。把这张表当作风格帧和每次自评时的检查清单。任何一条在片子创意要求时仍然可能是对的：把理由写进 look 笔记里，有意识地用它。

| 破绽 | 为什么读起来很generic | 该怎么做 |
|---|---|---|
| 紫到青的渐变背景 | 无数工具和模板的开箱配色；它在说「没有做过颜色决定」 | 从片子的世界和参考出发搭配色：一个主导色相，一个有活儿的强调色 |
| 发光粒子汤 | 几千颗闪片塞满空间却没有意义；眼睛找不到可读的东西 | 少量有物理成因的粒子（光柱里的浮尘、花粉、雨、余烬），带种子，按深度定尺寸 |
| 每个镜头都完美对称 | 真实地点和物体是不均匀的；恒定的对称看起来像算出来的 | 对称只留给一个有意为之的镜头；其他地方让主体偏离中心，用带种子的偏移改变间距 |
| 到处都是镜头光晕 | 画面里没有光源的光晕读起来就是装饰 | 光晕只能来自可见的明亮光源，与镜头匹配，出现在英雄时刻 |
| 每次切点都上 RGB 分离和故障 | 一种没有理由就说「这是数字」的滤镜；它把字体和人脸抹花 | 故障只出现在故事断裂的地方（信号丢失、记忆失效），一次就好 |
| 一切都在发霓虹光 | 每条边都重辉光会压平纵深，读起来像模板缩略图 | 只有存在光源的地方才发光；其余保持清晰的边 |
| 弹性和回弹缓动 | UI 库的弹簧式过冲；它让严肃的主体感觉像个 app | 按主体选 `motion.md` §5 里的那些弹簧词；回弹只给顽皮的片子 |
| 一切都放慢 | 抽掉节奏又拉长时长；慢只有靠对比才意味着「重要」 | 默认实时；只放慢那一个挣得来的时刻；速度逐镜头变化 |
| 漂浮的散景圆球 | 后面没有灯的失焦圆圈不属于任何场景 | 散景来自主体后面真实的实体灯，按对焦距离定尺寸 |
| 黑底居中标题 | 一个静止、可替换的开场，违反钩子规则 | 用运动开场；把文字放进世界里（在某个表面上、在光里），或者在钩子之后再带出来 |
| 连续几个镜头长得一样 | 同样的尺寸、角度和速度给不出推进感 | 每次切点至少改尺寸、角度、方向、速度里的两项 |
| 重重的青橙分级 | 一个用旧的预设，把每一片天空和每一种肤色都推向同一个方向 | 从片子自己的配色出发分级；逐色相有意识地设定饱和度 |
| 程序化噪声星云和云 | 着色器教程的默认样子：软、无形、无尺度 | 给大气结构：光线方向、有深度的层次、尺度线索；研究真实照片的形状和颜色 |
| 浮动 UI 卡片和玻璃面板 | 一个科技模板老套，用界面替换了世界 | 当界面就是故事时，把它落在设备、桌面或手上；否则就展示那个东西本身 |
| 默认字体 | 系统或工具默认字体说明没有做排印选择 | 挑一个符合年代和基调的字体（OFL 或其他开放授权），有意识地设定字号、字距和行长 |
| 每个表面都发光 | 到处辉光会压平对比；什么都不显得特别 | 高 bloom threshold；只让真实光源发光 |
| 相机毫无理由地漂移 | 没完没了的慢速环绕读起来像屏保 | 为每个运动给出动因：揭示、跟随、在拍点上推近；让有些镜头静止 |
| 一尘不染的均匀表面 | 平的 CG 干净读起来像塑料 | 加上磨损、粗糙度变化、灰尘，以及从参考里取来的边缘细节 |

## 9. 风格库（58 个配方）与 Quick 模式的 looks

上面那八个家族是底座。在它们之上，`KIT/styles/` 里坐着一套**58 个详细风格配方**的库（51 个来自 LemoLab，七个是工具包自己的，在 `styles/index.json` 里标 `source: cinecode`）（木刻、浮世绘、皮影、孔版印刷、80 年代赛璐璐动画、默片、瑞士风动态图形、暗黑发布会、玻璃产品、数据叙事……），每个都映射到最接近的基础家族。一个配方装着那种风格下对每部片子都成立的东西：本质与它不是什么、材质与渲染、颜色逻辑、字体与字幕、运动质感、镜头语法、声音调色板、标志性动作、坑与变化空间。有第 10 节（Engine）的配方会描述源项目的演示引擎，而那不属于这个工具包：在 `film.html` 或场景页里搭出同样的语法。

**设计上就是便宜的。** 绝不整库通读：

1. **从创意的词或合适的家族里找候选：** `python3 tools/styles.py list --q "<2-4 words>"` 或 `--base <family>`（每个一行短描述）。用户提到某个工作室、频道或片名的风格（"Vox gibi"（像 Vox）、"Ghibli"（吉卜力）、"Wes Anderson"（韦斯·安德森）、"Apple reklamı"（苹果广告））同样这样查：已知名称表（`KIT/styles/aliases.json`，`styles.py names`）会把最接近的配方排在前面，没有合适配方时就报出家族。名字指的是它的语法，绝不是照抄那个工作室的作品。
2. **Quick 模式板上的库行**（当没有指定 look 时；手写的行排在前面，`quick-mode.md` §1，第 3 问；Film 模式展示三个手写方向，`film-mode.md` Step 1）：彼此有差异的候选（家族、年代、材质）、基础家族或配方。从 `python3 tools/styles.py suggest "<idea>" --board research/style-board.jpg` 开始（六个不同的候选，按创意排序），用 `styles.py board <1> … <6>` 换掉任何弱的那个；在提问之前先用眼睛看，就在那一张带编号的板上（配方的 demo 海报，几秒内下载并拼好；只为挑选，绝不进片子）。
3. **只读被选中的那个配方**（`styles.py show <id>` 会打印它的路径）。从中取：颜色逻辑进 `PALETTE`，材质和纹理进绘制函数，运动质感（逐帧还是二帧、揭示语法）进 `MOTION` 的用法，镜头语法进相机那几行，字体规则进排版方向，声音调色板进声音身份，坑进预览检查。它的参考只当语法：绝不照抄它们的作品，也不在片子里提到它们。
4. 把这次挑选连同用户理由记进 `STATE.md`（look 一行）。

**Quick 模式的 looks。** `templates/quick/film.html` 把八个家族作为 `LOOKS` 带着，由 look pass 施加到整帧：分级、高光辉光、纹理（`paper` 纤维、`scanlines`、`halftone` 网点）、暗角、颗粒（运动或静态纤维）以及绘制速率（`hold: 12` 让画面按二帧绘制，而相机、字幕和声音保持 24）。绘制函数用 `shade(hex, light)`（平滑光照，或按 `steps` 分的卡通色带）和 `inkLine(g, S)`（该 look 的描边宽度），这样换家族会一路传导到绘制里。`?look=<id>` 为预览替换家族，所以导演卡上的一次风格改动只花一次 `make.py stills`（几秒）。一个配方从它基础家族的那一条出发，在绘制函数里加上自己的语法。

**58 个风格配方（英文名为风格标识符，中文为释义）：**

1. `archive-lighttable` — Archive Light Table（档案灯箱台）
2. `art-deco` — Art Deco（装饰艺术）
3. `ascii-crt` — ASCII / CRT Terminal（ASCII／CRT 终端）
4. `backrooms` — Liminal Found Footage（阈限空间伪纪录片）
5. `blueprint` — Blueprint（工程蓝图）
6. `brick-toy` — Brick Toy（积木玩具）
7. `briefing-board` — Engineering Briefing Board（工程简报板）
8. `cel-anime-80s` — 80s Cel Anime（80 年代赛璐璐动画）
9. `crayon-book` — Crayon Picture Book（蜡笔画绘本）
10. `dark-keynote` — Dark Tech Keynote（暗黑科技发布会）
11. `data-physical` — Data Physicalization（数据实体化）
12. `dataviz` — Data Storytelling（数据叙事）
13. `engraving` — Copperplate Engraving（铜版雕刻）
14. `game-show` — Game Show Flat（游戏节目平涂）
15. `glass-product` — Glass Product Render（玻璃产品渲染）
16. `halftone-dossier` — Halftone Dossier（网点档案卷宗）
17. `hd-2d` — HD-2D（HD-2D 像素风）
18. `hologram-hud` — Sci-fi Hologram HUD（科幻全息 HUD）
19. `impasto` — Impasto Oil Painting（厚涂油画）
20. `ink-wash` — Chinese Ink Wash（中国水墨）
21. `iso-infographic` — Isometric Infographic（等距信息图）
22. `lidar-survey` — LiDAR Survey（激光雷达测绘）
23. `living-screencast` — Living Screencast（活体录屏解说）
24. `lowpoly-island` — Low-poly Isometric Island（低多边形等距小岛）
25. `microgame` — Microgame Frenzy（小游戏狂潮）
26. `microscope` — Microscope Story（显微镜下的故事）
27. `midcentury-toon` — Mid-century Cartoon（世纪中期卡通）
28. `one-line` — One-line Drawing（单线画）
29. `paper-lantern` — Paper-cut Lightbox（纸雕灯箱）
30. `paper-popup` — Paper Pop-up Book（立体纸书）
31. `papercut-red` — Red Paper-cut（红色剪纸）
32. `pictogram-motion` — Pictogram Motion（象形图动态）
33. `pixel-rpg` — 16-bit Pixel RPG（16 位像素 RPG）
34. `risograph` — Risograph Print（孔版印刷）
35. `rubber-hose` — 1930s Rubber Hose Cartoon（1930 年代橡皮管动画）
36. `satellite-briefing` — Satellite Briefing（卫星简报）
37. `scifi-toon` — Sci-Fi Sitcom Toon（科幻情景喜剧卡通）
38. `shadow-puppet` — Shadow Puppetry（皮影戏）
39. `silent-film` — 1920s Silent Film（1920 年代默片）
40. `silkscreen-poster` — Silkscreen Travel Poster（丝网印刷旅行海报）
41. `spy-titles` — 60s Spy Title Sequence（60 年代间谍片标题序列）
42. `stained-glass` — Stained Glass（彩色玻璃窗）
43. `swiss-motion` — Swiss Motion Graphics（瑞士风动态图形）
44. `teardown` — Exploded Teardown（爆炸拆解图）
45. `tilt-shift` — Tilt-Shift Miniature（移轴微缩）
46. `ukiyoe` — Ukiyo-e（浮世绘）
47. `urban-sketch` — Urban Sketch · Pen & Wash（城市速写 · 钢笔淡彩）
48. `vox-explainer` — Vox Explainer（Vox 风格解说）
49. `watercolor` — Watercolor Brush（水彩笔触）
50. `white-model` — Projection on White Model（白模投影）
51. `whiteboard` — Whiteboard Explainer（白板解说）
52. `woodcut` — Woodcut Print（木刻版画）
53. `line-boil` — Line Boil（线条沸腾）
54. `liquid-motion` — Liquid Motion（流体动态）
55. `shape-morph` — Shape Morph Chain（形变串联）
56. `sticker-explainer` — Sticker Explainer（贴纸解说）
57. `variety-captions` — Variety Captions（综艺花字）
58. `visual-proof` — Visual Proof（视觉实证）

配方来源：[lemo-opuscar](https://github.com/lemomo-ai/lemo-opuscar)，作者 LemoLab，MIT（`styles/LICENSE-lemo-opuscar`），在 `styles/index.json` 里固定版本。

## 10. 方向板（Film 模式）

在 Film 模式里，风格问题是用眼睛、从真实影片里回答的。创意一确定，在风格问题之前：

1. **三个方向**，各自是这个创意的一个不同视觉答案（不是同一个东西的三种深浅），为这个故事手写：以一个基础家族作底（最多一行可以是真的合适的库配方，`film-mode.md` Step 1），从 `deal.py deal look` 发出的六个参考构建，配上承载它的那些影片或艺术家。每个方向在用户的语言里有一个短标签（"<label>"）。
2. **静帧，只看情绪：** `python3 tools/asset_search.py search --q "<film title>" --sources filmgrab --n 6 --max-wait 12 --out research/dir-a1`（或 `--q "cat:<cinematographer>"`），留下能体现这个方向的 3–4 张。FilmGrab 是受版权保护的影视静帧：每个一个链接加一张缩略图，只用于挑选和测量（`assets-and-licences.md`）。
3. **一张图：** `python3 tools/direction.py board --row "A|<label>|<style id>|research/dir-a1,research/dir-a2" --row "B|…" --row "C|…" --out review/directions.jpg`。每一行：配方的海报（只在配方行有）、静帧、下方是量出的配色；打印结果给出每行一行的实测词（key、contrast、saturation、warmth，以及主光的估计）。展示这张图，然后用 A、B、C（★ 加理由）和 "own" 来问风格问题。
4. **测量被选中项**（在风格帧处，`film-mode.md` Step 4）：`python3 tools/direction.py measure --dirs <the row's folders> --out research/direction` 写出 `direction.json`，里面有一份给片子自己 look（写在 `engine/looks.mjs` 里）的 **look patch**：配色（从暗到亮，强调色在最后）、光（`azimuthDeg`、`elevationDeg` 作为估计、`ambient`、`exposure`）以及后期数值（contrast、saturation、vignette）。复制选中的预设，套上这个 patch，然后构建静帧。
5. **拿静帧和方向对照：** `python3 tools/direction.py compare review/style-v1.png --against research/direction/direction.json` 检查 key、contrast、saturation、warmth 和暗部占比是否落在该方向静帧的范围内。每一个 "off" 都要么修掉，要么在关卡处说明是有意为之。

**长片（`craft.md` §0）：** 板保持三行；选定之后的覆盖度才扩展。超过 60 s，每一章从选中的方向里再找 2–3 张与该章光线相符的静帧，在同一些影片或同一位摄影师的页面里搜，并逐章测量（`direction.py measure --dirs … --out research/direction-C3`）；超过 3 分钟则逐幕。片子保持一个 look；一章的 patch 只在其中调整光线、色温和 key，每一章的静帧和它自己的测量值对照。

这张板花掉几次请求和一张图；一个方向是片子自己世界的起点，绝不是片中那些影片的复制（不用它们的镜头、角色、布景或标题）。

## 11. 光线与镜头词汇表

给导演卡上的 **Atmosfer / Atmosphere**（氛围）和 **Işığın hikâyesi / Light arc**（光的故事）这两行用的光线名称，以及各自怎么搭出来。Three.js：`look.light`（`azimuthDeg` 0 = 逆光，±90 侧光，180 来自相机方向；`elevationDeg`；`intensity`；`ambient`）。Canvas 2D：渐变和对主体的 `shade()`，加法混合（`'lighter'`）的辉光，`plane()` 纵深。每个场景只用一个光的想法，并且要有观众能信服的来源（太阳、窗户、灯、屏幕）。词汇参考 showheyohtaki/motion-prompt（MIT）。

| 名称 | 它是什么 · 感觉如何 | 搭法 |
|---|---|---|
| **Backlight**（逆光） | 光源在主体背后：一圈发亮的轮廓、一圈朦胧的光晕 · 渴望、到来 | three.js 方位角 0–20°，加雾；2D：背后一团明亮的径向辉光，主体暗一档，边缘加法辉光 |
| **Rim light**（轮廓光） | 主体远侧一道细亮的边 · 分离、戏剧感、产品光泽 | three.js 在主光背后加第二盏灯（方位角相对主光 160–200°）；2D：在主体下方用光色画一份偏移的剪影副本，或用 `'lighter'` 模式的 `shadowBlur` |
| **Silhouette**（剪影） | 明亮天空前的暗形 · 神秘、标志性形状 | 不补光：ambient 接近 0，底是明亮渐变；这个形状必须单靠黑色就读得出来 |
| **Split**（半分光） | 侧光，半张脸亮 · 冲突、双重性 | three.js 方位角 ±90°；2D：横切主体的硬边渐变，裁切在主体内 |
| **Rembrandt**（伦勃朗光） | 主光高在 45°，暗面颧骨上有一小块三角光 · 经典肖像、重量感 | 方位角 ±45°，仰角 35–45°，补光低；2D：亮的一侧加上暗侧一个小三角高光 |
| **Butterfly**（蝴蝶光） | 高位正面主光，鼻下和下巴下有小阴影 · 魅力、美妆 | 方位角 180°，仰角 55–65°，柔和 |
| **High-key**（高调） | 明亮，阴影被提起来，对比很少 · 干净、希望、医疗、美妆 | ambient 高，底接近白，LOOK 分级提亮，对比降低 |
| **Low-key**（低调） | 大部分暗，一潭光 · 张力、黑色电影、奢华 | ambient 接近 0，一束窄主光；主体的关键特征留在这潭光里 |
| **Golden hour**（黄金时刻） | 低位暖阳，长影子，暖主光配冷影子 · 怀旧、温暖、结尾 | 仰角 5–15°，主光约 3,000 K（暖橙），影子偏蓝，影子形状拉长 |
| **Blue hour**（蓝调时刻） | 没有直射阳光，深蓝天空，暖色实用光源 · 平静、忧郁、一座城市的苏醒或入睡 | ambient 偏蓝，没有硬影子，小的暖光源（窗、灯）作为强调 |
| **Bokeh**（散景） | 失焦的光点变成柔和的圆盘 · 亲密、夜、纵深 | 按深度定尺寸、边缘柔和、加法混合；只放在主体后面 |
| **Tilt-shift**（移轴） | 一条清晰带的上方和下方都模糊：真实地点成了微缩模型 · 玩味、总览 | 黏土定格家族的微缩景深，或用 `plane()` 做一条纵深模糊带 |
| **Long exposure**（长曝光） | 移动的光变成线条 · 时间、流、城市脉搏 | 把每盏灯在 0.5–2 s 的窗口里的轨迹画成一笔，向过去淡去 |
| **Panning shot**（摇摄） | 主体清晰，背景拖成条纹 · 速度、聚焦一件事 | 相机跟着主体；背景沿运动方向画成几份偏移副本（方向性拖影） |
| **Lens flare**（镜头光晕） | 明亮光源产生的光环与鬼影 · 热度、太阳、科幻 | 沿光源穿过画心的连线画鬼影，低 alpha，`'lighter'`；只来一次，且来自画面内的光源 |
| **Fisheye**（鱼眼） | 超广角鼓包 · 滑板、监控、喜剧 | 一个桶形畸变着色器 pass（three.js）；Canvas 2D 里避免 |
| **Chromatic aberration**（色差） | 朝边缘的色彩镶边 · 镜头质感、故障、VHS | 复古家族的边缘色散；只留在边缘，绝不落在正文文字上 |
| **Vignette**（暗角） | 四角压暗 · 聚焦中心、老镜头 | LOOK `vignette` |
| **Black and white**（黑白） | 只剩明度 · 记忆、文献、永恒 | 灰度分级；用明度而不是颜色来分开主体和底 |
| **Duotone**（双色调） | 暗部一种墨色，亮部另一种 · 图形、品牌、海报 | 一张亮度到两种颜色的表（Quick starter 里的色调曲线滤镜，两个停靠点） |
| **Teal and orange**（青橙） | 冷影子，暖皮肤和亮部 · 大片、能量 | 分离色调分级：影子偏青，亮部偏橙；肤色保持自然 |
| **Film grain**（胶片颗粒） | 细密的运动纹理 · 胶片、暖感 | LOOK `grain`（每帧或每二帧重新播种） |
| **Light leak**（漏光） | 一团暖色半透明辉光从边缘漂进来 · 怀旧、转场 | 帧边缘加法混合的暖色光斑，持续 0.3–1 s，常常跨一次切点 |
| **Double exposure**（双重曝光） | 一张图里两张图：人像剪影里的风景 · 记忆、身份 | 剪影当遮罩，第二张图用 `'screen'` 放在里面，底要亮 |
