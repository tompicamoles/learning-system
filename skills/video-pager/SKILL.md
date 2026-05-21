---
name: video-pager
description: |
  Use when the user types `/video-pager <url>`, says "make a pager from this video",
  "summarize this YouTube video", "turn this video into a learning note",
  "capture this talk for review", or shares a YouTube/Vimeo/conference video URL
  and asks for a Notion-ready takeaway. Produces a 300-500 word distilled pager,
  adds it to the Notion learning backlog as Taught, and seeds 3-4 real questions
  into the spaced-repetition ledger so refresher-exam picks them up next session.
  Trigger this even when the user just drops a video URL with a phrase like
  "save this for review" or "capture this" — passive video consumption is exactly
  what this skill is for.
---

# Video Pager

Turn a video (YouTube talk, conference recording, tutorial) into a Notion-ready pager and seed it into the spaced-repetition system. The watching already happened — this skill captures it as a durable artifact and queues review questions for later retention testing.

## When to use

- User shares a video URL (YouTube/Vimeo/conference recording) and wants a written takeaway
- After watching a talk, wanting it queued for later review without a live teaching session
- Capturing passive learning into the spaced-repetition system so it doesn't decay

## When NOT to use

- Article or blog post → use `add-to-backlog` directly, or `quick-tutorial` if they want a lesson
- Codebase concept → use `quick-tutorial` or `crash-course` (those ground in real code)
- Live teaching session with active recall → use `crash-course`

## Workflow

### Phase 1 — Fetch transcript and metadata

Extract the video ID from the URL (e.g. `https://www.youtube.com/watch?v=FqR5vESuKe0` → `FqR5vESuKe0`).

Try in order — stop at the first that yields a usable transcript:

1. `WebFetch` on a public YouTube transcript service. Known options (try one, then the next):
   - `https://youtubetotranscript.com/transcript?v=<VIDEO_ID>`
   - `https://www.youtubetranscript.com/?server_vid2=<VIDEO_ID>`
2. `WebFetch` on the YouTube page itself (`https://www.youtube.com/watch?v=<VIDEO_ID>`) — extract description, title, channel, and any inline transcript fragments.
3. If both fail, **ask the user to paste the transcript** (or a summary) in chat. Don't fabricate content from the title alone — that produces hollow pagers.

Also capture for the pager's subline: **title, creator/channel, URL, duration** (when available).

### Phase 2 — Distill the pager (the Pareto rule)

A video pager is **not a transcript** — it's the distilled takeaway. Apply the Pareto rule: pick the 20% of the video that gives 80% of practical leverage. Most talks have one core argument and 2–3 supporting ideas; everything else is colour.

Before writing, answer in one line: *If someone only reads one thing from this video, what should it be?* Every section of the pager must serve that one thing. Material that doesn't serve it is noise, however interesting the speaker found it.

### Pager format (output directly in chat as rendered markdown)

- H1 title — concise topic name, not the full video title
- Italic subline crediting the source: `*Source: [video title] by [creator] — [URL] ([duration if known])*`
- Numbered H2 sections (4–6)
- **At least one table** — comparison, cheat sheet, or "when to use / when not to"
- **At most one** concise code block, only if the video shows code worth keeping
- End with a single **bold one-sentence takeaway**
- No emojis, no `---` horizontal rules (Notion renders them as thin noise)
- Total length: **300–500 words**

### Phase 3 — Generate 3-4 review questions

Generate a spread of questions for spaced-repetition review:

- **1 conceptual** — does the user grasp the core mental model?
- **1 trade-off / design judgment** — "when would you use X vs Y?"
- **1 failure-mode** — "what breaks if…?"
- *Optionally* 1 more conceptual or trade-off

No code-reading questions — there's no codebase to re-read against. Keep each question answerable in under a minute.

Don't present these to the user in chat (no exam phase). They are persisted directly to the ledger and surface later via `/refresher-exam`.

### Phase 4 — Persist

**4a. Append questions to the exam-results ledger.** Read `~/.claude/skills/learning-system/learning-config.md` → `exam_results_path`. Append one entry per question (preserving existing entries):

```yaml
- topic: "<short topic name — same across all entries from this video>"
  date: <today YYYY-MM-DD>
  question: "<full question text, one line>"
  type: conceptual | trade-off | failure-mode
  result: taught-no-exam
  code_grounding: "video: <youtube-url>"
  session_type: video-pager
  ease_factor: 2.5
  next_review: <today + 1 day, YYYY-MM-DD>
```

The `result: taught-no-exam` value is **load-bearing** — `refresher-exam` treats it as highest priority, so these questions surface on the user's next refresher session. Do not use `correct` or `partial` (the user has not actually been tested).

Note for `refresher-exam`: when it encounters `code_grounding: "video: <url>"`, there is no live file to re-read — present the stored `question` text verbatim instead of trying to regenerate from code.

**4b. Create the Notion backlog card.** Invoke `add-to-backlog` in programmatic mode:

- **Title**: the distilled topic name (e.g. "Async Rust runtimes" — not the literal video title)
- **Priority**: P2-medium by default. Bump to P1-high only if the user explicitly asked or if the topic strongly aligns with an active learning goal
- **Size**: `quick-tutorial`
- **Status**: `Taught`
- **Goal Alignment**: read `~/.claude/skills/learning-system/learning-goals.md` and pick the tags that genuinely fit
- **Code Grounding**: `video: <youtube-url>`
- **Page body**: the pager content

### Phase 5 — Report back

In chat, tell the user:

1. The Notion card URL
2. How many questions were added to the ledger and when they'll surface (next refresher session)
3. The pager itself (already rendered in chat from Phase 2)

Keep this report to 2–3 short lines.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Producing a transcript instead of a pager | The pager is the distilled takeaway — apply the Pareto rule, cut ruthlessly |
| Pager over 500 words | Compress with tables; cut sections that don't serve the one-thing-to-remember |
| Fabricating content when transcript fetch failed | Ask the user to paste the transcript; don't invent from the title |
| Forgetting to append questions to the ledger | Every video produces 3-4 ledger entries — that's the spaced-repetition hook, the whole reason this skill exists |
| Marking questions as `result: correct` | Use `taught-no-exam` — the user has not been tested yet |
| Asking the questions in chat (Phase 3) | No exam phase; questions go straight to the ledger for later |
| Writing the pager to a file | Output directly in chat as rendered markdown |
| Skipping metadata fetch | The pager's subline must credit the source (title, creator, URL) |
| Including a code block when the video shows no code | The code block is optional — at most one, not required |
| Setting `next_review` more than 1 day out | Use today+1 so it surfaces on the next refresher session |
| Creating a duplicate Notion card when one already exists | If the user references an existing backlog card, update it instead — same pattern as `quick-tutorial` Scenario B |

## Red flags — stop and fix

- About to write a 5th section or 6th question → cut
- Pager passing 500 words → compress
- About to invent transcript content because fetch failed → ask the user instead
- About to skip the ledger append → STOP, that's the core value of this skill
- About to mark questions as `correct` → use `taught-no-exam`

## Why this shape works

Watching a video produces a brief spike of understanding that decays within days. This skill converts that spike into two durable artifacts: a Notion pager you can re-read in 2 minutes, and 3–4 review questions queued for spaced repetition. The pager handles short-term recall; the questions handle long-term retention. Without the questions, the pager rots in Notion unread; without the pager, the questions reference content you can't refresh on. Both earn their place.
