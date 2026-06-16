---
title: "What I Learned Building a Quality Gate for AI Content"
description: "Raw LLM output sounds like everyone else's raw LLM output. Not because it's wrong — because it's too consistent. Here's what I did about it."
date: "2026-06-13"
draft: false
---

If you generated a hundred posts from the same model, how many would you have to read before they started to feel like one post, repeated?

Not many. That was the uncomfortable discovery.

I've spent the last year building a content pipeline that generates social posts and blog drafts across four brands. The quality gate came out of a specific failure: posts that were technically correct, matched the brief, passed the voice guidelines, and still felt wrong. Not flagrantly machine-generated. Just slightly off in a way that was hard to name for longer than I'd like to admit.

The problem wasn't accuracy. It was signature.

Raw LLM output sounds like everyone else's raw LLM output. Not because the model is making mistakes, but because it's optimising for the same objective everyone else's model optimises for: coherent, fluent text that satisfies the prompt. That objective produces a recognisable style. Sentence lengths cluster in a narrow range. Certain punctuation appears at predictable intervals. Parallel structures accumulate. Abstract qualifiers stack up where a concrete detail would have been more honest.

None of these are individually damning. Together, in the same statistical pattern, they're diagnostic.

The first thing I tried was a checklist. After generating a post, I'd go through a list of banned words and constructions. This caught the obvious cases and missed most of the rest. The issue isn't individual word choices. It's rhythm. You can remove every flagged word and still have text that moves like a metronome.

## What the gate actually checks

The signature isn't any single thing. It's the absence of human entropy.

Human writers are inconsistent in ways that feel natural. Sentence lengths vary because thoughts have different lengths, not because someone ran a variance algorithm. Structure gets chosen because it fits the idea, not because it scores well on readability metrics. Words get chosen because they're right, and sometimes a writer reaches for an unusual one and it works, and sometimes they settle for the first one that fits.

AI text is too consistent. The variance exists, but it clusters differently than human writing does. There's a burstiness metric — the unevenness of sentence lengths — that runs higher in human writing. There are trigram repetition patterns that run higher in generated text. Vocabulary reuse shows up when the type-token ratio is lower than it should be for the length.

The gate measures these. It catches overused constructions and scores the text. Anything above a threshold gets flagged. The score doesn't tell you what to change. It tells you something is off and where to start looking.

## What changed operationally

The first version ran as a review pass after drafting. Generate the content, review it, run the gate, fix the issues. This didn't work. Fixing these patterns after the fact is harder than avoiding them while writing, because they're structural. They require rewriting sentences, not substituting words.

The gate now runs inline, during drafting. Not as a separate pass. Not as a review loop. During, before the draft exists as a finished thing.

The difference is significant. Inline runs produce better results than retroactive fixes because the model is still inside the piece when changes get suggested. Editing on finished text tends to produce patchwork: the fixed sections feel disconnected from the rest because they were written with different constraints active.

## The uncomfortable part

The gate is a set of rules. A sufficiently sophisticated model, given those rules, could simply follow them and produce output that passes. This is not a hypothetical.

This isn't unique to content generation. Every filter creates a gradient toward evading the filter. Email spam filters made spam look like legitimate mail. The quality gate makes AI writing look more like human writing, which is the goal, but it also means the gate needs to evolve as the models do.

What I haven't found is a version of this problem that goes away. The moving target is visible. The right response isn't to build a perfect detector. It's to keep the author voice specific enough that generic optimisation can't hit it, and to keep improving what specific means.

Whether you can tell depends on what you're reading and how carefully. The gate makes it harder to ignore. That's most of what it does.
