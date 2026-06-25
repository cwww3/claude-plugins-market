---
name: video-analyzer
description: >
  自动分析视频内容（B站/YouTube）——下载视频、拆帧、语音转录、AI分析，
  最终生成高质量的专题文档或实操教程，存入 Obsidian 笔记库。
  Use when user provides a video URL (Bilibili or YouTube),
  or mentions "分析视频", "bilibili分析", "youtube分析", "视频拆解",
  or /video-analyzer.
argument-hint: <video-url>
disable-model-invocation: false
metadata:
  short-description: 视频AI分析工具（B站/YouTube）
  version: 1.1.0
source:
  - name: FFmpeg
    documentation: https://ffmpeg.org/documentation.html
  - name: Bilibili API
    documentation: https://github.com/SocialSisterYi/bilibili-API-collect
  - name: yt-dlp
    documentation: https://github.com/yt-dlp/yt-dlp
  - name: mlx-whisper
    documentation: https://github.com/ml-explore/mlx-examples/tree/main/whisper
---

# Video Analyzer

## Description

B站 / YouTube 视频内容深度分析工具。基于 `curl + yt-dlp + ffmpeg + Python/OpenCV` 的核心技术栈，可选集成语音转录（mlx-whisper/faster-whisper）提升内容质量。

**核心能力**:
- **双平台支持**：Bilibili（curl + API）和 YouTube（yt-dlp），后续流程统一
- **轻量核心**：只需 ffmpeg + Python 即可跑通完整流程
- **音频转录（可选）**：集成 mlx-whisper 进行中文语音转文字，作为帧分析的补充
- **像素差去重**：Python/OpenCV 降采样帧差去重，对 PPT 类视频效果优于 PSNR
- **并行帧分析**：使用多个 Claude Sonnet Agent 分批并行读取帧图片
- **结果存入 Obsidian**：产出文件直接写入 `$NOTESDIR` 笔记库

## Prerequisites

依赖分为三层：核心必备（没有跑不起来）、语音增强（强烈推荐）、图像降级（平时用不到）。

### 🔴 Tier 0 — 核心必备

整个流程的骨架，**必须先装**。

| 工具 | macOS 安装 | Linux 安装 | 用途 |
|------|-----------|-----------|------|
| **ffmpeg** | `brew install ffmpeg` | `sudo apt install ffmpeg` | 拆帧 + 音频提取 |
| **Python 3** | 系统预装或 `brew install python` | 系统预装 | 帧去重脚本 |
| **opencv-python** | `pip install opencv-python numpy` | 同上 | 像素差去重 |
| **curl + jq** | 系统预装 | `sudo apt install curl jq` | Bilibili API |
| **yt-dlp** | `pip install yt-dlp` | 同上 | YouTube 下载 |
| **Node.js** | 系统预装或 `brew install node` | `sudo apt install nodejs` | yt-dlp JS 挑战求解（YouTube 需要） |

验证：
```bash
ffmpeg -version
python3 -c "import cv2, numpy; print('OK')"
curl --version | head -1
yt-dlp --version
node --version
```

**仅 Tier 0 就能跑完整流程**：下载 → 拆帧 → 去重 → Agent 分析 → 生成文档。差异是没有音频转录信息。

---

### 🟡 Tier 1 — 语音转录（强烈推荐）

安装后自动将视频语音转为文字，**大幅提升文档内容的完整性**（特别是对口语讲解密度高的视频）。

| 平台 | 安装命令 | 大小 |
|------|---------|------|
| Apple Silicon (M1-M4) | `pip install mlx-whisper` | 约 50MB（不含模型） |
| Intel Mac / Linux | `pip install faster-whisper` | 约 30MB（不含模型） |

首次运行时会自动下载模型（约 1.5GB），后续缓存复用。

验证：
```bash
# Apple Silicon
mlx_whisper --help
# Intel / Linux
faster-whisper --help
```

**不装 Tier 1 也能跑**，只是文档会缺少语音内容——对 PPT 密集型视频影响不大，对口语讲解密集型视频影响较大。

---

### 🟢 Tier 2 — 图像降级（仅极端情况需要）

**正常情况下完全不需要。** 只有当 Agent 模型不支持直接读取图片（如图片显示 `[Unsupported Image]`）时才会触发降级，按优先级尝试：

| 优先级 | 方案 | 安装 | 原理 |
|--------|------|------|------|
| 1（macOS 自动） | **Vision Framework OCR** | **零安装**，macOS 内置 | 系统级 OCR 引擎，中文识别好 |
| 2（手动） | **Tesseract OCR** | `brew install tesseract` | 开源 OCR，需额外装中文语言包 |

```bash
# Tesseract（仅当 Vision OCR 不可用时作为备选）
brew install tesseract
# 语言包 brew 默认不带中文，但 chi_sim 通常已包含
tesseract --list-langs | grep chi_sim
```

