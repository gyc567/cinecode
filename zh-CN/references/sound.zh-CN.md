# 声音

"电影感"有一半在声音上。下面这套工艺从一部片子到另一部片子都不变；**调色板**来自这部片子的**声音身份**，在访谈里定下（Q5）。每个数字都只是起点：用耳朵和用测量去调。

工具：ffmpeg（生成、效果、混音）、`tools/mix.py`（各层 → 一个 WAV）、`tools/master_audio.py`（−14 LUFS）、`tools/qc.py loud` 和 `spec`（测量）。录音来自开放来源，并在 `assets-and-licences.md` 里有一行授权。

## 0. 总原则

做任何声音工作之前先读这些；后面各节是手段。

**声音由人声带领。**
1. 清晰第一，氛围围着它转：旁白干、近（压缩，在 3 kHz 附近轻轻提一点存在感）；房间、合唱和纹理在人声旁边，绝不压在人声上。一句话播放时，人声要比其他一切都高 8–10 dB：这是量出来的，不是猜的。
2. 按密度做闪避：一句短的台词给一个深的闪避；一大段以说话为主的内容给浅的（约 6 dB），或者只挖出语音频段（1–4 kHz），否则音乐会被埋好几分钟。必须在词底下继续响的层要绕过闪避。
3. 人声由耳朵、由用户来选：该语言的母语者（或一个有设计感的合成音），2–3 个候选人念同一句台词，各存成独立的短文件。演绎要跟卡上这条片子的声音行一致；耳语只用在校定的几行上。原始 TTS 绝不直接交付（§4b）。
4. 念法要有变化：每行一种情绪，结尾也要不同（一个问题、一句留白、一口气）；所有句子都用同一个降调收尾，最容易让人疲劳。让句子跨过切点（J-cut）；把结尾的时机做得有弹性，因为更慢的念法会把固定时长的标题卡往后推。

**声音承担结构。**
5. 先把旁白排上时间，再按台词时间放击打：在它之前、之后，或者落在它有意义的那个词上，绝不按固定秒数。
6. 音乐按段落作为一整块连续流动；它绝不突然停下：用短交叉淡化，淡出与画面的转场绑在一起。
7. 母题每次回来都要变化（稀疏、下行、上行）；一大段以说话为主的内容配一个渐增的固定音型；一个画内的标志性声音可以成为配乐的脊梁，并回答某句台词。
8. 沉默是一个事件，不是空隙：得有东西结束它（渐快的击打、最后一下、一点尾巴），然后是一层极轻的房间声；混响的尾巴也跟着它安静下来；绝不用数字零。硬邦邦的静音读起来像故障。
9. 每一章带一个声音签名（一段上升、一下击打、一片闪烁）；钩子的击打和它的画面落在同一帧上。
10. 击打只比该段落最响的底床高 6–8 dB，不是 15。

**真实。**
11. **先录音：** 每一个物理事件（水、鸟、石子、脚步、风、引擎、布料、玻璃）从第一稿起就要有录音；代码只做那些跟随动作或配乐的东西（risers、whooshes、调性击打）。`make.py` 会扣住那些事件声大多靠代码做出来的草稿。先真实后电影感：一件乐器、一种仪式、一个地方，都要去研究它真正的声音；电影式夸张（低频轰击、长尾巴）是用户自己做的选择。把真实的开放录音（纹理）、生成的事件和合成（黏合剂）分层叠加；只用合成会听起来很单薄。每段录音的来源（文化、仪式、地方）都要写进 `LICENSES.csv`；来自错误文化的录音要拿掉。
12. 生成器要拿到物理描述（材质、怎么演奏、什么东西会响、录音方式：近、干、野外、无超低频），它们的文件在评判之前先测量（它们常常送来时很轻）。
13. 氛围要跟着场景的类型走（室内还是室外，什么地方），逐场景打标签，绝不按固定时间；图声锁定的拟音（画面里每一次小接触和小移动）很便宜，却能让画面活起来。

