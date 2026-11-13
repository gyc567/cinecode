# 引擎：渲染、编码、检查

电影的每一帧都是**时间的纯函数**：`frame = f(t, seed)`。任何一帧都能单独渲染，乱序渲染，并行渲染，得到的像素完全一致。无头 Chrome 里的 GPU 负责画出每一帧；`capture.mjs` 保存它们；ffmpeg 负责编码。

## 1. 帧契约

一个引擎页面（`engine/scene.html`、`engine/index.html`，或片子自己的页面）暴露：

```js
window.READY          // Promise<{ frames, fps, width, height, renderer }>, resolves after every asset and font loads
window.renderFrame(i) // draws frame i (t = i / fps); no state carried between frames
window.captureFrame(i, type, quality) // renderFrame(i), then returns a PNG/JPEG data URL
```

URL 参数：`w`、`h`、`seed`、`look`，以及 `capture=1`（由 capture.mjs 添加：不跑预览循环）。

两个起步文件：
- `engine/scene.html` + `engine/looks.mjs`：three.js 场景、look 预设、文本/字幕叠加层（three.js 与 postprocessing 从固定版本的 CDN 加载）。
- `engine/index.html`：单一片元着色器，零依赖，可离线运行。

一个镜头一个页面，或者整部片子一个页面（用 `shot` URL 参数选择场景），两种都行。

## 2. 抓帧

```bash
node tools/capture.mjs --url engine/scene.html --probe                       # which GPU? (warns on software rendering)
node tools/capture.mjs --url engine/scene.html --verify                      # deterministic? must say IDENTICAL
node tools/capture.mjs --url engine/scene.html --out review/frames/S01 --start 0 --end 0 --params "w=960&h=540"   # one still
node tools/capture.mjs --url engine/scene.html --out frames/S01 --start 0 --end 71 --workers 2 --params "w=1080&h=1920&shot=S01"
```

- 更多选项：`--frames "0,48,96"`（只出这几帧，用于预览）、`--video out.mp4`（把帧管道进 ffmpeg，用 §3 的编码参数，不落帧文件）、`--audio out.wav`（把页面的 `window.renderAudio()` 音轨导出为 32 位浮点 WAV；Quick 模式，见 `quick-mode.md` §4）。
- 本地抓帧只有在「页面、参数、以及 `READY` 之前加载的本地文件内容」与已保存的抓帧完全一致时才会续跑。这包括导入的模块、字体，以及任何文件夹里的 review 静帧，哪怕文件日期被保留。输出和状态文件不会让未改动的任务失效。`--fresh` 明确重新生成所请求的帧；旧的没有时间戳的帧会自动重新生成。HTTP 页面每次都是全新渲染；本地页面在远程资源（未固定版本）变化之后要用 `--fresh`。
- `--workers` 是正整数；`--start`、`--end` 和每个 `--frames` 条目都是非负整数。非法区间会直接失败，且不会清空已有的帧目录。页面的 `READY` 元数据必须提供正整数的 `frames`、`width`、`height`，以及正的 `fps`。
- **长任务在后台跑，配一个状态文件**，代理结束本轮而不是干等：用 shell 工具的后台选项启动命令（`run_in_background`，bash 和 PowerShell 里同名）：
  ```
  node tools/capture.mjs --url engine/scene.html --out frames/S03 --start 192 --end 335 --workers 2 --status renders/S03.status.json
  ```
  把这个任务记到 `STATE.md`（Running jobs）。下一轮：读状态文件（`state`、`done`、`total`、`eta_s`）。
- 机器上一次只跑一个重渲染；里面开 2–3 个 worker。
- 草稿：半尺寸 + `--format jpeg --quality 0.9`。成片：全尺寸 PNG。

## 3. 编码

逐个镜头编码，然后不重新编码地拼接（每条命令一行，bash 和 PowerShell 里都一样）：

```
ffmpeg -framerate 24 -start_number 0 -i "frames/S01/%06d.png" -vf "scale=out_color_matrix=bt709:out_range=tv,format=yuv420p,setparams=color_primaries=bt709:color_trc=bt709:colorspace=bt709:range=tv" -c:v libx264 -preset slow -crf 17 -maxrate 10M -bufsize 20M -g 48 -movflags +faststart renders/S01-v1.mp4
ffmpeg -f concat -safe 0 -i renders/list-v1.txt -c copy renders/picture-v1.mp4
```

