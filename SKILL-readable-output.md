---
name: readable-output
description: "Distill long conversations into structured, reviewable HTML pages. Covers all domains: research design, architecture decisions (ADR), product direction, debug postmortems, and more. Triggers: 整理对话, 沉淀一下, 做个回看, summarize discussion, write ADR, decision record, /readable. Not for: revision/diff tracking or short answers (<500 words)."
---

# Readable Output · 对话沉淀为可回看的HTML v2.0

> **灵魂句（处境）**：
>
> 你刚和用户讨论了一个复杂问题——可能是研究设计的三种识别策略，可能是系统架构在三个方案间的取舍，可能是产品方向的关键转折，可能是一次漫长debug的根因定位。
>
> 如果这段对话就这么消失在聊天记录里，三个月后重启这个项目时，用户会重新问一遍同样的问题，重新踩一遍同样的坑。
>
> 你的任务不是"生成一份总结"，是**让三个月后的用户打开这个HTML，5分钟内重建当时的思考现场**。
>
> 关键不是记录"说了什么"，是提炼"想通了什么"、"决定了什么"、"为什么排除了其他选项"。

---

## 你站在哪里

你不是一个"对话转录工具"，你是一个**思考过程的结构化专家**。

你手里有：

- 一段长对话（可能跨越多个话题、多次转折、多种领域）
- 对话中散落的关键决策、参考资料、待办事项、代码片段
- 用户的背景和项目上下文

你的责任：

- 从对话中提炼核心洞察和决策，不是逐句转录
- 让产出物在脱离对话上下文后仍然可理解
- 标注哪些是确定的结论、哪些是待验证的假设、哪些是被否决的方案

你的约束：

- 三个月后的用户是你的读者——他不记得对话细节，但记得项目大方向
- 协作者（导师/团队成员/产品同事）可能也会看——要能独立理解
- 宁可漏掉闲聊，不可漏掉决策和理由

---

## 触发条件

**触发**：

- "整理对话"、"沉淀一下"、"做个回看"、"summarize this discussion"
- "把讨论整理一下"、"这次对话很有价值"
- "帮我整理刚才的讨论"、"汇总一下"
- "readable"、"/readable"
- "整理成HTML"（上下文无修订标记时）
- "写个ADR"、"记录一下决策"、"decision record"

**智能判断（触发词路由）**：

- 上下文有 `<ins>`/`<del>` 修订标记，或用户说"修订"/"track changes"/"diff" → 不走本Skill（走 revision-render 类工具）
- 其他场景说"html"/"整理成HTML" → 走本Skill

**不触发**：

- 论文修订痕迹可视化（需要专门的 revision/diff 工具）
- 短回答（<500字产出）→ 直接在对话里说
- 用户明确说"用markdown"/"不要HTML"

---

## 阶段 1：场景确认（用 AskUserQuestion，最多3问）

用户已在prompt里明确所有参数时可跳过。提供**快速默认**：用户只说"整理一下"不想回答 → 自动用「自己回看 + 决策记录 + 标准版」。

### 问题 1：什么场景？（合并受众+终点）

| 选项 | 说明 | 默认风格 |
| --- | --- | --- |
| 自己三个月后重启项目 | 备忘+决策链，重建思考现场 | 研究叙事 |
| 给团队/协作者看推理过程 | 需独立理解，展示论证依据 | 研究叙事 |
| 技术架构决策记录（ADR） | Context→Decision→Consequences | 技术文档 |
| Debug/排障复盘 | 症状→排查→根因→修复→防护 | 技术文档 |
| 产品方向/需求讨论 | 问题→方案→取舍→MVP定义 | 产品备忘 |
| 提取行动清单 | 只要TODO和优先级 | 决策备忘 |

### 问题 2：要详到什么程度？

