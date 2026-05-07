---
name: learning-setup
description: "Use when the user types /learning-setup, or says 'setup learning goals', 'update my learning goals', or 'configure learning system'. One-time guided setup (or re-run to update goals) for the learning system."
---

# Learning Setup

One-time setup (or re-run to update) for the learning system. Two phases: goal coaching and Notion DB creation.

## Phase 1: Goal Coaching

Run a short coaching conversation to help the user articulate skill-up goals. Ask questions one at a time, wait for answers:

1. "What does 'next level' mean for you right now?"
2. "Where do you feel the biggest gaps in your day-to-day work?"
3. "What patterns do you encounter daily but don't fully understand?"
4. "Are there architectural or design topics you want to go deeper on?"
5. "What's your time horizon -- what do you want nailed this quarter vs this year?"

Synthesize answers into concrete, tagged goals. Present the proposed goals in a table for confirmation before writing.

## Phase 2: Write Goals File

Write `~/.claude/skills/learning-system/learning-goals.md` with this structure:

```markdown
# Learning Goals

## Current Quarter
| Goal | Tag |
|------|-----|
| ... | #tag |

## Current Year
| Goal | Tag |
|------|-----|
| ... | #tag |

## Active Goals
- [ ] Goal description (#tag)

## Completed
- [x] Goal description (#tag) -- completed YYYY-MM-DD
```

## Phase 3: Notion DB Setup

**Skip this phase if `~/.claude/skills/learning-system/learning-config.md` already contains a `notion_db_id` value.**

1. Ask the user for an empty Notion page URL.
2. REQUIRED SUB-SKILL: Use Notion MCP tools (`mcp__plugin_Notion_notion`) to authenticate if needed.
3. Create a database titled "Learning Backlog" inside that page with these properties:

| Property | Type | Details |
|----------|------|---------|
| Title | title | Topic name. One topic per card. |
| Priority | select | P0-critical, P1-high, P2-medium, P3-low |
| Size | select | quick-tutorial, crash-course |
| Status | status | To Do, Taught |
| Goal Alignment | multi_select | Tags matching goals from Phase 2 |
| Code Grounding | rich_text | File paths for codebase grounding |
| Last Taught | date | Date of last lesson |
| Notes | rich_text | Free-form notes |

Page body is reserved for pager content written by the `teacher` skill after teaching.

4. Resolve the exam results ledger path. The ledger must live in the user's project-specific Claude memory directory. Determine the path by:
   - Finding the current project's memory directory (typically `~/.claude/projects/<project-hash>/memory/`)
   - The file is named `exam-results.md`
   - If the memory directory does not exist, create it

5. Write `~/.claude/skills/learning-system/learning-config.md`:

```markdown
# Learning System Config

notion_db_id: <database-id>
notion_page_url: <original-page-url>
exam_results_path: <resolved-absolute-path-to-exam-results.md>
```

## Re-run Behavior

When re-run with existing config:
- Phase 1 and 2 always execute (goals may evolve).
- Phase 3 is skipped if `learning-config.md` already has a `notion_db_id`. Inform the user the DB is already set up and show the existing page URL.
