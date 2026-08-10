# 📦 论文分享技能包 | Paper Share Skills

**PDF 论文 → B 站讲解视频 一站式 AI 技能套件（Claude Code / Codex / OMP 通用）**

> 8 个即插即用技能 + 1 套模板，覆盖论文分享全流程：从 arXiv 下载论文源文件，到生成 SUSTech Beamer 幻灯片，再到配音讲解视频并发布 B 站。
>
> A drop-in skill suite for the paper-to-bilibili pipeline — from arXiv download and SUSTech Beamer slides to narrated videos and Bilibili publishing, installable in Claude Code, Codex CLI, and OMP.

> 👤 **杨昊波 (Haobo Yang)** — 2026 届计算机系博士 · [github.com/yhbcode000](https://github.com/yhbcode000) · yhbcode000@foxmail.com
> 🎨 配套 LaTeX 模板：[sustech-slides-template](https://github.com/yhbcode000/sustech-slides-template)

---

## 🎯 适用场景 | Use Cases

| 🏷️ 场景 | 说明 |
|:---|:---|
| 📄 **论文分享** Paper Share | 组会文献精读，自动生成 11 节结构幻灯片（方法、实验、贡献、附录） |
| 🎬 **讲解视频** Narrated Video | 幻灯片逐页配音，横版 + 竖版双 MP4，适配 B 站 / 小红书 |
| 📤 **B 站发布** Bilibili Upload | biliup 扫码登录，标题/分区/封面/定时发布一站完成 |
| 🏫 **实验室调研** Lab Survey | 配合 lab-survey 技能生成调研海报与视频封面 |
| 🎓 **课程组会复用** Reusable | 技能与模板解耦，环境变量配置后可在任意机器复用 |

---

## 🚀 快速开始 | Quick Start

### 安装技能 | Installation

每个技能都是一个普通目录：`SKILL.md`（YAML frontmatter：`name` + `description`）+ 脚本/模板，三种 Agent 通用：

| Agent | 安装位置 | 命令（在本仓库根目录） |
|-------|----------|------------------------|
| Claude Code | `~/.claude/skills/<name>/` 或项目 `.claude/skills/` | `cp -r skills/* ~/.claude/skills/` |
| Codex CLI | `~/.codex/skills/<name>/` 或项目 `.codex/skills/` | `cp -r skills/* ~/.codex/skills/` |
| OMP (Oh My Pi) | `<工作根目录>/.omp/skills/<name>/` | `cp -r skills/* .omp/skills/` |

> 💡 说明：额外的 frontmatter 字段（`user-invocable`、`argument-hint`）会被不识别它们的 Agent 忽略；`SKILL.md` 中引用的 `<SKILLS_DIR>/<name>/scripts/...` 即安装技能目录的上层目录；Windows 下请用 Explorer / `xcopy /E /I skills <target>` 复制（符号链接需要管理员权限）。

### 一行式流水线 | One-line pipeline

```text
PDF 论文 → MinerU Markdown → SUSTech Beamer 幻灯片 → 横竖双配音视频 → B 站投稿
```

由编排技能 `paper-to-bilibili` 一键驱动（`slides` / `video` / `bilibili` 三种模式），各环节技能详见下文。

---

## 🔧 流水线技能 | Pipeline Skills

| 技能 | 角色 |
|:---|:---|
| `paper-download-arxiv-paper-source` | 📥 arXiv 论文 TeX 源码 / PDF 下载 |
| `pdf-to-markdown` | 📄 PDF → 干净 Markdown（MinerU，GPU） |
| `paper-to-beamer` | 🎞️ 论文 → 编译好的 SUSTech Beamer 幻灯片 |
| `paper-video-cover` | 🖼️ 16:10 视频封面海报（tex/pdf/png） |
| `paper-slides-to-video` | 🎬 PDF 幻灯片 → 横版 + 竖版配音 MP4（edge-tts） |
| `paper-bilibili-uploader` | 📤 MP4 + 元数据 → B 站投稿（biliup） |
| `paper-to-bilibili` | 🧭 编排器：`slides` / `video` / `bilibili` 三模式 + 状态契约 |
| `sustech-beamer-theme-fix` | 🔧 修复旧模板缺失 `\setsource` 等宏的问题 |

点击展开每个技能的使用方式 👇

<details>
<summary>📥 <b>paper-download-arxiv-paper-source</b> — arXiv 论文源文件下载</summary>

**用途**：从 arXiv URL 或裸 ID 下载论文 TeX 源码（e-print tar.gz）并解包到 `paper_src/`；管道输入是 arXiv 链接且优先用 TeX 源码时**第一个**调用它（paper-to-beamer、paper-to-bilibili、paper-venue-discovery 的前置）。

**触发**：提供 arXiv 链接（abs/pdf/e-print、arxiv.org 或 export.arxiv.org）或裸 ID（如 `2509.07996v4`）时。

**运行**：
```bash
uv run --no-project python \
  "<SKILLS_DIR>/paper-download-arxiv-paper-source/scripts/download_source.py" \
  "<arxiv-url|arxiv-id>" [--output DIR] [--force]
```

</details>

<details>
<summary>📄 <b>pdf-to-markdown</b> — PDF 转 Markdown（MinerU）</summary>

**用途**：把 PDF（尤其学术论文）转换为干净 Markdown —— 保留阅读顺序、LaTeX 公式、表格与图片，GPU 加速。

**触发**：用户要求把 PDF 转成 Markdown / 文本，或说"把这篇论文提取成文字"。

**运行**：
```bash
"$MINERU_PYTHON" \
  "<SKILLS_DIR>/pdf-to-markdown/convert.py" \
  "<paper.pdf>" "<output_dir>" [--lang en]
```

</details>

<details>
<summary>🎞️ <b>paper-to-beamer</b> — 论文 → SUSTech Beamer 幻灯片</summary>

**用途**：自动化管道：论文（PDF 或 TeX 源码）→ SUSTech Beamer 幻灯片（11 节「论文分享」结构：背景/方法/实验/贡献/局限/附录）→ 编译好的 PDF。有 TeX 源码时跳过 MinerU，直接提取 LaTeX 内容。

**触发**：用户要求把论文做成幻灯片，或说"论文分享"、"paper to slides"、"make beamer from paper"。

**运行**：
```bash
uv run --project "<SKILLS_DIR>/paper-to-beamer" python \
  "<SKILLS_DIR>/paper-to-beamer/scripts/copy_template.py" \
  --output "论文分享/<DIR_NAME>/slides-template"
# 之后用 latexmk -xelatex 编译生成的 main.tex
```

</details>

<details>
<summary>🖼️ <b>paper-video-cover</b> — 视频封面海报生成</summary>

**用途**：从论文目录确定性生成 16:10 视频封面海报 —— 提取论文元数据与图片，原子化产出 `poster.tex` / `poster.pdf` / `poster.png`。用于**视频封面**，不是会议海报。

**运行**：
```powershell
uv run --project "<SKILLS_DIR>/paper-video-cover" `
  "<SKILLS_DIR>/paper-video-cover/scripts/generate_poster.py" `
  "<paper_dir>"
```

</details>

<details>
<summary>🎬 <b>paper-slides-to-video</b> — 幻灯片 → 配音讲解视频</summary>

**用途**：把 Beamer PDF 幻灯片转成 fail-closed 的配音 MP4：逐页对齐旁白（`% NARRATION:` 契约）、TTS 音频（edge-tts）、封面与上传元数据，横版 + 竖版双输出。

**触发**：用户要求"配音幻灯片 / 幻灯片视频 / 论文视频"。

**运行**（走编排器，或直接）：
```bash
python "<SKILLS_DIR>/paper-slides-to-video/scripts/batch.py" video \
  "<paper_dir>" --engine edge-tts
```

</details>

<details>
<summary>📤 <b>paper-bilibili-uploader</b> — B 站投稿</summary>

**用途**：用 biliup 校验并上传配音论文视频到 B 站 —— 处理横版优先/竖版其次的投稿顺序、多 P 系列上传、CST 定时发布、诊断、dry-run 校验与持久化上传回执。

**触发**：用户说 "upload to bilibili"、"bilibili upload"、"发布到B站"、"上传到B站"。

**运行**：
```bash
python "<SKILLS_DIR>/paper-bilibili-uploader/scripts/upload.py" "<paper_dir>"
```
首次上传会触发 B 站 App 扫码登录（cookie 存于 `~/.bilibili/cookies.json`）。

</details>

<details>
<summary>🧭 <b>paper-to-bilibili</b> — 全流程编排器</summary>

**用途**：把论文 PDF 编排走完 MinerU、SUSTech Beamer 幻灯片、完整中文旁白、横竖双视频、元数据门禁与 B 站投稿；支持单篇与批量，`slides` / `video` / `bilibili` 三种阶段模式，带磁盘状态契约（可断点续跑）。

**触发**：用户要求"论文转 B 站 / PDF 转视频发布 / 批量发论文"等完整流水线请求。

**运行**：
```bash
python "<SKILLS_DIR>/paper-to-bilibili/scripts/batch.py" slides <papers.json>
python "<SKILLS_DIR>/paper-to-bilibili/scripts/batch.py" video  <papers.json>
python "<SKILLS_DIR>/paper-to-bilibili/scripts/batch.py" bilibili <papers.json>
```

</details>

<details>
<summary>🔧 <b>sustech-beamer-theme-fix</b> — 主题宏修复</summary>

**用途**：当使用旧版模板编译 SUSTech Beamer 幻灯片报 `\setsource` / `\setdomains` / `\setpresenter` / `\setvenue` 未定义时，从参考实现复制扩展版 `beamerthemesustech.sty` 修复。

**触发**：任何 SUSTech 幻灯片编译报上述 undefined 命令时。

</details>

<details>
<summary>🧩 <b>可选配套技能</b> — Optional companion skills</summary>

以下技能通过 `skill://` 被引用，**不随本仓库分发**；自行安装或接受降级行为：

| 技能 | 作用 |
|:---|:---|
| `edge-tts-retry-video-driver` | edge-tts 批量 TTS 失败时的逐页重试驱动 |
| `index-tts-fallback` | IndexTTS 超时时的 edge-tts 回退 |
| `sustech-beamer-overflow` | 幻灯片溢出规则（表格/图片/密度控制） |
| `rednote-video-uploader` | 竖版视频 + 小红书发布 |

</details>

---

## ⚙️ 环境变量配置 | Configuration

换机器无需改文件 —— 所有机器相关值由环境变量控制：

| 环境变量 | 含义 | 默认值 |
|:---------|:-----|:-------|
| `PP_ROOT` | 流水线工作根目录（含 `论文分享/`） | 当前工作目录 |
| `MINERU_PYTHON` | 装有 MinerU 的 Python 解释器 | `python` |
| `PP_PYTHON` | 流水线脚本用 Python 解释器 | `python` |
| `LUALATEX` | `lualatex` 完整路径 | 从 PATH 解析 |
| `POPPLER_DIR` | 含 `pdftoppm` 的目录 | 从 PATH 解析 |
| `PRESENTER` | 幻灯片 `\author` 汇报人 | 询问用户 |
| `INSTITUTE` | 幻灯片 `\institute` 机构 | 询问用户 |

示例：

```bash
export PP_ROOT=/path/to/my/pipeline          # 含 论文分享/
export MINERU_PYTHON=/path/to/mineru-env/bin/python
export PRESENTER="Jane Doe"
export INSTITUTE="Example University, China"
```

---

## 📦 依赖 | Requirements

点击展开 👇

<details>
<summary><b>环境要求</b> — Python / TeX / ffmpeg / TTS / 上传</summary>

- **Python 3.10+**（PATH 上的 `python`；Windows 缺失时用 `py -3`）
- **TeX Live**（或 MiKTeX）：PATH 上含 `lualatex` 与 `pdftoppm`
- **ffmpeg**（视频合成）
- **edge-tts** — `python -m pip install edge-tts`
- **biliup**（仅上传技能）— `python -m pip install biliup`

**MinerU 虚拟环境**（`pdf-to-markdown`、`paper-to-beamer`、`paper-to-bilibili` 需要）：

```bash
uv venv --python 3.12 "<your-env>"
uv pip install --python "<your-env>/Scripts/python.exe" torch torchvision --index-url https://download.pytorch.org/whl/cu124
uv pip install --python "<your-env>/Scripts/python.exe" -U "mineru[core]"
"<your-env>/Scripts/mineru-models-download.exe" -s huggingface -m pipeline
export MINERU_PYTHON="<your-env>/Scripts/python.exe"   # Windows
# export MINERU_PYTHON="<your-env>/bin/python"          # macOS / Linux
```

无 NVIDIA GPU？安装 CPU 版 torch（`uv pip install --python ... torch torchvision`，不加 `--index-url`）—— 管道照常运行，只是更慢。

**B 站 cookie**：存于 `~/.bilibili/cookies.json`（跨用户共享）。首次上传触发二维码登录 —— 用 B 站手机 App 扫码。

</details>

---

## ✅ 安装自检 | Verification

点击展开 👇

<details>
<summary><b>六步自检</b> — 确认环境可用</summary>

```bash
# 1. Python 可用
python -c "print('OK')"

# 2. 工作根目录
ls "$PP_ROOT/"

# 3. MinerU 可导入
$MINERU_PYTHON -c "import torch, mineru; print('OK')"

# 4. TeX 工具
lualatex --version
pdftoppm -v

# 5. ffmpeg
ffmpeg -version

# 6. edge-tts / biliup
python -c "import edge_tts; print('OK')"
python -c "import biliup; print('OK')"
```

</details>

---

## 📂 部署结构 | Deployment Layout

点击展开 👇

<details>
<summary><b>推荐目录布局</b> — 技能与产物各归其位</summary>

```
<PP_ROOT>/
+- .omp/skills/                     # OMP 安装目标（或 ~/.claude/skills、~/.codex/skills）
|   +- paper-to-bilibili/
|   +- paper-to-beamer/
|   +- paper-video-cover/
|   +- paper-slides-to-video/
|   +- paper-bilibili-uploader/
|   +- paper-download-arxiv-paper-source/
|   +- pdf-to-markdown/
|   +- sustech-beamer-theme-fix/
+- sustech-slides-template/         # 配套模板仓库（sustech-slides-template）
|   +- main_template.tex
|   +- sustech-theme/
|   +- latexmkrc
+- 论文分享/                        # 流水线产物落盘目录
    +- <VENUE> - <TITLE>/
        +- slides-beamer/
        +- video/
        +- poster/
        +- md_output/
```

</details>

---

## 🎨 品牌定制 | Rebranding (optional)

点击展开 👇

<details>
<summary><b>换机构品牌</b> — 名称 / Logo / 配色 / 署名</summary>

模板自带 SUSTech 品牌（`sustech-slides-template/`）。如需为其他机构换肤：

1. **名称** — `main_template.tex` 中 `\institute[...]` → 你的机构。
2. **Logo** — 替换 `sustech-theme/assets/sustech_logo.png`（带透明通道 PNG，约 1600px 宽，保持文件名）或改 `beamerthemesustech.sty` 中的 `\logo{}`。
3. **配色** — 编辑 `sustech-theme/beamercolorthemesustech.sty`（`sustechorange` 主色、`sustechdark` 页眉、`sustechgrey` 辅助、`sustechlight` 表格行）；新色板也可注册为 `\sustechscheme{<name>}` 配色方案。
4. **署名 / 机构** — 设置 `PRESENTER` / `INSTITUTE`（见环境变量配置）；`paper-to-beamer` 会读取它们生成 `\author` / `\institute`。

</details>

---

## 📣 分发建议 | Distribution

本套件产出内容（幻灯片、讲解视频、博客等）欢迎自由分发，建议遵循：

| 渠道 | 建议 |
|:---|:---|
| 🎬 **B 站视频** | 讲解视频建议发布至 B 站，并 @ **白拾的物理组会** 获取选题 / 排版反馈，加入组会共创 |
| 🐙 **GitHub** | 在视频简介 / 博客文末附本仓库链接（`github.com/yhbcode000/paper-share-skills` 与配套模板 `sustech-slides-template`），方便观众取用 |
| 📝 **署名** | 幻灯片保留 `\setcreditauthor` 作者信息宏；二次创作请保留原论文引用与来源链接 |
| 🔄 **改造** | 允许 fork / 二次开发，衍生作品需保留 Apache-2.0 声明（见下） |

---

## 📜 许可 | License

[Apache License 2.0](LICENSE) © 2026 杨昊波 (Haobo Yang)

> 本仓库代码与技能文档以 **Apache-2.0** 协议开源：可自由使用、修改、分发（含商用），须保留版权声明与许可文本；衍生作品须注明修改并继续沿用 Apache-2.0。
> 附带的 SUSTech 主题（`paper-to-beamer` 内嵌模板）沿用其原始署名约定，欢迎自由使用与修改，保留主题作者署名即可。
