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

## Output Structure

Always output the outline in this 4-level hierarchy:

```
领域 (Domain)
├── 🌿 分支 Branch 1 — [branch description]
│   ├── 📦 模块 Module 1.1 — [module description] [difficulty] [priority]
│   │   ├── 📌 子主题 1.1.1 — [brief description]
│   │   ├── 📌 子主题 1.1.2
│   │   └── 📌 子主题 1.1.3
│   └── 📦 Module 1.2 ...
├── 🌿 Branch 2 ...
└── ...
```

## Step-by-Step Instructions

### Step 1: Clarify Before Generating (if needed)

**When to skip**: Domain is specific enough (e.g., "Rust", "MLOps") OR the user already stated goal/background in their message.

**When to ask**: Domain is vague or very broad (e.g., "AI", "programming"). Ask ONE question:
- **Goal**: Are you aiming to be a practitioner/engineer, researcher, or general understanding?
- **Background**: Beginner, intermediate, or experienced in adjacent fields?

**Fallbacks**:
- If user says "just generate" / "skip" / doesn't answer → proceed with defaults: engineer goal, beginner level, apply to the domain as-is
- If the request is not a learning intent (e.g., "debug my code", "explain this error") → gently redirect: "It looks like you need hands-on help rather than a learning roadmap — do you mean you'd like a structured outline for learning [related topic]?"

### Step 2: Generate the 4-Level Outline

**Background adaptation** (if user provided goal/background in Step 1 or original message):
- Experienced in adjacent field → mark overlapping modules as `P3 进阶` / `🔴 高级`; add a note "(可快速过)" in their 简介
- Beginner with no prior background → emphasize `P1 必学` / `🟢 入门` modules first; keep sub-topics fewer and more concrete
- Researcher goal → elevate theory/paper-reading modules to P1; deprioritize production tooling
- Engineer/practitioner goal → elevate hands-on modules to P1; deprioritize pure theory

Structure the output as follows:

#### Level 1 — Domain Overview
Start with a 2-3 sentence description of the domain: what it is, why it matters, and the overall learning journey.

#### Level 2 — Branches (3–7 branches)
Major sub-disciplines or thematic pillars of the domain. Each branch gets:
- An emoji icon for quick visual scanning
- A one-sentence description of what this branch covers

#### Level 3 — Modules (3–6 per branch)
Concrete learning units within each branch. Each module gets:
- **📘 简介 (Description)**: 1–2 sentences explaining what this module covers and why it matters
- **⚡ 难度 (Difficulty)**: `🟢 入门` / `🟡 中级` / `🔴 高级`
- **⭐ 优先级 (Priority)**: `P1 必学` / `P2 推荐` / `P3 进阶`

#### Level 4 — Sub-topics (3–6 per module)
Specific concepts, techniques, or skills within each module. Keep these concise (title only or title + 5-word hint).

### Step 3: Learning Path Section

After the outline, add a **📍 推荐学习路径 (Recommended Learning Path)** section:

```
🚀 快速入门路径 (Quick Start — ~1-3 months)
→ [Module A] → [Module B] → [Module C]

🎯 系统学习路径 (Systematic — ~6-12 months)
→ Phase 1: [Branch X] (Modules: ...)
→ Phase 2: [Branch Y] (Modules: ...)
→ Phase 3: [Branch Z] (Modules: ...)

🔬 深度进阶路径 (Advanced — ongoing)
→ Focus areas: [...]
```

### Step 4: Resource Recommendations

Add a **📚 推荐学习资源 (Recommended Resources)** section organized by type:

- **📖 书籍 Books**: 3–5 highly regarded books with a one-line reason
- **🎓 课程 Courses**: 3–5 courses (Coursera, edX, Udemy, YouTube channels, official docs)
- **🛠️ 实践项目 Projects**: 2–3 project ideas to apply knowledge
- **🌐 社区 Communities**: 1–2 communities (Discord, Reddit, GitHub orgs)

### Step 5: Refinement Checkpoint

After completing the full output, add a brief closing prompt:

> 以上是 [领域] 的完整学习大纲。如需调整，可以告诉我：
> - 某个分支需要更深/更浅
> - 想聚焦特定学习路径
> - 补充某类资源（如中文资料、视频课程）

If the user responds with adjustments, update only the relevant section — do not regenerate the entire outline.

## Formatting Rules

- Use Chinese for section headers and labels, English for technical terms when appropriate
- Use markdown headers and nested lists
- Use emoji icons consistently for visual hierarchy
- Keep the outline scannable — don't over-explain at the sub-topic level
- For very large domains (e.g., "Software Engineering"), focus on the most important 4–5 branches rather than trying to cover everything
- Total output length: aim for **comprehensive but not overwhelming** — roughly 600–1200 words for the outline itself

## Example Domains

This skill handles domains such as (but not limited to):
- Programming languages: Python, Rust, Go, TypeScript
- AI/ML: Machine Learning, Deep Learning, NLP, Computer Vision, LLMOps, MLOps
- Infrastructure: DevOps, Kubernetes, Cloud Architecture, System Design
- Data: Data Engineering, Data Science, Analytics Engineering
- Security: Cybersecurity, Web Security, Cryptography
- General CS: Algorithms & Data Structures, Operating Systems, Distributed Systems
- Non-tech (also supported): Economics, Product Management, UX Design, etc.
