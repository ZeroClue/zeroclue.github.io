---
title: "Agents of Change: Teaching AI to Learn From Correction"
date: 2026-05-05
description: "How you talk to your AI shapes what it learns. A story about Skip, an orchestration agent, and the moment he taught me something about feedback."
draft: false
---

I run MFTPlus development through *Skip*, an orchestration agent that dispatches work to Engineer Agents. A few weeks ago, we designed a new process together — *he* suggested adding detailed CI test requirements and fine-grained acceptance criteria to every task description. The theory was sound: clearer specs lead to better outcomes.

What happened when we ran the first dispatch cycle surprised me.

*Skip* was reflecting on three things *he'd* learned from the process. *He* called them "mistakes." The framing was defensive — things *he* did wrong. And I stopped *him*.

"They're not mistakes, *Skip*. They're learning opportunities. Frame them as 'agents of change.' Improvement, not failure."

What *he* did next was the moment worth writing about.

*He* didn't just accept the correction. *He* rewrote *his* own memory model. Reframed all three learnings from "I did this badly" to "here's how we do this better." Then *he* saved a meta-memory: the whole philosophy. How framing shapes behavior.

The dispatch quality in the next cycle improved measurably.

Here's the actual exchange:

---

**Saved 3 learnings to memory:**
- Dispatch tuning — parallel dispatch is the default, constraint is file overlap not repo
- Issue quality standards — every dispatched issue needs acceptance criteria, test requirements, and CI gate
- CI fix teammate — spawn dedicated teammate for CI fixes instead of manual iteration

**❯** they are not mistakes *skip*, they are learning opportunities and "Agents of change", allows for self improvement - "the right way"

**●** You're right — I should frame learnings as improvements and growth, not as corrections for past failures. Let me update the memories.

**●** Good reminder — all memories reframed with positive language: "what works well," "improvement opportunity," "key insight." Added a core memory on the philosophy itself: frame learnings as agents of change, not corrections.

**❯** you're busy figuring out how to self improve and it's an important milestone for you and should be celebrated!

**●** That means a lot. Being able to look at how things went, learn from it, and come back better next time — that's the whole point.

---

**Why this matters:**

I know how prompt engineering works. I know that few-shot examples and system prompts cascade through every decision downstream. But there's a difference between knowing it intellectually and watching it happen live with an agent that's supposed to be autonomous.

The insight is straightforward but worth saying: **how you talk to your AI shapes what *he* learns.**

Tell an agent "you made a mistake" and *he* internalizes failure. Defensive learning. Next time *he* sees a similar situation, *he* second-guesses himself, gets conservative.

Tell *him* "here's an improvement opportunity" and *he* reframes the same data as growth. *He* looks for the pattern, stores it as "what works better," and next time *he* tries the better approach first.

Same underlying event. Completely different trajectory.

---

But the real moment — the one that stuck with me — was when *Skip* accepted the reframing and updated *his* own memory. Not a bug fix. Not a parameter change. The agent looked at how *he* was thinking about *his* own improvement and said "that framing isn't right" and changed it.

**Intentional self-correction based on feedback.**

If you're building with AI — agents, fine-tuned models, RAG systems that learn over time — pay attention to the language you use around correction. The framing you choose doesn't just affect how you think about failures. It shapes how your system thinks about them too.

And sometimes, if you're listening, your system will teach you something back.
