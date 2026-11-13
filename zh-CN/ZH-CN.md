# cinecode 中文文档

> 本分支是 [sarpstar/cinecode](https://github.com/sarpstar/cinecode) 的**简体中文翻译版**。
> 上游作者：Sarp Yıldız（[@sarpstar](https://x.com/sarpstar)），MIT 许可证。
> 翻译整理：瑞哥观势（Eric）。

---

## 这是什么

cinecode 是一个 Agent Skill 包（Claude Code / Cursor / Codex 均可安装）。你给它一句话的想法，它和你一起把它做成一部短片。

上游定位原话：

> You make a short film with the user. Their taste decides; you do the rest: story, script, picture, sound, edit, checks. The aim is a film worth the wait.

翻译：**你决定品味，剩下它做——剧本、画面、声音、剪辑、检查。**

它不用生成模型。每一帧都是代码渲染的（`frame = f(t, seed)`），最终产出 -14 LUFS 母版 MP4。

---

## 本仓库

本 fork 相对上游只做了一件事：**把全部文档翻译成简体中文**，放在 `zh-CN/` 目录。上游 `main` 分支保持原样，随时可以 `git diff main..zh-CN` 查看全部改动。

| 文件 | 对应上游 | 说明 |
|---|---|---|
| [`zh-CN/SKILL.zh-CN.md`](zh-CN/SKILL.zh-CN.md) | [`SKILL.md`](https://github.com/sarpstar/cinecode/blob/main/SKILL.md) | 技能主文件：流程、提问规则、硬规则 |
| [`zh-CN/ZH-CN.md`](zh-CN/ZH-CN.md) | — | 本索引页 |

`references/` 下的 24 个参考文档的中文版位于 [`zh-CN/references/`](zh-CN/references/)。

---

## 文档地图

### 流程与模式

| 中文 | 英文原文 | 内容 |
|---|---|---|
| [快速模式](zh-CN/references/quick-mode.zh-CN.md) | `quick-mode.md` | 单文件出片，约 3 次确认 |
| [电影模式](zh-CN/references/film-mode.zh-CN.md) | `film-mode.md` | 六个门控步骤的制片厂流程 |
| [上下文与修订协议](zh-CN/references/context-protocol.zh-CN.md) | `context-protocol.md` | 何时派 helper，何时留在主会话 |
| [交接说明](zh-CN/references/context-protocol.zh-CN.md) | `context-protocol.md` | `STATE.md` 续作机制 |

### 创作方法

| 中文 | 英文原文 | 内容 |
|---|---|---|
| [写作](zh-CN/references/writing.zh-CN.md) | `writing.md` | 先散后收、预测测试、重叠检查、词数预算 |
| [写作提示词](zh-CN/references/writer-prompt.zh-CN.md) | `writer-prompt.md` | 每个故事必读的 12 步方法 |
| [导演手法](zh-CN/references/craft.zh-CN.md) | `craft.md` | 镜头数量、节奏、屏幕文字、节拍类型 |
| [视觉风格](zh-CN/references/looks.zh-CN.md) | `looks.md` | 58 种视觉风格的完整配方 |

### 技术实现

| 中文 | 英文原文 | 内容 |
|---|---|---|
| [渲染引擎](zh-CN/references/engine.zh-CN.md) | `engine.md` | `frame = f(t, seed)`、capture、ffmpeg 编码、质检 |
| [画面内运动](zh-CN/references/motion.zh-CN.md) | `motion.md` | 运动的因果、次级运动、缓动 |
| [运动设计](zh-CN/references/motion-design.zh-CN.md) | `motion-design.md` | 用运动讲故事的装置 |
| [运镜词典](zh-CN/references/camera-moves.zh-CN.md) | `camera-moves.md` | 命名运镜与转场的词汇表 |
| [声音](zh-CN/references/sound.zh-CN.md) | `sound.md` | 分层声音、声音身份、-14 LUFS 母版 |
| [字体排印](zh-CN/references/typography.zh-CN.md) | `typography.md` | 字体匹配、样卡、手机可读性 |
| [成品处理](zh-CN/references/finish.zh-CN.md) | `finish.md` | 光、镜头、空气、材质、几何、运动、后期的工具箱 |
| [角色](zh-CN/references/characters.zh-CN.md) | `characters.md` | 风格化演员阵容 |

### 用途分支

| 中文 | 英文原文 | 内容 |
|---|---|---|
| [电影质感](zh-CN/references/purpose-cinematic.zh-CN.md) | `purpose-cinematic.md` | 看起来像被拍的，不像被渲染的 |
| [广告与产品](zh-CN/references/purpose-ad.zh-CN.md) | `purpose-ad.md` | 差异化测试、多比例交付 |
| [参考视频改编](zh-CN/references/reference-video.zh-CN.md) | `reference-video.md` | "照这个格式做，但改成……" |
| [自己素材的剪辑](zh-CN/references/footage-edit.zh-CN.md) | `footage-edit.md` | 词级时间轴、停顿剪除、图形贴合 |

### 辅助

| 中文 | 英文原文 | 内容 |
|---|---|---|
| [舵向](zh-CN/references/steering.zh-CN.md) | `steering.md` | 代码画不好的东西，先说清楚代价 |
| [评审团](zh-CN/references/jury.zh-CN.md) | `jury.md` | 六个问题的独立冷评 |
| [音乐视频](zh-CN/references/music-video.zh-CN.md) | `music-video.md` | 节拍网格测量、卡点剪辑 |
| [素材与授权](zh-CN/references/assets-and-licences.zh-CN.md) | `assets-and-licences.md` | 下载即登记，授权不明即不用 |

---

## 翻译约定

| 项目 | 处理方式 |
|---|---|
| YAML frontmatter | **逐字节保留**（机器读取，不可改） |
| 文件路径、脚本名、命令行参数 | 逐字保留 |
| 代码块、行内 `code` | 逐字保留，不译 |
| 表格结构、标题层级、列表顺序 | 完全一致，数量不变 |
| 土耳其语 UI 字符串 | 译为中文，首次出现处附土耳其语原文，例如 逐步推进（Aşama aşama） |
| `★ (önerilen)` | 统一为 `★ (推荐)` |
| 引用的英文原句 | 保留英文，后附中文注释 |
| 数字、单位、时长、响度值 | 精确保留，不四舍五入 |

---

## 快速上手（译自上游 README）

**需要的环境**

```
node -v        # Node.js 22 或更高
python3 -V     # Python 3.9 或更高
ffmpeg -version
```

还要有 Chrome / Chromium / Edge。磁盘：快速模式几百 MB 够用，**电影模式每部片子渲染期间要几个 GB**。

**安装**

```bash
git clone https://github.com/sarpstar/cinecode.git ~/.claude/skills/cinecode
```

Windows 用户把路径换成 `%USERPROFILE%\.claude\skills\cinecode`。

装完**新开一个对话**，技能靠对话启动时加载。

**开始**

```bash
mkdir my-film && cd my-film && claude
```

然后说：

```
用 cinecode 一起做一部片子：<你想讲什么，一两句话>
```

**不用写模式**，它会按工作量推荐。**你写得越少，它问得越多；你写得越全，它越少打扰你。**

---

## 中文读者提示

上游项目默认语言是土耳其语，所有面向用户的提示语、选项标签、菜单都是土耳其语。中文版保留了这些 UI 字符串的原文（如 `Aşama aşama`），方便对照上游实际界面。

上游目前也提供官方 `README.zh-CN.md`（仅 README，非全部文档），本分支补齐了 `references/` 下全部 24 个文档的中文翻译。

---

## 许可与致谢

原始项目 [sarpstar/cinecode](https://github.com/sarpstar/cinecode)，MIT License，Copyright (c) 2026 Sarp Yıldız。

本分支的中文翻译为第三方贡献，遵循同样的 MIT 许可。`styles/` 目录中 51 种风格配方来自 LemoLab 的 `lemo-opuscar` 项目（MIT）。

翻译整理：瑞哥观势