- `renders/list-v1.txt` 按顺序每行一个镜头（`file 'S01-v1.mp4'`），用 Write 工具写。
- `-start_number` 是目录里第一帧的帧号（从第 192 帧开始抓的镜头就从 192 起）。
- 有颗粒时加上 `-tune grain`；保留 `-maxrate 10M` 这个上限（网页上的 1080p24 大约需要 8–10 Mbit/s；颗粒在 CRF 17 下单独就能跑到 25 Mbit/s，而每个平台都会重新编码）。只有用户明确要求的归档母版才考虑取消这个上限。
- 改动过的镜头单独重编；其他镜头保持比特级一致。
- 然后用 `master_audio.py` 加声音（`sound.md` §6）。

**磁盘：** 一张全尺寸 PNG 是 2–6 MB；24 fps 下 30 秒 ≈ 720 帧 ≈ 1.5–4 GB。渲染前先查剩余空间（`df -h .`；Windows：`Get-PSDrive`），任务体积 + 3 GB 空闲。某个镜头编码并检查完（帧数、`qc.py dup`）之后，它的帧目录可以删掉以腾出磁盘。版本遵循同一套策略（`SKILL.md` Hard rules：versions without clutter）：终稿之前，用户见过的每个草稿都保留；用户确认终稿之后，清理只问一次（`SKILL.md` Hard rules）。

## 4. 检查（`tools/qc.py`）

```bash
python3 tools/qc.py hook  renders/film-v1.mp4           # HOOK RULE (hard): motion in the first 2 s
python3 tools/qc.py all   renders/film-v1.mp4           # info, hook, contact sheet, black, freeze, duplicates, brightness, cuts, loudness, spectrogram
python3 tools/qc.py sheet renders/film-v1.mp4 2 6       # contact sheet: one tile every 2 s
python3 tools/qc.py strip renders/film-v1.mp4 12.0 1.2 5   # motion strip of a key moment
python3 tools/qc.py phone renders/film-v1.mp4 0.5,1.5,10    # frames at 390 px wide (phone readability)
python3 tools/qc.py thumb review/style-v1.png 768        # small copy to look at
python3 tools/qc.py thumb review/style-v1.png 96         # squint test: values and composition at a glance
python3 tools/qc.py action renders/film-v1.mp4 0,3,7.5,12   # subject action per shot (shot boundaries in seconds)
python3 tools/qc.py sceneloud renders/film-v1.mp4 0,3,7.5,12  # loudness and 2-5 kHz harshness per shot
python3 tools/qc.py pops renders/film-v1.mp4            # one-frame flashes: a frame unlike both neighbours (also in all)
python3 tools/qc.py loop renders/film-v1.mp4            # looping films: the last frame must flow into the first
python3 tools/scene_check.py action renders/film-v1.mp4 --shots shots.json   # the same, boundaries from shots.json
```

**逐镜头检查**（`scene_check.py`）：`action` 每秒采样 4 帧，在中心加权的裁切区域上消除采样之间最整体的位移和缩放（pan、tilt、truck、dolly、zoom），并标出那些剩余变化极小、或不到总变化 30% 的镜头（镜头在动，主体几乎没动作）。`sound` 测量每个镜头的整体响度和它的 2–5 kHz 占比，标出比中位数镜头高出 3 LU 以上、或刺耳程度高出 3 dB 以上的镜头：一个整体 −14 LUFS 的片子，可能藏着一场响得多或亮得多的戏。每个标记都要么修掉，要么说明是有意为之；被标记的秒数就是该去看、去听的地方。

**沉默的一句话测试**（第一稿之后，在有助手可用时）：一个对这个项目一无所知的全新助手，只拿到这一稿的小尺寸 contact sheet 和一行字："用一句话说这部电影讲了什么"。把它的这句话放在 `STATE.md` 的 logline 旁边。如果它说错了主体或转折，动得太多，或者关键画面缺失：在其他任何意见之前先修这个。成本很低（一张 sheet，一句回复），却能抓出创作者自己已经看不见的东西。

