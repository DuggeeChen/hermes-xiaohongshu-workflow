# Hermes Xiaohongshu Workflow &middot; 小红书 AI 内容工作流

> **AI-powered end-to-end content pipeline for Xiaohongshu (RED).**
> From a rough topic idea to a publish-ready post with image briefs — all driven by an AI agent that enforces quality, not just speed.
>
> **基于 Hermes Agent 的小红书 AI 内容生产线。**
> 从一个模糊选题到一篇待发布的笔记（含配图方案），全流程 AI 驱动，重质量不重速度。

---

## What Problem Does This Solve?

Writing a good Xiaohongshu post isn't just about typing words. It's a multi-stage process:

1. **Research** — Is this topic worth writing? What angles are trending?
2. **Drafting** — Title (≤20 chars), body (≤1000 chars), CTAs, tags. No fabricated facts.
3. **Image Planning** — Multi-image layouts, visual style, AI image-generation prompts.

Most AI tools stop at "generate a post." This workflow **connects all three stages** through a Notion database, with built-in quality gates at every step.

### Who Is This For?

- Xiaohongshu creators who want consistent quality without burning out
- Content teams that need a repeatable editorial pipeline
- Hermes Agent users looking for a production-grade skill example

---

## Architecture

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│  Topic Research     │     │  Content Production   │     │  Image Planning     │
│  选题探索            │ ──→ │  内容生产              │ ──→ │  配图方案            │
│                     │     │                       │     │                     │
│  • Web search       │     │  • Notion dedup       │     │  • Read post        │
│  • Trend analysis   │     │  • Fact-checking      │     │  • Plan layouts     │
│  • Angle discovery  │     │  • Title + body       │     │  • Generate prompts │
│  • Read-only        │     │  • Tags + CTAs        │     │  • Write to Notion  │
│                     │     │                       │     │                     │
│  Output: research   │     │  Status: 灵感池        │     │  Status: 待配图      │
│  report (console)   │     │       → 待配图         │     │       → 待发布       │
└─────────────────────┘     └──────────────────────┘     └─────────────────────┘
```

All state is tracked in a **Notion database** (`小红书内容工作流`). Each post flows through:

`灵感池 (Idea Pool)` → `待配图 (Ready for Images)` → `待发布 (Ready to Publish)` → `已发布 (Published)`

---

## Quick Start

### Prerequisites

| Requirement | Purpose |
|-------------|---------|
| [Hermes Agent](https://hermes-agent.nousresearch.com/) | AI agent runtime that loads and executes these skills |
| Notion account + API integration | Content database — create/read/update posts |
| Notion MCP tool installed | `notion_search`, `notion_fetch`, `notion_create_pages`, etc. |
| DuckDuckGo search | Topic research (built-in skill) |

### Installation

```bash
# 1. Clone this repo
git clone https://github.com/DuggeeChen/hermes-xiaohongshu-workflow.git

