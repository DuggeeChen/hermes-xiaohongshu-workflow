<p align="center">
  <img src="https://img.shields.io/badge/Hermes-Agent-6C5CE7?style=for-the-badge&logo=robotframework&logoColor=white">
  <img src="https://img.shields.io/badge/Platform-小红书-FF2442?style=for-the-badge">
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge">
</p>

<h1 align="center">📕 Hermes 小红书 Workflow</h1>

<p align="center">
  <i>AI-powered end-to-end content pipeline for Xiaohongshu (小红书 / RED)<br>
  AI 驱动的小红书内容生产线 — 从灵感到待发布，全流程自动化</i>
</p>

<br>

<table align="center"><tr>
<td align="center" width="260">

[<big>**🇨🇳 &nbsp;中文**</big>](#chinese)

</td>
<td align="center" width="260">

[<big>**🇬🇧 &nbsp;English**</big>](#english)

</td>
</tr></table>

<br>

---

<div id="chinese">

<details open>
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

### 👥 适合谁用

| 人群 | 场景 |
|------|------|
| 小红书创作者 | 想保持内容质量又不想每次都从零开始 |
| 内容团队 | 需要可复现的编辑流程，新人上手快 |
| AI Agent 开发者 | 想参考生产级 Skill 的写法和质量门控设计 |

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

### 🔄 状态流转

每篇笔记在 Notion 中经历四个状态：

```
灵感池 → 待配图 → 待发布 → 已发布
```

| 状态 | 说明 | 谁来执行 |
|------|------|---------|
| 灵感池 | 只有选题，尚未生成内容 | 用户或 topic-research |
| 待配图 | 内容已完成，需要配图方案 | content-production 自动推进 |
| 待发布 | 配图方案已完成，可人工复审发布 | image-planning 自动推进 |
| 已发布 | 发布完成，待填写数据表现 | 用户手动 |

<br>

### 💡 设计理念

| 🛡️ | **编辑守门员，不是模具** — 约束「不能做什么」比「怎么做」更重要：不编造数据、不套模板CTA、不覆盖已有内容 |
|🎨| **边界内自由创作** — Agent 自主决定风格、结构、节奏、图片数量 |
| 🔍 | **信任但核验** — 涉及政策、算法、数据的敏感选题，必须多来源交叉验证 |

<br>

### 🛠️ 技术栈

| 层级 | 工具 / 平台 | 作用 |
|------|-----------|------|
| AI Agent | [Hermes Agent](https://hermes-agent.nousresearch.com/) | 运行时引擎，加载并执行 Skill |
| 数据库 | Notion + MCP | 内容管理、状态追踪、多字段记录 |
| 搜索 | DuckDuckGo / ddgs | 选题研究、事实核验 |
| 图片生成 | ChatGPT Images 2.0 | 根据配图方案生成实际图片 |

<br>

### ❌ 错误处理

| 场景 | 处理方式 |
|------|--------|
| 找不到 Notion 数据库 | 立即停止，提示检查连接和 Token |
| 选题已存在且有内容 | 不覆盖，返回已有记录链接 |
| 无法核验敏感事实 | 停止并标注「需人工确认」，不猜测 |
| 写入成功但状态更新失败 | 告知内容已保存，状态需手动更新 |

<br>

### 🚫 边界：不做什么

- 生成实际图片（→ 输出提示词，配合 ChatGPT Images 2.0 等工具）
- 上传或发布到小红书
- 抓取站内数据
- 填写发布后数据（点赞、收藏等）

工作流终点是 **「待发布」** — 人工最终审核是刻意保留的设计。

<br>

### 📚 每个 Skill 详解

#### `topic-research`

| 项目 | 说明 |
|------|------|
| **触发词** | 「帮我探索选题」「找一下方向」「深度研究」 |
| **输入** | 一个主题方向（如「AI 工具」「效率提升」） |
| **输出** | 近期话题 + 常青选题，含推荐理由、可信度、来源链接 |
| **模式** | 默认轻量探索；说「深度研究」触发严格核验模式 |
| **写 Notion?** | ❌ 只读，不写入数据库 |

#### `content-production`

| 项目 | 说明 |
|------|------|
| **触发词** | 「写一篇小红书」「小红书选题」「内容工作流」「新建选题」 |
| **输入** | 选题 + 内容类型（干货教程/经验分享/产品种草/观点讨论/生活记录） |
| **输出** | 完整笔记：标题（≤20 字）、正文（≤1000 字）、互动引导、标签、话题标签 |
| **质量门** | 6 项写入前复查：标题字数、正文字数、无捏造数据、无伪造经历、无政策断言、标签匹配 |
| **写 Notion?** | ✅ 新建记录，状态设为「待配图」 |

#### `image-planning`

| 项目 | 说明 |
|------|------|
| **触发词** | 「完成待发布」「配图」「出图」 |
| **输入** | Notion 中「待配图」状态的笔记（自动取最新） |
| **输出** | 多图布局方案：每页作用、展示文案、构图建议、AI 生图提示词 |
| **模式** | A·氛围引导型（观点/情绪，留白大） vs B·信息结构型（干货/清单，编号+色块） |
| **写 Notion?** | ✅ 写入「配图方案」字段，状态设为「待发布」 |

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

</div>

<div id="english">

<details>
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

### 👥 Who Is This For

| Audience | Use Case |
|----------|----------|
| Xiaohongshu creators | Want consistent quality without starting from scratch every time |
| Content teams | Need a repeatable editorial pipeline that new members can pick up quickly |
| AI Agent developers | Want a production-grade Skill example with quality gates and error handling |

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

### 🔄 State Machine

Each post moves through four states in Notion:

```
灵感池 → 待配图 → 待发布 → 已发布
```

| State | Description | Who advances it |
|-------|-------------|----------------|
| 灵感池 (Idea Pool) | Topic only, no content yet | User or topic-research |
| 待配图 (Ready for Images) | Content complete, needs image plan | content-production auto-advances |
| 待发布 (Ready to Publish) | Image plan complete, human review ready | image-planning auto-advances |
| 已发布 (Published) | Published, awaiting analytics | User manually |

<br>

### 💡 Design Philosophy

| 🛡️ | **Gatekeeper, not mold** — Enforces what *not* to do: no fake data, no template CTAs, no overwriting existing content |
|🎨| **Freedom within guardrails** — Agent decides style, structure, pacing within hard boundaries |
| 🔍 | **Trust but verify** — Sensitive claims require ≥2 independent sources before proceeding |

<br>

### 🛠️ Tech Stack

| Layer | Tool / Platform | Role |
|-------|-----------------|------|
| AI Agent | [Hermes Agent](https://hermes-agent.nousresearch.com/) | Runtime engine — loads and executes skills |
| Database | Notion + MCP | Content management, state tracking, multi-field records |
| Search | DuckDuckGo / ddgs | Topic research and fact-checking |
| Image Gen | ChatGPT Images 2.0 | Generates actual images from image briefs |

<br>

### ❌ Error Handling

| Scenario | Behavior |
|----------|----------|
| Notion database not found | Stop immediately, prompt to check connection and token |
| Topic exists with content | Do not overwrite; return link to existing record |
| Unverifiable sensitive fact | Stop and flag "needs human review"; no guessing |
| Write succeeded but status update failed | Inform user content is saved; status needs manual update |

<br>

### 🚫 What These Skills Do NOT Do

- Generate actual images (→ output prompts for ChatGPT Images 2.0, Midjourney, etc.)
- Upload or publish to Xiaohongshu
- Scrape platform data
- Fill post-publication analytics

The pipeline stops at **"待发布 (Ready to Publish)"** — manual final review is by design.

<br>

### 📚 Skill Breakdown

#### `topic-research`

| Item | Description |
|------|-------------|
| **Triggers** | "Explore topics for me", "Find directions", "Deep research" |
| **Input** | A topic direction (e.g., "AI tools", "productivity hacks") |
| **Output** | Trending + evergreen topics with rationale, credibility, source links |
| **Modes** | Lightweight by default; "deep research" triggers strict verification |
| **Writes Notion?** | ❌ Read-only |

#### `content-production`

| Item | Description |
|------|-------------|
| **Triggers** | "Write a Xiaohongshu post", "New topic", "Content workflow" |
| **Input** | Topic + content type (tutorial / experience / recommendation / opinion / lifestyle) |
| **Output** | Full post: title (≤20 chars), body (≤1000 chars), CTA, tags, hashtags |
| **Quality Gates** | 6-item pre-write review: title length, body length, no fabricated data, no fake experiences, no policy claims, tag validation |
| **Writes Notion?** | ✅ Creates record, sets status to 待配图 |

#### `image-planning`

| Item | Description |
|------|-------------|
| **Triggers** | "Finish ready to publish", "Image plan", "Generate images" |
| **Input** | Post with status 待配图 in Notion (auto-picks latest) |
| **Output** | Multi-image layout: per-page role, copy, composition, AI generation prompt |
| **Modes** | A: Atmosphere-driven (opinion/emotion, heavy whitespace) vs B: Info-dense (tutorial/checklist, numbered + color blocks) |
| **Writes Notion?** | ✅ Writes to 配图方案 field, sets status to 待发布 |

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

</div>

<br>

---

<p align="center">
  <sub>Built with <a href="https://hermes-agent.nousresearch.com/">Hermes Agent</a> · For creators who care about quality</sub>
</p>