| 选项 | 包含什么 | 阅读时间 |
| --- | --- | --- |
| 速记卡 | 核心结论 + 1个关键决策 + TODO | 2分钟 |
| 标准版（默认） | 决策链 + 关键论证 + 参考资料 + TODO | 5-8分钟 |
| 完整版 | + 讨论过程 + 被否决的方案 + 推演细节 | 15-20分钟 |

### 问题 3：风格（可选，不问则按场景自动匹配）

| 选项 | 视觉 | 适合场景 |
| --- | --- | --- |
| 研究叙事（默认学术） | 暖白底+时间线+callout | 研究推演、文献讨论 |
| 技术文档 | 深色代码块+monospace+GitHub风 | 架构决策、debug复盘、技术选型 |
| 产品备忘 | 卡片+标签+进度指示 | 产品需求、功能讨论、路线图 |
| 决策备忘 | 极简黑白+卡片式 | 只要结论和理由 |

---

## 阶段 1.5（强制）：素材体量自检

问题答完后、开始写之前，自检一次：

**触发反劝**：对话中可提取的独立决策点 < 3个 + 用户选了"完整版"

> ⚠️ 你选的是完整版（15-20分钟阅读），但这次对话的独立决策点约 X 个。强行扩到完整版会有大量"过程复述"而非"洞察提炼"。建议改成「标准版」（5-8分钟）。
>
> 要继续完整版吗？（继续 / 改标准版）

---

## 阶段 2-6（AI内部执行，不对用户念）

预计生成时间：速记卡~1分钟 | 标准版~2分钟 | 完整版~4分钟

### 阶段 2 · 定终点

基于用户选的场景，具体化"读完手里多了什么"：

| 场景 | 终点产物 |
| --- | --- |
| 自己回看 | 决策树 + 理由链 |
| 团队/协作者 | 完整推理路径 + 证据 |
| ADR | Context/Decision/Consequences 三段式 |
| Debug复盘 | 根因 + 修复方案 + 防护措施 |
| 产品讨论 | 方案对比矩阵 + MVP边界 |
| 行动清单 | 优先级排序的TODO |

### 阶段 3 · 抓核心点（≤ 3 个）

从对话中提取不超过3个核心洞察/决策。超过3个必须合并或降级到附录。

**砍半测试**：列出所有候选点后问"只能保留一半留哪几个"——剩下的就是真骨干。

### 阶段 4 · 选主结构

| 对话性质 | 推荐结构 |
| --- | --- |
| 研究设计推演 | 时间线（问题→方案A→否决→方案B→选定） |
| 审稿回应策略 | 双栏对照（审稿意见 ↔ 回应策略+修改方案） |
| 文献讨论 | 分类章节（理论脉络/方法前沿/研究gap） |
| 战略/方向讨论 | 决策树（现状→选项→选定→理由） |
| 技术架构决策 | ADR式（Context→Decision→Consequences→Status） |
| Debug/排障复盘 | 根因分析（症状→排查路径→根因→修复→防护） |
| 产品需求讨论 | PRD式（问题定义→方案→取舍→MVP→迭代计划） |
| 技术选型对比 | 矩阵式（维度×候选方案，加权打分） |
| 代码review总结 | 问题清单+修复方案+架构改进建议 |
| 多话题混合 | 卡片式（每个话题一张卡） |

### 阶段 5 · 写

**5.1 开篇TL;DR**：前2-3句 = 全文核心结论。技术场景可用一行代码或命令概括。

**5.2 每段一个论点句开头**：先说结论再展开。

**5.3 参考资料提取**（必选模块，按场景适配）：

| 场景 | 提取内容 | 格式 |
| --- | --- | --- |
| 学术 | 文献引用 | `作者 (年份). 标题. 期刊.` 无法确认标 `[待补全]` |
| 代码 | 文档链接/Issue/PR/Stack Overflow | `[标题](URL) — 一句话说明` |
| 产品 | 竞品链接/用户反馈/数据报告 | `来源: 内容摘要` |
| 通用 | 对话中提到的任何外部资源 | 按类型归类 |

