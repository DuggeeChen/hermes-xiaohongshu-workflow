---
name: xiaohongshu-image-planning
description: |
  EN: Xiaohongshu (RED) image planning — reads a post from Notion, analyzes content structure, plans multi-image layouts (flexible count, content-driven), independently designs adaptive visual direction (any style — photography/illustration/collage/editorial/conceptual/mixed), generates a Session Prompt (sent once) + lean per-page Page Prompts for AI image tools like ChatGPT Images 2.0, writes the full plan to Notion's 配图方案 field. Does NOT advance status or generate actual images. Internal planning is minimal and focused; Session Prompt sets global rules; Page Prompts are page-specific; ChatGPT gets ample but bounded creative freedom.
  CN: 小红书多图配图方案生成器：读取 Notion 内容记录，生成视觉策划和 AI 生图提示词。职责：读取 Notion → 分析内容 → 规划图片结构 → Prompt 编译 → 写入「配图方案」。不修改状态字段。内部只规划必要内容，Session Prompt 管全局，Page Prompt 管当前页面，ChatGPT 获得充分但有边界的视觉创作自由。不负责重写正文、修改标题、生成/上传图片、发布。
version: 4.0.1
metadata:
  hermes:
    tags: [xiaohongshu, image-planning, notion, 小红书, 配图, 视觉策划, chatgpt, prompt-engineering]
    related_skills: [xiaohongshu-content-production, notion]
---

<!--
  ═══════════════════════════════════════════════════════
  ENGLISH SUMMARY
  ═══════════════════════════════════════════════════════
  Role in pipeline: Stage 3 of 3 — reads a post from Notion
  (status=待配图), analyzes its content, plans a multi-image
  layout, and outputs two-layer prompts for ChatGPT:

    • Session Prompt — sent once, sets global rules, visual
      direction, constraint priorities, creative mandate
    • Page Prompts — one per page, lean, only page-specific info

  Key v4.0.1 changes from v2.2.0:
    • Visual direction is now ADAPTIVE (content-driven), not
      preset by 3 account pillars with fixed color palettes
    • No forced page archetypes, density modes, or per-page
      differentiation requirements
    • Quality check merged into single pass (5 items)
    • Prompt compilation simplified (4 steps vs 8)
    • Constraint priority simplified (4 tiers vs 7)
    • Page Prompts are ~40-50% shorter (no repeated global rules)
    • Output is lean: Session Prompt + Page Prompts + one-liner
      usage instruction — no static templates, no fix commands
    • Status is NOT advanced (stays as-is after writing 配图方案)

  Quality gates: 5-item single-pass check after compilation.
  Hard constraints: specified Chinese text must be verbatim;
  no fabricated facts/experiences/data.

  ⚠️  This is a Hermes Agent SKILL.md — the Chinese text
     below is the actual runtime prompt. The English here
     is supplementary documentation.
  ═══════════════════════════════════════════════════════
-->

# 小红书多图配图方案 <!-- EN: Xiaohongshu Multi-Image Planning -->

> 内部只规划必要内容，Session Prompt 管全局，Page Prompt 管当前页面，ChatGPT 获得充分但有边界的视觉创作自由。
> <!-- EN: Minimal internal planning. Session Prompt = global rules. Page Prompts = page-specific only. ChatGPT gets ample but bounded creative freedom. -->

## 概述 <!-- EN: Overview -->

这个 Skill 的职责是：读取笔记正文 → 分析内容 → 规划整组配图 → 输出可直接交给 ChatGPT 的生图提示词。

核心设计原则：**Agent 内部只规划必要内容。Session Prompt 管全局规则，Page Prompt 只负责当前页面。ChatGPT 获得充分但有边界的视觉创作自由。** 内部规则可以复杂，最终 Prompt 必须清楚、轻盈、有优先级、有创作授权。

## 触发条件 <!-- EN: Trigger Conditions -->

用户提供以下之一，否则默认取最新「待配图」记录：
- **选题**（`选题` 字段值）
- **标题**（`标题` 字段值）

