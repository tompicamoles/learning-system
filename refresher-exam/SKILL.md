---
name: refresher-exam
description: >
  Use when the user types `/refresher-exam`, "refresher exam", "quiz me",
  "review session", "spaced repetition", or "test my knowledge".
---

# Refresher Exam Skill

Administer a spaced-repetition exam drawn from the exam history ledger,
aligned to the student's learning goals.

## 1. Read Inputs

1. Read `~/.claude/skills/learning-system/learning-goals.md` for goal-alignment weighting.
2. Read `the exam results ledger (path from `~/.claude/skills/learning-system/learning-config.md` → `exam_results_path`)` for the exam history ledger. If the file does not exist, treat the ledger as empty (first exam).

## 2. Select Questions -- Spaced Repetition Algorithm (SM-2 Inspired)

### Per-Topic State

Each topic in the ledger carries an `ease_factor` (EF) and `next_review` date.

| Parameter       | Rule                                                        |
|-----------------|-------------------------------------------------------------|
| Initial EF      | 2.5                                                         |
| EF after miss   | EF - 0.3 (floor 1.3)                                       |
| EF after partial| EF - 0.1 (floor 1.3)                                       |
| EF after correct| EF + 0.1 (cap 3.0)                                         |
| 1st interval    | 1 day                                                       |
| 2nd interval    | 3 days                                                      |
| nth interval    | previous_interval x EF (round to nearest day)               |

### Priority Score

```
days_overdue = today - next_review_date   (positive = overdue, negative = not yet due)
goal_weight  = 1.5 if topic aligns with an active goal, else 1.0
priority     = (days_overdue / interval) * goal_weight
```

### Selection Rules

1. Entries with `result: taught-no-exam` (from quick-tutorials that were never tested) get **highest priority** -- always pick these first.
2. Sort remaining entries by priority score descending.
3. Prefer previously-incorrect (`missed` or `partial`) questions over `correct` ones at the same priority.
4. Pick the top 3-4 topics.
5. Ensure the question mix includes **at least 1 conceptual, 1 code-reading, and 1 trade-off** question. Swap in a lower-priority topic if needed to satisfy this constraint.

### Handling Edge Cases

- **Empty ledger**: Tell the student there are no topics to review yet and suggest running a crash-course or quick-tutorial first.
- **Nothing overdue**: Tell the student all topics are up to date. Optionally offer to quiz on the least-recently-reviewed topics if they insist.

## 3. Reuse Original Code Grounding

For each selected topic, re-read the files referenced in the `code_grounding` field from the ledger entry. Check whether the code has changed since the original lesson. Use the **live code** (not stale references) when constructing questions.

If a file no longer exists or the referenced lines have shifted significantly, note this and adapt the question accordingly.

## 4. Administer Exam

Present 3-4 questions sequentially. For each question, state:

- The topic being tested
- The question type (conceptual / code-reading / trade-off / failure-mode)
- The question itself

Wait for the student's answers before correcting.

## 5. Correct

For each answer:

1. Tag as `correct`, `partial`, or `missed`.
2. **Lead with specific strengths** -- what the student got right and why it matters.
3. Explain why mistakes are mistakes with references to the live code.
4. For partial answers, identify what was missing and why it matters.

## 6. Persist Results

Append new entries to `the exam results ledger (path from `~/.claude/skills/learning-system/learning-config.md` → `exam_results_path`)`.

### Entry Format

```yaml
- topic: "Topic name"
  date: YYYY-MM-DD
  question: "The question that was asked"
  type: conceptual | code-reading | trade-off | failure-mode
  result: correct | partial | missed
  code_grounding: "file:line"
  session_type: crash-course | quick-tutorial | refresher
  ease_factor: N.N
  next_review: YYYY-MM-DD
```

Compute `ease_factor` and `next_review` using the SM-2 rules above, based on the previous entry for that topic (or defaults if first exam).

Set `session_type: refresher` for all entries created by this skill.

## 7. Suggest Rescheduling

If the student scores `missed` on a topic, suggest rescheduling the original lesson format:

- If the ledger shows `session_type: crash-course` for that topic, suggest a crash-course.
- If it shows `session_type: quick-tutorial`, suggest a quick-tutorial.
- If it shows `session_type: refresher` (failed a refresher), suggest escalating to a crash-course.

## Common Mistakes

| Mistake                                  | Why it matters                                                        |
|------------------------------------------|-----------------------------------------------------------------------|
| Asking questions on topics not yet due   | Defeats spaced repetition; wastes review time on strong topics        |
| Ignoring `taught-no-exam` entries        | These have never been tested; they must be prioritized                |
| Skipping code re-read                    | Code may have changed; stale questions teach wrong patterns           |
| All questions same type                  | Mix of conceptual/code-reading/trade-off tests deeper understanding   |
| Correcting without citing code           | Corrections must reference live code to ground the learning           |
| Forgetting to persist results            | Breaks the spaced repetition loop; progress is lost                   |
| Using stale EF from wrong topic entry    | Always use the most recent entry per topic for EF and interval        |
| Setting next_review without rounding     | Round interval to nearest day; fractional days cause scheduling drift |