- macOS 用户：Vision Framework 是系统内置的，**零安装**，永远可用
- Linux 用户：需手动 `sudo apt install tesseract-ocr tesseract-ocr-chi-sim`
- 这两种降级方案都**不需要额外 Python 包**（通过 subprocess 调用）

**再次强调**：Tier 2 只在"模型不能看图"的极端情况触发。正常情况下 Agent 直接用视觉分析帧图片，完全不需要 OCR。

## Environment

Target directory:
!`echo "${NOTESDIR:-⚠️  NOTESDIR is not set — results will go to ./video-analysis-<id> instead}"`

If `$NOTESDIR` is not set, stop and tell the user to set it.

## Workflow

### Step 0: 识别平台

根据 URL 判断平台：

```text
包含 "bilibili.com" → B站 → 用 Bilibili API
包含 "youtube.com" 或 "youtu.be" → YouTube → 用 yt-dlp
```

### Step 1: 获取视频信息

**B站**：
```bash
curl -L -s \
  -H 'Referer: https://www.bilibili.com/video/<BV号>' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' \
  "https://api.bilibili.com/x/web-interface/view?bvid=<BV号>" \
  | jq '{title:.data.title, duration:.data.duration, cid:.data.cid, owner:.data.owner.name}'
```

**YouTube**：
```bash
yt-dlp --print "%(title)s|%(duration)s|%(uploader)s" "<URL>"
```

**根据时长选择帧率**（两平台通用）:

| 时长 | fps | 预期帧数 |
|------|-----|---------|
| <10分钟 | 1.0 | ~600 |
| 10-30分钟 | 0.5 | ~600-900 |
| >30分钟 | 0.2 | ~400-900 |

### Step 2: 下载视频并拆帧 + 提取音频

**B站**：
```bash
OUTDIR="${NOTESDIR:-.}/video-analysis-<BV号>"
mkdir -p "$OUTDIR/images"

VIDEO_URL=$(curl -L -s \
  -H 'Referer: https://www.bilibili.com/video/<BV号>' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' \
  "https://api.bilibili.com/x/player/playurl?bvid=<BV号>&cid=<cid>&qn=80&fnval=1" \
  | jq -r '.data.durl[0].url')

curl -L --retry 3 \
  -H 'Referer: https://www.bilibili.com/video/<BV号>' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36' \
  "$VIDEO_URL" -o "$OUTDIR/video.mp4"
```

**YouTube**：
```bash
OUTDIR="${NOTESDIR:-.}/video-analysis-<video-id>"
mkdir -p "$OUTDIR/images"

# --cookies-from-browser 导入浏览器登录态
# --remote-components ejs:github 安装 JS 挑战求解器（绕过 bot 检测）
# --js-runtimes node 指定 JS 运行时
yt-dlp \
  --cookies-from-browser chrome \
  --js-runtimes node \
  --remote-components ejs:github \
  -f "bestvideo[height<=720]+bestaudio/best[height<=720]" \
  --merge-output-format mp4 \
  -o "$OUTDIR/video.mp4" \
  "<YouTube URL>"
```

> 如果 `--cookies-from-browser chrome` 失败，尝试 `safari` 或 `firefox`。如果仍然失败，告诉用户需要在浏览器中登录 YouTube 后再试。

**后续步骤（两平台统一）**：
```bash
# 拆帧
ffmpeg -hide_banner -y -i "$OUTDIR/video.mp4" \
  -vf "fps=<fps>" -q:v 2 "$OUTDIR/images/frame_%04d.jpg"

# 提取音频（用于转录）
ffmpeg -hide_banner -y -i "$OUTDIR/video.mp4" \
  -q:a 0 -map a "$OUTDIR/audio.wav"
```

### Step 3: 关键帧去重

使用 Python/OpenCV 降采样灰度图 + 均值绝对差进行相邻帧去重：

```python
import cv2, numpy as np
from pathlib import Path

src = Path("$OUTDIR/images")
dst = Path("$OUTDIR/keyframes")
dst.mkdir(exist_ok=True)

THRESHOLD = 2.0  # 像素差阈值 (0-255)
paths = sorted(src.glob("frame_*.jpg"))
prev = None
kept = []

for p in paths:
    img = cv2.imread(str(p), cv2.IMREAD_GRAYSCALE)
    small = cv2.resize(img, (160, 120))
    if prev is not None:
        diff = float(np.mean(cv2.absdiff(prev, small)))
        if diff < THRESHOLD:
            prev = small
            continue
    prev = small
    kept.append(p)

# 复制并重新编号
for i, p in enumerate(kept):
    shutil.copy2(p, dst / f"frame_{i+1:04d}.jpg")
```

