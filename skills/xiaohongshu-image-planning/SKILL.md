---
name: xiaohongshu-image-planning
description: |
  EN: Xiaohongshu (RED) image planning — reads a post from Notion, analyzes content structure, plans multi-image layouts (typically 4–6 images), designs visual direction (color palette, mood, consistency anchor), generates AI image-generation prompts per page for tools like ChatGPT Images 2.0, writes the full plan back to Notion, and advances status from 待配图 to 待发布. Does NOT generate actual images, rewrite posts, or publish.
  CN: 小红书多图配图方案生成器：读取 Notion 内容记录，生成视觉策划和 AI 生图提示词。职责：读取 Notion → 分析内容 → 规划图片结构 → 生成提示词 → 写入「配图方案」 → 状态改为「待发布」。不负责重写正文、修改标题、生成/上传图片、发布。适配 ChatGPT Images 2.0 及同类中文生图工具，提示词用自然中文撰写。
version: 2.2.0
metadata:
  hermes:
    tags: [xiaohongshu, image-planning, notion, 小红书, 配图, 视觉策划, chatgpt]
    related_skills: [xiaohongshu-content-production, notion]
---

<!--
  ═══════════════════════════════════════════════════════════════
  ENGLISH SUMMARY
  ═══════════════════════════════════════════════════════════════
  This skill is the final stage of the Xiaohongshu content pipeline.
  It takes a drafted post (status: 待配图) and produces a complete
  image brief with AI generation prompts for each page.

  Key behaviors:
  • Reads post content from Notion (topic, title, body, content type)
  • Plans image count based on content (4–6, flexible)
  • Two density modes (mixable within one post):
    - Mode A (atmosphere-driven): big concept, heavy whitespace, mood
    - Mode B (information-dense): numbered lists, icons+labels, color blocks
  • Account visual system with 3 content pillars:
    1. Workflow/Methods   → calm, professional (deep blue-grey, cool white)
    2. Craft/Handmade     → warm, artisanal (cream, charcoal, coral orange)
    3. Opinion/Attitude   → sharp, contrasting (deep blue-grey, warm yellow)
  • Anti-laziness checklist: 4 items to prevent template thinking
  • Status flow: 待配图 → 待发布

  Prompt writing conventions:
  • Natural Chinese (not translated English)
  • Image text in 「」 with "逐字准确" (verbatim requirement)
  • "不要其他文字、不要英文" (no extra text, no English)
  • Layout hints are suggestions, not pixel-precise specs

  ⚠️  This is a Hermes Agent SKILL.md — the Chinese text below is
     the actual runtime prompt. The English here is supplementary
     documentation for open-source readers.
  ═══════════════════════════════════════════════════════════════
-->

# 小红书多图配图方案 <!-- EN: Xiaohongshu Multi-Image Planning -->

> 视觉策划助手，不是固定模板机器。 <!-- EN: Visual planning assistant, not a template machine. -->

## 触发条件 <!-- EN: Trigger Conditions -->

用户提供以下之一，否则默认取最新「待配图」记录：
- **选题**（`选题` 字段值）
- **标题**（`标题` 字段值）

## 目标数据库 <!-- EN: Target Database -->

Notion 数据库：**小红书内容工作流**。数据库 ID / 数据源 ID 动态获取，不硬编码。

## 执行流程 <!-- EN: Execution Flow -->

### Step 1：定位记录 <!-- EN: Locate Record -->

```
notion_search(query="小红书内容工作流", query_type="internal")
→ database_id

notion_fetch(id="<database_id>")
→ data_source_id（collection://...）
```

按选题或标题查询，或取最新待配图：

```sql
SELECT * FROM "collection://<data_source_id>"
WHERE "选题" = '<选题>' OR "标题" = '<标题>'

-- 或默认：
SELECT * FROM "collection://<data_source_id>"
WHERE "状态" = '待配图'
ORDER BY createdTime DESC
LIMIT 1
```

未找到 → 提示并停止。

### Step 2：提取内容 <!-- EN: Extract Content -->

读取字段及用途：