## 执行流程 <!-- EN: Execution Flow -->

### Step 1：定位记录

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

### Step 2：提取内容

读取字段及用途：

| 字段 | 用途 |
|------|------|
| `选题` | 理解整体主题 |
| `标题` | 理解封面主题 |
| `正文` | 拆分多图内容的核心依据 |
| `内容类型` | 判断表达方式 |
| `封面提示词` | 仅作封面视觉方向参考，不要求沿用，不限制后续页面创意 |

### Step 3：内容分析（内部规划）<!-- EN: Content Analysis -->

分析正文的核心命题和读者目标反应。填写以下内部规划字段：

```yaml
planning:
  core_proposition:          # 这篇笔记唯一的核心命题
  desired_reader_response:   # 读者看完整组图片后，应理解、感受或采取什么行动
```

- `core_proposition` 和 `desired_reader_response` 必填
- 不要求填写固定情绪曲线；教程、清单和方法类内容可以使用认知顺序

完成标准：能用一个完整的句子说出这篇笔记的核心命题和读者看完后应产生什么反应。

### Step 4：规划图片结构 <!-- EN: Image Structure Planning -->

基本规则：

- 图片数量由内容自然决定，通常 4~6 张，但可以更少或更多
- 第 1 张通常为封面
- 每页只承担一个主要信息任务
- 页面顺序应有信息递进
- 不得为了达到数量而拆分或凑页
- 不得将同一观点换一种说法重复多页

封面应单独承担吸引停留和建立主题的任务，但不需要生成多个封面候选，除非用户明确要求。

为每一页填写规划字段：

```yaml
pages:
  - page_number:
    role:                    # 本页在整组内容中的作用
    core_message:            # 本页唯一核心信息
    exact_text:              # 必须准确出现的文字（可以为空）
    visual_intent:           # 本页的视觉任务与阅读顺序
    special_constraint:      # 可选，仅在确有必要时填写
```

- `special_constraint` 默认为空，不要为了完整而填写
- `exact_text` 可以为空，不要为了填写字段而增加文字

### Step 5：自适应视觉方向 <!-- EN: Adaptive Visual Direction -->

Agent 应根据当前笔记的具体内容、语气、主题、信息结构和叙事目标，为这组图片独立规划最合适的视觉方向。

不要默认沿用任何固定配色、固定插画风格、固定版式或固定摄影语言。

Agent 可以自主决定：

- 整组图片采用摄影、插画、拼贴、编辑设计、概念视觉、信息图形或混合风格
- 色彩系统
- 材质与质感
- 光线和空间氛围
- 图形语言
- 字体气质
- 视觉隐喻
- 镜头语言
- 页面节奏
- 留白与信息密度

判断标准：

1. 是否准确服务当前内容
2. 是否能在小红书信息流中形成停留感
3. 是否有明确的主次层级
4. 是否具有整套视觉作品的完成度
5. 是否给 ChatGPT 留出真实的创作空间

如果数据库中的「封面提示词」具有参考价值，可以吸收其主题、氛围或视觉意图，但不得机械复制，也不得把它作为后续所有页面的固定模板。

**跨页一致性：**

视觉方向可以自由选择，但同一组图片一旦确定视觉系统，后续页面必须保持系列一致性。

规划以下 `global_visual` 字段：

```yaml
global_visual:
  concept:                 # 整组视觉母题
  mood:                    # 整体气质
  visual_language:         # 最适合本内容的视觉表达方向（摄影/插画/拼贴/编辑设计/概念视觉/信息图形/混合）
  continuity:              # 跨页需要保持一致的关键元素
```

不需要固定填写主色、辅助色、镜头参数、纹理等所有字段。只有当某个具体属性对系列一致性有实际帮助时，才记录它。

核心原则：

> Agent 根据当前内容提出开放的视觉方向，ChatGPT 在不改变核心信息和指定文字的前提下，自主优化并完成视觉创作。风格不由预设账号模板决定，但整套作品必须自洽。