**THRESHOLD 说明**:
- `2.0`（推荐）：对 PPT 幻灯片视频有较好的区分度
- `1.0`：更激进去重，适合大量重复画面的视频
- `3.0`：保守去重，保留更多帧

### Step 4: 音频转录（可选，推荐）

如果安装了 Tier 1 工具，在后台运行转录，与 Step 5 帧分析并行。

```bash
# macOS Apple Silicon
mlx_whisper "$OUTDIR/audio.wav" \
  --model mlx-community/whisper-large-v3-turbo \
  --output-dir "$OUTDIR" \
  --output-format txt \
  --language zh &

# Linux / Intel Mac
faster-whisper "$OUTDIR/audio.wav" \
  --model large-v3 \
  --language zh \
  --output_dir "$OUTDIR" &
```

**如果未安装 Tier 1**：跳过此步。文档生成时仅基于帧分析内容，对 PPT 密集型视频影响很小。

### Step 5: 分批并行分析关键帧

**分批策略**（根据关键帧总数动态计算）:

| 关键帧数 | 分批数量 | 每批约 |
|---------|---------|--------|
| 1-30 | 1 批 | 全部 |
| 31-60 | 2 批 | 15-30 |
| 61-120 | 3 批 | 20-40 |
| 121-200 | 4 批 | 30-50 |
| 200+ | 5 批 | 40-70 |

使用多个 **Claude Sonnet Agent**（`model: "sonnet"`）并行分析。每个 Agent 读取一批关键帧图片，提取文字、代码、架构图等。

**Agent Prompt 模板**:

```
请逐个读取 {关键帧目录} 下的 frame_XXXX.jpg 到 frame_YYYY.jpg（共 N 张）。

对每张图片详细记录：
1. 帧号、场景类型（PPT/代码/终端/浏览器/架构图/其他）
2. 界面内容：窗口布局、IDE/编辑器、UI元素
3. 文字内容：完整转录屏幕上所有文字
4. 代码内容：如有代码请完整复制（保留缩进）
5. 操作动作：正在进行的操作
6. 关键信息：重要配置、步骤、错误信息

按帧号顺序输出。不要省略任何图片。代码和文字必须完整转录。
```

### Step 6: 生成文档

将音频转录 + Agent 帧分析结果综合整理，按**知识体系**重新组织（不是按时间线），生成 `$OUTDIR/视频分析.md`。

**判断视频类型**:
- 实操类（编程教程、软件操作）→ 操作教程模板
- 知识类（概念讲解、原理分析）→ 专题文档模板

**文档结构要求**:
- 内容重新组织，章节有逻辑顺序
- 不看视频也能完全理解
- 图片与内容严格对应：`![frame_xxxx: 图片实际内容](./keyframes/frame_xxxx.jpg)`
- 代码来自帧中的实际代码，标注来源
- 图文必须对应——图片描述反映实际内容，不是期望内容

详见下方的 Output Format 规范。

### Step 7: 质量检查

逐项验证：
1. 所有图片引用都指向存在的文件
2. 章节结构完整、逻辑清晰
3. 代码块来自视频实际内容
4. 文档可独立阅读
5. 音频转录已整合（如果完成）

### Step 8: 输出技术总结

文档生成完成后，**必须**向用户展示本次分析用到的技术栈和处理统计。格式如下：

```
## ✅ 分析完成

**视频**：《{标题}》（{平台}，{时长}）
**产出**：`{完整路径}/视频分析.md`

| 步骤 | 技术 | 结果 |
|------|------|------|
| 视频信息 | curl + Bilibili API / yt-dlp | {标题}，{时长} |
| 下载 | curl / yt-dlp | {大小}，{格式} |
| 拆帧 | ffmpeg fps={fps} | {N}帧 |
| 去重 | Python OpenCV 像素差 | {N}→{M}张关键帧 |
| 转录 | mlx-whisper / faster-whisper | {模型}，{语言}，{大小}文本 |
| 帧分析 | Claude Sonnet Agent ×{N} | {N}个Agent并行，{M}张/批 |
| 文档 | Claude 综合撰写 | {行数}行，{章节}章，{图片}图 |
| 质量 | 文件存在性检查 | {N}/{N}图片引用有效 |

{如有跳过的步骤，注明原因}
```

如果某步骤被跳过（如未安装 Tier 1 导致无转录），标注为 `⏭️ 跳过（原因）`。

统计数据的获取方式：
- 视频信息：Step 1 的 API 返回
- 帧数：`ls "$OUTDIR/images/"*.jpg | wc -l`
- 关键帧数：`ls "$OUTDIR/keyframes/"*.jpg | wc -l`
- 文档行数/图片数：`grep -c` 统计
- 图片引用验证：`grep -oP '\./keyframes/frame_\d+\.jpg' | while read f; do [ -f "$f" ] && echo OK; done`

---

## Output Format

### 知识文档类