| 字段 | 用途 |
|------|------|
| `选题` | 理解整体主题 |
| `标题` | 理解封面主题 |
| `正文` | 拆分多图内容的核心依据 |
| `内容类型` | 判断表达方式和视觉风格方向 |
| `封面提示词` | 仅作封面视觉方向参考，不要求沿用，不限制后续页面创意 |

### Step 3：分析内容 <!-- EN: Analyze Content -->

理解正文的核心观点、信息层次和阅读节奏。

### Step 4：规划图片结构 <!-- EN: Plan Image Structure -->

- 根据内容自然判断图片数量，通常 4~6 张；内容简单时可更少，不凑数
- 第 1 张为封面
- 后续按正文自然段落拆分，每页只表达一个主要信息
- 不使用固定模板，拆分逻辑和页面顺序由内容决定
- 参考但不限于：痛点、背景、流程、方法、对比、案例、注意事项、总结、互动

**先判断内容类型属于哪种密度模式（见 Step 6「两种信息密度模式」），再决定每页的文字量和版式，不要不管内容类型都套用同一套"标题+一句话"模板。**

- 模式A·氛围引导型 —— 适用：观点类、情绪类、悬念类、系列衔接页（详见 Step 6）
- 模式B·信息结构型 —— 适用：干货教程、清单、步骤流程、对比、工作流类（详见 Step 6）

同一篇笔记内，六张图可以混用两种模式（封面/收尾用模式A留白，中间干货页用模式B铺信息），不强制全篇统一密度。

### Step 5：设计视觉方向 <!-- EN: Design Visual Direction -->

- 风格与内容类型匹配
- 配色 2~3 主色 + 背景色
- 统一视觉锚点贯穿所有页面（如统一色温、统一线条元素、统一画面质感）
- 每页构图允许变化，保持生动

#### 账号视觉系统（内容支柱分类） <!-- EN: Account Visual System (Content Pillars) -->

本账号有三条内容支柱，各有情绪锚点和视觉方向，供配图时参考，不是精确色值表：

| 支柱 | 情绪锚点 | 视觉方向 |
|------|----------|----------|
| 工作流方法类 | 冷静专业 | 深蓝灰、冷白等 |
| 去味手艺类 | 温度手艺感 | 奶油白、炭黑、珊瑙橙等 |
| 观点态度类 | 犀利有对比 | 深蓝灰、暖黄等 |

硬性要求（必须执行）：
1. 左上角小标不能省 <!-- EN: top-left badge always required -->
2. 如果是系列笔记，加一个系列小尾标（如 "1/3"） <!-- EN: series badge for multi-part posts -->

深浅、构图、留白、文字量、辅助色全部自主判断——只要还在这个情绪基调的大方向里，怎么好看怎么来，别为了"跟上次色值一致"束手束脚。

### Step 6：生成提示词 <!-- EN: Generate Prompts -->

#### 提示词撰写规范 <!-- EN: Prompt Writing Conventions -->

面向 ChatGPT Images 2.0 等中文生图工具，遵循以下原则：

**1. 用自然中文直接描述**
- 不需要先翻译成英文
- 可理解：日常中文、网络用语、抽象氛围（"潮湿、克制、疏离"）、摄影语言（"85mm 人像、浅景深、逆光"）、设计语言（"新中式、赛博东方、极简留白"）
- 可表达：人物位置、动作、光线、前后景、镜头角度、否定条件（"不要塑料皮肤、不要多余手指"）

**2. 文字要求**
- 图片中的文字放中文引号里，明确写「逐字准确」
- 限制额外文字，写明「不要其他文字、不要英文」
- 可以有多行文字，但要知道排版是近似的（居中、上下两行、左对齐可以要求，但字距行距不像专业设计软件那样精确）
- 氛围型内容图片文字少而精；信息结构型内容可以多文字块并列，但每个独立块仍建议 ≤15 字避免糊字
**3. 内容密度（按内容类型分两种模式）** <!-- EN: Content Density (Two Modes by Content Type) -->

不再一刀切"图片只放标题/金句/引导"，按内容类型选择模式：

