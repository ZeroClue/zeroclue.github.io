---
title: "The Architect Nobody Hired"
date: 2026-05-07
description: "Every expensive rewrite starts somewhere. Usually with a meeting that never happened."
draft: false
---

The client wants to know why his payment gateway isn't working.

He calls me in to review the code. Cross-border project — his team here, the development team somewhere else. The code is the problem. That's the assumption. Find the bad code, fix it, ship.

So I look at the code.

It's not great. SQL scattered everywhere instead of stored procedures. JavaScript functions duplicated across files. The kind of mess that happens when a team is moving fast without a standard. But it's not catastrophically broken. A competent team wrote it under pressure. That much is clear.

The problem isn't the code.

---

Nobody has designed this thing.

I know this because when the development team delivers something, the response from the client isn't "this is wrong because X." It's "this doesn't work." That's the tell. You can only say "this doesn't work" when you don't know what "working" looks like. Which means nobody wrote it down. Which means nobody designed it.

The remote team is building against a target that doesn't exist yet. Professional, well-paid guessing. Every sprint they guess wrong and get told so. Every sprint they adjust and guess again.

---

I raise it. The architecture. The design sessions that should have happened before the first line of code. The need for someone whose actual job is to hold the whole picture.

The client hears me. Then goes back to talking about go-live. There's a date. There are commitments. The market isn't going to wait while we sort out the design. We need to ship.

At some point the conversation stops being about the project. It's about me. I'm being difficult. I'm an outsider who doesn't understand the relationship. Nobody wants to hear it anymore.

So I stop. Not dramatically. I just stop caring. The project stops feeling like mine to save. I walk away — or get pushed out, depending on your perspective. Both are probably true.

---

The gateway went live eventually. Whether it made money, I don't know. I stopped asking.

What I know: somewhere in that codebase is a design decision nobody made consciously. An assumption that became a rule. A gap the code works around in ways nobody fully understands. And some developer, years from now, is going to spend six months on a rewrite that could have been avoided in a single afternoon — before anyone wrote a line.

---

I've been in that room more than once. The room where nobody has budget for an architect but everyone has budget for the rewrite. The room where the deadline is real and the design is optional. The room where "we need to ship" ends every conversation that matters.

If you're building something and the design lives only in the developers' heads — or worse, in nobody's head — that's where it starts.

You don't need me after it breaks. You need me before.
