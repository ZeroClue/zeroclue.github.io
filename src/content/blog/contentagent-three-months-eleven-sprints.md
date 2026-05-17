---
title: "Three Months, Eleven Sprints"
date: 2026-05-17
description: "ContentAgent went from empty folder to billing-enabled SaaS in 11 sprints. Here's what each one actually took — and what I learned building a product with AI that catches other AI's mistakes."
draft: false
---

Sprint 10. Two weeks from launch. I sat down to actually use the product I'd been building for three months and discovered that content wasn't saving.

Not a fringe case. Not some edge state triggered by a specific input sequence. The main loop. The thing the entire product exists to do. You generate content, hit save, come back later. Empty screen.

I had sixteen issues by the time I stopped testing.

Some were minor. A loading state that never appeared. A template description clipped at 768px. A dropdown that stayed open when you clicked outside it. Those are fixable in an afternoon. But content not saving isn't minor. It's the only job. The form collects voice settings, the generator produces output, the save button saves it. That's ContentAgent. And the save button didn't work.

There's a specific quality of humbling that comes from this. Ten sprints of architecture decisions. Drizzle ORM schema for eight tables, Polar.sh billing integration, OpenRouter API calls through Vercel AI SDK v6, production Clerk auth, competitor voice scanning logic, a specificity radar, 14 content templates. None of that matters if the save button doesn't save.

---

The problem ContentAgent was built to solve isn't a capability problem.

AI can write. The problem is that every AI content tool optimises for writing that looks correct, and because they all train on similar data, they converge on the same version of correct. Which means every LinkedIn feed is slowly becoming the same post by the same person.

You know the patterns. Em dashes between clauses that should be separate sentences. Five-point frameworks with alliterative headers. "In today's landscape" to open. Something about "exciting times ahead" to close. "Delve." "Tapestry." "Transformative." The sycophantic opener that builds artificial tension before delivering an unsurprising point.

These aren't random failures. They're what you get when generation is optimised for plausibility without constraint on voice. The model produces output that is statistically likely to be praised, by readers trained on the same kind of output. It's circular. Every tool makes it worse.

ContentAgent's thesis: this isn't an inference problem. It's a detection and constraint problem. Catch the patterns before the user sees them. Constrain generation with something specific about the person doing the writing.

Two things. A gate. And a voice.

---

Sprint 2 was the quality gate.

Eight detection categories: sycophantic openers, generic conclusions, AI vocabulary flagging 500+ terms, structural tell-tales like the five-point list, platform format violations, hedge stacking, significance inflation, em dash overuse.

The key decision: deterministic scanning, not LLM evaluation. I looked at using another model to review the generation model's output and decided against it. You're asking the same class of system that produced the problem to identify the problem. The quality gate doesn't have opinions. If a draft contains "delve," that's a fact. If it has more than two em dashes, that's a fact. The flag either fires or it doesn't.

What I didn't anticipate was how much the post-sprint code reviews would matter. Sprint 2's review found three critical issues and five high-severity ones the build phase had missed entirely. This became the rhythm across all eleven sprints: build produces something that mostly works, review finds what doesn't. Sprint 2's scoring function was returning undefined for certain content structures. The gate was flagging nothing because the gate was broken. Clean bill of health that was just empty.

Every sprint had a version of this. Something looked right until it was examined. The build agent was very good at constructing code that handled the cases it had been asked to handle. The review agent was better at asking whether those were the right cases. Those are different skills and I needed both.

I fixed the critical issues before Sprint 3. Most of them.

---

Sprint 5 was voice calibration.

The challenge: people are bad at describing their writing style. "Direct and data-driven" is what every consultant on LinkedIn says about themselves. Pasting writing samples has a different problem: people submit their best work, which is usually their most polished and least distinctive. The writing someone does without thinking is often more revealing than the writing they've laboured over.

The interview approach routes around both problems. Three questions. Not "describe your style." Specific, contextual questions. What would you never want to sound like? What topic do you know well enough to argue about? What would you say out loud but never write in a professional context?

The model extracts patterns from the answers: sentence rhythm, argument structure, what you avoid, what you reach for when you're being precise. Those patterns become generation constraints.

Then the calibration sandbox. Two versions of the same output, one constrained and one not. User picks which sounds more like them. If they pick the generic one, the system recalibrates and tries again.

What I didn't expect: users were better at recognising their voice than describing it. They couldn't always explain why one version felt closer. But they could pick. Interview mode outperformed paste mode consistently, not because the model was better at processing answers than samples, but because the answers surfaced authentic patterns the samples obscured. People describe their voice honestly when answering specific questions. They perform it when they know they're being read.