- 模式A（氛围引导型）：一页金句/概念，留白大，氛围带情绪 <!-- EN: Mode A: Atmosphere-driven — one concept per page, heavy whitespace, mood -->
- 模式B（信息结构型）：允许编号列表、多标签并列、图标+短词、简易流程图/对比表，单张图 50~80 字可以，靠编号/图标/色块切分 <!-- EN: Mode B: Information-dense — numbered lists, icon+label, flowcharts, 50–80 chars/page -->

画面描述策略：
- 氛围型内容多用视觉隐喻和氛围（光线、色温、材质、情绪）
- 信息结构型内容也可以用功能性版式词（编号列表、图标+标签、色块分区、判断分支箭头），不必强行套氛围隐喻

#### 反偷懒质检清单（写入 Notion 前必做） <!-- EN: Anti-Laziness Checklist (Before Writing to Notion) -->

完成方案后、写入 Notion 前，逐页自查以下四条，任何一条不通过则回到 Step 4 重新规划，不带问题写入：

- [ ] 这页的文案是不是从**这篇稿子的具体内容**提炼出来的？不是套用别的选题的句式换几个词
- [ ] 6 张图连起来看，有没有信息递进关系？不是在换着花样复读同一句话
- [ ] 如果这篇是干货/清单/流程类，有没有把具体的干货点铺进画面里，而不是只放了一个标题糊弄过去
- [ ] 每页提示词是不是都不一样——即使视觉锚点（色调/质感）相同，构图和文字量也要按这一页的信息量走，不是复制上一页只换文字

#### 输出结构 <!-- EN: Output Structure -->

```
📸 小红书配图方案

选题：《<选题>》
内容类型：<内容类型>
建议图片总数：<N> 张

━━━ 🎨 统一视觉锚点 ━━━
（控制整体风格一致性：色调、质感、画面比例、统一元素）

━━━ 📄 第 1 页：封面 ━━━
作用：...
展示文案：（计划出现在画面上的文字）
构图/氛围：...
完整提示词：（直接复制给 ChatGPT）

（重复第 2~N 页）
```

文字原则：手机端清晰可读。氛围引导型内容优先标题和金句，信息结构型内容按模式B铺开具体干货点。

### Step 7：直接写入 <!-- EN: Write Directly (No Confirmation) -->

汇报完方案后直接写入 `配图方案` 字段，不需要等用户确认。包含：

- 图片总数
- 每页作用
- 每页展示文案
- 每页构图/氛围建议
- 统一视觉锚点
- 每页完整提示词

```json
{
"page_id": "<page_id>",
"command": "update_properties",
"properties": {
  "配图方案": "<完整方案>"
}
}
```

写入后简短确认。

### Step 8：状态 →「待发布」 <!-- EN: Status → Ready to Publish -->

```json
{
  "page_id": "<page_id>",
  "command": "update_properties",
  "properties": {"状态": "待发布"}
}
```

## 错误处理 <!-- EN: Error Handling -->

| 错误 | 处理 |
|------|------|
| 找不到数据库 | 提示检查 Notion 连接，停止 |
| 找不到指定记录 | 提示未匹配，停止 |
| 无待配图记录 | 提示，停止 |
| 正文为空/太短 | 提示内容不足，停止 |
| 字段名不匹配 | 记录差异，停止 |
| Notion 写入失败 | 告知原因，已输出方案文本仍可用 |

## 事实底线 <!-- EN: Fact Floor -->

禁止在画面描述中编造正文不存在的经历、数据或效果。信息不足时用图形化表达或提出简短补充问题。

## 创作自由 <!-- EN: Creative Freedom -->

Hermes 可自主决定：
- 图片数量（以内容自然分段为准，4~6 张，简单内容可更少）
- 拆分逻辑和页面顺序
- 每页构图方式（居中、上下、左右、卡片、网格等）
- 视觉风格、配色、画面质感
- 叙事节奏和页面间视觉过渡
- 提示词措辞和细节
- 每页走模式A还是模式B，以及每页的信息密度

## 边界 <!-- EN: Boundaries / What This Skill Does NOT Do -->

不负责：重写正文、修改标题、生成/上传图片、发布小红书、修改其他记录。

## 依赖 <!-- EN: Dependencies -->

- `notion` MCP（notion_search / notion_fetch / notion_query_data_sources / notion_update_page）