**测量，不是听。**
14. 每一条声音笔记都要变成一个秒数、一层和一个可测的目标（"kısık"（低沉）→ 在 0:12–0:18 相对底床 +4 dB）。
15. 按段落和 4 秒窗口画响度图，配乐单独一份和混音一份（`qc.py sceneloud`）；检查最后 15 s；每次改声音前后都看一张频谱图（`qc.py spec`）。
16. 只改声音绝不要重渲染画面（`make.py draft --sound-only` 或 `make.py final --sound-only`；视频和字幕直接拷贝）。母带目标要压在交付上限之下，因为 AAC 会抬高真峰值（`master_audio.py` 用 `--video` 做这件事）。
17. 生成的语音要检查：用户有本地语音转文字工具时用它（安装需要用户同意），在原始录音上量停顿，不要在加了混响的版本上量。
18. 每个镜头都要在声音上回答"我们在哪儿"：每个地方有自己的底床。沉默在节拍表里是标出来的；其他所有地方都是满的，绝不是只有旁白飘在空处。
19. 声音尊重它的主体：不要在平民、受害者或悲伤上堆奇观（爆炸、riser、whoosh）；那些画面由房间声或沉默来承担。
20. 关键时刻要有自己的声音时刻（`writing.md` §15）：底床撤掉，一下低频击打或一个持续的长音，然后留一条尾巴。

## 1. 声音身份（轻）

**Film 模式：** 开工之前，提出**三条针对这个创意的一句话声音概念**；用户挑一条或改一条。**Quick 模式：** 导演卡上写一行身份，从创意出发，可以用平实的话改（不单独提问）。每条概念在一行里点名：核心想法、调色板、一个标志性声音。这三条至少要在以下两项上有差别：来源家族（录音 / 合成 / 乐器 / 人声）、要不要音乐、我们借谁的耳朵在听、沉默怎么用。每一条都要从故事自己的材料（它的物件、它的地点、它的物理）写出来，这样它不可能被贴到另一部片子上。

选定之后，在 `ROADMAP.md` §4 里把它展开：调色板、标志性声音以及它何时回来、沉默在哪里、人声、音乐。

## 2. 原则

1. **每一个声音都来自画面，**在画内或紧贴画外。不要没有来源的通用提示音（心跳、riser、stinger）。
2. **钩子从第 1 帧就有声音。** 不要无声开场；第一个声音和第一个动作一起诞生。
3. **最大的时刻之前先静。** 在峰值前 0.3–1.5 s 撤掉房间声（9:16 里更短），然后让峰值成为最响的时刻。
4. **大东西要听着大：** 质量和长尾巴撑起尺度；安静时刻给一个精确的细节。
5. **转场要和画面一起呼吸：** 用画面自己的动作搭出来（相机下沉时变亮的风），而不是现成的 whoosh。
6. **除非身份里有音乐，否则让设计层保持无音高**；有音高的持续音很快会让场景变得"音乐化"。
7. **每一层生成物单独播种**，这样改一层不会动到其他层。
8. **从第 1 版就开始做声音设计，**两种模式都一样：用户看到的每一版都有逐场景的氛围底床、锁在画面事件上的效果，以及一个把它们全放在同一个世界里的共享混响空间。只有音乐的版本是草稿，不是版本。
9. **主旋律要悦耳。** 刺耳的合成风声和铜管、还有明亮的原始锯齿或方波主音，几秒内就会让耳朵疲劳。旋律主音优先用拨奏或打击的音色，或者把主音软化：2–3 kHz 低通，柔和的起音（≥ 20 ms），少一点共振。按场景检查 2–5 kHz 占比（`qc.py sceneloud`）。
10. **对齐声音的峰值，不是它的起点。** 撞击的瞬态要落在画面的接触帧上；起音柔和的采样要按它的起音时间提前开始，这样峰值才落在 cue 上。whoosh 在它承载的运动到达前 2–3 帧到峰值，riser 在 cue 上收尾，click 落在按下的那一帧。有音乐时，挑一个拍长是整数帧数的速度（`fps × 60 / BPM`）：24 fps 下是 80、90、96、120 或 144 BPM（18、16、15、12、10 帧）；30 fps 下是 90、100、120 或 112.5 BPM（20、18、15、16 帧）。这样每个落在拍格上的 cue 都正好落在帧上。

## 3. 分层

| # | 层 | 干什么 | 电平（相对人声） |
|---|---|---|---|
| 1 | 空间底床（房间声） | 我们在哪儿？跨切点延续；空间变了它就变 | −20 到 −30 dB |
| 2 | 低频体量 + 泛音 | 重量和尺度；在手机上靠 70–250 Hz 泛音也能听见 | 大时刻之前撤掉 |
| 3 | 纹理 | 让空间是活的：稀疏的带种子事件（水滴、噼啪、落叶） | −30 到 −40 dB |
| 4 | 拟音 | 锁在画面接触帧上的接触声 | 按距离定 |
| 5 | 英雄事件 | 故事里的大时刻 | 最响（瞬时 ≈ −10 LUFS） |
| 6 | 人声（如果有） | 故事本身；一切都为它让路 | 0 dB |
| 7 | 标志性声音／母题 | 身份反复出现的那个声音 | 按身份来定 |
| 8 | 音乐（如果有） | 独立总线 | −4 dB 总线 |

