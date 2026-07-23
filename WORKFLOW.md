# Workflow Deep Dive &middot; 工作流详解

> How the three skills connect, what happens at each stage, and why the pipeline is designed this way.
>
> 三个 skill 如何串联、每个阶段做什么、为什么这样设计。

---

## Pipeline Overview

```
USER INPUT                 SKILL                      NOTION STATE           OUTPUT
───────────                ─────                      ────────────           ──────

"探索 AI 工具选题"  ──→  topic-research  ──→  (no write)           ──→  Research report
                              │                                          (console only)
                              ▼
"写一篇：干货教程，        content-production  ──→  灵感池 → 待配图      ──→  Full post in
 《用 AI 写周报的 3 个坑》"       │                                      Notion
                              ▼
"完成待发布"        ──→  image-planning     ──→  待配图（写入方案）      ──→  Image briefs
                                                                          in Notion
```

---

## Stage 1: Topic Research (选题探索)

**Skill**: `xiaohongshu-topic-research`
**Notion**: Read-only — does not write anything

### What it does

Given a broad direction (e.g., "AI tools", "productivity hacks"), it:

1. Searches the web for current discussions, trends, and angles
2. Categorizes results into **trending topics** and **evergreen topics**
3. Assesses credibility (verified / reliable source / synthetic / unverified)
4. Recommends which topics to write first

### Quality gates

- **Sensitive topics** (platform policy, algorithm changes, specific data) → requires ≥2 independent sources
- **Unverifiable claims** → downgraded or removed
- **404/anti-scraped pages** → not counted as valid sources

### Design rationale

Topic research is deliberately **decoupled** from content production. This lets you:

- Brainstorm dozens of topics in one session without committing
- Share research results with a team before writing
- Keep the Notion database clean (no half-baked records)

### Trigger examples

```
"帮我探索一下 AI 工具 方向的小红书选题"
"深度研究：小红书最新算法变化"
"找几个 效率提升 方向的常青选题"
```

---

## Stage 2: Content Production (内容生产)

**Skill**: `xiaohongshu-content-production`
**Notion**: Creates or updates records

### What it does

Given a **topic** + **content type** (one of: `干货教程` / `经验分享` / `产品种草` / `观点讨论` / `生活记录`), it:

1. Searches the Notion database for duplicate topics
2. If new: creates a record with status `灵感池`
3. If existing but empty: reuses the record to fill content
4. If existing with content: **stops** — no overwrite
5. (Conditional) Fact-checks sensitive claims via web search
6. Generates: title, body, CTAs, tags, hashtags
7. Runs a **pre-write review checklist** (6 items)
8. Writes everything to Notion
9. Advances status: `灵感池` → `待配图`

### Hard constraints (violated → stop, don't write)

| # | Check | How |
|---|-------|-----|
| 1 | Title ≤ 20 characters | Count every character (Chinese, English, digits, punctuation — all count as 1) |
| 2 | Body ≤ 1000 characters | Count total characters |
| 3 | No fabricated data | Scan for `\d+%`, `\d+ 万` patterns; ask "does this have a source?" |
| 4 | No fake personal experience | Scan for first-person + specific event combos |
| 5 | No platform policy claims | Scan for "小红书官方", "根据政策", "新规" |
| 6 | Tags match existing options | Cross-check against Notion database schema |

### Creative freedom

Within these constraints, the AI agent decides:

- Title angle and phrasing
- Body structure (story, list, Q&A, opinion, tutorial, comparison, experience)
- Pacing and tone (varies by content type)
- CTA approach (no template phrases like "你怎么看？评论区聊聊")

### Content types and tones

| Type | Tone |
|------|------|
| 干货教程 (Tutorial) | Clear, direct |
| 经验分享 (Experience) | Natural, authentic |
| 产品种草 (Recommendation) | Specific, restrained |
| 观点讨论 (Opinion) | Takes a stance, not extreme |
| 生活记录 (Lifestyle) | Narrative feel |

### Trigger examples

```
"写一篇小红书：干货教程，《用 AI 写周报的 3 个坑》"
"小红书选题：「Notion 搭建内容管理系统」— 经验分享"
```

---

## Stage 3: Image Planning (配图方案)

**Skill**: `xiaohongshu-image-planning`
**Notion**: Reads post content, writes image briefs

### What it does

1. Finds the target post (by topic/title, or auto-picks latest `待配图`)
2. Reads: topic, title, body, content type
3. Analyzes content structure and core proposition
4. Plans image count (flexible, content-driven, usually 4–6)
5. Independently designs adaptive visual direction (no preset account color palettes)
6. Compiles prompts: Session Prompt + lean per-page Page Prompts
7. Runs a **quality check** (5 items)
8. Writes the full plan to Notion's `配图方案` field
9. Status remains unchanged (no auto-advance to `待发布`)

