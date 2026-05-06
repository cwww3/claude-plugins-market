---
name: notes
description: >
  Save knowledge, conclusions, or key insights from the current conversation to the user's Obsidian
  notes vault. Use when user says things like: "保存到笔记", "记录这个结论", "记下来", "记到笔记里",
  "把这个知识点记一下", "save this to notes", "add to my notes", "write this to notes", or calls /notes.
  Reads the vault path from the $NOTESDIR environment variable. Intelligently categorizes notes into
  subfolders, and appends to existing files rather than overwriting.
argument-hint: "[topic hint — optional]"
allowed-tools: Bash(test -f *), Bash(mkdir -p *), Bash(ls *), Bash(date *), Read, Write
---

# Notes — Save to Obsidian Vault

Save key knowledge or conclusions from the current conversation to the user's Obsidian vault.

## Environment Check

Vault path: !`echo "${NOTESDIR:-⚠️  NOTESDIR is not set}"`

Existing categories:
!`[ -d "${NOTESDIR}" ] && ls -1d "${NOTESDIR}"/*/  2>/dev/null | while IFS= read -r d; do basename "$d"; done | sort || echo "(vault not found or no categories yet)"`

Today's date: !`date +%Y-%m-%d`

## Step 1 — Extract Content

Look at the current conversation and identify what to save:
- If `$ARGUMENTS` was provided, treat it as a topic hint and find the relevant conclusions
- Otherwise, identify the most recent and significant knowledge/conclusion discussed

**Completeness requirements — do NOT omit:**
- The core problem or question being addressed
- All key concepts, mechanisms, or principles explained
- Every important conclusion or decision and the reasoning behind it
- All code snippets, commands, or configuration examples
- Caveats, pitfalls, and edge cases mentioned
- Any "why" explanations (not just "what")

## Step 2 — Determine Topic and Category

**Topic name** rules:
- Concise noun phrase (3–8 words), no date prefix
- Use the same language as the primary content (Chinese or English)
- Good examples: `Docker网络模型`, `Git Rebase最佳实践`, `React useEffect清理函数`, `JWT认证流程`
- Bad examples: `今天学到的关于Docker的知识`, `Conversation about networking`

**Category folder** rules:
- Pick from existing categories if one fits well
- Common categories to create if none exists: `AI`, `Backend`, `Frontend`, `DevOps`, `Database`,
  `Networking`, `Security`, `Linux`, `Tools`, `Architecture`, `Languages`, `Algorithms`
- If the topic spans multiple categories, pick the most specific one

## Step 3 — Format Note Content

Use this Obsidian-compatible markdown structure. **Omit any section that has no content — do not add empty headings.**

```
# <Topic Name>

> 来源：Claude Code 对话
> 日期：<YYYY-MM-DD>

## 背景 / 问题

<What problem or question prompted this topic. Why does it matter.>

## 核心概念 / 解决方案

<Full explanation of the mechanism, concept, or solution. Cover all key points discussed.
Use sub-headings (###) freely to organize multi-part content.>

## 示例

<Code snippets, commands, or concrete examples. Use fenced code blocks with language tags.>

## 注意事项

- <Caveat, pitfall, or edge case 1>
- <Caveat, pitfall, or edge case 2>

## 要点总结

- <Key takeaway 1>
- <Key takeaway 2>
- <Key takeaway 3>
```

**Writing rules:**
- Write in the same language as the primary content (Chinese or English)
- Prefer completeness over brevity — a longer note that captures everything is better than a short one that loses context
- Sub-headings and bullet lists over dense paragraphs
- Every code example from the conversation must be preserved verbatim

## Step 4 — Write to Vault

Target path: `$NOTESDIR/<category>/<topic>.md`

**If the file does NOT exist:**
1. `mkdir -p "$NOTESDIR/<category>"`
2. Use the `Write` tool to create the file with the formatted content from Step 3

**If the file ALREADY exists:**
1. Use the `Read` tool to get the existing content
2. Construct the combined content: existing content + separator + new section:
   ```
   <existing content>

   ---

   > 补充日期：<YYYY-MM-DD>

   <new content from Step 3, omitting the # header since it's already there>
   ```
3. Use the `Write` tool to overwrite the file with the combined content

## Step 5 — Report

After writing, tell the user:
- ✅ File path saved to (full path)
- Category chosen and why
- Whether it was a new file or an append
- Topic name used

## Error Handling

- If `$NOTESDIR` is not set: stop and tell the user to run `export NOTESDIR=/path/to/vault` and add it to their shell profile (`~/.zshrc` or `~/.bashrc`)
- If `$NOTESDIR` path doesn't exist as a directory: stop and confirm with the user before creating it
