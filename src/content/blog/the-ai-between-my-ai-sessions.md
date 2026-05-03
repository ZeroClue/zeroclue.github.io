---
title: "The AI Between My AI Sessions"
date: 2026-05-03
description: "There's an AI sitting between me and my AI coding sessions. This is what that looks like on day one."
draft: false
---

I'm currently using an AI to give instructions to two other AIs.

I'm aware of how that sounds.

---

The setup: Hermes — NousResearch's Hermes agent — runs in one tmux window as a TUI. Two Claude Code sessions run in another window, one per pane. I talk to Hermes. Hermes reads the panes, interprets what's happening, gives me a picture. When I tell it what I want next, it decides which session gets it and dispatches accordingly.

The Claude Code sessions don't know about each other. They don't know about Hermes. They only know what Hermes tells them.

None of this required anything exotic. Tmux has been around for twenty years. Pane captures are basic. The new part is a model sitting in the middle, making sense of the output and deciding what to do with it.

---

Working directly across two Claude Code sessions means the overhead lives in your head. You track what each one knows. You phrase things differently depending on where they are. You remember what you told Session A that Session B doesn't know yet. You're simultaneously the director and the context file.

With Hermes in the middle, that shifts. I say what I want to happen. Hermes reads the current state, decides which session handles what, and translates intent into instruction.

I'm operating on the problem. Hermes is operating on the sessions.

I didn't expect that distinction to feel as significant as it does.

---

The part I haven't resolved: Hermes isn't a router.

When I ask what's happening, Hermes doesn't give me a transcript. It gives me its read. When I push something back down, it decides how to frame it for each session. Something I say to Hermes arrives at Claude Code already transformed. The Claude Code sessions are responding to Hermes's version of what I meant.

I don't know yet what that costs. Whether Hermes is adding something in translation or just handling noise I'd have introduced anyway — I genuinely can't tell from one day. There's a layer of interpretation I didn't have before, and it isn't mine.

---

I'm watching. That's all it is right now.

Though technically, Hermes is watching and giving me its read.

Which is maybe the whole question.
