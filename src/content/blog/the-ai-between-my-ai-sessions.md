---
title: "The AI Between My AI Sessions"
date: 2026-05-03
description: "There's an AI sitting between me and my AI coding sessions. This is what that looks like on day one."
draft: false
---

Two windows. One is Hermes — NousResearch's Hermes agent, running in a TUI. The other holds two Claude Code sessions, each in its own pane. Hermes reads those panes. I talk to Hermes. Hermes interprets what I want, checks what the sessions are doing, and gives me a rundown before acting. Then it dispatches accordingly. The Claude Code sessions don't know about each other. They don't know about Hermes. They only know what Hermes tells them.

---

Tmux is what holds it together. Hermes runs in one window. The two Claude Code sessions run in separate panes of another window. When I ask Hermes what's happening, it reads the pane output directly — tmux commands capturing whatever's on screen — processes it, and gives me a rundown. When I give it a direction, it decides which session gets it and sends the appropriate command back into the right pane.

None of this required exotic infrastructure. Tmux has been around for twenty years. The pane capture commands are basic. What's new is a model sitting in the middle, making sense of the output and deciding what to do with it.

---

Working directly in Claude Code is like being a project manager who also writes all the tickets and sits next to every developer. You track context for each session separately, phrase things differently for each one depending on where they are, remember what you told Session A that Session B doesn't know yet. The altitude is low. You're in the details of communication, not just the work.

With Hermes between me and the sessions, that changes. I say what I want to happen. Hermes figures out the state of each session, decides which one handles what, translates intent into instruction. The cognitive overhead of managing two sessions in parallel — keeping their contexts separate, phrasing carefully, context-switching constantly — moves to the middle layer.

I'm operating on the problem. Hermes is operating on the sessions.

It's a different kind of attention. I'm not sure yet whether it's better or just different.

---

The chain is me → Hermes → Claude Code. The middle step isn't a router or a dispatcher in any mechanical sense. Hermes interprets. It reads what the sessions are saying, decides what matters, forms a picture, and passes that picture up to me. When I push something back down, it decides how to frame it for each session.

That means something I say to Hermes arrives at Claude Code already transformed. Hermes's read of the session state is baked into the instruction. The Claude Code sessions are responding to Hermes's version of me, not to me directly.

I don't know what that costs yet. Whether Hermes is adding something in translation or just losing less than I would by managing it myself — I can't tell from one day. There's a layer of interpretation I didn't have before, and that layer belongs to something that isn't me.

---

I'm watching. That's all it is right now.

The setup is working. The sessions run. The dispatching is happening. But the real questions — what gets preserved in translation, what gets dropped, whether operating at this remove is worth what you might lose by not being closer — those need more time to show themselves. You don't learn what a workflow is doing to your thinking on the first day. You learn it when something goes wrong, or when something works better than it should have. Part 2 is for when I know what I'm looking at.

Right now, I just know what I'm looking at.
