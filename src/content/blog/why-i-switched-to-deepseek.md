---
title: Why I Switched to DeepSeek
description: I replaced Claude Sonnet with DeepSeek V4 Flash as my daily driver across OpenCode, content generation, and API routing. Not because it's better across the board — because 17x cheaper with 1M context changes what you build.
author: Armin Marxer
date: 2026-05-29
wordCount: 1400
image: /og-deepseek-switch.png
---

The model powering this response is deepseek-v4-flash. If you've read any of my recent posts here, every one of them passed through it at some stage — draft, edit, or the agent loop that produced the output.

That wasn't the plan three months ago. I was on Claude Sonnet like half the solo builders I know. It worked. Good coding, decent writing, reliable API. The switch wasn't a protest. It was a spreadsheet.

One month in. Here's where the numbers landed and where they didn't.

## What I Was Running

Before April 24, my stack was straightforward: Claude Sonnet for agentic coding via Claude Code, GPT-4.1-mini for cheap bulk work, and a mix of both for content. Monthly API bill sat around $80-120. Nothing dramatic, but it was the kind of number you don't want to explain to yourself when you're bootstrapping three products.

The Claude Code sessions were the expensive part. A single extended refactor session — the kind where you leave an agent running while you make coffee — could burn $3-5 in output tokens alone. I'd work around it by splitting tasks into small, isolated prompts, which is exactly the wrong way to use an agentic tool. The agent's strength is maintaining context across a session. Splitting sessions to save money undermines the whole point.

## What DeepSeek V4 Actually Is

DeepSeek released V4 on April 24. Two MoE models under MIT: V4-Pro at 1.6T total / 49B active, and V4-Flash at 284B total / 13B active. Both with 1M-token context windows. Both supporting thinking and non-thinking modes.

The architecture deserves a footnote — hybrid attention combining Compressed Sparse Attention and Heavily Compressed Attention, cutting KV cache by 90% at 1M tokens. What matters in practice is the pricing. At launch promo rates: V4-Flash at $0.14/M input and $0.28/M output. V4-Pro at $0.435/M and $0.87/M.

Compare that to Claude Sonnet at $3/M input and $15/M output. Even at V4-Pro's list price ($1.74/$3.48), the output gap is 4-17x depending on tier.

Nobody likes reading pricing tables. The single number that made me switch: V4-Flash scoring 79.0% on SWE-Bench Verified against V4-Pro's 80.6%. A 1.6-point gap for 12x less cost. That's not a budget model. That's a serious default with a premium option above it.

## Running It in Practice

I switched OpenCode to deepseek-v4-flash as the default model and reserved deepseek-v4-pro for complex multi-file refactors that need the extra reasoning depth. The cost difference hits you on the first day — a full session that used to burn $8-12 now costs about $0.50-1.00.

The 1M context window matters more than I expected. Claude Code's context compaction is good, but I've had sessions where it summarised away something I needed ten turns later. With 1M tokens, I almost never hit the ceiling. The agent loop keeps more of its history, and the coherence stays consistent through longer sessions.

There are quirks. DeepSeek's thinking mode produces a `reasoning_content` field that has to be passed back to the API on subsequent calls. OpenCode had issues with this — there's a well-documented bug where session continuation breaks if `reasoning_content` gets stripped during serialization. I hit it twice before pinning the right version. The DeepSeek API also has stricter rate limits than OpenAI or Anthropic. I've had two brief outages in four weeks. Both resolved within 20 minutes, but if your workflow can't tolerate a brief API blip, you need a fallback route.

## Where It Falls Short

DeepSeek's writing has a different character than Claude's. For technical documentation and this kind of blog content, it works fine — the dry, direct register matches what I'd write anyway. But when I need nuanced long-form writing that requires a specific tone shift, I still reach for Claude. The gap is narrow but real.

Multimodal is weaker. DeepSeek V4 can handle images in prompts but the vision capability isn't as sharp as GPT-5.4 or Claude. For UI screenshot analysis and visual debugging, I keep the GPT fallback active.

The ecosystem is thinner. No first-party coding agent. No IDE plugin. OpenCode works well with it, Cursor and Continue both support the OpenAI-compatible endpoint, but you don't get the polished integration that Claude Code offers. It's a BYO-tool setup.

## The Honest Calculation

No benchmark drove this. No ideology. Just a number: I was paying 12-17x more for output that was, at best, 5-10% better on the tasks I actually run. That gap wasn't delivering $80-100/month of value.

The 1M context was the second factor. Once I had it, I realised how often I'd been working around the 200K ceiling on Claude. Not always — most sessions don't need a million tokens. But the ones that do are disproportionately valuable: long refactors, multi-file architecture changes, extended content planning. Having the ceiling disappear changed what I attempted in a single session.

## What I Tell Other Solo Builders

If your API bill is under $50/month, stay where you are. The switching cost isn't worth it. Optimise for familiarity, not line items.

If your bill is above $100/month and you're doing coding work, run the numbers on V4-Flash as a primary with Claude as a fallback for hard problems. The split config in OpenCode takes ten minutes to set up. Fifteen if you count the API key.

The model powering this paragraph cost about $0.002 to generate. Including the thinking trace. That's a different economic regime than what I was operating in three months ago. It means I can afford to iterate more, throw more at the agent, and stop optimising token count in my prompts.

I care about build velocity right now. DeepSeek V4 gives me more of it per dollar than anything else available.

That's the switch. Spreadsheet-driven, not principled. The moment where the cost curve intersects what you actually need is hard to ignore once you've seen it.