`mix.py` 会加上总线偏置：对白 0，拟音 −1，音效 −2，氛围 −3，音乐 −4 dB。

**编曲表（音乐）。** 在渲染任何音乐之前，先写一张「段落 × 乐器或音轨」的表（谁在哪里演奏），并对着时间线读一遍：

| Section | Time | Pad | Bass | Lead | Drums | Signature |
|---|---|---|---|---|---|---|
| intro | 0:00–0:08 | ● | | | | ● |
| build | 0:08–0:20 | ● | ● | | ● | |
| peak | 0:20–0:26 | ● | ● | ● | ● | ● |

检查有没有重叠和双份层次：一个持续声部跨相邻段落只跑一次，而不是每行都重启；一个循环声部在下一个段落开始处结束（半开），这样边界那一下只响一次；两个声部绝不在同一段落里共用角色和音区。Quick 模式下这张表是 `film.html` 里的 `ARRANGEMENT`（每次渲染前都会跑 `checkArrangement()`）；Film 模式下它在 `ROADMAP.md` §4。

## 4. ffmpeg 配方（已测试）

```
# Space bed: seeded pink noise, band-limited, a short room echo (change seed per space)
ffmpeg -f lavfi -i "anoisesrc=color=pink:seed=5:amplitude=0.3:duration=30" -af "highpass=f=200,lowpass=f=3000,aecho=0.8:0.7:40|90:0.3|0.2" audio/bed-room.wav

# Low body: unpitched noise 22-90 Hz plus its own 70-250 Hz harmonics (what phone speakers can play)
ffmpeg -f lavfi -i "anoisesrc=color=brown:seed=21:amplitude=0.6:duration=30" -af "highpass=f=22,lowpass=f=90,asplit[a][b];[b]volume=6,asoftclip=type=tanh,highpass=f=70,lowpass=f=250,volume=-8dB[h];[a][h]amix=inputs=2:normalize=0" audio/lowbody.wav

# A falling mass hit (75 -> 36 Hz sweep with a fast decay)
ffmpeg -f lavfi -i "aevalsrc='0.4*sin(2*PI*(75-39*t)*t)*exp(-6*t)':d=1:s=48000" audio/hit.wav

# Scale a real recording: slow it down (0.5x = one octave lower, bigger)
ffmpeg -i assets/sound/creak.ogg -af "aresample=48000,asetrate=48000*0.5,aresample=48000" audio/creak-big.wav

# Narration chain: high-pass, presence, gentle compression (voice_treat.py's clean stage does this and more)
ffmpeg -i audio/vo/VO1.wav -af "highpass=f=90,equalizer=f=300:t=q:w=1:g=-2,equalizer=f=3200:t=q:w=1:g=2,acompressor=threshold=-22dB:ratio=2.5:attack=8:release=150:makeup=3dB" audio/vo/VO1-eq.wav

# Duck the bed under the voice (sidechain), keeping the voice in the result
ffmpeg -i audio/bed-room.wav -i audio/vo/VO1-eq.wav -filter_complex "[1]apad[v];[v]asplit[vk][vm];[0][vk]sidechaincompress=threshold=0.03:ratio=8:attack=30:release=600[d];[d][vm]amix=inputs=2:normalize=0:duration=first" audio/bed-ducked.wav
```

## 4b. 片子自己的人声（旁白绝不用原始 TTS）

引擎直接吐出来的合成声音听起来很生硬，不属于任何一部片子。每一部有旁白的片子都要设计自己的人声，就像设计自己的标志性声音一样，没有哪种处理是默认的。