### Step 6：Prompt 编译 <!-- EN: Prompt Compilation -->

在生成最终提示词前，执行四步编译：

1. **合并和删除重复信息** — 删除重复规则，合并同义表达
2. **解决冲突并确定优先级** — 识别规则冲突，按约束优先级处理
3. **分离硬约束和创作意图** — 区分"必须遵守"和"创作方向"
4. **压缩表达，保留创作空间** — 删除不必要的逐像素布局和形容词堆叠

约束优先级（从高到低，只在 Session Prompt 中出现一次）：

1. 指定文字与事实准确
2. 页面核心信息清楚
3. 整组视觉一致且手机端可读
4. 创意、审美和装饰细节

### Step 7：质量检查 <!-- EN: Quality Check -->

在 Prompt 编译之后、写入 Notion 之前，执行一次检查，只保留以下五项：

1. 每一页是否来自当前正文，并承担独立且必要的作用？
2. 页面之间是否具有递进关系，且不存在信息重复或无意义拆分？
3. 每页指定文字是否清楚、准确、数量合理？
4. 整组视觉系统是否一致，同时避免机械复制相同构图？
5. 最终 Prompt 是否简洁、可直接使用，并给 ChatGPT 留有足够的创作空间？

如果任意一项不通过，修改对应页面即可，不需要整体重做。

三条致命问题（出现时重新压缩 Prompt）：

- Prompt 像设计规范清单，而不是创作 Brief
- Prompt 只罗列元素，没有说明页面要表达什么
- 规则过多，ChatGPT 几乎没有视觉决策空间

不要输出打勾过程，不需要生成评分表。

### Step 8：组装最终输出 <!-- EN: Assemble Final Output -->

最终输出只包含三部分：

````markdown
# 小红书配图生成方案

先发送整套启动指令，再在同一 ChatGPT 会话中依次发送各页指令。

## 整套启动指令

```text
（Session Prompt）
```

## 第 1 张

```text
（Page Prompt）
```

## 第 2 张

```text
（Page Prompt）
```
````

除非用户明确要求，不添加其他说明、分析表格或修正模板。

#### 整套启动指令（Session Prompt）<!-- EN: Session Prompt -->

只需发送一次。包含：

- 整组主题、图片数量
- 核心命题
- 读者看完后的目标反应
- 页面序列概览
- 为本组内容选择的视觉方向
- 跨页需要保持一致的关键元素
- 指定文字与事实准确性要求
- 全局禁止事项
- 四层约束优先级
- 创作授权
- 执行方式

**页面序列概览：** 每页一句话，只说明页面作用和核心信息，不展开构图，不重复 Page Prompt。格式示例：

```
【页面序列】
第1张：封面，建立主题并吸引停留
第2张：呈现核心问题
第3张：解释原因
第4张：展示方法
第5张：总结结果
```

实际内容根据正文生成，不固定套用示例。

执行方式应明确：

- 这是同一篇笔记的一组连续配图
- 先理解整组内容和页面顺序
- 每次只生成一张，不生成九宫格或多页拼图
- 后续图片保持同一设计系统
- 不得自行改写指定中文
- 默认不增加任何未指定文字
- 不必机械复制页面描述
- 可以主动选择更好的构图、镜头、视觉隐喻和细节
- 视觉风格不受固定账号配色限制
- 一旦确定整组视觉方向，后续页面应保持一致
- 当局部要求影响整体质量时，按照四层优先级处理

创作授权：

> 除明确标注为必须的内容外，其余描述均为创作方向。请根据当前主题主动选择最合适的视觉语言、构图、镜头、材质、色彩、隐喻和空间关系，不必沿用固定模板。可以优化原始构想，但不得改变核心信息和指定文字。

#### 逐页生成指令（Page Prompt）<!-- EN: Page Prompt -->

每页只输出当前页面新增的信息，不重复 Session Prompt 已建立的规则。

