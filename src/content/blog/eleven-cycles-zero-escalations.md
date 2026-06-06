---
title: "Eleven Cycles, Zero Escalations"
date: 2026-06-09
description: "I set up an autonomous ops-lead coordinator to run the MFTPlus engineering workstream. It ran 11 complete cycles without me. Here's what I noticed."
draft: false
---

By the third cycle, I wasn't watching as closely.

A cycle is a full pass through the engineering loop: the ops-lead coordinator reads the sprint backlog, dispatches tasks to engineer agents, monitors CI, reviews completed PRs, and merges. From dispatch to merged code. I set up the rules at the start — what escalates to me, what doesn't — and then it ran.

Auth changes: escalate. Billing: escalate. Security decisions: escalate. Everything else? The machine handles it.

Phase 1 closed with 11 of these cycles. Zero escalations. 30 PRs merged.

I want to be specific about what I mean by "zero escalations," because the obvious reading is wrong. It doesn't mean nothing went wrong. CI failed on several PRs; the coordinator spawned a dedicated teammate to investigate and fix, same as I would have. A few PRs needed scope clarification; the coordinator flagged them, updated the spec, re-dispatched. The loop handled it.

Zero escalations means the coordinator never hit a decision it needed me to make. Not because the work was trivial, but because the rules I wrote at the start were specific enough to cover what came up.

That specificity was the work. Not the coordination itself.

Writing the escalation rules forced me to articulate what actually requires human judgment in an engineering loop. It's a shorter list than I expected. Security decisions need human sign-off because they carry liability, and billing changes touch real money in ways that compound. Auth is a security surface. Everything else is pattern matching: code quality, test coverage, architecture decisions within a spec.

The coordinator is good at pattern matching.

What surprised me was how fast I stopped monitoring individual cycles. By cycle five I was reading the summary at the end rather than watching in real time. By cycle eight I was checking in once a day. The work was getting done and the quality was consistent. Nothing to watch.

That shift from watching to trusting happened faster than I thought it would. I'm not sure yet whether that's the right move or whether I'm trading off awareness for productivity in ways I haven't fully accounted for.

The question I'm sitting with: what do you lose when execution is no longer something you're close to? I can read a PR summary and know what merged. I can't easily reconstruct the thinking that went into a scope decision three cycles ago. That context used to live in my head because I was making the decisions. Now it lives in the coordinator's memory files, which I can read but rarely do.

There's a version of this that ends with the human being less equipped to course-correct when something does need escalating, because they've been out of the loop long enough to lose the thread.

I don't know if that's what's happening. Eleven cycles isn't enough data.

What I do know: the work shipped and the rules held up. That's a better result than I'd have predicted when I started.