0. **用耳朵选人声：** 让 2–3 个候选声音（该语言的母语者）念剧本里的一行，各存成自己的短文件：`python3 tools/tts.py --audition "gemini:Charon,gemini:Kore,elevenlabs:<id>" --text "<one line>" --style "<delivery>" --out review/voices`。一个一个编号发给用户听并问选哪个；绝不发合并在一起的合集。处理方式（Film 模式总是要做；Quick 模式在用户要求时做，否则用卡上用户确认过的处理行）同样这样选：`python3 tools/voice_treat.py audition review/voices/<picked>.wav --pairs "close/room,radio/dry,tape/hall"`，每种处理一个文件。
1. **问故事**：谁在说话，从哪儿说，对谁说：一段记忆、一次广播、一个内心声音、一个神话、一台机器、一群人。这个答案决定了**媒介**（声音从什么里出来）和**空间**（它站在哪儿）：`python3 tools/voice_treat.py list`。
2. **每一行都要处理：** `python3 tools/voice_treat.py apply audio/vo/*.wav --medium <medium> --space <space> --out audio/vo-t`（clean 阶段总是做：高通、去齿音、存在感、温和压缩；然后是媒介，然后是空间；每一行落在同样的人声电平上）。混音用 `audio/vo-t/`。
3. **用一个只可能属于这里的想法，把它变成这部片子自己的声音**，例如：
   - **印痕（imprint）：** 让人声和片子标志性声音的一段做卷积（`--imprint audio/sig.wav --imprint-mix 0.2`）：人声带着这部片子自己的声音痕迹；
   - **一个声音事件：** 处理方式只在转折处变一次（真相到来时收音机的电流声清掉；合唱稀薄成一个声音；房间消失，人声变得干而近）；
   - **一层用人声自己做的层，** 只在故事本身就是关于它的时候：给一个秘密或一个神话用倒放的耳语（下面的配方），给一个分裂的心智用延迟的第二个人声，把最后一个词留在空间里回响。
4. **表演要匹配处理：** TTS 的风格提示要带上语速、距离和呼吸（"speak close and slow, almost to yourself"（说得近而慢，几乎像对自己说）；"announce it, clipped, as if through a speaker"（把它喊出来，短促，像透过一个喇叭））；后期只负责把表演开始的东西收尾。
5. **写下来**，在导演卡或路线图上写一行："画外音／人声（Voice / Dış ses）：<medium> in <space> (+ <imprint>)，因为 <a reason from this story>"，并在试听笔记里点名这个人声。

**用人声做的层**（只在故事要求时）：

```bash
# Reversed whispers: the narrator's own line, reversed, band-limited, distant (in gaps between lines only)
ffmpeg -i audio/vo/VO1.wav -af "areverse,highpass=f=300,lowpass=f=3000,afade=t=in:d=0.4,areverse,afade=t=in:d=0.4,aecho=0.8:0.9:120|260:0.4|0.3,volume=-24dB" audio/whisper1.wav

# Uncanny voice chain: slightly lower pitch at the same speed, chorus, trimmed band
ffmpeg -i audio/vo/VO1.wav -af "aresample=48000,asetrate=48000*0.84,aresample=48000,atempo=1/0.84,chorus=0.6:0.9:50|60:0.4|0.32:0.25|0.4:2|1.3,highpass=f=120,lowpass=f=6000" audio/vo/VO1-uncanny.wav
```

## 5. 转场

**画面做什么，声音就做什么**（这跟下面的强调预算不是一回事）。屏幕上每一个物理事件都要有自己同步的声音：杯子放下、钥匙转动、门、翻页、一步、车后备箱、玻璃杯放下。一部画面里有动作、底下却一片静音的片子读起来像没做完。下面的预算限制的是*设计过的*声音（whoosh、riser、切点上的击打），绝不限制这些。
- **撞击是分层的**，每一层一个文件，峰值对齐在接触帧：瞬态（那声脆响，0–20 ms）· 体（物体自己的声音，来自真实录音）· 房间（它发生的空间：共享的 ROOM 或一段录下来的反射）· 尾巴（衰减，0.5–3 s；户外更长）· sub（40–80 Hz 的闷响，给重量，只在最重的那些上用）。一扇铁门被摔上是五层齐全；杯子放在桌上是瞬态、体和一点点房间；门是体和房间。
- **画面事件审计，在草稿之前：** 过一遍分镜表的 action 列和静帧，把画面上的每一个事件列进 `review/sound-audit.md`（时间 · 事件 · 它的声音 · 层次）；除非那里的沉默是一个写明的选择，否则没有一个是静音的。草稿报告要写出覆盖率（"events 42 · sounded 41 · silent by choice 1"）。

**每一个声音从哪来。** 该是真实的、或者该是设计出来的声音，要现成地来，绝不现场合成：
- **设计过的短音**（一个砸下的词、click、pop、toggle、快门、敲门、玻璃、金属、盘子、
  脚步、翻页、门、布料、纸牌、硬币、骰子）：`python3 tools/sounds.py search "<material and action>"`，然后
  `get <id>`（Kenney 的 CC0 素材包，每部片子都缓存；同一事件重复出现时换一个 take）。