# 2. Copy skills into your Hermes profile
cp -r skills/* ~/AppData/Local/hermes/skills/productivity/

# 3. Set up Notion connection in Hermes
#    (notion_search / notion_fetch / notion_query_data_sources / notion_update_page)
```

### Usage

```
# Start with topic research
"帮我探索一下 AI 工具 方向的小红书选题"

# Pick a topic and create content
"写一篇小红书：干货教程，《用 AI 写周报的 3 个坑》"

# Generate image briefs (auto-picks latest 待配图 post)
"完成待发布"
```

---

## Skills Overview

| Skill | Role | Input | Output | Writes to Notion? |
|-------|------|-------|--------|-------------------|
| **xiaohongshu-topic-research** | Topic discovery | A topic direction (e.g. "AI tools") | Curated topic list with angles, credibility scores, source links | No (read-only) |
| **xiaohongshu-content-production** | Content drafting | Topic + content type | Full post: title, body, CTAs, tags, hashtags | Yes (new record + content) |
| **xiaohongshu-image-planning** | Image brief generation | Post content (from Notion) | Multi-image layout plan + AI prompts per page | Yes (配图方案 field) |

---

## Design Principles

### 🛡️ Editor Gatekeeper, Not Writing Mold

These skills enforce **what not to do** more than **what to do**. They have:

- **Fact floor** — No fabricated data, personal experiences, or platform policy claims
- **Hard checks** — Title ≤ 20 chars, body ≤ 1000 chars, tags must match existing Notion options
- **Dedup guard** — If a topic already has content, the skill stops and reports — no overwrites

### 🎨 Creative Freedom Within Guardrails

Within the hard boundaries, the AI agent has full creative autonomy:

- Title style, body structure, pacing — all agent's choice
- Image count, layout logic, visual direction — decided per post
- No forced templates, no mandatory CTA formats

### 🔍 Trust but Verify

For sensitive topics (policy changes, platform rules, specific data), the skills perform **multi-source fact-checking** before proceeding. Unverifiable claims are downgraded or removed.

---

## Limitations

These skills do **NOT**:

- Generate actual images (they produce prompts for tools like ChatGPT Images 2.0)
- Upload or publish posts to Xiaohongshu
- Scrape Xiaohongshu's platform for internal data
- Fill in post-publication analytics (likes, saves, comments, etc.)

The pipeline stops at **"待发布 (Ready to Publish)"**. The final publish step is manual — by design.

---

## Repository Structure

```
hermes-xiaohongshu-workflow/
├── README.md                          # You are here
├── WORKFLOW.md                        # Detailed pipeline documentation
├── LICENSE                            # MIT
└── skills/
    ├── xiaohongshu-topic-research/
    │   └── SKILL.md                   # Topic discovery skill
    ├── xiaohongshu-content-production/
    │   └── SKILL.md                   # Content drafting skill
    └── xiaohongshu-image-planning/
        └── SKILL.md                   # Image planning skill
```

---

## License

MIT — see [LICENSE](./LICENSE).

---

---

# 中文说明

## 解决什么问题？

写好一篇小红书笔记不是打字那么简单，它是一个多阶段流程：

1. **选题研究** — 这个话题值不值得写？有什么切入角度？
2. **内容生产** — 标题（≤20 字）、正文（≤1000 字）、互动引导、标签。不能编造事实。
3. **配图方案** — 多图布局、视觉风格、AI 生图提示词。

大多数 AI 工具只做到「帮你写一段文案」。本工作流把**三个阶段串联起来**，通过 Notion 数据库追踪状态，每一步都有质量检查。

### 适合谁用？

- 想保持内容质量又不想每次都从零开始的小红书创作者
- 需要可复现编辑流程的内容团队
- 想参考 Hermes Agent 生产级 skill 写法的开发者

---

## 架构

```
┌─────────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│  选题探索            │     │  内容生产              │     │  配图方案            │
│  Topic Research     │ ──→ │  Content Production   │ ──→ │  Image Planning     │
│                     │     │                       │     │                     │
│  • 网络搜索          │     │  • Notion 去重         │     │  • 读取正文         │
│  • 趋势分析          │     │  • 事实核验            │     │  • 规划图片结构      │
│  • 角度挖掘          │     │  • 标题 + 正文         │     │  • 生成提示词        │
│  • 只读不写          │     │  • 标签 + 互动引导      │     │  • 写入 Notion      │
│                     │     │                       │     │                     │
│  输出：选题报告       │     │  状态：灵感池 → 待配图   │     │  状态：待配图 → 待发布 │
└─────────────────────┘     └──────────────────────┘     └─────────────────────┘
```

所有状态追踪在一个 **Notion 数据库**（`小红书内容工作流`）中。每篇笔记经历：

`灵感池` → `待配图` → `待发布` → `已发布`

---

## 快速开始

### 前提条件

| 条件 | 用途 |
|------|------|
| [Hermes Agent](https://hermes-agent.nousresearch.com/) | 加载和执行 skill 的 AI agent 运行时 |
| Notion 账号 + API 集成 | 内容数据库的读写 |
| Notion MCP 工具 | `notion_search`、`notion_fetch`、`notion_create_pages` 等 |
| DuckDuckGo 搜索 | 选题研究（内置 skill） |

### 安装

```bash
# 1. 克隆仓库
git clone https://github.com/DuggeeChen/hermes-xiaohongshu-workflow.git

# 2. 复制 skill 到 Hermes profile 目录
cp -r skills/* ~/AppData/Local/hermes/skills/productivity/

# 3. 在 Hermes 中配置 Notion 连接
```

### 使用方式

```
# 开始选题研究
"帮我探索一下 AI 工具 方向的小红书选题"

# 选定选题，生成内容
"写一篇小红书：干货教程，《用 AI 写周报的 3 个坑》"

# 生成配图方案（自动取最新待配图记录）
"完成待发布"
```

---

## 三个 Skill 概览

| Skill | 职责 | 输入 | 输出 | 写入 Notion？ |
|-------|------|------|------|--------------|
| **xiaohongshu-topic-research** | 选题探索 | 一个方向（如「AI工具」） | 选题列表 + 角度 + 可信度 + 来源 | 否（只读） |
| **xiaohongshu-content-production** | 内容生产 | 选题 + 内容类型 | 完整笔记：标题、正文、互动引导、标签 | 是（新建记录 + 内容） |
| **xiaohongshu-image-planning** | 配图方案 | 笔记内容（从 Notion 读取） | 多图布局方案 + 每页 AI 提示词 | 是（配图方案字段） |

---

## 设计理念

### 🛡️ 编辑守门员，不是写作模具

这些 skill 的核心是**约束而非模板**：

- **事实底线** — 不编造数据、经历、平台政策
- **硬性检查** — 标题 ≤ 20 字、正文 ≤ 1000 字、标签必须匹配数据库已有选项
- **去重保护** — 选题已有内容则停止并报告，不会覆盖

### 🎨 边界内自由创作

在硬性规则之内，AI agent 有完全自主权：

- 标题风格、正文结构、节奏由 agent 决定
- 图片数量、拆分逻辑、视觉方向按每篇内容灵活调整
- 不强制模板、不固定开头结尾

### 🔍 信任但核验

涉及平台政策、算法变化、具体数据等敏感选题时，skill 会执行**多来源事实核验**，无法确认的内容降级或删除。

---

## 边界 — 不做什么

这些 skill **不负责**：

- 生成实际图片（只生成提示词，可配合 ChatGPT Images 2.0 等工具使用）
- 上传或发布到小红书平台
- 抓取小红书站内数据
- 填写发布后的数据表现（点赞、收藏、评论等）

工作流终点是 **「待发布」**，最终发布由人工完成——这是设计的选择，不是功能缺失。

---

## 仓库结构

```
hermes-xiaohongshu-workflow/
├── README.md                          # 你在看这里
├── WORKFLOW.md                        # 工作流详细文档
├── LICENSE                            # MIT
└── skills/
    ├── xiaohongshu-topic-research/
    │   └── SKILL.md                   # 选题探索 skill
    ├── xiaohongshu-content-production/
    │   └── SKILL.md                   # 内容生产 skill
    └── xiaohongshu-image-planning/
        └── SKILL.md                   # 配图方案 skill
```

---

## 许可

MIT — 详见 [LICENSE](./LICENSE)。