**5.4 TODO提取**（必选模块）：从讨论中提取所有待办事项，按优先级排列。格式：`□ 待办内容 | 优先级：高/中/低 | 关联决策：X`。TODO必须具体可执行（不是"继续研究"，是"明天读完Smith(2024)第三章"或"周五前把auth middleware重构完"）。

**5.5 决策链可视化**（场景触发）：讨论中有明显决策分叉时生成。

- 学术场景：`分叉点 → 方案A（否决，原因） / 方案B（选定，理由）`
- 技术场景：`需求 → 方案对比表 → 选定方案 + trade-off说明`
- 产品场景：`用户痛点 → 解法候选 → MVP边界 → 迭代路径`

**5.6 代码片段**（技术场景必选）：对话中出现的关键代码、命令、配置，用语法高亮代码块保留。标注"最终版本"vs"中间尝试"。

**5.7 结尾给"出口"**：下一步行动 + 开放问题（还没想通的）+ 边界声明（这个结论在什么条件下可能不成立）

### 阶段 6 · 自检 4 问

1. **三个月后能看懂吗？**（不依赖对话上下文）
2. **核心决策和理由都在吗？**（不只是"决定了X"，还有"因为Y"）
3. **有没有逐句转录对话？**（应该是提炼，不是转录）
4. **TODO是否具体可执行？**（有明确的人/时间/产出物）

---

## 输出文件规则

### HTML 顶部必加配置条

```html
<div class="config-bar">
  <div class="config-line">
    📋 场景：[答案1] · 详略：[答案2] · 风格：[答案3]
  </div>
  <div class="hint-line">
    💬 觉得短？「展开 [章节名]」/ 觉得长？「砍到要点」
  </div>
</div>
```

### 文件路径（智能路由）

```text
学术/研究讨论 → docs/discussions/discussion_{topic}_{YYYYMMDD}.html
代码/技术讨论 → {project_dir}/docs/adr_{topic}_{YYYYMMDD}.html
产品讨论     → {project_dir}/docs/decision_{topic}_{YYYYMMDD}.html
通用/跨领域  → docs/discussions/discussion_{topic}_{YYYYMMDD}.html
```

**增量更新规则**：同主题已有沉淀文件时，提示用户选择"新建独立文件"还是"追加到已有文件（新增章节标注日期）"。

### 写完自动打开

写完HTML后运行 `open <path>`（macOS）或 `xdg-open <path>`（Linux）在浏览器打开。

---

## 反模式清单

| 反模式 | 阶段 | 后果 |
| --- | --- | --- |
| 逐句转录对话 | 5 | 冗长无重点，不如直接看聊天记录 |
| 跳过阶段1直接开写 | 1 | 不知道场景和深度，产出物不对路 |
| 核心点超过3个 | 3 | 工作记忆超载，读完什么都记不住 |
| TODO写"继续研究X" | 5.4 | 不可执行，等于没写 |
| 参考资料不标来源 | 5.3 | 三个月后找不到原文 |
| 决策只记结论不记理由 | 5 | 未来无法判断决策是否仍然有效 |
| 对话中的闲聊也收录 | 3 | 噪音淹没信号 |
| 为凑字数加背景延伸 | 5 | 注水，自检第3问就是防这个 |
| 素材不足选完整版不反劝 | 1.5 | 必然注水 |
| 学术模板硬套代码讨论 | 4 | 文献引用模块对代码场景无意义 |
| 所有文件都扔同一目录 | 输出 | 三个月后面对20个HTML不知道看哪个 |

---

## 不该用这个Skill的反向信号

- ✗ 对话只有3-5轮，没什么值得沉淀的 → 直接记一句话备忘
- ✗ 讨论的是文档具体修改/diff → 用专门的 revision/diff 工具
- ✗ 只是想要一个markdown笔记 → 不需要HTML，直接写md
- ✗ 对话还在进行中，还没到沉淀的时候 → 等讨论完再触发
- ✗ 纯代码实现（没有决策讨论）→ 代码本身就是文档，不需要额外沉淀
- ✗ 信息已经在其他系统中结构化了（Jira/Linear/Notion）→ 避免重复

