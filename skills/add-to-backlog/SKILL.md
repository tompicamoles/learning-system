---
name: add-to-backlog
description: >
  Use when the user says /add-to-backlog, "add to backlog", "add to learning backlog",
  "I want to learn X", or "queue topic for learning". Also used programmatically by
  the quick-tutorial skill after a lesson to create a Taught card.
---

# Add to Learning Backlog

Two modes: **interactive** (user-triggered) and **programmatic** (called by quick-tutorial).

## Mode 1: Interactive

User wants to queue a topic for later learning.

1. Read `~/.claude/skills/learning-system/learning-goals.md` for priority context
2. Read `~/.claude/skills/learning-system/learning-config.md` for the Notion DB name
3. Guided conversation:
   - Clarify what the user wants to learn
   - Recommend size: **quick-tutorial** (5-10 min, focused concept) or **crash-course** (30 min, broader topic)
   - Assign priority (P0-P3) based on goal alignment
   - If the topic is too large for a 30-min crash course, propose splitting into separate cards (one topic per card)
4. **Code Grounding**: accept whatever the user provides (file paths, snippets, references). Do NOT search the codebase -- that is the `teacher` skill's job during the actual lesson.
5. Create card in Notion "Learning Backlog" using `Notion:create-database-row`. Fields:
   - Title, Priority, Size, Status = "To Do", Goal Alignment tags, Code Grounding (if provided)

## Mode 2: Programmatic

Called by the quick-tutorial skill after delivering a lesson. Skip the guided conversation.

1. Read `~/.claude/skills/learning-system/learning-config.md` for the Notion DB name
2. Create card with:
   - Status = "Taught"
   - Code Grounding = references from the lesson
   - Page body = pager content from the lesson
3. All other fields (Title, Priority, Size, Goal Alignment) are passed by the caller

## Common Mistakes

| Mistake | Correct behavior |
|---|---|
| Searching the codebase for grounding | Only accept what the user provides; codebase search is the teacher's job |
| Skipping learning-goals.md in interactive mode | Always read it to assign accurate priority |
| Putting multiple topics on one card | One topic per card; split if too large for a 30-min crash course |
| Creating card with Status "To Do" in programmatic mode | Programmatic mode uses Status "Taught" |
| Forgetting to read learning-config.md | Always read it to get the correct Notion DB name |
