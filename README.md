<p align="center">
  <img src="https://img.shields.io/badge/Hermes-Agent-6C5CE7?style=for-the-badge&logo=robotframework&logoColor=white">
  <img src="https://img.shields.io/badge/Platform-Xiaohongshu-FF2442?style=for-the-badge">
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge">
</p>

<h1 align="center">📕 Hermes Xiaohongshu Workflow</h1>
<p align="center">
  <i>AI-powered end-to-end content pipeline for Xiaohongshu (RED)<br>
  AI 驱动的小红书内容生产线 — 从灵感到待发布，全流程自动化</i>
</p>

<br>

<!-- ══════════════════════════  LANGUAGE PANELS  ══════════════════════════ -->

<details open>
<summary><b>🇬🇧 &nbsp;English</b></summary>

<br>

> **From a rough topic idea to a publish-ready Xiaohongshu post with image briefs — all driven by an AI agent that enforces quality, not just speed.**

<br>

### 🎯 The Problem

Writing a good Xiaohongshu post is a **multi-stage pipeline**, not a single "generate" button:

| Stage | What Happens | Quality Gate |
|-------|-------------|--------------|
| 🔍 **Research** | Is this topic worth writing? What angles are trending? | Multi-source verification for sensitive claims |
| ✍️ **Drafting** | Title ≤20 chars, body ≤1000 chars, CTAs, tags | 6-item pre-write review checklist |
| 🎨 **Image Planning** | Multi-image layouts, visual style, AI prompts | 4-item anti-laziness quality audit |

Most AI tools stop at "generate a post." This workflow **connects all three stages** through a Notion database with built-in quality gates.

<br>

### 🏗️ Architecture

```
  🔍 TOPIC RESEARCH          ✍️ CONTENT PRODUCTION        🎨 IMAGE PLANNING
  ┌──────────────────┐       ┌──────────────────────┐       ┌──────────────────┐
  │                  │       │                      │       │                  │
  │  Web search      │       │  Notion dedup        │       │  Read post       │
  │  Trend analysis  │  ───→ │  Fact-checking       │  ───→ │  Plan layouts    │
  │  Angle discovery │       │  Title + body + tags │       │  AI prompts      │
  │                  │       │                      │       │                  │
  │  Read-only       │       │  灵感池  →  待配图     │       │  待配图  →  待发布  │
  └──────────────────┘       └──────────────────────┘       └──────────────────┘
```

<br>

### ⚡ Quick Start

```bash
git clone https://github.com/DuggeeChen/hermes-xiaohongshu-workflow.git
cp -r skills/* ~/AppData/Local/hermes/skills/productivity/
# Configure Notion connection in Hermes
```

```bash
"帮我探索一下 AI 工具 方向的小红书选题"            # → Topic research
"写一篇小红书：干货教程，《用 AI 写周报的 3 个坑》"  # → Content drafting
"完成待发布"                                        # → Image planning
```

<br>

### 🧩 Three Skills

| Skill | Role | Input → Output | Writes to Notion? |
|-------|------|----------------|-------------------|
| `topic-research` | Discover angles | Direction → Curated topic list + credibility + sources | No |
| `content-production` | Draft posts | Topic + type → Full post (title, body, tags, CTAs) | ✅ |
| `image-planning` | Plan visuals | Post content → Multi-image layout + AI prompts | ✅ |

<br>

### 💡 Design Philosophy

| 🛡️ | **Gatekeeper, not mold** — Enforces what *not* to do: no fake data, no template CTAs, no overwriting existing content |
|🎨| **Freedom within guardrails** — Agent decides style, structure, pacing within hard boundaries |
| 🔍 | **Trust but verify** — Sensitive claims require ≥2 independent sources before proceeding |

<br>

### 🚫 What These Skills Do NOT Do

- Generate actual images (→ output prompts for ChatGPT Images 2.0, Midjourney, etc.)
- Upload or publish to Xiaohongshu
- Scrape platform data
- Fill post-publication analytics

The pipeline stops at **"待发布 (Ready to Publish)"** — manual final review is by design.

<br>

### 📂 Structure

```
hermes-xiaohongshu-workflow/
├── README.md
├── WORKFLOW.md              ← Deep-dive pipeline docs (bilingual)
├── LICENSE                  ← MIT
└── skills/
    ├── xiaohongshu-topic-research/SKILL.md
    ├── xiaohongshu-content-production/SKILL.md
    └── xiaohongshu-image-planning/SKILL.md
```

