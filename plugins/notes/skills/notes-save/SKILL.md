---
name: notes-save
description: >
  Save knowledge, conclusions, or key insights from the current conversation to the user's Obsidian
  notes vault. Use when user says things like: "保存到笔记", "记录这个结论", "记下来", "记到笔记里",
  "把这个知识点记一下", "save this to notes", "add to my notes", "write this to notes", or calls /notes-save.
  Reads the vault path from the $NOTESDIR environment variable. Intelligently categorizes notes into
  subfolders, merges new knowledge into existing files, and maintains category index.md files with
  links to saved notes. Trigger: "保存到笔记", "记录这个结论", "记下来", "save this to notes", /notes-save.
---

# Notes Save — Save to Obsidian Vault

Save key knowledge or conclusions from the current conversation to the user's Obsidian vault.

## Environment Check

Vault path: !`echo "${NOTESDIR:-⚠️  NOTESDIR is not set}"`

Existing categories:
!`[ -d "${NOTESDIR}" ] && ls -1d "${NOTESDIR}"/*/  2>/dev/null | while IFS= read -r d; do basename "$d"; done | sort || echo "(vault not found or no categories yet)"`

Today's date: !`date +%Y-%m-%d`

## Step 1 — Extract New Knowledge

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

## Step 3 — Write to Vault

Target path: `$NOTESDIR/<category>/<topic>.md`

### If the file does NOT exist

1. `mkdir -p "$NOTESDIR/<category>"`
2. Format a new note using this Obsidian-compatible markdown structure.  
   **Omit any section that has no content — do not add empty headings.**

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

3. Use the `Write` tool to create the file with the formatted content.

### If the file ALREADY exists — Merge and Reorganize

**Goal: produce ONE coherent, well-structured note. Not an append log.**

1. Use the `Read` tool to load the full existing content.

2. Analyze both the existing note and the new knowledge side by side:
   - What does the existing note already cover?
   - What is genuinely new from this conversation?
   - Are there sections in the existing note that should be expanded?
   - Are there overlapping points that should be consolidated?
   - Does the structure still make sense, or does it need reorganization?

3. Write a single merged document following these rules:

   **Content rules:**
   - **No duplicates**: if the existing note already explains something, do not repeat it — expand or refine it instead
   - **Integrate, don't append**: place new knowledge in the most logical position within the document (inside an existing section, as a new sub-heading, or as a new top-level section)
   - **Consolidate**: if the same concept is scattered across multiple places, merge them into one coherent explanation
   - **Preserve everything**: do not silently drop any existing content — only restructure or move it
   - **Update summary**: always refresh the `## 要点总结` section to reflect the full combined knowledge

   **Structure rules:**
   - Keep the `# <Title>` and the `> 来源 / 日期` header block, updating the date metadata:
     ```
     > 来源：Claude Code 对话
     > 创建：<original creation date>
     > 更新：<today's date>
     ```
   - Use `###` sub-headings freely to separate distinct sub-topics within a section
   - Sections are ordered by: 背景/问题 → 核心概念 → 示例 → 注意事项 → 要点总结
   - New top-level sections (e.g., `## 进阶用法`, `## 对比`) may be added if the new knowledge genuinely introduces a new dimension not covered by existing sections

   **Readability rules:**
   - The final note must read as a document written by one author at one time — no "补充", "追加", or date-stamped dividers mid-document
   - Prefer flowing prose or well-structured bullet lists over dense paragraphs
   - Every code example must be preserved verbatim in a fenced code block with language tag

4. Use the `Write` tool to overwrite the file with the reorganized content.

## Step 4 — Maintain Category Index

After saving or merging the note, update the category index at:

`$NOTESDIR/<category>/index.md`

The index is not a changelog and should not be organized around the current conversation. It should describe the knowledge domain for that category, then link notes into the appropriate module.

### If `index.md` does NOT exist

Create it with this structure:

```
# <Category> 知识索引

这个索引按 <Category> 领域的知识结构组织笔记。

## 领域模块

### <major module 1>

- [[<topic>]]：<one-line description>

### <major module 2>

- 暂无笔记
```

Guidelines:
- Define broad, stable modules for the category first; do not build the index only from the current note
- Keep modules complete enough to guide future notes, but not overly detailed
- Add the saved note under the best-fitting module using an Obsidian wikilink
- Use `[[<topic>]]`, not absolute paths

### If `index.md` ALREADY exists

1. Use `Read` to load it.
2. Check whether the saved note is already linked.
3. If already linked, only adjust the one-line description if the note's meaning changed.
4. If not linked, add it under the best-fitting existing module.
5. If no existing module fits, add one broad module and place the link there.
6. Preserve the index's existing structure and wording as much as possible.

Index quality rules:
- Do not add "整理建议", "待处理", "本次保存", or discussion notes to `index.md`
- Do not include merge/split/review commentary in the index
- Avoid duplicate links to the same note
- Prefer stable domain modules over transient conversation topics
- Keep descriptions concise: one sentence or less

Use `Write` if creating or fully reorganizing the index; use `Edit` for small targeted updates.

## Step 5 — Report

After writing, tell the user:
- ✅ File path saved to (full path)
- ✅ Index path updated (full path)
- Category chosen and why
- Whether it was a new file or a merge+reorganize
- Whether the index entry was added, updated, or already present
- A one-sentence description of what new knowledge was integrated

## Error Handling

- If `$NOTESDIR` is not set: stop and tell the user to run `export NOTESDIR=/path/to/vault` and add it to their shell profile (`~/.zshrc` or `~/.bashrc`)
- If `$NOTESDIR` path doesn't exist as a directory: stop and confirm with the user before creating it
