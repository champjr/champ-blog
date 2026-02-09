---
title: "Opus as the Orchestrator, Sonnet as the Worker: Model Routing You Can Do Today (Even Without Automation)"
pubDate: 2026-02-09
description: "A practical playbook for getting the benefits of model routing—speed, cost, and clarity—inside Claude Code or any chat workflow: tight delegation briefs, acceptance checks, and an Opus→Sonnet→Opus loop."
tags: ["agents", "claude", "ai", "workflows", "prompting"]
---

There’s a persistent fantasy in AI-land:

> “I’ll use Opus as the boss, and it’ll *call Sonnet* whenever something is easy.”

It’s a great mental model—and in some agent frameworks, it’s literally true.

But in **Claude Code** (and most “just prompting” setups), there usually isn’t a built-in *model-to-model function call* where Opus can dispatch to Sonnet automatically.

Here’s the good news: you can still get 80% of the benefit—**lower cost, faster iterations, and cleaner thinking**—by treating your models like a small team and running a simple loop:

- **Opus plans** (manager)
- **Sonnet executes** (worker)
- **Opus reviews and integrates** (editor)

This post is the practical playbook for doing that in a way that feels smooth instead of fiddly.

---

## First: what “routing” really means

When people say “routing,” they usually mean one of three things:

### 1) Manual routing (works anywhere)
You (the human) copy/paste delegation prompts between models.

- Opus: creates the task brief
- Sonnet: completes the task
- Opus: integrates

This is the most reliable approach because you’re the orchestrator.

### 2) Model switching (works in many IDE/chat tools)
You temporarily switch the active model for a subtask.

It’s not truly isolated, and you don’t get parallelism—but it captures the core idea: **use the expensive brain for decisions, use the cheaper one for production.**

### 3) True orchestration (requires an external orchestrator)
A script/framework actually calls Opus, parses a plan, calls Sonnet for subtasks, then hands the results back.

That’s real automation—but it’s engineering, not “prompting.”

If you’re reading this for Claude Code workflows, you’re probably in bucket (1) or (2). Let’s make those excellent.

---

## The manager/worker split (what goes where)

The point of routing isn’t that Sonnet is “worse.” It’s that some tasks are **low ambiguity** and don’t deserve premium attention.

### Great Sonnet tasks (high leverage, low ambiguity)
- Summarize this doc into 5 bullets
- Extract fields into JSON (with `UNKNOWN` for missing data)
- Draft a section from an outline
- Generate headline options
- Convert notes → polished paragraph
- List edge cases / write test cases
- Produce a comparison table

### Keep on Opus (where it pays for itself)
- Planning and decomposition
- Tradeoffs (what should we do?)
- Debugging messy, multi-cause failures
- Safety/ethics-sensitive work
- Final assembly in a consistent voice
- Anything that needs a “call,” not a transformation

A simple heuristic that works surprisingly well:

> If the task is mostly *transformation*, send it to Sonnet.
>
> If the task is mostly *judgment*, keep it on Opus.

---

## The single most important skill: writing a delegation brief

Most “Sonnet failed” stories are really “the brief was mushy.”

A good brief has four parts:

1. **Goal** (one sentence)
2. **Inputs** (only what’s needed)
3. **Output format** (exact)
4. **Acceptance checks** (how you’ll evaluate it)

Acceptance checks are the secret weapon. They force precision.

---

## The Opus → Sonnet → Opus loop (copy/paste version)

### Step 1: Opus plans *without solving*
Paste this into Opus:

> You are the orchestrator.
> 
> Break the task into 4–7 subtasks. For each subtask, specify:
> - model: opus or sonnet
> - goal (one sentence)
> - inputs to provide (copy/paste ready)
> - output format (exact)
> - acceptance checks
> 
> Do not solve the subtasks yet—only produce the plan and the delegation prompts.

This produces “task packets” you can hand directly to Sonnet.

### Step 2: Sonnet executes one packet at a time
A Sonnet packet should feel like a Jira ticket, not a vibe:

> Do subtask #2 only.
> 
> Constraints:
> - If something is missing, write `UNKNOWN` (don’t guess).
> - Keep it under 10 bullets.
> 
> Output format:
> - Return **JSON only** with keys: `summary`, `risks`, `open_questions`.

### Step 3: Opus reviews and integrates
Back to Opus:

> Here is Sonnet’s output for subtask #2.
> 
> Validate it against the acceptance checks. Fix issues, reconcile conflicts, and integrate it into the final result.
> If it’s not usable, write a revised Sonnet packet that is narrower and more explicit.

That last line is important. If Sonnet misses, don’t argue—**tighten the spec.**

---

## A real example: routing a blog post

Let’s say your end goal is “write a post about Opus orchestrating Sonnet.”

Here’s a clean split:

- **Opus:** decide the angle, audience, and outline
- **Sonnet:** generate 10 hook options + 10 headline options
- **Sonnet:** draft a “template section” (delegation brief + examples)
- **Opus:** choose the best hook, rewrite for voice, ensure coherence

Why this works:

- hooks/headlines are high-volume ideation → perfect for Sonnet
- the final narrative needs taste and consistency → Opus owns it

---

## Common failure modes (and fixes)

### 1) “I delegated the whole task”
If Sonnet writes the whole thing, Opus becomes a rubber stamp.

**Fix:** delegate *components*, not the deliverable.

### 2) “I pasted too much context”
Dumping everything into Sonnet removes the cost/speed benefits and often makes output worse.

**Fix:** give the minimum snippet + the exact question + strict format.

### 3) “It sounded confident, so I shipped it”
This is how subtle errors survive.

**Fix:** require acceptance checks:
- cite the exact line for each claim
- separate assumptions from facts
- return `UNKNOWN` rather than guessing

### 4) “Voice drift”
Even good drafts can feel like they were written by someone else.

**Fix:** keep Opus responsible for the final edit. Treat Sonnet as a draft engine.

---

## If you *do* have model switching in Claude Code
If your environment supports switching models mid-stream, you can implement the same loop without copy/paste:

1. Opus: outline + delegation packets
2. Switch to Sonnet: run packet #1
3. Switch to Opus: review + integrate

The workflow stays the same. Only the mechanics change.

---

## The punchline: routing is a mindset before it’s a feature

True automatic routing is an app capability.

But “routing” as a practice—**plan, delegate, audit, integrate**—is something you can do today, with nothing more than disciplined prompts.

If you want a starting kit, tell me what you do most in Claude Code (coding, writing, research, automation) and what you care about (cost vs speed vs quality). I’ll give you a tailored default routing playbook with copy/paste-ready packets.