### Adaptive visual direction

Visual direction is now content-driven rather than preset by account pillars. The skill independently decides:

- Visual language: photography, illustration, collage, editorial design, conceptual, infographic, or mixed
- Color system, materials/textures, lighting, spatial mood
- Graphic language, typographic character, visual metaphors, page rhythm

Once a visual system is chosen for a post, subsequent pages maintain consistency — but no two pages mechanically copy the same composition.

### Prompt architecture

Two-layer output:
- **Session Prompt** (sent once) — theme, proposition, desired reader response, page sequence overview, visual direction, global rules, constraint priorities, creative mandate
- **Page Prompts** (one per page) — only page-specific info: task, exact text, visual intent, special constraints (if any). No repeated global rules.

Constraint priorities (4 tiers, in Session Prompt only):
1. Specified text and factual accuracy
2. Clear page-level core message
3. Visual consistency across pages + mobile readability
4. Creative, aesthetic, and decorative details

### Quality check (before writing to Notion)

1. Is each page from the current post and serving an independent, necessary role?
2. Is there progressive relationship between pages, with no information duplication?
3. Is each page's specified text clear, accurate, and reasonable in quantity?
4. Is the visual system consistent while avoiding mechanical composition copy?
5. Is the final Prompt concise, directly usable, and leaving adequate creative space?

Three fatal issues (re-compress Prompt if found):
- Prompt reads like a design spec checklist, not a creative brief
- Prompt lists elements without explaining what the page should convey
- Rules are so dense that ChatGPT has almost no visual decision space

### Prompt writing for AI image tools

Same as before — prompts in natural Chinese for ChatGPT Images 2.0, using `「」` for verbatim text, specifying layout intent while accepting approximate rendering.

### Trigger examples

```
"完成待发布"
"配图"
"给《用 AI 写周报的 3 个坑》做配图方案"
```

---

## Error Handling Across the Pipeline

| Scenario | Behavior |
|----------|----------|
| Notion database not found | Stop, prompt user to check connection |
| Field name/type mismatch | Stop, report the mismatch |
| Notion connection failed | Stop, suggest network/Token check |
| Duplicate topic with content | Stop, show existing record — no overwrite |
| Duplicate topic, empty content | Reuse record, fill content normally |
| Unverifiable high-risk fact | Stop, flag for human review |
| Content write succeeds, status update fails | Report: content saved, status needs manual update |
| Image plan generation succeeds, Notion write fails | Report: plan text available in console, manual write needed |

### Core principle

> Never pretend a failure succeeded. Never create a half-finished record without disclosure. Never modify any other existing record.

---

## Design Decisions

### Why Notion?

- **Structured state tracking** — `灵感池 → 待配图 → 待发布 → 已发布` is clearer than a flat file
- **Multi-field records** — separate fields for title, body, tags, image briefs, analytics — each stage writes to different fields
- **Human-readable** — at any point, a human can open Notion and see exactly where each post is

### Why decouple research from production?

Research is exploratory and high-volume. Production is committed and high-effort. Mixing them would:
- Clutter the database with abandoned ideas
- Make the production skill do search work it's not designed for
- Prevent batch research sessions (find 10 topics, write 3)

### Why manual final publish?

The `待发布 → 已发布` transition is intentionally left manual:
- Xiaohongshu has no public write API
- Image generation is done in a separate tool (ChatGPT Images 2.0)
- Final review by a human before hitting "publish" is a feature, not a bug

---

## Prerequisites Setup Guide

### Notion

1. Create a Notion integration at https://www.notion.so/my-integrations
2. Create a database named `小红书内容工作流` with these fields:
   - `选题` (title), `内容类型` (select), `状态` (select)
   - `标题` (text), `正文` (text), `互动引导` (text)
   - `标签` (multi_select), `发布话题标签` (text), `配图方案` (text)
   - `核心角度` (text) — optional, for editorial guidance
3. Connect the integration to the database (Share → Invite)
4. Configure the Notion MCP tool in Hermes

### Hermes Agent

```bash
# Ensure Notion tools are available
hermes tools list | grep notion

# Load the skills
hermes skills load xiaohongshu-topic-research
hermes skills load xiaohongshu-content-production
hermes skills load xiaohongshu-image-planning
```

---

---

# 中文版

## Pipeline 总览

