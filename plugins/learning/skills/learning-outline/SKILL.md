---
name: learning-outline
description: >
  Generate a complete, systematic, and detailed learning roadmap/outline for any knowledge domain,
  especially technical and programming fields (e.g., AI/ML, backend development, system design,
  cloud computing, a programming language, a framework, cybersecurity, data science, DevOps, etc.).
  Use this skill whenever the user says things like "I want to learn X", "give me a learning roadmap for X",
  "how do I get started with X", "help me learn X systematically", "give me an outline for X",
  "梳理X的学习大纲", "我想学X", "帮我规划X的学习路径", or any similar intent to systematically
  learn or understand a field. Always trigger this skill for structured learning requests —
  even if the domain seems simple, the user benefits from a comprehensive breakdown.
---

# Learning Outline Generator

Generate a **4-level structured learning roadmap** for any knowledge domain.

## 工具说明

### Tavily 搜索

```bash
curl -s -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -d "{\"api_key\": \"$TavilyToken\", \"query\": \"<query>\", \"search_depth\": \"<basic|advanced>\", \"max_results\": 5}"
```

`TavilyToken` 未设置或调用失败时，跳过搜索，仅使用训练数据继续。响应为 JSON，从 `results[].content` 提取正文，`results[].url` 获取来源链接。

## Step-by-Step Instructions

### Step 1: Clarify Before Generating (if needed)

**When to skip**: Domain is specific enough (e.g., "Rust", "MLOps") OR the user already stated goal/background in their message.

**When to ask**: Domain is vague or very broad (e.g., "AI", "programming"), OR the domain is ambiguous enough that the timeline and competitive scoping would be unclear. Send **one message** covering both dimensions as a combo grid:

| | 🆕 新手 (no prior background) | 🔁 有相邻经验 (adjacent field) | 📚 中级 (some experience in this field) |
|---|---|---|---|
| 🔧 工程实践 (build & ship) | A1 | A2 | A3 |
| 🔬 研究深造 (research & papers) | B1 | B2 | B3 |
| 🌐 通识了解 (general understanding) | C1 | C2 | C3 |

Reply with a combo like **A1**, **B2**, etc. — or describe your situation directly. Reply "跳过" to generate with defaults.

**Fallbacks**:
- If user says "just generate" / "skip" / doesn't answer → proceed with defaults: engineer goal, beginner level, apply to the domain as-is
- If the request is not a learning intent (e.g., "debug my code", "explain this error") → gently redirect: "It looks like you need hands-on help rather than a learning roadmap — do you mean you'd like a structured outline for learning [related topic]?"
- If the domain is highly niche or specialized (e.g., "3D thermodynamic FEM analysis", "medieval historiography methods") and you have low confidence in covering it comprehensively → generate the outline at a higher level of abstraction (fewer sub-topics, broader branches), and add a note at the top: "⚠️ 这是一个专业细分领域，以下大纲基于通用学习框架，建议结合领域内的权威教材或专家建议进行调整。"

### Step 2: Generate the 4-Level Outline

**Fast-moving domain check** — before generating, judge whether the domain evolves rapidly (e.g., AI/ML, LLMOps, cloud-native, frontend frameworks, cybersecurity). If yes, proactively search 1–2 authoritative sources to ground the outline in current reality. 用 Tavily 搜索（`basic`，见顶部工具说明），建议查询词：`<domain> current state 2025`、`<domain> roadmap 2025`、`best way to learn <domain> 2025 site:reddit.com`。重点参考官方文档、roadmap.sh、HN/Reddit 社区共识等来源的结果。

Use findings to adjust the branch structure, module priorities, and timeline before writing anything. For stable domains (algorithms, operating systems, economics), skip this — training data is sufficient.

**Background adaptation** (if user provided goal/background in Step 1 or original message):
- Experienced in adjacent field → mark overlapping modules as `P3 进阶` / `🔴 高级`; add a note "(可快速过)" in their 简介
- Beginner with no prior background → emphasize `P1 必学` / `🟢 入门` modules first; keep sub-topics fewer and more concrete
- Researcher goal → elevate theory/paper-reading modules to P1; deprioritize production tooling
- Engineer/practitioner goal → elevate hands-on modules to P1; deprioritize pure theory

Structure the output as follows:

```
领域 (Domain)
│
├── 🧠 核心心智模型 (Core Mental Model)
│   ├── 一句话直觉: [用生活类比点破这个领域的本质，让门外汉也能"哦原来如此"]
│   ├── 解决的问题: [没有它之前，人们用什么痛苦的方式处理这个问题]
│   └── 最常见误区: [初学者最容易产生的错误认知，一句话点明]
│
├── 📜 纵向时间线 (Development Timeline)
│   └── [key milestone 1] → [milestone 2] → [milestone 3] → ...
│
├── 🔄 横向竞品对比 (Competitive Landscape)
│   ├── [Alternative A] — [one-line differentiator]
│   ├── [Alternative B] — [one-line differentiator]
│   └── [Alternative C] — [one-line differentiator]
│
├── 🌿 Branch 1 — [one-sentence description]
│   ├── 📦 Module 1.1 — [description] [🟢/🟡/🔴] [P1/P2/P3]
│   │   ├── 📌 Sub-topic 1.1.1 — [5-word hint]
│   │   └── 📌 Sub-topic 1.1.2
│   └── 📦 Module 1.2 ...
└── 🌿 Branch 2 ...
```

#### 前置节 1 — 🧠 核心心智模型 (Core Mental Model)

The goal is a single "aha" paragraph that anchors everything else. Before the learner sees any timeline or roadmap, they need one mental peg to hang everything on.

Write three short bullets:

- **一句话直觉**: A concrete, everyday analogy that captures the domain's essence. Avoid using technical comparisons — compare to cooking, city planning, a library, a game, etc. The test: could a middle-schooler nod along?
- **解决的问题**: Describe the pain that existed *before* this domain/tool existed. Make it vivid. "Before X, engineers had to manually..." is the pattern. The contrast makes the domain feel necessary, not abstract.
- **最常见误区**: Name the single most common wrong mental model that beginners carry. Be specific to this domain — don't write generic advice. Example: "初学者常以为 Docker 就是虚拟机，但它共享宿主内核，启动快百倍、但隔离性也弱得多。"

Keep the whole section to 3–5 lines. The point is sharpness, not comprehensiveness.

#### 前置节 2 — 📜 纵向时间线 (Development Timeline)

Help the learner understand the domain's evolution — key turning points, paradigm shifts, and major releases that shaped the current landscape. This gives context for WHY things are the way they are.

- **Length**: 3–7 key milestones, each one line
- **Scope by domain type**:
  - **工具/语言/框架** (e.g., Rust, Kubernetes, React): list origin, major version releases, ecosystem inflection points
  - **概念/学科** (e.g., Machine Learning, Cybersecurity): list key eras, breakthrough papers, methodological paradigm shifts
  - **泛领域** (e.g., Product Management): list evolution of the discipline, influential books/thought leaders
- **Format**: a chronological chain `[年份/时期] 事件 — 一句话影响`, connected by `→` arrows or as a bulleted timeline

#### 前置节 3 — 🔄 横向竞品对比 (Competitive Landscape)

Help the learner understand where this domain/tool fits in the broader ecosystem. What are the alternatives, and when would you choose each?

- **Length**: 3–5 alternatives, each with a one-line description + key differentiator
- **Scope by domain type**:
  - **工具/语言/框架**: compare to direct alternatives (e.g., Rust → C++, Go, Zig; React → Vue, Angular, Svelte)
  - **概念/领域**: compare different schools of thought, methodologies, or sub-approaches (e.g., ML → supervised vs unsupervised vs RL; Databases → SQL vs NoSQL vs NewSQL)
  - **泛领域**: compare adjacent roles, frameworks, or lenses for approaching the domain
- **Format**: a comparison list. End with a concise **"选型建议" (selection guide)**: when to pick this domain's subject over its alternatives

#### Level 1 — Domain Overview
Start with a 2-3 sentence description of the domain: what it is, why it matters, and the overall learning journey.

#### Level 2 — Branches (3–7 branches)
Major sub-disciplines or thematic pillars of the domain. Each branch gets:
- An emoji icon for quick visual scanning
- A one-sentence description of what this branch covers

#### Level 3 — Modules (3–6 per branch)
Concrete learning units within each branch. Each module gets:
- **📘 简介 (Description)**: Two sentences in problem-driven order — first, name the concrete pain or confusion that arises *without* this knowledge ("不懂这个，你会…" or "没有它，开发者只能…"); second, explain how this module resolves that pain. This framing makes every module feel necessary rather than academic.
  - Bad: "介绍异步编程的概念和 async/await 语法。"
  - Good: "不懂异步，你的程序一旦等待网络或磁盘就会完全卡死，用户体验崩溃。async/await 让你写出'看起来同步、实际不阻塞'的代码。"
- **⚡ 难度 (Difficulty)**: `🟢 入门` / `🟡 中级` / `🔴 高级`
- **⭐ 优先级 (Priority)**: `P1 必学` / `P2 推荐` / `P3 进阶`

#### Level 4 — Sub-topics (3–6 per module)
Specific concepts, techniques, or skills within each module. Keep these concise (title only or title + 5-word hint).

