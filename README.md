# Learning System

A structured skill-up system for software engineers, powered by Claude Code skills. It combines daily crash courses, on-the-fly tutorials, spaced-repetition exams, and a prioritized Notion backlog -- all grounded in your actual codebase.

## The Problem

As a software engineer working in a large codebase, you encounter patterns, architectures, and design decisions daily that you understand enough to work with but not enough to reason about independently. Reading documentation helps, but without active recall and spaced repetition, knowledge fades within days.

## The Solution

Six Claude Code skills that form a complete learning loop:

```
  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
  │ add-to-backlog│─────>│ crash-course │─────>│ refresher-exam│
  │ (queue topic) │      │ (30 min)     │      │ (spaced rep.) │
  └──────────────┘      └──────────────┘      └───────┬───────┘
         ▲                      │                      │
         │                      │                      │ poorly scored?
         │               ┌──────▼──────┐               │ reschedule
         └───────────────│quick-tutorial│<──────────────┘
                         │ (5-10 min)  │
                         └─────────────┘
```

Every lesson is grounded in real code from your codebase (file:line citations), and every exam question references actual code you work with daily.

## Architecture

### Skills

| Skill | Command | Duration | What it does |
|-------|---------|----------|-------------|
| `learning-setup` | `/learning-setup` | 10-15 min | One-time setup: coaching conversation to define learning goals + Notion DB creation |
| `crash-course` | `/crash-course` | 30 min | Lesson + exam + correction + Notion pager. Full active-recall loop. |
| `quick-tutorial` | `/quick-tutorial` | 5-10 min | Micro-lesson + pager. No exam. For on-the-fly learning. |
| `refresher-exam` | `/refresher-exam` | 10-15 min | Spaced-repetition quiz on previously taught topics. |
| `add-to-backlog` | `/add-to-backlog` | 2-5 min | Add a topic to the Notion learning backlog with priority and size. |
| `teacher` | (sub-skill) | Variable | Core teaching engine. Invoked by crash-course and quick-tutorial. |

### Data Flow

```
learning-goals.md ──────── read by all skills for priority context
       │
       ├── crash-course ──> exam-results.md ──> Notion card (pager)
       ├── quick-tutorial ─> exam-results.md ──> Notion card (pager)
       └── refresher-exam ─> exam-results.md (read + append)
                                                  │
                                    add-to-backlog ├── Notion "Learning Backlog" DB
```

### Local-first Design

- **Source of truth**: Local `.md` files (zero latency, no network dependency)
  - `learning-goals.md` -- north-star goals, read by every skill
  - `learning-config.md` -- Notion DB reference + exam ledger path
  - `exam-results.md` -- append-only exam history ledger (stored in your project's Claude memory directory, path resolved during `/learning-setup`)
- **Derived view**: Notion (nice UI, mobile access, shareable pagers)
  - "Learning Backlog" database with topic cards
  - Each card's page body contains the lesson pager

### Spaced Repetition (SM-2 Inspired)

The refresher-exam skill uses a simplified SM-2 algorithm:

1. Each topic has an ease factor (EF) starting at 2.5
2. Incorrect answers decrease EF (-0.3), correct answers increase it (+0.1)
3. Review intervals grow with EF: 1 day, 3 days, then previous_interval x EF
4. Topics aligned with learning goals get a 1.5x priority boost
5. Topics from quick-tutorials (never tested) get highest priority

## Setup Tutorial

### Prerequisites

- Claude Code with skills support
- A Notion workspace
- A codebase to learn from

### Step 1: Install the skills

```bash
git clone https://github.com/tompicamoles/learning-system.git ~/.claude/skills/learning-system
```

Or manually copy the `learning-system/` folder to `~/.claude/skills/learning-system/`.

### Step 2: Run the setup

```
/learning-setup
```

This starts a guided coaching conversation to define your learning goals. You'll be asked:
- What "next level" means for you
- Where your biggest gaps are
- What patterns you encounter daily but don't fully understand
- Your time horizon (this quarter vs this year)

Then provide an empty Notion page URL. The skill creates the "Learning Backlog" database inside it with the correct schema. It also auto-detects your project's memory directory and creates the exam results ledger there.

### Step 3: Populate the backlog

```
/add-to-backlog "outbox pattern"
```

The skill guides you through priority and size assignment based on your goals.

## Daily Usage

### Morning routine (30 min)

Pick a topic from your Notion backlog and run:

```
/crash-course https://notion.so/your-topic-card
```

You get: a lesson grounded in your codebase, a 3-4 question exam, correction, and a Notion pager saved to the card.

### During the day (5-10 min)

Encounter something unfamiliar in the code? Run:

```
/quick-tutorial "explain the ACL pattern I see in this file"
```

You get: a micro-lesson with a pager. A Notion card is automatically created so the topic enters the review cycle.

### Spot something to learn later

```
/add-to-backlog "NestJS dynamic modules"
```

Creates a prioritized card in your backlog.

### Weekly review (10-15 min)

```
/refresher-exam
```

The skill picks 3-4 questions from topics due for review, weighted by your goals. Topics you got wrong come back more often. Topics from quick-tutorials (never tested) come first.

### Update goals periodically

```
/learning-setup
```

Re-run anytime to update your goals as they evolve.

## How It Works Under the Hood

### The Teacher Skill

Every lesson passes through the `teacher` skill, which enforces:

1. **Three mandatory sections**: Underlying Principles, System Design Angle, What to Take Away
2. **Codebase grounding**: search for real code examples before explaining, cite file:line numbers
3. **Teaching loop**: diagnose, explain with analogy, demonstrate with real code, break down step by step, name common pitfalls, suggest practice

This ensures consistent quality whether invoked by crash-course or quick-tutorial.

### Why Separate Skills Instead of One

Each skill has a distinct trigger, duration, and output shape. Claude Code skills trigger most accurately when focused -- a single mega-skill with branching logic would have a vague description that either fires too often or not enough. The `teacher` skill serves as the shared atomic unit that all teaching skills compose.

### The Exam Results Ledger

An append-only YAML ledger stored in your project's Claude memory directory (path resolved during `/learning-setup` and saved in `learning-config.md`). Each entry records: topic, date, question, result (correct/partial/missed/taught-no-exam), code grounding, ease factor, and next review date. The refresher-exam reads this file to determine which topics need review, using the SM-2 algorithm to schedule reviews at optimal intervals.

## Notion Database Schema

| Property | Type | Values |
|----------|------|--------|
| Title | title | Topic name (one per card) |
| Priority | select | P0-critical, P1-high, P2-medium, P3-low |
| Size | select | quick-tutorial, crash-course |
| Status | status | To Do, Taught |
| Goal Alignment | multi_select | Tags matching your goals |
| Code Grounding | rich_text | File paths for codebase grounding |
| Last Taught | date | Date of last lesson |
| Notes | rich_text | Free-form notes |

Page body contains the lesson pager after teaching.

## Folder Structure

```
~/.claude/skills/learning-system/
├── README.md                      # This file
├── learning-goals.md              # North-star goals (shared data)
├── learning-config.md             # Notion DB reference (shared config)
├── teacher/
│   └── SKILL.md                   # Core teaching sub-skill
├── crash-course/
│   └── SKILL.md                   # 30-min crash course + exam
├── quick-tutorial/
│   └── SKILL.md                   # 5-10 min micro-lesson
├── refresher-exam/
│   └── SKILL.md                   # Spaced repetition exam
├── add-to-backlog/
│   └── SKILL.md                   # Notion backlog card creation
└── learning-setup/
    └── SKILL.md                   # One-time setup + goal coaching
```
