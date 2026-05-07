---
name: notes-discuss
description: >
  Read and discuss web clippings saved via Obsidian Web Clipper. Lists articles from
  $NOTESDIR/articles/, reads the selected one, presents a structured summary, and guides the
  user through discussion to form conclusions. After discussion, remind the user they can save
  conclusions by saying "保存到笔记" (which triggers notes-save). Trigger: "讨论文章",
  "看看收集的文章", "处理文章", "帮我看这篇文章", "discuss this article", or /notes-discuss.
  Uses $NOTESDIR environment variable.
argument-hint: "[filename — optional, picks a specific article]"
allowed-tools: Bash(test -f *), Bash(ls *), Bash(date *), Read
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

## Step 3 — Present Structured Summary

After reading, present a concise summary using this structure (keep it brief):

```
## 📋 文章概览

**标题：** <article title — extract from content>

**核心主题：** <one-line summary of what this article is about>

**关键要点：**
- <point 1>
- <point 2>
- <point 3>

**值得深入讨论的方向：**
1. <aspect to discuss 1>
2. <aspect to discuss 2>
3. <aspect to discuss 3>
```

Then ask the user: "你对这篇文章的哪些方面感兴趣？我们可以一起讨论，形成结论后你可以说'保存到笔记'来记录。"

## Step 4 — Guide Discussion

During the discussion:
- Ask critical questions about the article's claims
- Connect concepts to the user's existing notes (check categories in vault for related topics)
- Help the user form their own conclusions
- When the discussion reaches a meaningful conclusion, explicitly remind the user: "这些结论需要记录下来吗？说'保存到笔记'就可以存入你的 Obsidian 笔记。"

## Error Handling

- If `$NOTESDIR` is not set: stop and tell the user to run `export NOTESDIR=/path/to/vault` and add it to their shell profile (`~/.zshrc` or `~/.bashrc`)
- If `$NOTESDIR/articles/` does not exist: tell the user the directory doesn't exist yet and suggest creating it or checking the `$NOTESDIR` path
- If the specified file does not exist in `$NOTESDIR/articles/`: list available files and ask the user to pick from the list