```markdown
# {主题}

> 视频作者：{作者} | 时长：{时长} | 来源：B站 {BV号}

## 概述
{背景和核心问题}

## {章节1}
{内容+配图}

## 核心要点
- 要点列表

## 延伸阅读
{相关资源}

> 📝 本文档基于视频分析和音频转录整理。
```

### 实操教程类

```markdown
# {教程主题}

## 简介
{目标和前置条件}

## 环境准备
{安装和配置}

## 操作步骤
### 1. {步骤标题}
{说明 + 代码 + 截图}

## 完整代码
{汇总}

## 总结
{核心回顾}
```

### 图片插入规范

| 规则 | 格式 |
|------|------|
| 帧号标注 | `![frame_0015: 实际描述](./keyframes/frame_0015.jpg)` |
| 来源标注 | `<!-- 代码来自 frame_0025 -->` |
| 图文对应 | 图片必须与上下文内容匹配 |

---

## Examples

### 分析知识类视频（B站）

```bash
/video-analyzer https://www.bilibili.com/video/BV1HTGy68EUF
```

→ 生成 `$NOTESDIR/video-analysis-BV1HTGy68EUF/视频分析.md`（知识文档）

### 分析实操类视频（B站）

```bash
/video-analyzer https://www.bilibili.com/video/BV1aj5t6KER8
```

→ 生成 `$NOTESDIR/video-analysis-BV1aj5t6KER8/视频分析.md`（专题文档）

### 分析YouTube视频

```bash
/video-analyzer https://www.youtube.com/watch?v=Du9Xlg-JRZc
```

→ 生成 `$NOTESDIR/video-analysis-Du9Xlg-JRZc/视频分析.md`

---

## Output Directory Structure

```
$NOTESDIR/video-analysis-<id>/
├── 视频分析.md          # 最终文档
├── video.mp4            # 原视频
├── audio.wav            # 提取的音频（转录后可删除以节省空间）
├── audio.txt            # 语音转录文本
├── images/              # 原始帧
└── keyframes/           # 去重后的关键帧（文档中引用的图源）
```

---

## Technical Notes

### 去重算法选择

| 方法 | 工具 | 适用场景 |
|------|------|---------|
| 像素差 (Mean AbsDiff) | Python/OpenCV | PPT 幻灯片、屏幕录制 |
| PSNR (Peak Signal-to-Noise) | ffmpeg | 实景视频、电影 |
| SSIM (Structural Similarity) | ffmpeg | 需要感知相似度的场景 |

本技能默认使用像素差方法，因为它对 PPT 幻灯片中的文字/图表变化更敏感，不会漏掉关键的幻灯片切换。

### 音频转录模型选择

| 平台 | 推荐模型 | 速度 |
|------|---------|------|
| Apple Silicon (M1/M2/M3/M4) | mlx-whisper large-v3-turbo | 最快（MLX 加速） |
| Intel Mac / Linux | faster-whisper large-v3 | 较快（CTranslate2） |
| 通用 | openai-whisper large-v3 | 较慢（标准 PyTorch） |

### 当 Agent 无法读取图片时（降级路径）

正常情况下 **Agent 直接通过视觉读取帧图片**，不需要任何额外工具。

只有当模型不支持视觉（图片返回 `[Unsupported Image]`）时才触发降级：

```
Agent 视觉可用？
  ├── 是 → 直接分析图片（默认路径，零额外依赖）
  └── 否 → 降级路径：
            ├── macOS: Vision Framework OCR（系统内置，零安装）
            ├── 非 macOS: Tesseract OCR（brew/apt install）
            └── 最终兜底: 纯音频转录文本分析
```

| 降级方案 | 安装成本 | 触发条件 |
|---------|---------|---------|
| Agent 视觉 | **零** | 默认 |
| Vision OCR | **零**（macOS 内置） | Agent 不可用 + 是 Mac |
| Tesseract | `brew install tesseract` | Agent 不可用 + 不是 Mac |
| 纯音频 | **零**（依赖 Tier 1） | OCR 也失败时 |

此时文档中可能无法插入图片，但内容质量仍然基于帧 OCR 文本 + 音频转录保证。

---

## Quality Checklist

### 内容质量
- [ ] 内容按知识体系重新组织，不是时间线流水账
- [ ] 章节结构清晰，有逻辑顺序
- [ ] 不看视频也能完全理解
- [ ] 包含总结和核心要点

### 图文对应
- [ ] 每张图片标注帧号 + 实际内容描述
- [ ] 图片上下文的文字与图片内容直接相关
- [ ] 代码块标注来源帧号

### 代码质量
- [ ] 代码来自帧中的实际代码，不是编造的
- [ ] 代码片段完整，可直接复制使用

---

## Tags

`bilibili`, `youtube`, `video-analysis`, `ffmpeg`, `python`, `whisper`, `obsidian`, `markdown`
