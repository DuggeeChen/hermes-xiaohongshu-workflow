---
name: xiaohongshu-topic-research
description: |
  EN: Xiaohongshu (RED) topic discovery — user gives a direction → web search → curated topic list with angles, credibility scores, and source links. Read-only research; does not write to Notion or generate post content.
  CN: 小红书选题探索助手 — 用户给方向 → 搜索公开信息 → 输出可用的选题。只读研究，不写 Notion，不生正文。
version: 3.0.0
metadata:
  hermes:
    tags: [xiaohongshu, topic-research, 小红书, 选题研究]
    related_skills: [xiaohongshu-content-production, duckduckgo-search]
---

<!--
  ═══════════════════════════════════════════════════════════════
  ENGLISH SUMMARY
  ═══════════════════════════════════════════════════════════════
  This skill is the research stage of the Xiaohongshu content
  pipeline. It takes a broad topic direction (e.g. "AI tools",
  "productivity hacks"), searches the web for current discussions
  and angles, and returns a curated list of potential post topics.

  Two modes:
  • Default (lightweight): 1+ reliable source per topic, outputs
    topic + type + rationale + credibility
  • Deep mode (triggered by "深度研究"): multi-source verification,
    detailed scoring, risk analysis

  Quality gates:
  • Sensitive topics (platform policy, algorithm changes, specific
    data) require ≥2 independent sources
  • Unverifiable claims are downgraded or removed
  • 404/anti-scraped pages don't count as valid sources

  ⚠️  This is a Hermes Agent SKILL.md — the Chinese text below is
     the actual runtime prompt. The English here is supplementary
     documentation for open-source readers.
  ═══════════════════════════════════════════════════════════════
-->

# 小红书选题探索 <!-- EN: Xiaohongshu Topic Research -->

> 用户给方向 → 搜索公开信息 → 输出可用的选题。默认轻量探索，需要时才深入核验。 <!-- EN: Direction → search → usable topics. Lightweight by default, deep on demand. -->

## 触发条件 <!-- EN: Trigger Conditions -->

用户提供**主题方向**即可。可选附带：时间偏好、目标人群、账号定位。

## 默认模式：灵感探索 <!-- EN: Default Mode: Inspiration Discovery -->

### 搜索 <!-- EN: Search -->

围绕主题方向自由搜索公开信息。

Hermes 自行决定：搜索关键词、搜索角度、是否扩展相邻话题、是否将一个宽泛热点拆成多个具体角度、是否合并重复话题。

搜索不限轮数，以信息收益为准。不设最低/最高候选数量——根据搜索质量自然输出，通常 3～8 个。

### 可输出的选题类型 <!-- EN: Topic Types That Can Be Output -->

- **近期话题** — 当前讨论较多、有新鲜感的角度 <!-- EN: Trending topics — currently active, fresh angles -->
- **常青选题** — 不依赖时效、长期可用的角度 <!-- EN: Evergreen topics — timeless, always relevant -->

不强制每个选题都挂钩新闻热点。

### 来源要求 <!-- EN: Source Requirements -->

**普通选题**（工具、经验、趋势、使用心得等）：
- ≥1 个可靠可访问来源即可
- 也可基于多个公开信号综合形成创作角度
- 但不得虚构具体事实

**高风险选题**（涉及以下任一）→ 进入严格核验： <!-- EN: High-risk topics → strict verification -->
- 平台政策
- 算法调整
- 封禁规则
- 新功能发布（含内测）
- 具体数据
- 重大新闻事件
- 争议性事实判断

严格核验要求：
- ≥2 个独立可访问来源
- 优先官方公告、帮助中心
- 无法核验时不得下确定结论，降低结论强度或删除该选题

### 页面访问失败时 <!-- EN: When Pages Are Inaccessible -->

页面 404 / 反爬 / 需登录 / 无日期 → 不算有效来源。可以：
- 寻找替代来源
- 找不到时降低结论强度或删除该选题
- 不得脑补

搜索摘要 ≠ 已核验事实。 <!-- EN: Search snippets ≠ verified facts. -->

### 选题评估 <!-- EN: Topic Evaluation -->

每个选题默认只需给出：
- **推荐理由**（一句话）
- **适合的内容类型**：干货教程 / 经验分享 / 产品种草 / 观点讨论 / 生活记录
- **可信度**：已核验 / 有可靠来源 / 基于公开信号合成 / 待验证 <!-- EN: Credibility: verified / reliable source / synthetic / unverified -->
- **是否适合立即创作**

不强制数字评分。只有用户明确要求「评分」时才给出分值。

### 排除说明 <!-- EN: Exclusion Notes -->

只简要提及明显被排除的高风险或低可信选题及原因。不强制每个选题都写删除报告。

## 深度模式（用户明确要求时触发） <!-- EN: Deep Mode (triggered explicitly) -->

触发词：「深度研究」「严格核验」「完整报告」「政策核验」。

在默认模式基础上增加：
- 多来源交叉核验
- 来源发布时间
- 详细评分（维度由选题特性决定）
- 删除原因
- 风险说明

## 硬性边界 <!-- EN: Hard Boundaries -->

- 不编造来源 <!-- EN: Don't fabricate sources -->
- 不把搜索摘要当成已核验事实
- 不绕过反爬机制
- 不写入 Notion <!-- EN: Don't write to Notion -->
- 不生成正文 <!-- EN: Don't generate post content -->
- 搜索失败时明确说明，宁可少给也不编造
- 涉及政策、算法、封禁、数据时必须严格核验
- 不凑数量降低标准

## 默认输出格式 <!-- EN: Default Output Format -->

```markdown
## 选题探索

**主题方向**：xxx

### 近期话题

| # | 选题 | 类型 | 推荐理由 | 可信度 | 来源 |
|---|------|------|----------|--------|------|
| 1 | ... | ... | ... | ... | [链接](url) |

### 常青选题

| # | 选题 | 类型 | 推荐理由 | 可信度 | 来源 |
|---|------|------|----------|--------|------|
| 1 | ... | ... | ... | ... | [链接](url) |

### 建议优先创作
1. 选题X — 理由
2. 选题Y — 理由

### 注意事项（如有）
- 排除：选题Z — 来源无法确认，暂不建议
```

## 自由发挥空间 <!-- EN: Creative Freedom -->

Hermes 自行决定：
- 搜索关键词和角度
- 是否扩展相邻话题
- 选题包装方式
- 受众切入点
- 内容类型判断
- 输出数量（不设固定上下限）
- 是否拆分宽泛热点
- 是否合并重复话题
- 搜索深度和轮数

唯一约束：不违反事实规则和硬性边界。 <!-- EN: Only constraint: don't violate fact rules and hard boundaries. -->

## 依赖 <!-- EN: Dependencies -->

- `duckduckgo-search` Skill / `ddgs` CLI
  - 搜索：`ddgs text -k "关键词" -m 15`
  - 提取页面：`ddgs extract -u "<url>"`
  - 注意：当前版本用 `-k`（或 `-q`）传查询词，`-m` 设最大结果数（旧版 `--max` 已废弃）
- `web_search` / `web_extract` / 浏览器工具（用于打开页面核验）