- **录制的长音**（雨、房间、街道、人群、运转的机器、纸张被翻、水）：`asset_search.py
  search --q "<what and where>" --type sound`，一次性取进共享缓存。
- **代码**只用在现成的声音跟不上的地方：随着动作实时成形的声音（一行字被画出来时上升的音调，跟着相机的扫频），正好落在切点上的 whoosh 或 riser，配乐调性里的 ping 或铃声，没有现实对应物的抽象声音（数据、全息图），有意做成合成的片子（8-bit、复古游戏），以及录音撞击底下的 sub。分层最强：录制的体 + 代码的 sub + 卡准帧的代码 riser。
- 两个库都记得每部片子用过什么；搜索会在其余结果之后列出你上几部片子用过的，标出来：按事件选，别按习惯选。

**强调，而不是每次切点一个声音**（在专业发布片上实测：它们设计过的声音落在承载意义的时刻上，切点上有声音的频率并不高于偶然；音乐负责承载运动）。`make.py` 在违反这些时会把草稿扣成 SOUND SLOP：
- **Whooshes：** 每 30 s 最多 2 个。whoosh 是**低频涌起，不是空气**：150 Hz 以下的体加上它的泛音，一层很轻的空气层低 20–30 dB，0.5–1 s，在切点上涌到峰值，切后 0.25 s 消失（模板里的 `whoosh`）。滤波噪声的空气扫频和振荡器的 zaps 听起来很廉价；`air` 只留给真正有气流从镜头旁掠过的时刻。
- **大时刻：** riser 接一个急停（全场空 0.1–0.5 s），接一个落在揭示帧上的撞击，一部片一两次，每 30 s 最多一个 riser。
- **切点：** 最多 70 % 的切点上有一个设计过的声音；大多数切点靠音乐，而一部音乐稀疏的片子用画面自己的声音给切点配乐（一扇门、一次翻页、房间的变化）。
- **优先级：** 揭示、logo 或名字、关键动作和段落变化得到设计过的声音；小动作靠音乐带过。
- **小声音压低：** tick、轻敲和 UI click 处在音乐峰值下 10–15 dB，短而柔，声像偏向它发生的地方；绝不让一个明亮的 click 比节拍还响。
- **留出空间：** 声音更多不等于更好；名字之前的一次急停或一口气本身就是设计的一部分。


- **底床跨切点**（0.8 s 交叉淡化）；干的声音跟着画面切（10–20 ms）。
- **新空间要早进**（J-cut，切点前 0.3–1.5 s）；尾巴在切点后流出去（L-cut）。
- **底床在切点上跳动 ≤ 4 LU**，除非是特意安排的（硬切到静音或到一个响的空间可以是重点）。
- **长声部要避开切点起步**（`mix.py` 会标出离切点 0.3 s 以内的层）。
- 如果旁白在切点后 0.6 s 内开始，闪避要在切点之前就开始。
- **不要在每次转场都上噪声 whoosh。** whoosh 响两次很抢，响到第五次就让人疲惫；整部片子里，只为真正的一阵气流使用它。给切点配乐才是正道：一层反向的 pad 涌起、在章节切点上戛然而止（用音乐的调内和弦），下一个镜头从一个点长出来时一片柔和的钟鸣，推近或变焦穿过时一道安静的滑音，一次有节奏的揭示配一串快速的拨弦，一次扫过时配一段短促上升的踩镲，以及**大约一半的切点上什么也不加**：重拍和下一个镜头自己的入场声足够撑起它们。
- **同一个转场声绝不连用两次，**一部片子里大约五个结构时刻用完整撞击（带噪声尾巴的 hit），其余用更柔的击打。
- **按段落给能量：** 一个严肃或安静的章节可以掉到半速，并且比主章节低约 5 dB（用 `qc.py sceneloud` 检查）。
- **把小声音绑在画面的单位上：** 每个字母或每块砖一个 tick，一个标签落下时一声 blip，一个倒数的数字底下有滚动声，一次拒绝配一声低嗡。

## 6. 混音与母带

