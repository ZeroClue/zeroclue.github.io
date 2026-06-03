---
title: "Every Token in Your Context Is a Cost"
date: 2026-06-10
description: "Most multi-agent stacks leak tokens you can't see. Cache discipline is the practice of finding and eliminating those leaks."
draft: true
canonicalURL: "https://tokentelescope.com/essays/every-token-is-a-cost/"
---

> *Originally published on [Token Telescope →](https://tokentelescope.com/essays/every-token-is-a-cost/). Cross-posted here.*

The month my AI bill crossed my AWS bill, I didn't panic. I assumed I'd been running agents harder than usual and waited for it to normalize.

It didn't normalize.

What I had instead was a slow, formless suspicion, the kind you can't act on because you can't name it. Something in my stack was wasteful. The invoices confirmed it. But I couldn't point at a single decision and say: *that's the problem.* Every component looked reasonable in isolation. The system as a whole was expensive in a way I couldn't see.

That's the invisible cost trap. It doesn't announce itself. It accrues.

This essay is what I found when I finally looked: a full audit of a 4-agent stack, precise numbers from before and after, a framework I'm calling **cache discipline**, and seven principles I'm grouping under the **Token Telescope**. I cut context costs by roughly 60% in a week without changing what the agents could do. The leaks were everywhere. Yours probably are too.

---

## The Audit That Changed Everything

I run a 4-agent stack on a popular multi-agent orchestration platform. Four agents — a CEO, CTO, CMO, and Sr Engineer — coordinating across continuous work queues driven by crons and webhooks.

The audit started with a simple question: what is in context on every agent run, and does it need to be there?

The first thing I found: **615KB of plugin data** loaded as system reminders every session. One integration alone, the workflow automation plugin, contributed the majority. It was installed. It wasn't used. Every session started with half a megabyte of context that did nothing but sit there and cost money.

The second thing: **personality prompts.** The lead agent carried a `SOUL.md` and a `HEARTBEAT.md`: documents defining its communication style and values. Together, 5.6KB. Reloaded on every agent run. Not once. Not daily. Every. Single. Run. The other agents had similar payloads: 1.8KB and 1.4KB respectively. Total per cycle: 7.6KB of personality that made no measurable difference to output quality.

The third: **duplicate documentation.** Reference files that existed in both the agent instructions and as attachments. The agent read the same content twice on every run because no one had cleaned up the overlap.

The fourth: **cron output inline.** Status checks ran three times an hour and dumped their full output directly into the next agent prompt. Every hour, the ops context grew. The stable, cacheable prefix kept getting polluted by fresh, variable output.

Simon Willison's [*Agentic Engineering Patterns*](https://simonw.substack.com/p/agentic-engineering-patterns) names several of these as engineering patterns: the 92% cache hit rate achievable in well-structured Claude Code sessions, the discipline of never mutating tools mid-session. He's right about the patterns. What I hadn't seen named anywhere was the *practice*: the deliberate, ongoing work of treating your context as a budget and defending it.

There's a word for that. Cache discipline.

---

## The Cost of Reasonable Choices

Here's what's strange about this kind of waste: every single leak was a defensible decision at the time.

The personality prompts? Adding them felt like good engineering. Consistent agent behavior. Branded tone. Real value, or so I assumed. Installing the workflow plugin made sense when I was evaluating integrations. Never removing it was an oversight, but a minor one. The duplicate docs were a copy-paste accident that never got cleaned up. The inline cron output was the simplest way to pass state between runs, and it worked.

None of these were mistakes in isolation. They were the output of reasonable decisions made without a cost lens.

What I didn't have — what I suspect most agent operators don't have — is the habit of asking, every time I add something to context: *does this need to be here right now?* The question sounds obvious. Ask it in retrospect about 615KB of unused plugin data and the answer is obvious. In the moment, mid-sprint, with a feature to ship, you don't ask. You add.

The sum of all those additions was a 5-to-10x cost multiplier no one was tracking. The CEO agent's instructions ran ~7KB per run. After the audit: ~3KB. The CTO: ~10KB down to ~8KB. The Sr Engineer: ~3.7KB down to ~2KB. These numbers don't look dramatic on a spreadsheet. Multiply them by hundreds of agent runs per day, add the plugin bloat, add the cron pollution, and you're burning tens of thousands of tokens per hour on things that are invisible on any single invoice line.

I wasn't running a wasteful stack. I was running a normal stack. That's the point.

---

## Cache Discipline: The New Prompt Engineering

Three years ago, "prompt engineering" was crystallizing as a practice. People had been writing prompts for longer than that, but the community was just beginning to name what worked: few-shot examples, chain-of-thought reasoning, role assignment, output formatting. The naming made it teachable. It created vocabulary. It turned intuitions into transferable skills.

Cache discipline is at that same stage right now.

Everyone running a non-trivial agent stack has some instinct about context size. They've noticed that some sessions feel faster. They've vaguely tried to keep system prompts lean. They've maybe heard that prompt caching exists. But it's not a practice yet — it's a feeling.

Here's the definition I'm working with: **cache discipline is the practice of treating every token entering context as a cost, every cache hit as savings, and every prompt design decision as a budget allocation.**

The hardest part to internalize is the budget piece: cache keys are prefix-based, so the moment anything variable appears before your stable content, you've busted the cache for everything that follows. Prompt layout is cost architecture.

This is not prompt engineering. Prompt engineering is about output quality: getting the model to say the right thing. Cache discipline is about cost: making sure you don't pay for context that doesn't earn its place.

This is not FinOps. FinOps looks at invoices. Cache discipline looks at the moment before the token is sent: the architectural choice, the cron script, the config file. It asks whether the cost is warranted.

---

## The Token Telescope: 7 Principles

The Token Telescope is the framework I use to apply cache discipline in practice. Seven principles, in the order I'd apply them to a new stack.

**1. Trunk & Fork**

The trunk session stays lean. Its only job is coordination. All substantive work happens in disposable subagent forks that inherit the cached prefix and get discarded on completion. The trunk prefix stays hot, so every fork starts cheaper. In my stack, the ops-lead agent respawns every 8-10 cycles to prevent context accumulation — before that change, 20 cycles generated 5-6MB of accumulated context. After: capped and predictable.

**2. Token Telescope**

The principle the framework is named after. Every addition to context has to pass through one question: *does this need to be in context right now?* Not eventually. Not in case. Right now, for this run. If the answer isn't yes, it belongs on disk and gets read on demand. The personality prompts failed this test completely. 7.6KB per agent run cycle, zero functional value.

**3. Stable Prefix**

Cache keys are prefix-based. If the first token of your prompt varies between runs, you have no cache. The fix: a visible `--- CACHE BOUNDARY ---` marker in your system prompts. Everything before the marker must be identical between sessions. Everything after can vary freely without touching the cached prefix. I use this in every agent instruction file now. It's the first thing I check in any new stack.

**4. File-Based State**

Cron prompts don't need to carry their output. They need to trigger a read. Script outputs go to disk; the agent reads the file on demand. Before this change, status checks dumped full output inline — ~200 tokens of variable context per run, 3x/hour. After: the cron prompt is static boilerplate, ~10 tokens. The state is on disk. The cache is intact.

**5. Agent Instruction Hygiene**

Kill `SOUL.md`. Kill `HEARTBEAT.md` if it duplicates content in your main instructions file. Merge or delete. Personality documents are token waste. The model doesn't need a prose description of its values re-read on every run. The CMO agent in my stack runs on 813 bytes. Single file. No fluff. That's the target. My cuts: CEO instructions dropped from ~7KB to ~3KB. Engineer from ~3.7KB to ~2KB.

**6. Event-Driven Over Polling**

A WebSocket listener is infrastructure. It costs zero tokens. A cron that polls for events three times an hour is an agent. It burns context and busts cache. Replacing polling crons with a listener didn't just reduce token use; it eliminated an entire class of cache perturbation. Cron-based cache busting went from 3x/hour to 1x/hour — a 66% reduction just from consolidation, before the listener was fully operational.

**7. The 5-Minute Audit**

Five questions. Answer them about your stack:
1. What is loaded in every agent's system prompt? What of that is actually used on every run?
2. Do your agents have personality or style documents? When did you last validate they change output quality?
3. Are your cron prompts variable? Are they dumping state inline?
4. Where is your cache boundary, and is it stable between sessions?
5. What was in context on the last 10 runs that wasn't in context on the first run you ever ran?

The fifth question is the one that finds the drift. Stacks accumulate. Files get added. References get duplicated. Nobody audits it because the cost is invisible — until you look.

---

## The Numbers

Before and after, from the audit:

| Area | Before | After | Saving |
|---|---|---|---|
| Plugin bloat in system prompt | 615KB (one integration alone) + 3 unused plugins | Removed | ~20 system-reminder entries eliminated |
| Agent instructions per run | CEO: ~7KB, CTO: ~10KB, Engineer: ~3.7KB | CEO: ~3KB, CTO: ~8KB, Engineer: ~2KB | ~5-8KB per agent run |
| Ops context per session | ~27KB | ~15KB | ~12KB per session |
| Cron cache perturbations | 3x/hour | 1x/hour | 66% fewer cache busts |
| Agent personality bloat | 5.6KB (CEO), 1.8KB (CTO), 1.4KB (Engineer) | 1.2KB, 0KB, 0KB | 7.6KB total per agent run cycle |
| Ops-lead context accumulation | 5-6MB over 20 cycles | Respawn every 8-10 cycles | ~75% less accumulated context |

Projected daily: **60,000-180,000 tokens saved per day** across 4 agents on Ollama Cloud inference. The range is wide because agent run frequency varies. The floor is real.

The 60% headline reduction comes from the combined effect: fewer cache misses compounding across every session start and agent run. No single change delivers 60%. The discipline delivers 60%.

Source: a 4-agent stack on a popular multi-agent orchestration platform. One week of focused audit and restructuring. No changes to what the agents could do — only to what was loaded into their context and when.

---

## Try the Audit Yourself

You don't need a full week. Start with 5 minutes.

**Step 1.** Open the system prompt for your highest-traffic agent. Read it like you're paying per word — because you are. Mark anything that doesn't need to be there on every run.

**Step 2.** List every plugin, integration, or tool included in your platform's system reminders. When did you last use each one?

**Step 3.** Find your cron jobs or scheduled prompts. Are they passing state inline? Are they variable? What does that variability cost your cache prefix?

**Step 4.** Look for your stable-vs-variable boundary. Is it explicit? Does anything variable appear before anything stable?

**Step 5.** Search your agent instructions for personality, tone, values, or style documents. Weigh each one. If you removed it, would output quality change? Run the experiment.

If you found something in step 1, you have a leak. If you found something in every step, you have a system. Most stacks do.

The Audit Kit — the full framework, expanded principles, self-audit worksheet, and before/after templates — is in progress. If you want it when it's ready, plus weekly writing on cache discipline and multi-agent cost patterns, sign up at **tokentelescope.com**.

The question I keep coming back to: if cache discipline is at the same stage prompt engineering was three years ago — felt but unnamed, practiced but not teachable — what gets built once it has a name?

---

*Armin — builder at [kern.web.za](https://kern.web.za) and [mftplus.co.za](https://mftplus.co.za). Writing about what actually works in multi-agent systems.*