---

Sprint 6 added the specificity radar. It came from watching early test generations and noticing a pattern: calibrated output still contained claims that were vague in structure even when the vocabulary was right. "We help businesses grow." "Our solution drives results." These pass tone checks. They don't pass a specificity check.

The radar flags vague claims and asks: can you be more specific? "We reduced onboarding time from 14 days to 3" passes. "We improve efficiency" doesn't. The detection isn't about vocabulary. It's about whether a claim has a verifiable anchor.

False positive rate on this is unknown. UAT found it flagging some technical precision as vague. I don't know how often that happens in normal use. Post-launch backlog.

---

Sprint 7 added the competitor voice scanner. Not on the original spec.

If you can profile your own voice, you can profile someone else's. If you can profile someone else's, you can compare. Paste one to three competitor writing samples, the system runs the same extraction, then surfaces where the voices overlap and where they diverge. The recommendation comes out as one of three: converge, differentiate, or selective convergence.

Nobody asked for this feature. Every person who's seen it running immediately wants it. There's a difference between things that make it onto a roadmap and things people actually reach for.

---

Now the uncomfortable part.

I used AI to build a tool that catches AI writing patterns. The same class of models that produces the patterns I scan for wrote most of the code that does the scanning.

The devil's advocate approach I described in an earlier post ran through the ContentAgent build. Before any significant feature got built, it went through at least one cold session with a model that had no prior context. No shared history, no bias toward the idea that already existed. The competitor scanner went two rounds before it made it to the spec. Features that didn't survive the argument didn't get built. The ones that made it through were partially stress-tested before a line of code was written.

ContentAgent also wrote the marketing content for ContentAgent. Every landing page section. Every email draft. Every LinkedIn post about the launch. All of it went through the quality gate before I saw the output.

The gate caught AI vocabulary in content generated specifically to not contain AI vocabulary.

Which is either evidence the detection works, or evidence the problem is harder than I thought, or both. The recursion is real: I'm using models to constrain model output, evaluated those constraints with models, and built with models that produce the exact patterns the constraints are designed to catch. Models reasoning about models all the way down.

No clean resolution. The gate works deterministically. The patterns it catches are ones I'd catch on a careful read. The calibrated output is closer to something I'd want to read than the unconstrained alternative. Whether that means the problem is solved or just moved upstream is a question I'm still sitting with.

The product launched anyway.

---

What shipped: 14 content templates across LinkedIn posts, Twitter threads, blog articles, email sequences, Instagram captions, and three research-oriented formats. The quality gate, all eight categories, blocking on every generation. Voice calibration in both interview and sample modes. The calibration sandbox. The competitor voice scanner. The specificity radar. Guided revision presets. Polar.sh billing: free at 10 generations a month, Pro at $19 for 50 plus model picker. Production Clerk auth. 26 API routes. Deployed to contentagent.kern.web.za.

The $19 price: below $20, no approval friction in most organizations. Above $9 to $12, where tools feel like experiments. Jasper is $69 a seat. Copy.ai is $49. ContentAgent isn't competing on features yet.

What didn't ship: tests. None. Zero test coverage, listed in build notes as large effort, post-launch backlog. The middleware uses a deprecated Next.js 16 API that currently works and will eventually become a problem. LinkedIn and Twitter direct posting aren't built. The specificity radar's false positive rate is unknown. The billing integration worked end-to-end the day I tested it properly in Sprint 10, which was also the first time I had tested it properly. Three sprints of individual billing components, none of them run as a complete purchase-to-generation sequence until the UAT that found sixteen other things broken.

That one is on me. The components were right. The flow wasn't tested as a system until I actually sat down and used the product.

---

The content-not-saving bug was a stale closure. The save function referenced a local variable the generated content was updating, not the state the function could see. One useCallback dependency fix. Five minutes once you know what you're looking for.

Sprint 11 fixed twelve of the sixteen issues. The other four went to post-launch backlog.

The first quality gate score after the fix came back at 71. The draft was for a LinkedIn post about project management software. Two flags: an em dash, and a generic conclusion. The conclusion had opened with "In today's competitive landscape."

Exactly what the tool was built to catch.

The voice profile was still thin. First interview session, sandbox not yet calibrated. But the output sounded less like a template and more like someone who had a specific opinion about the thing they were writing about.

Not clean. Closer. That's what three months and eleven sprints bought.