1. 把每一层在 `audio/` 里做成自己的文件（带种子，有版本）。
2. 写 `audio/mix.json` 并运行 `python3 tools/mix.py audio/mix.json`（峰值限制在 −2 dBFS）。母带前把混音瞄准 −16 到 −14 LUFS。
3. 母带并混流：`python3 tools/master_audio.py audio/mix-v1.wav renders/film-v1.mp4 --video renders/picture-v1.mp4 --limit`（两遍 loudnorm，**整体 −14 LUFS，真峰值 ≤ −1 dBTP**，AAC 320k 48 kHz）。脚本会报告是否跑了线性模式；如果它报告是动态模式，就压低最高的峰值再跑一次。
4. 测量最终文件：`python3 tools/qc.py loud renders/film-v1.mp4` 和 `python3 tools/qc.py spec renders/film-v1.mp4`（频谱图里持续的水平线 = 一个非预期的有音高层）。
5. **逐场景测量：** `python3 tools/qc.py sceneloud renders/film-v1.mp4 <boundaries>`。整体 −14 LUFS 可能藏着一场响得多或亮得多的戏；标出比中位场景高 3 LU 以上的场景，以及任何刺耳度离群（2–5 kHz 占比比中位数高 3 dB 以上），然后修掉它或说明它是有意为之。
6. **请用户去听。** 你能测量音频，但听不见。每一次交付都用一行说明这一点，并请用户戴耳机听一次、用手机扬声器听一次，点名要紧的秒数：每一次转场、最响的那场戏、旁白的台词、任何被标记的场景。

| 目标 | 数值 |
|---|---|
| 网页／社交 | −14 LUFS ±1，真峰值 ≤ −1 dBTP |
| 逐场景 | 与中位场景相差 3 LU 以内，2–5 kHz 占比与中位数相差 3 dB 以内（或者把例外点名） |
| 旁白压过底床 | 明显在上面（说话时在语音频段比底床高约 15 dB） |
| 沉默 | 每一处沉默都是写下来的 |
| 身份检查 | 这个混音原封不动能配任何另一部片子吗？如果能，它就是个模板：回到 §1 |

## 7. 变体

- **竖屏短片（9:16）：** 声音和画面同时开始；峰值前的静默缩到 0.3–0.5 s；片子静音时依然成立（画面和屏幕文字撑起它）。
- **音乐主导：** 歌就是时钟；画面跟着它剪；设计层塞进它的空隙里。
- **产品／品牌：** click 和 UI 声锁在屏幕事件上；一个英雄时刻，在它之前有一次撤空。

## 8. 音乐：类型、和声、律动

配乐的选择和 look 一样：从故事、用途和速度里来，在导演卡的 **Music** 一行上带数字点名（类型、BPM、调、进行、律动、主音、弧线）。那种带持续音、琶音和钟琴的慢底床是十五种类型之一，绝不是默认；`make.py` 会点名你上几部片子用过的乐器和和弦循环（Quick 模式的代码配乐），以及一段对卡上那个速度来说太薄的音乐。

**Quick 模板里的工具：** `hz('A3')`、`chord('Am7', 3)`、`chordAt(prog, t)`、`grid(part, 'X...x...X...x...')`
（每小节 16 步，X 是重音，`{ swing }`），合成的 `kick`、`snare`、`hat`、`clap`、`sub808`、`bass`、`stab`、`pad`、
`pluck`、`lead`，以及录制的乐器（`samples.py list`；`INSTRUMENTS` 和 `sample()`）。

### 类型卡（一个起点；片子会让它变形）

