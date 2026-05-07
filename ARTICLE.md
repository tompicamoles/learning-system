# How I Use AI to Skill Up as a Junior Engineer (And Why That's the Point)

There's a growing narrative in tech: AI writes code now, so what's the point of junior engineers? I think that's exactly backwards. AI doesn't replace the need to learn — it gives you the best learning partner you've ever had.

I'm a junior software engineer at PayFit. I work in a large TypeScript monorepo — NestJS, MongoDB, DDD, hexagonal architecture, AWS Lambdas, SQS. A year ago, I was building React components from a Codecademy syllabus. Now I want to close the gap to confirmed engineer and bring as much value as I can to my team. So I built a system that teaches me 30 minutes a day, grounded in the actual code I ship.

This article is about that system.

## The controversy: juniors in the age of AI

Let's address the elephant in the room. AI coding agents can ship features faster than most junior engineers. Claude, Copilot, Cursor — they generate working code in seconds. So why hire a junior?

Because shipping code is not the job. Understanding *why* that code works, *when* it will break, and *what* to build next — that's the job. An AI agent can implement the outbox pattern. It can't decide whether your system needs one. It can write a NestJS module. It can't explain why the dependency injection is structured the way it is, or redesign it when the requirements change.

I believe the junior engineers who will thrive are the ones who use AI to accelerate their understanding, not just their output. Not by memorizing the implementation details that AI already handles perfectly, but by building the judgment to challenge what an agent generates, learn from it, and eventually surpass it.

That's what I'm building toward.

## My goals

I'm not trying to memorize APIs or learn every framework feature. AI agents will always be better at that. Instead, I'm focused on the things AI can't do for me:

- **Read and understand any part of the codebase independently** — not just the files I touch, but the ones I've never opened
- **Understand *why* things are coded a certain way** — justify architectural choices, not just follow patterns
- **Master the core patterns** — DDD, hexagonal architecture, event-driven systems — well enough to apply them in new situations
- **Design and propose solutions** — not just implement specs, but be the person who writes them
- **Lead technical studies end-to-end** — evaluate trade-offs, propose designs, present them to the team

These are the skills that make a confirmed engineer. And they're exactly the skills that AI can't shortcut.

## The philosophy: spaced repetition meets real code

The learning system I built is inspired by how flashcard apps like Anki work. The core idea is simple: you review what you don't know more often than what you do.

In Anki, when you get a card wrong, you see it again soon. When you get it right, the interval grows — 1 day, 3 days, a week, a month. Over time, knowledge that sticks gets reviewed less, and knowledge that doesn't gets reinforced more. It's called spaced repetition, and it's one of the most effective learning techniques ever studied.

I applied this principle to software engineering, with one critical difference: **every lesson is grounded in real code from the codebase I work in every day.**

Not hypothetical examples. Not textbook diagrams. Real file paths, real patterns, real architectural decisions from the monorepo I ship to production. When I learn about domain events, I see the collect-then-dispatch pattern in our `SubmitDsnUseCase`. When I learn about aggregate design, I walk through our `ScheduledDeclaration` model. The code is the textbook.

## The routine: ship, learn, ship

The system only works with consistency. Here's my routine:

- **Every morning, 30 minutes before I start my day**: one crash course from my backlog. A focused lesson on one topic, followed by a short exam (3-4 questions), correction, and a one-page summary saved to Notion.
- **Once a week**: a refresher exam. The system picks 3-4 questions from topics I've already learned, weighted by how well I answered last time. Topics I got wrong come back sooner. Topics I nailed wait longer.
- **During the day, on the fly**: when I encounter something I don't understand while coding — a pattern I can't explain, an agent suggestion I can't evaluate — I ask for a 5-minute tutorial. It creates a card in my backlog automatically, so it enters the review cycle.

The key insight is that **the backlog fills itself from real work**. I don't sit down and plan what to learn in the abstract. I learn what I need because I ran into it while delivering. Ship code, hit a gap, learn it, ship better code. It's a virtuous cycle.

