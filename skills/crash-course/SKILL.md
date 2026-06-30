---
name: crash-course
description: Use when the user asks for a crash course, a timed learning session, "teach me X in 20 minutes", wants to learn a technology or concept quickly with codebase examples, or wants a teaching session that ends with an exam and a shareable one-pager for Notion
---

# Crash Course Session

## Overview

A timed, five-phase teaching session: ~10 min crash course, 5 min exam, 5 min correction, open follow-up, then a Notion-ready pager. Total budget is ~30 minutes — treat this as a **hard ceiling, not a target**. Designed to convert passive reading into durable understanding by forcing active recall (exam) and producing a lasting artifact.

**REQUIRED SUB-SKILL:** Use the `teacher` skill for Phase 2 and any follow-up deep-dives. It enforces the three mandatory sections (Underlying principles, System design angle, What to take away) and codebase grounding.

## The Pareto rule (most important constraint)

This is a **crash** course. The student is time-boxed and will not re-read a long lesson. Your single most important job is **selection**: pick the 20% of the topic that gives 80% of practical leverage, and ruthlessly cut the rest.

A good crash course teaches one core mental model plus 2–3 supporting ideas — enough that the student can reason about the rest of the topic on their own later. A bad one tries to survey the field and drowns the signal.

If you find yourself writing a fifth section, a third code walkthrough, or a sixth exam question, **stop and cut**. The bar for inclusion is "the student cannot reason correctly without this," not "this is interesting" or "this is technically part of the topic."

## When to Use

- "Give me a 20 min crash course on X"
- "Teach me X then quiz me"
- "I want to learn X with examples from `<codebase>`"
- User wants both depth AND a takeaway artifact they can keep

## When NOT to Use

- One-off factual question → just answer
- User wants open-ended exploration without an exam
- Pure implementation task (write/fix code)

## Workflow

### Phase 1 — Intake (brief)

Confirm: **topic**, **codebase path** (if any), **subtopics to emphasize**. If the user is working in a repo but didn't mention it, ask whether to ground examples there.

### Phase 2 — The crash course (target: ~10 min read)

Invoke the `teacher` skill. Deliver a lesson that:

- Opens with **one** strong analogy or mental model — the thing the whole lesson hangs off
- Picks **ONE primary codebase example** that demonstrates the core pattern. At most one contrasting example if it teaches something the primary cannot. Not three.
- Names 2–3 supporting ideas (the "80% of leverage" set) and stops
- Ends with the three mandatory teacher sections

**Length: 600–1000 words.** Scannable, table-heavy. If you're over 1000 words, something is not earning its place — cut it.

Before drafting, write a one-line answer to: *"If the student only remembers one thing from this session, what should it be?"* Everything in the lesson should serve that one thing. Material that doesn't serve it is noise, however interesting.

### Phase 3 — The exam (target: ~5 min to answer)

Generate **3–4 questions**. Each question should be answerable in under a minute. Cover a spread, don't pile on:

- 1 conceptual (mental model check)
- 1 code-reading against the real codebase (only if one was used)
- 1 trade-off / design judgment ("would you use X or Y here? why?")
- 1 "what breaks if…" scenario *or* a second code-reading question — not both

Questions should be short. A question that takes the student two minutes to *parse* is too long. Ask for all answers in a single reply.

### Phase 4 — Correction (target: ~5 min)

For each answer, use this exact ordering so the student can grade themselves at a glance:

1. **The question (verbatim)** — restate the original question label *and full question text* first. The student may have scrolled past it. Format: `**Q2 (Code-reading):** Which DDD zoom level does…` Do NOT paraphrase or shorten — quote the question as originally posed.
2. **The student's answer (quoted)** — show what they actually wrote, in italics or a blockquote, so the comparison is visible.
3. **The correction** — lead with what was **specifically** right ("you correctly identified X"), not empty praise. Then explain **why** any mistakes are mistakes, not just what the right answer is. Tag: correct / partial / missed.

The shape of one corrected item should look like:

```
**Q2 (Code-reading):** <full original question text>

> <student's answer, quoted>

Verdict: correct. You nailed <specific thing>. One refinement: <…>
```

Never collapse steps 1 and 2 into "Q2: <student's answer>" — that loses the question and makes the correction unreadable when the student re-reads later.

**Match correction length to answer length.** A one-sentence student answer deserves a one-paragraph correction, not a one-page lecture. If the student skipped a question or said "no time" / "don't know", give a short model answer (2–3 sentences) and move on — don't turn a skip into a full retelling of the lesson.

