---
name: xiaohongshu-content-production
description: |
  EN: Xiaohongshu (RED) content drafting — takes a topic + content type, checks Notion for duplicates, optionally fact-checks sensitive claims, generates a full post (title ≤20 chars, body ≤1000 chars, CTAs, tags, hashtags), writes back to Notion, runs a 6-item pre-write review, and advances status from 灵感池 to 待配图. Does NOT generate images, upload, or publish.
  CN: 小红书内容生产工作流：接收选题和内容类型，在 Notion「小红书内容工作流」数据库中创建完整记录。职责：选题去重 → 新建记录 → 生成标题、正文、互动引导、标签 → 写回 Notion → 状态更新为「待配图」。不负责图片生成、图片上传、发布、数据回填。触发词：「小红书选题」「写一篇小红书」「内容工作流」「新建选题」。
version: 2.0.0
metadata:
  hermes:
    tags: [xiaohongshu, content-production, notion, workflow, 小红书, 内容创作]
    related_skills: [notion]
---

<!--
  ═══════════════════════════════════════════════════════════════
  ENGLISH SUMMARY
  ═══════════════════════════════════════════════════════════════
  This skill is the content drafting stage of the Xiaohongshu
  pipeline. It bridges topic research and image planning.

  Key behaviors:
  • Dedup: won't overwrite existing content — stops and reports
  • Fact-check: verifies sensitive claims (policy, data, events)
  • Review: runs a mandatory 6-item checklist before writing
  • Status flow: 灵感池 → 待配图

  Hard constraints (failure → stop, don't write):
  1. Title ≤ 20 characters (every char counts, including punctuation)
  2. Body  ≤ 1000 characters
  3. No fabricated statistics or percentages
  4. No fake personal experiences
  5. No platform policy assertions without verification
  6. Tags must match Notion database's existing options

  Between these constraints, the AI agent has full creative freedom
  over title angle, body structure, tone, pacing, and CTAs.

  ⚠️  This is a Hermes Agent SKILL.md — the Chinese text below is
     the actual runtime prompt. The English here is supplementary
     documentation for open-source readers.
  ═══════════════════════════════════════════════════════════════
-->

# 小红书内容生产工作流 <!-- EN: Xiaohongshu Content Production Workflow -->

> 编辑守门员，不是写作模具。 <!-- EN: Editor gatekeeper, not a writing mold. -->

## 触发条件 <!-- EN: Trigger Conditions -->

用户必须同时提供：

- **选题** — 要写的笔记主题。也可以是已有记录（灵感池或内容为空的待配图），此时只需提供选题即可触发内容填充，内容类型可选（已有则沿用）
- **内容类型** — 必须是以下之一：`干货教程` / `经验分享` / `产品种草` / `观点讨论` / `生活记录`。填充已有记录时，如果记录已有内容类型可省略

缺失内容类型且现有记录也未设置 → 提示补全，停止。类型不在列表中 → 提示选择，停止。

## 目标数据库 <!-- EN: Target Database -->

Notion 数据库：**小红书内容工作流**

完整字段（20 个）：

**📝 可写入字段：** <!-- EN: Writable fields -->
- `选题`（title）
- `内容类型`（select）：干货教程 / 经验分享 / 产品种草 / 观点讨论 / 生活记录
- `状态`（select）：灵感池 / 待配图 / 待发布 / 已发布
- `标题`（text）
- `互动引导`（text）
- `标签`（multi_select）：效率提升 / 内容创作 / 小红书运营 / AI工具
- `发布话题标签`（text）——自由文本，每篇单独拟 5~8 个具体贴题的标签（广泛词+精准长尾词混用）

**🔒 不自动填写的字段：** <!-- EN: Fields never auto-filled -->
- `发布日期`（date）
- `数据表现`（number）
- `笔记链接`（url）
- `曝光`（number）
- `点赞`（number）
- `收藏`（number）
- `评论`（number）
- `涨粉`（number）
- `发布时间`（date）
- `复盘结论`（text）
- `优化建议`（text）

> ⚠️ 数据库 ID/数据源 ID 每次执行时通过 `notion_search` + `notion_fetch` 动态获取，不硬编码。 <!-- EN: DB ID fetched dynamically, never hardcoded. -->

## 执行流程 <!-- EN: Execution Flow -->

### Step 1: 搜索数据库并定位 <!-- EN: Search & Locate Database -->

```bash
# 1a. 搜索数据库
notion_search(query="小红书内容工作流", query_type="internal")

# 1b. 获取 schema 和 data_source_id
notion_fetch(id="<database_id>")

# 1c. 提取 data_source_id — 注意区分两种用途：
#   • SQL 查询用 collection:// 完整 URI：collection://3a2d6eb8-03e1-80fd-90cd-000b80336ebf
#   • notion_create_pages 的 parent.data_source_id 只用裸 UUID：3a2d6eb8-03e1-80fd-90cd-000b80336ebf
#   ⚠️ 传 collection:// URI 给 create_pages 会报 validation_error
```

### Step 2: 选题去重 <!-- EN: Topic Deduplication -->

```sql
SELECT * FROM "collection://<data_source_id>" WHERE "选题" = '<选题>'
```

**未命中 →** 继续 Step 4 新建记录。

**命中 + 正文和标题均非空 → 停止。** 不创建、不覆盖。返回已有记录（选题、状态、链接）并提示用户。

**命中 + 正文或标题为空 → 复用该记录。** 使用现有 `page_id`，跳过 Step 4，直接从 Step 5 生成内容。适用于：灵感池选题写内容、内容为空的待配图记录补内容。如果现有记录未设置 `内容类型`，以用户本次指定的为准。

### Step 3: 事实核验（条件触发） <!-- EN: Fact-Checking (Conditional) -->

**触发条件**：选题涉及平台政策、算法变化、封禁规则、新功能/内测、具体数据、新闻事件、官方声明。

**核验方法**：
1. 使用 `duckduckgo-search` 或终端 `ddgs` 搜索：
   - 搜索：`ddgs text -k "关键词" -m 10`
   - 提取页面：`ddgs extract -u "<url>"`
   - 注意：ddgs 当前版本用 `-k`（或 `-q`）传查询词，`-m` 设最大结果数（旧版 `--max` 已废弃）
2. 优先官方公告、帮助中心
3. 至少 2 个独立来源交叉验证
4. 记录来源链接和日期

**核验失败 → 停止。** 不猜测、不包装。提示用户需要人工确认。

### Step 4: 新建记录（仅去重未命中时） <!-- EN: Create Record (only if no duplicate) -->

```json
{
  "parent": {"data_source_id": "collection://..."},
  "pages": [{
    "properties": {
      "选题": "<选题>",
      "状态": "灵感池",
      "内容类型": "<内容类型>"
    }
  }]
}
```

获取返回的 `page_id`。

> 如果 Step 2 命中了内容为空的现有记录，跳过此步骤，直接使用现有 `page_id`。

### Step 5: 生成内容 <!-- EN: Generate Content -->

#### 核心角度（生成前必读） <!-- EN: Core Angle (must read before generating) -->

⚠️ **生成任何内容前，先从 Notion 记录中读取「核心角度」字段。** 该字段由选题人和舵手反复讨论确定，是这篇笔记的独特切入点和要避开的坑。

核心角度的作用：
- **定方向**：这篇笔记从哪个角度切入？与其他同类选题的差异在哪？
- **避坑**：这篇笔记不能写成什么样？常见的偏离方向是什么？

核心角度**不规定**怎么写——分几段、举什么例子、用什么结构、怎么开头结尾，全由 Hermes 根据选题和内容类型自主决定。

> 一句话：核心角度是罗盘（定方向），不是轨道（不限制路线）。方向跟着它走，怎么走你说了算。

#### 事实底线（不可违反） <!-- EN: Fact Floor (Non-Negotiable) -->

禁止伪造或声称：
- 个人经历（"我用了 3 个月涨粉 5 万"——除非是用户的真实数据）
- 具体数据（"80% 的博主都……"——除非有可查证来源）
- 案例细节（除非是公开、可验证的案例）
- 平台政策或官方结论（"根据小红书最新政策……"——除非已完成核验）
- 用户评价或市场表现（"被 xxx 万用户推荐"——除非有来源）

不确定的事实改用：`建议`、`可以考虑`、`一种思路是`、`常见的做法是`。

#### 自由创作空间 <!-- EN: Creative Freedom -->

以下方面由 Hermes 根据选题和内容类型自主决定：

- **标题**：要有冲击力，别陈述事实。黑名单：「XX的真相」「不是你不努力」「教你XX」「你还在XX吗」等被用烂的模板句。≤20 字（硬性限制），写完逐字出声数一遍。
- **正文结构**：故事式、清单式、问答式、观点式、教程式、对比式、体验式——选择最合适的
- **内容长度**：由选题深度决定，不设硬性字数限制
- **语气节奏**：
  - 干货教程 → 清晰直接
  - 经验分享 → 自然真实
  - 产品种草 → 具体克制
  - 观点讨论 → 有立场但不极端
  - 生活记录 → 有叙事感
- **封面创意**：手机端清晰易读即可，不限定字数
- **互动方式**：别用「你平时...呢？评论区交流一下」「聊聊你的看法」「你怎么看」这类模板句，要让人有非答不可的理由。

#### 默认建议（非强制） <!-- EN: Default Suggestions (Not Mandatory) -->

- 标题 ≤ 20 字（硬性限制），在有限字数内做到简洁有力
- **正文不超过 1000 字**（小红书笔记字数上限），分段清晰，口语化，像真人说话
- 标签通常 3～6 个
- 不强制固定开头/结尾/CTA
- 不强制标题包含数字、冲突词、利益点或情绪词

#### 输出策略 <!-- EN: Output Strategy -->

- **默认**：直接选择最合适的方案，生成一个版本
- **仅在以下情况提供多个方向**：
  - 选题存在明显多种切入角度
  - 用户明确要求多个版本
  - 观点型内容可能存在立场分歧

不要为了显得完整而机械输出多个版本。

#### 生成后复查（写入前必须执行） <!-- EN: Pre-Write Review (Mandatory) -->

⚠️ **这是写入前的硬性检查关口，逐项确认，不得跳过。**

| # | 检查项 | 怎么查 | 不通过时 |
|---|--------|--------|----------|
| 1 | **标题 ≤ 20 字** | **逐字出声数一遍**：中文字符、英文字母、数字、标点全部计入。一个标点也算 1 字。 | 缩短或重写标题。特别注意：含引号、括号、破折号的标题，这些标点占用字数容易被忽略。 |
| 2 | **正文 ≤ 1000 字** | 数字符总数 | 压缩冗余段落、合并短句 |
| 3 | **无捏造数据** | 搜索 `\d+%`、`\d+ 万`、`\d+ 成` 等数字模式；对每个数字问「这个有来源吗？」 | 删除或改为模糊表述（「很多」「大多数」「据观察」） |
| 4 | **无伪造经历** | 搜索「我」「我的朋友」「我认识」等第一人称 + 具体事件的组合 | 改为一般性建议或删除 |
| 5 | **无平台政策断言** | 搜索「小红书官方」「根据政策」「新规」 | 删除或标注「个人理解」 |
| 6 | **标签均为已有选项** | 交叉核对数据库 schema 中的标签选项 | 替换为已有选项 |

**复查失败 → 修改内容 → 重新复查 → 通过后才能进入 Step 6。**

常见漏网模式（来自实际翻车案例）：
- `我认识的万粉博主里，80%前50条笔记赞藏不过百` — **虚构百分比，删除**
- `我用了 3 个月涨粉 5 万` — **伪造经历，删除**
- 标题 24 字但自认为「差不多」→ **严格数，一个标点也算**

### Step 6: 写入内容 <!-- EN: Write Content -->

⚠️ **标签验证**：`标签` 是 multi_select，值必须是数据库已有选项（效率提升 / 内容创作 / 小红书运营 / AI工具）。传入不在列表中的值会直接报 `validation_error`，不会自动创建新选项。如果选题需要新标签类别，需先通过 `notion_update_data_source` 添加。

```json
{
  "page_id": "<page_id>",
  "command": "update_properties",
  "properties": {
    "标题": "<标题>",
    "正文": "<正文>",
    "互动引导": "<互动引导>",
    "标签": ["标签1", "标签2", "标签3"],
    "发布话题标签": "<5~8个具体标签，广泛词+长尾词混用>"
  }
}
```

### Step 7: 状态 →「待配图」 <!-- EN: Status → Ready for Images -->

```json
{
  "page_id": "<page_id>",
  "command": "update_properties",
  "properties": {"状态": "待配图"}
}
```

### Step 8: 输出结果 <!-- EN: Output Result -->

```
✅ 小红书内容生产完成

| 项目 | 内容 |
|------|------|
| 选题 | <选题> |
| 内容类型 | <内容类型> |
| 标题 | <标题> |
| 状态 | 待配图 |
| 记录链接 | <Notion URL> |

已写入：标题 / 正文 / 互动引导 / 标签 / 发布话题标签
事实核验：<已完成 / 不适用>
```

不输出数据库 ID、UUID、工具参数、技术日志。

## 错误处理 <!-- EN: Error Handling -->

| 错误 | 处理 |
|------|------|
| 找不到数据库 | 提示检查 Notion 连接，停止 |
| 字段名/类型不匹配 | 记录差异，停止 |
| Notion 连接失败 | 提示检查网络和 Token，停止 |
| 内容类型不在列表 | 提示从列表中选择，停止 |
| 无法核验高风险事实 | 提示需人工确认，停止 |
| 选题已存在且有内容 | 返回已有记录，提示已有内容，停止 |
| 选题已存在但内容为空 | 复用该记录生成内容，正常流转 |
| 创建成功但写入失败 | 告知已创建记录（附链接）和写入失败的字段 |
| 写入成功但状态更新失败 | 告知内容已写入、状态未更新，建议手动改 |
| data_source_id 格式错误 | notion_create_pages 的 data_source_id 必须用裸 UUID，不能带 collection:// 前缀。SQL 查询才用完整 URI |

**原则**：
- 不在失败后假装成功
- 不创建半成品记录后不说明
- 不修改任何其他已有记录

## 边界 <!-- EN: Boundaries / What This Skill Does NOT Do -->

此 Skill **不负责**：
- 生成图片（提供给用户自行使用图片工具）
- 上传图片
- 发布小红书
- 抓取小红书站内数据
- 填写发布后数据（数据表现 / 笔记链接 / 曝光 / 点赞 / 收藏 / 评论 / 涨粉 / 发布时间 / 复盘结论 / 优化建议）

## 依赖 <!-- EN: Dependencies -->

- `notion` MCP（notion_search / notion_fetch / notion_create_pages / notion_update_page / notion_query_data_sources）
- `duckduckgo-search`（事实核验，条件触发）