<br>

### 📄 License

MIT · See [LICENSE](./LICENSE)

</details>

<details>
<summary><b>🇨🇳 &nbsp;中文</b></summary>

<br>

> **从一个模糊选题到一篇待发布的小红书笔记（含配图方案），全流程 AI 驱动，重质量不重速度。**

<br>

### 🎯 解决什么问题？

写一篇好的小红书笔记不是打字，是**多阶段流程**：

| 阶段 | 做什么 | 质量关卡 |
|------|--------|----------|
| 🔍 **选题探索** | 这个话题值不值得写？有什么切入角度？ | 敏感选题需 ≥2 个独立来源核验 |
| ✍️ **内容生产** | 标题 ≤20 字、正文 ≤1000 字、互动引导、标签 | 写入前 6 项硬性复查清单 |
| 🎨 **配图方案** | 多图布局、视觉风格、AI 生图提示词 | 4 项反偷懒质检 |

大多数 AI 工具只做到「帮你写一段」，本工作流把**三个阶段串联**，每一步都有关卡。

<br>

### 🏗️ 架构

```
  🔍 选题探索              ✍️ 内容生产                🎨 配图方案
  ┌──────────────────┐    ┌──────────────────────┐    ┌──────────────────┐
  │                  │    │                      │    │                  │
  │  网络搜索         │    │  Notion 去重          │    │  读取正文         │
  │  趋势分析         │ ─→ │  事实核验              │ ─→ │  规划图片结构     │
  │  角度挖掘         │    │  标题 + 正文 + 标签    │    │  AI 提示词        │
  │                  │    │                      │    │                  │
  │  只读，输出到终端  │    │  灵感池  →  待配图      │    │  待配图  →  待发布  │
  └──────────────────┘    └──────────────────────┘    └──────────────────┘
```

<br>

### ⚡ 快速开始

```bash
git clone https://github.com/DuggeeChen/hermes-xiaohongshu-workflow.git
cp -r skills/* ~/AppData/Local/hermes/skills/productivity/
# 在 Hermes 中配置 Notion 连接
```

```bash
"帮我探索一下 AI 工具 方向的小红书选题"            # → 选题研究
"写一篇小红书：干货教程，《用 AI 写周报的 3 个坑》"  # → 内容生产
"完成待发布"                                        # → 配图方案
```

<br>

### 🧩 三个 Skill

| Skill | 职责 | 输入 → 输出 | 写 Notion？ |
|-------|------|-------------|------------|
| `topic-research` | 选题探索 | 方向 → 选题列表 + 可信度 + 来源 | 否 |
| `content-production` | 内容生产 | 选题 + 类型 → 完整笔记（标题、正文、标签） | ✅ |
| `image-planning` | 配图方案 | 笔记内容 → 多图布局 + AI 提示词 | ✅ |

<br>

### 💡 设计理念

| 🛡️ | **编辑守门员，不是模具** — 约束「不能做什么」比「怎么做」更重要：不编造数据、不套模板CTA、不覆盖已有内容 |
|🎨| **边界内自由创作** — Agent 自主决定风格、结构、节奏、图片数量 |
| 🔍 | **信任但核验** — 涉及政策、算法、数据的敏感选题，必须多来源交叉验证 |

<br>

### 🚫 边界：不做什么

- 生成实际图片（→ 输出提示词，配合 ChatGPT Images 2.0 等工具）
- 上传或发布到小红书
- 抓取站内数据
- 填写发布后数据（点赞、收藏等）

工作流终点是 **「待发布」** — 人工最终审核是刻意保留的设计。

<br>

### 📂 仓库结构

```
hermes-xiaohongshu-workflow/
├── README.md
├── WORKFLOW.md              ← 工作流详细文档（中英双语）
├── LICENSE                  ← MIT
└── skills/
    ├── xiaohongshu-topic-research/SKILL.md
    ├── xiaohongshu-content-production/SKILL.md
    └── xiaohongshu-image-planning/SKILL.md
```

<br>

### 📄 许可

MIT · 详见 [LICENSE](./LICENSE)

</details>

<br>

---

<p align="center">
  <sub>Built with <a href="https://hermes-agent.nousresearch.com/">Hermes Agent</a> · For creators who care about quality</sub>
</p>
