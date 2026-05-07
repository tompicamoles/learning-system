---
name: learning-setup
description: "Use when the user types /learning-setup, or says 'setup learning goals', 'update my learning goals', or 'configure learning system'. One-time guided setup (or re-run to update goals) for the learning system."
---

# Learning Setup

One-time setup (or re-run to update) for the learning system. Three phases: goal coaching, write goals, and Notion DB creation.

**First run**: Create the data directory `~/.claude/skills/learning-system/` if it does not exist. This is where user-specific config and goals are stored.

## Phase 1: Goal Coaching

Run a short coaching conversation. Ask questions one at a time, wait for answers.

### Q1: Context and objective

"What are you trying to achieve? What's the context?"

Give examples to help the user frame their answer:
- "I'm a junior SW engineer and I want to reach the confirmed/senior level"
- "I'm new to this company and I want to learn the codebase and its architecture"
- "I used to work in a Java environment and I want to learn TypeScript idioms"
- "I want to skill up on DDD -- I know the theory but struggle to apply it in real code"
- "I'm preparing for a system design interview"
- "I want to understand our infrastructure (Terraform, AWS, CI/CD) better"

### Q2: Tell me more

**Codebase analysis (optional, before asking Q2):**
- If running inside a codebase, ask: "I can scan this codebase to tailor my suggestions — want me to?"
- If NOT running inside a codebase, ask: "Do you have a codebase you'd like me to analyze? If so, share the path(s)."
- If the user agrees, scan: project structure, package.json, tsconfig, Dockerfile, CI config, terraform files. Identify languages, frameworks, libraries, architectural patterns, infra tools.
- Use findings to enrich the Q2 suggestions below. If the user declines or no codebase is available, use generic suggestions based on Q1.

Then ask: "Tell me more about your situation so I can tailor the goals."

Suggest 3-4 dimensions to elaborate on, tailored to Q1 and enriched with codebase findings. For instance:

- If Q1 = "junior wanting to level up" and codebase uses NestJS + MongoDB + hexagonal:
  - "I see the codebase uses NestJS, MongoDB, DDD with hexagonal architecture, SQS, Lambdas, and Terraform. Which of these do you work with most?"
  - "What kind of tasks do you handle day-to-day? (features, bug fixes, reviews, ops)"
  - "How long have you been in the role / with the codebase?"
- If Q1 = "new to the company":
  - "I scanned the repo — here's what I found: [tech summary]. What's your previous experience with these?"
  - "What team/domain are you joining?"
- If Q1 = "learning a specific topic" (e.g. DDD):
  - "I found these DDD patterns in the codebase: [list]. Which ones have you seen vs which are new to you?"
  - "What triggered the interest — a task, a code review, curiosity?"

The user can answer all, some, or just freeform. No pressure to pick or narrow.

### Q3: Time horizon

"What's your time horizon? What do you want solid this quarter vs this year?"

### Synthesize

After the 3 questions and optional codebase analysis, synthesize into concrete, tagged goals. Present them in a table for the user to confirm, adjust, or add to before writing.

**Goals vs backlog — do not confuse them:**
- **Goals** describe the level or capability the user wants to reach. They answer "what does success look like?" Examples: "be able to justify architectural choices", "lead technical studies", "read any part of the codebase independently".
- **Backlog items** are specific topics or technologies to learn. They answer "what do I need to study?" Examples: "NestJS dynamic modules", "Lambda patterns", "MongoDB transactions".

Goals go in `learning-goals.md`. Backlog items go in the Notion backlog as cards. Do NOT list specific technologies or frameworks as goals — those are backlog cards prioritized against the goals.

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

**Important:** Notion may default Status to the workspace language (e.g. French: "Pas commencé", "Terminé"). After creating the database, verify Status values are `To Do` and `Taught`. If not, update them using the Notion MCP update-data-source tool.

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