End with **one** sentence naming the principle most worth internalizing. Not a summary. One sentence.

### Phase 4b — Persist exam results

After correction, append results to the exam ledger. For each question:

Append to `the exam results ledger (path from `~/.claude/skills/learning-system/learning-config.md` → `exam_results_path`)`:

```yaml
- topic: "<session topic>"
  date: <today ISO>
  question: "<question text, one line>"
  type: conceptual | code-reading | trade-off | failure-mode
  result: correct | partial | missed
  code_grounding: "<file:line reference used>"
  session_type: crash-course
  ease_factor: <computed per SM-2: start 2.5, -0.3 miss, -0.1 partial, +0.1 correct>
  next_review: <computed: 1st=1d, 2nd=3d, nth=prev*EF>
```

If the file does not exist, create it with the format header from the ledger template.

### Phase 5 — Follow-up

Invite targeted questions. For any genuine deep-dive, re-invoke the `teacher` skill so the mandatory sections stay enforced.

### Phase 6 — Pager

**Output the pager directly in chat as rendered markdown.** Do NOT write it to a file.

The pager is the artifact of the **entire session**, not just Phase 2. It must incorporate insights and topics that emerged during the exam correction (Phase 4) and follow-up questions (Phase 5). If the student asked clarifying questions that revealed new concepts, distinctions, or mental models not in the original lesson, those belong in the pager.

### Pager content requirements

- **Length**: 1–2 pages equivalent (≈400–700 words)
- **Structure**: H1 title, italic subline referencing codebase if any, numbered H2 sections (6–8)
- **Coverage**: Must include topics surfaced during follow-up questions (Phase 5), not only the initial lesson. The pager reflects the full session.
- **Tables**: at least one ("when to use / when not to" or "cheat sheet")
- **Code**: at most one concise code block
- **Ending**: a single bold "one sentence to keep"
- **No emojis** unless the user asked
- **No `---` horizontal rules** — H2 headings already delimit sections, HR adds noise on Notion import

### Phase 6b — Update Notion backlog

Read `~/.claude/skills/learning-system/learning-config.md` for the Notion DB reference. If the session was invoked from a backlog card (user provided a Notion link), update the existing card:

- Status → "Taught"
- Last Taught → today
- Page body → pager content

If the session was ad-hoc (no Notion link), use `add-to-backlog` in programmatic mode to create a new card with Status = "Taught" and the pager as page body.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Trying to cover the whole topic | Pick the 20% that gives 80% leverage. Cut the rest, however interesting. |
| Lesson over 1000 words | Compress or delete. A crash course that takes 30 min to read isn't a crash course. |
| Multiple codebase walkthroughs when one would teach the pattern | One primary example. One contrast at most. |
| Exam with only conceptual questions | Mix conceptual + code-reading + trade-off + failure-mode (3-4 total) |
| Correcting a skipped/brief answer at full length | Match correction length to answer length. "No time" = 2-3 sentence model answer. |
| Correction starting with "good try" | Lead with a **specific** strength before correcting |
| Quoting the student's answer instead of the question | Show the full original **question** first, then the student's quoted answer, then the correction. Three blocks, in that order — never collapse. |
| Skipping the `teacher` skill invocation | It's required, not decorative |
| Writing the pager to a file instead of chat | Output directly in chat as rendered markdown |
| Pager longer than 2 pages | Compress with tables; a pager is not a chapter |
| Using `---` separators in the pager | Omit; Notion's import renders them as thin noise |
| Pager only covers the initial lesson | Include insights from exam correction and follow-up questions — the pager reflects the full session |

## Red Flags — stop and fix

- About to write a fifth section or a third codebase walkthrough → cut it
- Word count on the crash course passing 1000 → stop, compress, or delete a section
- About to ask a sixth exam question → cut it; 4 is the ceiling
- About to write a multi-paragraph correction for a skipped answer → 2-3 sentences instead
- About to write the pager to a file → STOP, output it directly in chat
- About to grade the exam before the user has answered → wait for the reply
- About to skip Phase 2's `teacher` skill because "the topic is simple" → invoke it anyway
- About to generate a pager that only covers the initial lesson → review Phase 4–5 for topics to include

## Why this shape works

A crash course alone fades within days. Active recall (exam) plus immediate correction plus a durable artifact (Notion pager) converts the session into long-term retention. Each phase earns its place.
