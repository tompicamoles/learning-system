---
name: quick-tutorial
description: |
  Use when the user types `/quick-tutorial`, says "quick tutorial on X", "briefly teach me X", "5 min lesson on X", or encounters an unfamiliar concept in the codebase (e.g. a pattern, annotation, or abstraction they haven't seen before). Also trigger when a coding agent suggestion references a concept the user seems unsure about.
---

# Quick Tutorial

A single-pass, exam-free micro-lesson that produces a Notion-ready pager and records the topic for future review. Target reading time: 5-10 minutes.

**REQUIRED SUB-SKILL:** `teacher` -- invoke it for every lesson. It enforces the three mandatory sections (Underlying principles, System design angle, What to take away) and codebase grounding.

## Scenario A -- Ad-hoc concept (no existing backlog card)

User encounters something and wants to understand it. Input is a topic or question, not a Notion link.

1. Invoke `teacher` for the lesson (all 3 mandatory sections + codebase grounding). 400-700 words max.
2. Output a **pager** directly in chat (format below).
3. Invoke `add-to-backlog` skill in non-interactive/programmatic mode to create a new Notion card: Status = "Taught", page body = pager content, Code Grounding = file:line references used in the lesson.
4. Append to `the exam results ledger (path from `~/.claude/skills/learning-system/learning-config.md` → `exam_results_path`)`: topic, date, code grounding paths, result = `taught-no-exam`.

## Scenario B -- Existing backlog card (user provides Notion link)

User picks a topic from the backlog. Input contains a Notion URL.

1. Read the Notion card to get: topic, code grounding paths.
2. Invoke `teacher` for the lesson, using code grounding from the card. 400-700 words max.
3. Output a **pager** directly in chat (format below).
4. Update the existing Notion card: Status -> "Taught", Last Taught -> today, page body = pager content.
5. Append to `the exam results ledger (path from `~/.claude/skills/learning-system/learning-config.md` → `exam_results_path`)`: topic, date, code grounding paths, result = `taught-no-exam`.

## Pager format

Output directly in chat as rendered markdown. Do NOT write to a file.

- H1 title, italic subline referencing codebase if applicable
- Numbered H2 sections (4-6)
- At least 1 table (cheat sheet, comparison, or "when to use / when not to")
- At most 1 concise code block
- End with a single bold one-sentence takeaway
- No emojis, no `---` horizontal rules
- 300-500 words

## Exam results ledger

Path: `the exam results ledger (path from `~/.claude/skills/learning-system/learning-config.md` → `exam_results_path`)`

Each entry: `| <topic> | <date> | <code-grounding-paths> | taught-no-exam |`

The `taught-no-exam` result signals highest priority for upcoming refresher-exam reviews.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Skipping the `teacher` skill | It is required for every lesson, no exceptions |
| Lesson over 700 words | Compress. This is a quick tutorial, not a crash course |
| Pager over 500 words | Cut. Use tables to compress information |
| Writing the pager to a file | Output directly in chat |
| Forgetting to update exam-results.md | Always append, even for Scenario B |
| Creating a new Notion card in Scenario B | Update the existing card; do not duplicate |
| Skipping Code Grounding on the Notion card | Fill it with file:line references from the lesson |
| Using `---` separators in the pager | Omit; H2 headings delimit sections |