**冷静自评：** 做完东西之后，像别人做的一样重新审视它：只对着 contact sheet、strip、手机尺寸画面和数字，对照已确认的风格帧和路线图。每个镜头按构图、光线与色彩、风格契合度、运动、读得懂这几项各打 0–2 分（共 10 分）；8 分意味着「可以给人看了」。运动依据 strip 上的九项检查判定（`motion.md` §7）；只有全部通过才给 2 分。始终报告**最弱的 3 个镜头**并附时间戳。

## 5. 终稿清单

- [ ] Hook 检查通过；strip 显示从第 1 帧起就有真实运动，由一个动作带动
- [ ] 每个镜头都有主体的动作（`qc.py action`：没有未解释的标记）
- [ ] 关键动作的 strip 通过运动检查（`motion.md` §7）：每个时刻只有一个主导，没有线性的活体运动，没有跳帧（pops），有重叠，落在节拍点上
- [ ] 每个镜头都符合已确认的风格帧（冷静自评 ≥ 8/10）；主体一眼能认出来
- [ ] 帧数 = 预期值；没有黑帧、冻结帧（除非有意）或重复帧
- [ ] 关键形状和文字在 390 px 宽下可读；文字在安全区内
- [ ] −14 LUFS ±1，真峰值 ≤ −1 dBTP，在最终 MP4 上测得；逐镜头与中位数相差在 3 LU 内且无刺耳度离群（`qc.py sceneloud`），或者例外情况已写明
- [ ] 交付消息说明声音是「测量」而非「听」出来的，并请用户在指定秒数处试听
- [ ] H.264 yuv420p、BT.709 标签、AAC 48 kHz、faststart（`qc.py info`）
- [ ] 每个外部文件在 `LICENSES.csv` 里都有一行；说明用的署名准备好了
- [ ] 一个被静音的观众也能用一句话说出发生了什么

## 6. 排障

| 症状 | 处理 |
|---|---|
| `no Chrome/Chromium found` | 设置 `CHROME_PATH=/path/to/chrome` |
| `--probe` 报 SwiftShader / llvmpipe | 无头模式下没有 GPU：Linux 上试试 `--flags "--use-angle=vulkan --enable-features=Vulkan"`；Windows 上在图形设置里把 Chrome 调成 "High performance"；或者换台机器渲染 |
| `page never defined window.READY` 带报错，或 `the page threw while loading` | 脚本或着色器错误，会带上文件与行号：修那一行 |
| `SyntaxError: Unary operator used immediately before exponentiation` | JavaScript 不接受 `-x ** 2`：写成 `-(x ** 2)` 或 `x * x` |
| 模块或贴图在 `file://` 下加载失败 | 把目录起个服务：`python3 -m http.server 8000`，然后 `--url http://localhost:8000/engine/scene.html` |
| `--verify` 报 DIFFERENT | 墙上时钟时间、`Math.random`、帧之间残留的状态、资源加载过晚 |
| `READY` 失败并报 "Cannot access 'SEED' before initialization"，或者每个 seed 出来的数据都一样 | 带种子的数据在顶层、`SEED` 设定之前就构建了：改成首次使用时惰性构建（`lazy()`，见 `looks.md` §7） |
| 文本框里变音符号或下伸部被切掉 | 按字体的 ascent 和 descent 再加内边距来定框尺寸（`label()`）；用 `İĞÜŞÖÇ ÉÅ Ñ` 测试 |
| 黑帧 | 绘制和读取必须在同一次调用里；`preserveDrawingBuffer: true` |
| 某个物体或整个 shader pass 渲染成黑色 | 着色器里出现了 NaN：GLSL 的 `pow(x, y)` 在 `x < 0` 时（以及 `sqrt`、`log` 作用于负数时）行为未定义：用 `pow(max(x, 0.0), y)`、`x * x` |
| GLSL 在某个名字上报语法错误 | 那是保留字（`flat`、`sample`、`input`、`output`、`filter`）：改名 |
| `height not divisible by 2` | 渲染偶数尺寸 |
| macOS 上长时间渲染会休眠 | 任务期间跑 `caffeinate -dimsu &` |
| Windows：找不到 `python3` | 用 `py -3`（python.org 的启动器）或 `python`；所有工具（含 `qc.py`）都能在 Windows 的 Python 和 Node 上跑，不需要 bash |
