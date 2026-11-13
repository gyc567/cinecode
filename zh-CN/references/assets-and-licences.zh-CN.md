# 素材与授权

这个工具包里的画面是用代码搭出来的，而一部长的事实类片子会在代码无法呈现真实之物的地方补上真素材（档案照片和影像、图库照片和视频）；声音通常是录制的配乐和现成的音效（Kenney CC0，通过 `sounds.py` 取得，`sound.md` §5），在更合适的地方，或当片子有意做成合成感时，用代码生成的声音。每个外部文件（录音、音效、贴图、HDRI 天空、档案图像）都来自开放授权来源，并在到达的那一刻登记在册。

## 1. 两类参考

| 类别 | 用途 | 允许的来源 |
|---|---|---|
| **情绪参考** | 研究它的质感、光线、颜色、构图；绝不进入画面 | 用户分享或链接的任何东西，包括受版权保护的影片和图像 |
| **素材** | 出现在片子里（贴图、底片、录音） | 仅限开放授权：CC0、公有领域、CC BY、CC BY-SA（需署名），以及用于图库照片和视频的 Pixabay 内容许可 |

用户自己的参考放进 `refs/`（图像）和 `refs/links.txt`（链接、带时间戳的影片名）。绝不索要密码，也绝不登录任何站点。

## 2. 搜索（无浏览器；只有 Pixabay 需要 key）

```bash
python3 tools/asset_search.py search --q "<a sound>" --type sound --out research/<sound>
python3 tools/asset_search.py search --q "<a subject>" --out research/<subject>           # wikimedia, aic, cleveland, nasa, pixabay
python3 tools/asset_search.py search --q "<what and where>" --type video --out research/<clip> # pixabay, wikimedia, archive
python3 tools/asset_search.py search --q "<a sky or a room>" --type hdri --out research/hdri # polyhaven
python3 tools/asset_search.py search --q "<subject>" --sources met,cleveland --out research/<subject>
python3 tools/asset_search.py fetch --results research/<sound>/results.json --pick 2 --dest assets/sound
```

| 来源 | 提供什么 | 授权 |
|---|---|---|
| Pixabay | 图库照片和视频（`--type image`、`--type video`） | Pixabay 内容许可：可免费商用，无须署名（我们仍然署名）；绝不转售或作为文件分享；使用用户的免费 key，只问一次：`keys.py open PIXABAY_API_KEY` |
| Wikimedia Commons | 图像、声音和视频 | 按文件：保留 CC0、PD、CC BY、CC BY-SA；其余丢弃 |
| Internet Archive | 音乐（`--type music`：其 Jamendo 与 netlabel 合集里已发行的曲目）、录音和影片（`--type sound`、`video`） | 仅限公有领域、CC0 和 CC BY |
| Art Institute of Chicago | 公有领域艺术品 | CC0 图像 |
| Cleveland Museum of Art（开放获取） | 艺术品和器物，大文件（最大 3400 px） | CC0 图像 |
| The Met（开放获取） | 公有领域艺术品和器物 | CC0 图像（`isPublicDomain`） |
| Rijksmuseum（`--sources rijks`） | 绘画、版画、素描和早期照片；其搜索按标题匹配 | 逐件为 Public Domain Mark 或 CC0 |
| Openverse（在默认的声音和音乐搜索里；经常宕机，然后它会在这部片子的搜索里缺席 30 分钟） | 从众多站点（ Flickr、博物馆、Freesound 等）收集的开放授权图像和声音 | 过滤为 CC0、Public Domain Mark、CC BY、CC BY-SA；保留每件的确切许可版本、许可 URL 和署名行；使用前在条目的来源页面上确认 |
| Poly Haven | HDRI、贴图 | CC0 |
| NASA Image and Video Library | 太空和地球图像 | NASA 媒体准则：署名 NASA，不暗示背书，检查第三方署名 |
| FilmGrab（`--sources filmgrab`） | 按片名查找的剧照，或 `cat:<cinematographer, director or genre>` | **仅限情绪参考**：受版权保护的影片；只为方向板及其测量提供链接和小缩略图（Film mode），绝不整段抓取，绝不进入画面；`fetch` 会拒绝它们；请求间隔 2 秒 |

看 `research/<name>/sheet.jpg`（一张小联系表），而不是逐张打开图像。只有当选择确实重要时，才给用户一份候选清单。