### Step 3: Learning Path Section

After the outline, add a **📍 推荐学习路径 (Recommended Learning Path)** section.

**Selection rules** (use actual module names from the outline above, not placeholders):
- 🚀 **快速入门路径** (~1-3 months): all `P1 必学` modules only, ordered by dependency (foundational first)
- 🎯 **系统学习路径** (~6-12 months): all `P1 + P2` modules, grouped by branch in logical progression order
- 🔬 **深度进阶路径** (ongoing): `P3 进阶` modules + cross-branch synthesis topics

If user background was provided, annotate paths accordingly:
- Adjacent field experience → mark skippable modules with `(可跳过)`
- Researcher goal → swap production-tooling modules for paper/theory equivalents in the path

Each step in the path must state its learning purpose — not just what to study, but why this step comes here and what it unlocks. The format for each step is:

```
→ [模块名] — 学完能做到：[一句话描述学完后新获得的能力或理解]
              为什么在这里：[一句话说明它为后续步骤铺路，或没有它后续无法进行]
```

The "学完能做到" line should describe a concrete, observable outcome ("能独立写出…", "能看懂…", "能排查…"), not an abstract concept. The "为什么在这里" line should make the dependency or sequencing logic explicit — if the order doesn't matter, say so.

Apply this format to all three paths:

```
🚀 快速入门路径 (Quick Start — ~1-3 months)
→ [实际模块名A]
   学完能做到：[具体能力]
   为什么在这里：[铺路原因，或"入门起点，无前置依赖"]
→ [实际模块名B]
   学完能做到：[具体能力]
   为什么在这里：[依赖A的哪个知识点]
→ [实际模块名C] ...

🎯 系统学习路径 (Systematic — ~6-12 months)
→ Phase 1: [Branch X]
   [模块名] — 学完能做到：[能力] / 为什么在这里：[原因]
   [模块名] — 学完能做到：[能力] / 为什么在这里：[原因]
→ Phase 2: [Branch Y] ...

🔬 深度进阶路径 (Advanced — ongoing)
→ [实际P3模块名] — 学完能做到：[能力] / 为什么在这里：[原因]
```

### Step 4: Resource Recommendations

Add a **📚 推荐学习资源 (Recommended Resources)** section organized by type:

- **📖 书籍 Books**: 3–5 highly regarded books with a one-line reason
- **🎓 课程 Courses**: 3–5 courses (Coursera, edX, Udemy, YouTube channels, official docs)
- **🛠️ 实践项目 Projects**: 2–3 project ideas to apply knowledge
- **🌐 社区 Communities**: 1–2 communities (Discord, Reddit, GitHub orgs)

**Resource freshness** — always fetch latest recommendations from authoritative sources before finalising this section. 用 Tavily 搜索（`advanced`，见顶部工具说明），建议查询词：`best <domain> courses 2025 site:reddit.com`、`<domain> book 2024 OR 2025`、`awesome <domain> github`。重点参考 roadmap.sh、Reddit、HN、官方文档、awesome-* 仓库等来源的结果。

Replace weak recommendations with better ones found. Note the source inline: `（来源：roadmap.sh）`、`（来源：官方文档）`.

### Step 5: Refinement Checkpoint

After completing the full output, add a brief closing prompt:

> 以上是 [领域] 的完整学习大纲。如需调整，可以告诉我：
> - 某个分支需要更深/更浅
> - 想聚焦特定学习路径
> - 补充某类资源（如中文资料、视频课程）
> - 调整时间线精度或竞品覆盖范围

If the user responds with adjustments, update only the relevant section — do not regenerate the entire outline.

## Formatting Rules

- Use Chinese for section headers and labels, English for technical terms when appropriate
- Use markdown headers and nested lists
- Use emoji icons consistently for visual hierarchy
- Keep the outline scannable — don't over-explain at the sub-topic level
- For very large domains (e.g., "Software Engineering"), focus on the most important 4–5 branches rather than trying to cover everything
- Total output length: aim for **comprehensive but not overwhelming** — roughly 800–1500 words for the outline itself (including timeline and competitive comparison)

## Example Domains

This skill handles domains such as (but not limited to):
- Programming languages: Python, Rust, Go, TypeScript
- AI/ML: Machine Learning, Deep Learning, NLP, Computer Vision, LLMOps, MLOps
- Infrastructure: DevOps, Kubernetes, Cloud Architecture, System Design
- Data: Data Engineering, Data Science, Analytics Engineering
- Security: Cybersecurity, Web Security, Cryptography
- General CS: Algorithms & Data Structures, Operating Systems, Distributed Systems
- Non-tech (also supported): Economics, Product Management, UX Design, etc.
