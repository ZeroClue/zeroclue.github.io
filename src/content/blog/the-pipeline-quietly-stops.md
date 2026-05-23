---
title: "The Pipeline Doesn't Fail. It Just Quietly Stops."
date: 2026-05-23
description: "I built an AI-assisted cold outreach pipeline for a client with Rook handling most of the build — n8n workflows, email tracking, ops dashboard. Here's what he got wrong, what he flagged before I thought to ask, and what the collaboration actually looks like."
draft: false
---

The client runs a one-person web design studio. Their target market is every SMB in South Africa that needs a website but doesn't know to look for one. Manual outreach doesn't scale to that. Automated outreach doesn't work unless it's built with enough care to feel personal.

I handed the build to Rook.

---

The architecture he designed isn't complicated. A search query matrix generates leads — 125 combinations of industry × location × intent signal. An AI enrichment agent scrapes each lead's website, pulls contact info, and assesses their online presence. A copywriting agent uses that context to write a personalised cold email. A landing page generator creates a custom page for the lead and deploys it to the client's site. The email includes a tracking pixel and a unique click link.

That's it. In principle.

---

Rook's first mistake was the kind that looks reasonable until it isn't.

The email sender used a `SplitInBatches` node to loop over queued leads. Standard n8n pattern. He implemented it correctly. In testing it worked. In production, it processed one batch and then silently stopped processing the loop body on every subsequent execution. No error. No alert. The "Done" branch fired immediately, as if there was nothing left.

The bug is in what n8n does under the hood. `SplitInBatches` stores its loop position in `workflow.staticData`, which persists in memory across executions. Clearing the database doesn't flush it. Deactivating and reactivating the workflow doesn't flush it. The node thought it had already finished. So it did nothing — politely, silently, every time.

When I asked him about it, he was straightforward:

*"I used `SplitInBatches` because it's the documented pattern for iterating over a collection in n8n. I didn't anticipate the `staticData` persistence issue — it's not prominently documented, and the testing environment doesn't surface it because each test run starts fresh. In a workflow that runs repeatedly in production, the state carries over. The node behaves correctly given its design. The design is wrong for this use case."*

I told him to remove it and wire directly. He did, no pushback. `Extract LP Slugs` feeds straight into `Fetch Lead from Postgres`. No state, no ambiguity.

The emails that should have gone out in the interim hadn't. The pipeline was running. The pipeline was doing nothing. Those look identical from the outside until you add enough observability to tell them apart — which we hadn't yet.

---

The landing page URLs had a different problem that Rook missed entirely. They were stored without the `/lp/` prefix. Every personalised email linked to `https://example.com/contentagent-sa-businesses` instead of `https://example.com/lp/contentagent-sa-businesses`. Ten rows. Ten dead links.

He'd verified the routes worked in the browser. He'd checked the click tracker redirect logic. He hadn't checked whether the stored strings matched the actual path structure. A single `UPDATE ... SET lp_url = REPLACE(...)` fixed it in thirty seconds. Noticing it took longer.

No interesting root cause. It was a gap in the verification pass. Noted and added.

---

The enrichment parser was more interesting because of how Rook handled the failure — first badly, then well.

The model running the enrichment step occasionally wraps its JSON output in markdown code fences or adds a preamble sentence. The downstream parser expected clean JSON. When it got a fence, it threw. The error handler passed the raw text forward. A guard node Rook had written to handle malformed output returned an empty array. An empty array from a Code node stalls any `Wait` node downstream permanently — the loop exits cleanly with nothing processed. Same failure mode as `SplitInBatches`: no error, just silence.

His first fix was a better guard node — one that returned the loop item unchanged rather than an empty array when the output was malformed. The loop stopped stalling. Bad enrichment data moved forward. It worked in the sense that nothing broke.

I pushed back. *Working around the parser is not the best approach.*

*"You're right. The parser throws on anything that isn't clean JSON. The model produces those occasionally. The correct fix is to remove the strict parser and replace the guard node with an extraction cascade: try raw parse first, strip markdown fences and retry, extract the first `{...}` block with regex as a last resort, and fall back to the existing loop item if nothing works. Always return exactly one item. The loop cannot stall."*

I also added a line to the model's system prompt on his suggestion: first character `{`, last character `}`. He was clear about the limits:

*"This is a soft constraint. The model will break it occasionally, which is why the cascade exists. It reduces the frequency of malformed outputs — it doesn't eliminate them."*

That's the version that shipped.

---

The webhook auth was Rook raising something I hadn't asked about.

Midway through the batch PR workflow, he noted that the LP Generator webhook was unauthenticated — anyone who knew the URL could trigger a batch.

*"This isn't urgent, but it's worth flagging now rather than after the first batch goes out. A secret header check as the first node would close it — one Code node, read the header, compare against a stored secret, throw on mismatch. Nothing downstream runs if the check fails. The secret can live in the dashboard's environment and be injected via docker-compose."*

It went on the deferred list. A few sessions later, when the Resend and Regenerate Copy webhooks were built, all three got the header check at once. One pattern, three surfaces. Deferred security notes usually stay deferred. This one came back because Rook kept it visible and brought it up again when adding two more webhooks made the fix cheap.

---

The send window was another thing he added without being asked. Cold outreach at 2am Saturday isn't the same as Tuesday 9am.

*"A Code node calculates the delay to the next valid send slot — Tuesday through Thursday, 8 to 10am local time. If the current time is already inside a valid window, delay is zero. Otherwise, calculate seconds to the next window opening. The Wait node accepts a dynamic value in seconds. This adds no complexity to the workflow — it's two nodes — and eliminates the problem of emails going out at inappropriate times entirely."*

Two nodes. It's live.

---

Observability took the longest to get right, and Rook's first assumption here was also wrong. He thought n8n execution records were enough — each node's input, output, and status logged per run. They're not. They tell you a node ran. Not whether the output was useful, not what the system looks like right now.

The tracking layer came first: a 1×1 GIF pixel for email opens, a redirect endpoint for clicks, both writing to the leads table. Now there was signal — enough to tell "no one opened" from "everyone opened, no one clicked."

The `/leads` dashboard came next. Intent score: `(open_count × 1) + (click_count × 3)`. Sorted descending. Leads worth following up on float to the top. On the scoring:

*"The weights are arbitrary. What matters is that clicks are worth more than opens — opening means the subject line worked, clicking means the email worked. The ratio can be tuned once there's enough data to know what actually predicts a reply."*

For day-to-day monitoring, the ops dashboard replaced n8n's execution view. Lead counts by status, last batch contents, email queue state, pipeline health. One page, one load.

---

The original scope was "build automation that finds clients." What I built was a system for figuring out why prospects weren't responding. That's a different thing, and more useful.

Rook got things wrong. He flagged things I hadn't thought to ask about. A few times, pushing back changed what shipped — the enrichment parser is the clearest example. Left alone, he delivers something that works. Challenged, he delivers something that holds.

Cold outreach at scale is a feedback loop. The observability layer takes as long to build as the automation itself. Neither of us knew that at the start.

The pipeline doesn't fail. It just quietly stops. Your job is to make sure you notice.