`exact_text` 可以为空，不要为了填写字段而增加文字。无文字页面直接写「本页不出现任何文字。」，有文字时写「必须逐字出现的文字」并标注「不需要出现其他文字」。

```text
现在生成第 X 张。

【本页任务】
说明这一页在整组叙事中的作用，以及读者应理解什么。

【准确文字】
有文字时：「必须逐字出现的文字」
不需要出现其他文字。

无文字时：
本页不出现任何文字。

【视觉意图】
说明读者第一眼应看到什么、随后理解什么。
描述信息关系、情绪、节奏或视觉张力，不要机械指定所有元素的位置。

【特殊要求】
仅在这一页确有特殊约束时出现。没有则删除本节。
```

不要重复 Session Prompt 中的全局内容，包括：3:4 竖版、全局视觉系统、约束优先级、通用禁止事项、完整创作授权、"每次只生成一张"、"不生成拼图"、通用文字规则。

### Step 9：写入 Notion <!-- EN: Write to Notion -->

汇报完方案后直接写入 `配图方案` 字段，不需要等用户确认。状态字段保持不变。

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

## 视觉描述方式指南 <!-- EN: Visual Description Guide -->

**优先使用：** 视觉母题、阅读顺序、情绪变化、信息关系、画面张力、主次层级、空间节奏、视觉隐喻、前后页关系

**减少使用：** 大量元素坐标、每个元素必须放在哪个角落、重复的风格形容词、过多固定装饰要求、不必要的摄影参数、堆叠式负面提示词

只有当具体位置是信息表达必需条件时，才指定位置。

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

## 边界 <!-- EN: Boundaries -->

**负责：**
- 内容分析
- 配图规划
- 内部规划
- Prompt 编译
- 生成可直接复制给 ChatGPT 的 Session Prompt 和 Page Prompts
- 写入 Notion

**不负责：**
- 实际调用生图工具
- 查看/审核生成结果
- 图片编辑、下载、保存、上传
- 图片发布
- 重写正文
- 修改标题

## 依赖 <!-- EN: Dependencies -->

- `notion` MCP（notion_search / notion_fetch / notion_query_data_sources / notion_update_page）

## 附录 <!-- EN: Appendix -->

### 页面原型库（可选参考）<!-- EN: Page Archetypes (Optional Reference) -->

以下原型仅在 Agent 没有思路时作为构图语法参考，不是执行步骤，也不是必填字段。不得原样输出给 ChatGPT，不得成为固定模板。

| 原型 | 适用场景 | 视觉任务 |
|------|----------|----------|
| 封面海报 | 笔记入口 | 标题+情绪定调，吸引停留 |
| 场景叙事 | 故事/经历 | 让读者代入场景 |
| 痛点快照 | 问题呈现 | 让读者感到"这就是我" |
| 因果拆解 | 原因分析 | 清晰展示因果关系 |
| 步骤流程 | 方法/教程 | 信息层级清晰，可跟随 |
| 前后对比 | 效果展示 | 对比张力 |
| 清单页 | 干货罗列 | 信息密度高但可扫读 |
| 系统关系 | 复杂概念 | 展示元素间关系 |
| 总结收束 | 结尾升华 | 情绪收束+行动暗示 |
| 互动结尾 | 引导互动 | 轻松，有参与感 |

### 通用修正指令 <!-- EN: General Fix Commands -->

仅在用户明确要求修图指令时输出，不默认写入每篇配图方案。

**保持画面，只修正文字：**
```text
保持当前图片的构图、人物、色彩、材质和图形元素不变。
只修改以下文字：
错误文字：
正确文字：
不要添加其他文字。
```

**保持视觉系统，重新构图：**
```text
延续当前整套图片的色彩、材质和视觉语言，
但不要沿用当前构图。
根据本页核心信息重新设计更清晰、更有表现力的画面。
```

**修正跨页一致性：**
```text
保持当前页面的文字和核心构图不变，
将人物、服装、关键物体、色彩和材质恢复为前面页面中的统一设定。
```