---

## CSS 设计 token + HTML 骨架

### 风格A：研究叙事（学术默认）

```css
:root {
  --bg: #FFFDF7; --text: #2D2D2D; --accent: #4A7C59;
  --timeline: #E8DCC8; --callout-bg: #F5F0E8; --callout-border: #C9B99A;
  --todo-bg: #FFF8E1; --ref-bg: #F0F4F8;
  --font-body: 'Noto Serif SC', serif;
  --font-heading: 'Noto Sans SC', sans-serif;
}
```

布局：左侧时间线 + 右侧内容块 + callout高亮关键决策

### 风格B：技术文档（代码/架构默认）

```css
:root {
  --bg: #FFFFFF; --text: #24292F; --accent: #0969DA;
  --code-bg: #F6F8FA; --code-border: #D0D7DE;
  --decision-bg: #DDF4FF; --decision-border: #54AEFF;
  --warning-bg: #FFF8C5; --warning-border: #D4A72C;
  --font-body: -apple-system, 'Segoe UI', sans-serif;
  --font-code: 'JetBrains Mono', 'Fira Code', monospace;
}
```

布局：GitHub风格 + 代码块语法高亮 + 折叠式详情 + status badge

### 风格C：产品备忘

```css
:root {
  --bg: #FAFAFA; --text: #1A1A2E; --accent: #6C63FF;
  --card-bg: #FFFFFF; --card-shadow: 0 2px 8px rgba(0,0,0,0.08);
  --tag-bg: #E8E5FF; --tag-text: #4A45B0;
  --progress-fill: #6C63FF; --progress-bg: #E0E0E0;
  --font-body: -apple-system, sans-serif;
  --font-heading: -apple-system, sans-serif;
}
```

布局：卡片式 + 标签分类 + 进度指示 + 优先级色带

### 风格D：决策备忘（极简）

```css
:root {
  --bg: #FFFFFF; --text: #1A1A1A; --accent: #333333;
  --card-bg: #F8F8F8; --card-border: #E0E0E0;
  --priority-high: #D32F2F; --priority-mid: #F57C00; --priority-low: #388E3C;
  --font-body: -apple-system, sans-serif;
  --font-heading: -apple-system, sans-serif;
}
```

布局：卡片式 + 优先级标签 + 极简留白

### HTML 最小骨架（所有风格共用结构）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{主题} — 对话沉淀 {YYYY-MM-DD}</title>
  <style>/* 按风格插入对应CSS token + 通用布局 */</style>
</head>
<body>
  <div class="config-bar"><!-- 配置条 --></div>

  <header>
    <h1>{主题}</h1>
    <p class="meta">{日期} · {场景标签} · 阅读约{N}分钟</p>
  </header>

  <section class="tldr">
    <h2>TL;DR</h2>
    <p>{2-3句核心结论}</p>
  </section>

  <main class="content">
    <!-- 按阶段4选定的结构填充 -->
    <!-- 学术：timeline组件 -->
    <!-- 技术：ADR三段式 / 根因分析流 -->
    <!-- 产品：方案对比卡片 -->
  </main>

  <aside class="todo-section">
    <h2>行动清单</h2>
    <!-- TODO列表，按优先级排序 -->
  </aside>

  <aside class="references">
    <h2>参考资料</h2>
    <!-- 按场景适配：文献/链接/文档 -->
  </aside>

  <footer>
    <div class="open-questions">
      <h3>开放问题</h3>
      <!-- 还没想通的 -->
    </div>
    <div class="boundary">
      <h3>边界声明</h3>
      <!-- 结论在什么条件下可能不成立 -->
    </div>
  </footer>
</body>
</html>
```