```
用户输入                    SKILL                     Notion 状态            输出
────────                   ─────                     ───────────            ────

"探索 AI 工具选题"  ──→  topic-research  ──→  (不写 Notion)        ──→  选题报告
                              │                                        (仅终端)
                              ▼
"写一篇：干货教程，        content-production  ──→  灵感池 → 待配图     ──→  完整笔记
 《用 AI 写周报的 3 个坑》"       │                                    写入 Notion
                              ▼
"完成待发布"        ──→  image-planning     ──→  待配图（写入方案）     ──→  配图方案
                                                                        写入 Notion
```

---

## 第一阶段：选题探索

**Skill**: `xiaohongshu-topic-research`
**Notion**: 只读，不写入

### 做什么

给定一个方向（如「AI工具」「效率提升」），它：

1. 搜索网络上的当前讨论、趋势、切入角度
2. 分为**近期话题**和**常青选题**
3. 评估可信度（已核验 / 有可靠来源 / 基于公开信号合成 / 待验证）
4. 推荐优先创作的选题

### 质量关卡

- **敏感选题**（平台政策、算法变化、具体数据）→ 要求 ≥2 个独立来源
- **无法核验** → 降级或删除
- **404/反扒页面** → 不算有效来源

### 设计理由

选题探索和内容生产刻意解耦，好处：

- 一次探索几十个选题，不用每个都建 Notion 记录
- 选题报告可以分享给团队讨论后再写
- 保持 Notion 数据库干净（没有半成品记录）

---

## 第二阶段：内容生产

**Skill**: `xiaohongshu-content-production`
**Notion**: 创建或更新记录

### 做什么

给定**选题 + 内容类型**（干货教程 / 经验分享 / 产品种草 / 观点讨论 / 生活记录）：

1. 在 Notion 中搜索去重
2. 新选题 → 创建记录，状态 `灵感池`
3. 已有但内容为空 → 复用记录填充
4. 已有且有内容 → **停止**，不覆盖
5. （条件触发）敏感选题进行事实核验
6. 生成：标题、正文、互动引导、标签、发布话题标签
7. 执行**写入前复查清单**（6 项）
8. 写入 Notion
9. 状态推进：`灵感池` → `待配图`

### 硬性约束

详见上方英文版第 6 项检查表。

### 创作自由度

在约束范围内，AI agent 自主决定标题角度、正文结构、节奏语气、互动方式。

---

## 第三阶段：配图方案

**Skill**: `xiaohongshu-image-planning`
**Notion**: 读取笔记内容，写入配图方案

### 自适应视觉方向

视觉方向由内容决定，不再按账号预设配色。Agent 自主选择摄影、插画、拼贴、编辑设计、概念视觉、信息图形或混合风格，以及色彩、材质、光线、图形语言等。一旦确定整组视觉系统，后续页面保持一致但不机械复制。

### Prompt 架构

两层输出：
- **Session Prompt**（发送一次）— 主题、核心命题、读者目标反应、页面序列概览、视觉方向、全局规则、约束优先级、创作授权
- **Page Prompts**（每页一个）— 仅当前页面信息：任务、准确文字、视觉意图、特殊约束（如有）。不重复全局规则。

### 质量检查

写入前检查 5 项：内容来源、信息递进、文字准确性、视觉一致性、Prompt 简洁度。详见上方英文版。

---

## 设计决策

### 为什么用 Notion？

- 结构化状态追踪，比 flat file 清晰
- 多字段记录，不同阶段写不同字段
- 任何阶段人都能打开 Notion 看到进度

### 为什么研究和生产解耦？

- 研究是高量探索，生产是高投入承诺
- 混在一起会导致数据库被废弃选题污染
- 允许批量研究 → 挑选 → 逐个生产

### 为什么最终发布是手动的？

- 小红书无公开写入 API
- 图片在独立工具中生成
- 发布前人工最终审核是特性不是缺陷

---

## 常见问题

**Q: 我没有 Notion，能用吗？**
A: 目前不行。Notion 是整个工作流的状态存储。如果你用其他工具（飞书多维表格、Airtable），核心逻辑类似但需要适配字段。

**Q: 能直接生成图片吗？**
A: 不能。配图 skill 生成的是 AI 绘图工具的提示词，你需要复制到 ChatGPT Images 2.0 等工具中生成实际图片。

**Q: 标题 20 字限制能放宽吗？**
A: 这是 SKILL.md 中的硬性约束，编辑源码中的检查项即可。

**Q: 为什么中文 skill 不翻译成英文？**
A: SKILL.md 是 Hermes Agent 的运行时代码（prompt），保持中文确保运行时效果不变。英文说明以摘要块的形式补充在文件头部。