**搜索绝不静默阻塞。** 前台搜索会让各个来源并行作答，每个约 20 s（Openverse 约 30 s，因为它的音频要在 10–25 s 才回），所以耗时取决于最慢的来源，最多约 30 s，并打印它跳过的来源；没有应答的来源会在 30 分钟内退出这部片子的搜索（用 `--sources` 指定后，它仍会被问到）；在启动一个可能要花点时间的搜索之前先说明你在做什么（"正在 Internet Archive 上找配乐"）。要想把所有来源彻底搜一遍且不设时限，传 `--patient` 并放到后台跑（shell 工具的 `run_in_background`），之后再读它的输出——绝不在前台等 `--patient`。`fetch` 在某个来源连续三次连接失败后停止（它挂了：换一个，或晚点再试）；单个 `404` 只说明那一个文件没了，于是它转到下一个候选。

**片中的实拍素材：** 一段素材以编号帧的形式进入画面，这样每次渲染都精确一致：`ffmpeg -ss <start> -t <seconds> -i assets/video/<file> -vf "fps=<the film's fps>,scale=<width>:-2" -q:v 3 assets/footage/<shot>/%05d.jpg`；页面在时间 t 绘制该文件夹的第 `floor((t − shot start) × fps) + 1` 帧，先加载解码再绘制，绝不通过 `<video>` 元素——它无法定位到精确的帧。图库素材负责回答代码无法呈现的真实世界（真实地点、人群、建筑外观）。使用一段图库片段或照片之前：它的标签确实指向该主体；它不是 AI 生成、卡通或渲染；画面里没有可辨识的品牌、logo 或屏幕（裁掉或柔化）；而且它绝不冒充这部片子的真实主体（§5）。

## 3. 授权台账

`LICENSES.csv` 在下载时为每个文件写一行（`asset_search.py fetch` 负责写）：file、title、author、source URL、licence、licence URL、attribution text、modifications、date、used in、notes。**没有行，就不用。** 授权不明 = 不使用。文件进入片子时填上 `used_in` 和 `modifications`（例如 "slowed 0.5×, low-passed"）。

还要登记：
- 引擎所用的库（three.js MIT、postprocessing Zlib：已在模板里）；
- 用户选择加入的任何 AI 生成语音、音乐或图像（引擎、模型或音色、日期、条款）；
- 用户自己的录音和图像（"own work"）。

## 4. 署名

**画面上不署名：** 来源、照片署名和许可文字绝不放在画面里（不贴角标，不在图下加说明）；它们放进视频简介（事实则进 `research/facts.md`），也绝不作为片尾清单出现。只有用户要求，或者来源本身是故事的一部分时（一封文件抬头，镜头要读它）才出现在画面上。

CC BY 和 CC BY-SA 的文件需要署名，对视频来说简介是合适的位置（许可要求以对该媒介而言合理的方式署名）；`attribution_text` 列存放一行现成的文字。CC0 和公有领域不需要，但署名是好习惯。NASA 素材："Courtesy NASA"，加上来源页面上的任何第三方署名。

把署名写成视频简介里的**一行**（`Images: "A" by X, CC BY 4.0 · "B", The Met, CC0 · Sound: …`），更长的清单就在那里多写几行，或者链到一个页面。`make.py final` 会从 `LICENSES.csv` 把它们写进片子旁边的 `<title>-credits.txt`。

## 5. 诚实的使用

- 找到的图像展示它真实的样子。它绝不代替客户或品牌自己的产品、地点或人物，也绝不在展示别的东西时冒充这部片子的真实主体：用真品（客户提供的）、一个明确风格化或代码搭建的版本，或者一张标明它到底是什么的现成图像。
- 在授权之上还要遵守每个来源自己的条款：署名行按来源的原文写，不暗示背书（NASA、博物馆），遵守 API 速率限制和 user-agent 规则；当 Openverse 这样的聚合器列出条目时，以条目自己的来源页面为准。

## 6. 绝不

- 当片子会被分享时，使用标记为 NC（非商业）或 ND（禁止演绎），或"仅供个人使用"的素材。
- 把受版权保护影片的帧、音乐或声音放进交付物。
- 抓取不允许抓取的站点，或从需要登录的站点下载。
- 未经用户明确同意就使用付费服务或 API 调用。