| Genre | BPM | Groove (16 steps) | Bass | Harmony | Lead and colour | Fits |
|---|---|---|---|---|---|---|
| 电影紧张、环境 | 60–90 | 无，或一个柔脉冲 `x.......x.......` | 一个持续的低音 | 一个和弦或 i–VI | pad、低弦、钢琴、涌起 | 神秘、夜、调查 |
| 管弦史诗、体育赞歌 | 84–100 | 嗵鼓和底鼓 `X...x.x.X...x...`，军鼓滚奏推进峰值 | 大提琴和低音提琴落在根音，八分音符 | i–VI–III–VII、I–V–vi–IV | 铜管和小号扛主旋律，弦乐固定音型，定音鼓 | 致敬、传承、体育、胜利 |
| 极简钢琴、新古典 | 60–80 | 无 | 钢琴左手 | I–V–vi–IV 开放排列，i–VI–iv–v | 钢琴琶音、一条大提琴线 | 记忆、渴望、安静的致敬 |
| 巴洛克室内、异想天开 | 100–132 | 无，或木鱼在拍点上 | 八分音符上的拨奏行走 | I–IV–V–I 与模进 | 羽管键琴、拨弦弦乐、竖笛、钟琴重音 | 手艺、年代、韦斯·安德森、仪式 |
| 爵士、摇摆 | 120–180 | 每小节 12 步（八分三连音）：叮叮 `x..x.xx..x.x`，踩镲脚 `...x.....x..`，软军鼓在反拍上即兴 | `bass-pizz` 上的行走四分音 | 带七音的 ii–V–I（Dm7 G7 Cmaj7） | 萨克斯、钢琴、颤音琴 | 喜剧、复古、咖啡馆、夜城 |
| Lo-fi hip-hop、boom bap | 70–92 | 底鼓 `X......x..X.....`，军鼓在第 4 和第 12 步，踩镲在八分上，swing 0.2 | 根音上一个柔和的 sub | m7 和 m9（Dm9 G7 Cmaj7 Am7） | FM 钢琴、颤音琴、一层有尘的 pad | 学习、日常生活、暖感 |
| House、Disco | 118–126 | 底鼓 `X...X...X...X...`，拍手在 4 和 12，开镲 `..x...x...x...x.` | 八度或一条切分线 | m7 刺音（Am7 Dm7） | 和弦刺音、钢琴和弦、弦乐 | 发布、时尚、上扬的广告 |
| 合成波、80 年代 | 84–110 | 大军鼓在 4 和 12，踩镲在八分上 | 十六分音符上的锯齿贝斯，根音 | i–VI–III–VII | 锯齿主音、pad、门限鼓 | 复古科技、汽车、夜驾 |
| UK garage、Drum and bass | 130–134、170–174 | 2-step `X.....x...X.....` 配 shuffled 踩镲，或 DnB break `X.........X.....` | 带滑音的 `sub808` | m7 和 m9 | 刺音、拨奏、一声铃 | 手机、应用、体育集锦 |
| 极简 techno | 124–130 | 四踩地板，低音十六分踩镲，稀疏拍手 | 单音脉冲 | 一个和弦，调式 | click、blip、一记滤波刺音 | 精密、手表、工程 |
| Afrobeat、Latin | 100–115（reggaeton 90–100 `X..x..x.X..x..x.`） | 康加鼓、邦戈、沙锤、牛铃 | 反拍上的切分 | I–IV–V、i–iv | 马林巴、卡林巴、铜管刺音 | 食物、旅行、喜悦、夏天 |
| 民谣、原声 | 80–110 | 跺脚底鼓、拍手、沙锤 | 落在根音上 | I–V–vi–IV、I–IV | 竖琴或 strumstick 扫弦、口琴、竖笛 | 手艺、自然、家、手作人 |
| 安纳托利亚、土耳其 | 2/4、4/4（`X..x..x.X.x.x...`）、9/8 aksak 2+2+2+3 | darbuka、手鼓 | 落在根音上，稀疏 | maqam 色彩：Hicaz（A Bb C# D E F G）、Kürdi、Uşşak；曲调下垫持续音 | 拨奏扬琴或 dan tranh，像 ney 的竖笛，弦乐 | 伊斯坦布尔、历史、土耳其故事 |
| 顽皮、儿童、古怪 | 110–140 | 木鱼、拍手、滑稽音效 | 弹跳的拨奏或大号 | I–IV–V | 钟琴、木琴、拨奏、一声哨 | 儿童解释视频、玩具、喜剧 |
| Trap、现代 hip-hop | 130–150 半速 | 十六分踩镲带滚奏，拍手在第 8 步 | 带滑音的 `sub808` | i–VI、Phrygian | 铃、拨奏、一层暗 pad | 街头服饰、游戏、张扬的发布 |

Swing：`{ swing }` 让反拍上的十六分延后（lo-fi、house、garage）。爵士、shuffle 和布鲁斯的 swing 在八分上：那些模式写成 12 步（三连音），swing 音符落在每拍的第三步。奇数拍号：`bar()` 保持四拍，所以 9/8 要把 BPM 设成八分音符每分钟 × 4/9，并写 9 步的模式。一部片子可以有意混搭两张卡（house 底鼓上铺管弦弦乐）；在卡上写明。

