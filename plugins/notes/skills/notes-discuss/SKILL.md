---
name: notes-discuss
description: >
  Use when the user wants to discuss, analyze, or process web-clipped articles in
  $NOTESDIR/articles/. Trigger on phrases like "讨论文章", "看看收集的文章",
  "处理文章", "帮我看这篇文章", "discuss this article", or /notes-discuss.
---

# Notes Discuss — Discuss Collected Articles

Read a collected article from the Obsidian article directory, summarize it, and guide the user through discussion to form conclusions.

## Environment Check

Vault path: !`echo "${NOTESDIR:-⚠️  NOTESDIR is not set}"`

文章目录: !`echo "${NOTESDIR}/articles"`

已有文章:
!`[ -d "${NOTESDIR}/articles" ] && ls -1 "${NOTESDIR}/articles/"*.md 2>/dev/null | while IFS= read -r f; do basename "$f"; done | sort || echo "(还没有文章)"`

Today's date: !`date +%Y-%m-%d`

## Step 1 — Identify the Article

- If `$ARGUMENTS` contains a filename (e.g., `/notes-discuss my-article.md`), use that file from `$NOTESDIR/articles/`
- If `$ARGUMENTS` is empty, show the list of available articles from the Environment Check and ask the user to pick one (by number or name)
- If the user references an article by topic in conversation, fuzzy-match against the file list

**文章目录:** `$NOTESDIR/articles/`

## Step 2 — Read the Article

Read the selected article file using the `Read` tool.

## Step 3 — Present Comprehensive Analysis

After reading, present a thorough, detailed analysis of the article. Do NOT summarize briefly — go deep. The user expects a complete understanding before discussion begins.

Use this structure (expand each section with substantive detail; skip only if truly inapplicable):

```
## 📋 文章深度分析

### 基本信息
- **标题：** <article title>
- **作者/来源：** <if available>
- **发表日期：** <if available>
- **文章类型：** <tutorial / opinion / technical deep-dive / news / research / review / case-study>

### 核心论题

<What is the central question, problem, or argument the article addresses? State it clearly in 2–4 sentences.>

### 内容详析

<Break down the article's content section by section. For each major section or logical block, explain:
- What the author is saying
- The key arguments, data, or evidence presented
- How this fits into the article's overall argument>

### 关键概念解析

<Identify and explain every important concept, term, technique, or tool mentioned in the article. For each:
- Definition and context
- How the article uses it
- Any nuance or subtlety worth noting>

### 论据与佐证

<What evidence does the article provide for its claims?
- Data, benchmarks, case studies, cited research
- Code examples, demos, or practical illustrations
- Assess the quality and persuasiveness of the evidence>

### 实践启示 / 可操作要点

<What can the reader DO with this information?
- Concrete techniques, workflows, or approaches
- Code patterns, commands, or configurations worth adopting
- Decision-making frameworks or mental models>

### 局限性与批判性思考

<What does the article miss or gloss over?
- Assumptions the author makes
- Counterarguments or alternative viewpoints
- Scenarios where the advice might not apply
- Outdated or evolving information to be aware of>

### 与已有知识体系的关联

<Connect to broader knowledge domains:
- How does this relate to well-known principles, patterns, or technologies?
- Does it reinforce, contradict, or extend common understanding?
- Check the user's vault categories for related notes and category `index.md` files; mention where the article's conclusions would fit in the existing knowledge structure>
```

After presenting the complete analysis, ask: "分析完了。你对这篇文章的哪些方面想深入讨论？"

## Step 4 — Deep Discussion

Now engage in substantive, detailed discussion. This is the core value — don't rush through it.

**Discussion principles:**
- Probe the article's claims with specific, critical questions — don't just accept them
- Ask the user for their own experience or opinion on each key point
- When the user mentions a related concept, explore that connection in depth
- Challenge assumptions: "作者这里假设了X，但如果在Y场景下呢？"
- Offer concrete scenarios or counter-examples to test the article's ideas
- Draw connections to the user's existing notes (use the vault categories from the Environment Check)

**Depth over breadth:** Pick 2–3 of the most interesting directions and explore them thoroughly, rather than skimming all topics. Follow the user's curiosity — if they latch onto one aspect, go deeper on that.

**Conclusion prompting:** When a thread of discussion reaches a meaningful insight or decision, pause and explicitly remind: "这个结论值得记录下来。说'保存到笔记'就可以存入你的 Obsidian 笔记，并自动维护对应分类的 index.md。"

**Index handoff:** If the discussion identifies a likely category or module for the conclusion, say it explicitly so notes-save can use it as context. Example: "这个结论适合放到 AI / Transformer 与大模型架构 模块。" Do not edit index.md from notes-discuss; notes-save owns writing notes and maintaining indexes.

## Error Handling

- If `$NOTESDIR` is not set: stop and tell the user to run `export NOTESDIR=/path/to/vault` and add it to their shell profile (`~/.zshrc` or `~/.bashrc`)
- If `$NOTESDIR/articles/` does not exist: tell the user the directory doesn't exist yet and suggest creating it or checking the `$NOTESDIR` path
- If the specified file does not exist in `$NOTESDIR/articles/`: list available files and ask the user to pick from the list