## Prerequisites: you need a foundation

This system doesn't work from zero. You need to know how to code before you can learn *why* code is structured a certain way. In my case, before switching to software engineering I spent three years as a product builder (low-code engineer) at PayFit. That gave me strong product intuition that I still rely on daily. I followed the Codecademy Frontend Engineer syllabus on my own time, which gave me a solid foundation in JavaScript and React — enough to start contributing, but not enough to understand a production codebase.

The learning system picks up where bootcamps leave off. It works for anyone who can write code but wants to go deeper: a junior wanting to understand systems, a confirmed engineer joining a new company on an unfamiliar tech stack, or someone who needs to ramp up on DDD or event-driven architecture for the first time.

You also need to know what you don't know — or at least be honest about it. The backlog requires you to identify gaps. That said, the system helps: when I run the setup, it can scan the codebase and suggest topics based on the technologies and patterns it finds. And every time I encounter something unfamiliar while coding, it becomes a backlog item automatically.

## What I focus on (and what I don't)

As a junior working alongside AI coding agents, I made a deliberate choice about where to invest my learning time.

**I prioritize**: system design, architectural patterns, trade-off reasoning, understanding *why* code is structured a certain way. These are the skills that compound over time and that AI agents are bad at.

**I don't prioritize**: memorizing API signatures or syntax details. I still need to read code fluently — you can't evaluate what an agent generates if you can't read it — but I let AI handle the recall. When I need the exact parameters for a MongoDB aggregation pipeline, I ask the agent. When I need to decide *whether* an aggregation pipeline is the right approach, or spot that the agent's implementation has a subtle flaw, that's judgment I need to build myself.

The two feed each other. Implementation experience deepens design understanding, and design understanding makes you a better judge of implementations. I'm not skipping implementation — I'm choosing to spend my dedicated learning time on the layer above it.

This distinction shapes my entire backlog. Topics like "DDD tactical vs strategic", "Hexagonal architecture", "Resilience patterns" are P1. Topics like "Base64 encoding" are P3. The system prioritizes automatically based on my goals.

## The system

The learning system is built as a set of Claude Code skills (think: reusable AI behaviors). Six skills that compose together:

1. **`/learning-setup`** — the starting point. A guided conversation where you define your goals, which the system relies on to prioritize your backlog and select questions for exams. It also creates your Notion backlog and connects everything together.
2. **`/crash-course`** — 30-minute session: lesson grounded in your codebase, exam, correction, Notion summary
3. **`/quick-tutorial`** — 5-10 minute micro-lesson for on-the-fly questions
4. **`/refresher-exam`** — spaced repetition quiz on previously learned topics
5. **`/add-to-backlog`** — queue a topic to learn later with priority and size
6. **`teacher`** — the core teaching engine that every lesson flows through

Every lesson enforces three mandatory sections: *underlying principles* (the mental model), *system design angle* (how it affects scalability, reliability, coupling), and *what to take away* (one sentence to remember). Every lesson cites real files with line numbers. Every exam question references actual code.

The backlog lives in Notion. Exam results live in a local file that the spaced repetition algorithm reads. Goals live in a config file that every skill references when prioritizing.

It's open source: [github.com/tompicamoles/learning-system](https://github.com/tompicamoles/learning-system)

## What changed

Six months ago, I couldn't explain why our domain layer has no external dependencies. I didn't understand why we use SQS FIFO queues instead of standard ones. I couldn't read a Terraform file.

Today, I can trace a declaration submission from the API gateway through the use case, through SQS, to the Lambda that processes it. I can explain *why* each layer exists and what would break if you removed it. I can propose infrastructure changes and justify them in a technical study.

I'm not there yet. But the gap between junior and confirmed feels like a backlog with a deadline, not a cliff.

Thirty minutes a day. Real code. Spaced repetition. That's the system.