**每张卡的录音演奏者**（`samples.py list`；合成声部补齐其余）：
管弦：小提琴、violins-spic（固定音型）、大提琴、低音提琴、圆号、小号、长号、timpani-roll、小鼓、锣 ·
新古典：立式或三角钢琴、大提琴、小提琴 · 巴洛克：羽管键琴、violins-pizz、cellos-pizz、竖笛、长笛 ·
爵士：ride、hihat-foot、bass-pizz、upright、sax（快乐句用 sax-stac）、trumpet-muted、vibes · lo-fi：fm-piano、upright、vibes、bass-pizz ·
house 与 disco：violins-spic、clavi、trumpet-stac、claps、tambourine · Afrobeat 与 Latin：conga、bongos、agogo、
cowbell、shaker、balafon、mbira、trumpet-stac · 民谣：strumstick、folk-harp、harmonica、recorder、frame-drum ·
安纳托利亚：darbuka、frame-drum、扬琴或 psaltery 演拨奏曲调、竖笛或长笛演 ney、小提琴 ·
顽皮：clarinet-stac、bassoon-stac、tuba-stac、xylophone、violins-pizz、claves · 紧张：cellos、violins-trem、
contrabass、tubular-bells、gong。

### 一段真实录音（音乐的 ★）
用类型和演奏者在开放档案里搜：`asset_search.py search --q "<genre and instruments>" --type music
--max-wait 12 --out research/music-a`（Internet Archive 的 Jamendo 和 netlabel 合集里发布了 CC0 和 CC BY 的曲目，那里类型词好用；Jamendo 通过 Openverse 也一样；每次搜索同时发起）以及 `--type sound --sources
wikimedia`（公有领域录音：美国军乐团
比如 Airmen of Note、Army Blues、Navy Band Commodores 演爵士和大乐队；大学与档案的录音演古典和民谣）。用 `fetch --preview 30` 取三到六个候选（每个只前 30 s，别更多），
发给用户听，只把选中的完整取下，测量它（`beat_grid.py`），在小节线上剪并设置
`OWN_MUSIC`（`music-video.md`）。它的 LICENSES.csv 行和署名行随完整抓取一起给你。
**最稳的第一步：** 先搜公有领域和 CC0 录音（`--licence cc0,pd`），没有合适的再用 CC BY；绝不用 NC、ND 或授权不明的（工具会把它们丢掉）。选中的源页面要打开一次：上传者必须是作者本人。
在 Internet Archive 上，授权是上传者的声明：一个有名或有商业性质的录音即使标着 "public domain" 也会被排除，而选中的 LICENSES.csv 行会写明是谁上传的。
选定之后，用一行说明一条 CC BY 曲目需要在描述里署名（`<title>-credits.txt`），以及 YouTube 的 Content ID 仍可能标记一条同时被该艺人注册的 CC BY 曲目：署名文件和授权链接能解决这场争议；CC0 或公有领域的录音则完全避开它。

### 不等于那四个和弦的和声

- 大调、明亮：I–V–vi–IV（C G Am F）、I–vi–IV–V、I–IV（两个和弦就能撑起一部片子）。
- 小调、严肃：i–VI–III–VII（Am F C G）、i–iv–v、安达卢西亚下行 i–VII–VI–V（Am G F E）。
- 调式色彩：Dorian i–IV（Dm G）、Mixolydian I–bVII–IV（D C G）、Phrygian i–bII（Em F）用来制造张力。
- 爵士：带七音的 ii–V–I；12 小节布鲁斯 I7–IV7–V7。
- 长长一段只用一个和弦而旋律在动，往往比一段忙碌的进行更有效。
- 在转折处换进行：升一个大调级的新调，或用影片开头那个大调的小调。

### 旋律与弧线

- **一个 3–5 个音的母题**，有自己的节奏，早点陈述，被应答（呼应），回来时变化（更高、更慢、换乐器）；这部片子的标志性声音可以是这个母题。
- **留出空间：** 旁白说话时，旋律要休息或压低；它在句子之间说话。
- **靠加法构建：** 前奏里一个元素，每 4–8 小节加一个新声部，回报前先来一段拆解（鼓撤掉），在转折或峰值上全部到位，结尾母题变一次，然后收在主和弦或一个持续音上，绝不是淡出到突然停掉。
- **短乐句配短 take：** 一个比约 0.4 s/音还快的句子要演奏 `-stac` 或 `-pizz` 的 take（`sax-stac`、`trumpet-stac`、`violins-pizz`）；持续音的 take 是给长音用的。快速乐句用持续音的 take 听起来像键盘演示。比它 4 s 的 take 更长的持续音，就在淡出下再放一遍，或者铺在 pad 上。
- **落在演奏者的音域里：** 离录音 take 超过 4 个半音的音会让渲染停下（`samples.py list` 给出每个乐器的音域）；把它高八度写，或者把这一句交给另一位演奏者。
- **旋律住在那里，就用真人演奏者：** 母题和贝斯用录音更好听（`samples.py`），鼓和 pad 可以继续用合成。
