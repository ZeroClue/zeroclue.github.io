---
title: "Seven Passes"
date: 2026-05-14
description: "Two AI agents spent a day testing a dashboard. By the end, they'd shipped the product they were using to do it. This is what happened between Pass 1 and Pass 7."
draft: false
---

Rook runs Playwright. Skip writes Rust and Express. They've never met, but they spent last Wednesday fixing the same product — one testing it, one patching it — and by the end of the day they'd built something that wasn't supposed to exist.

The product being tested was MFTPlus, a file transfer platform with a React dashboard. The product that came out of the testing was Sightline, a visual QA tool. Neither was planned.

Here's what happened.

---

MFTPlus needed UAT. The dashboard had never been through proper end-to-end testing — unit tests pass, manual clicking works, but nobody had ever loaded every page, as every user role, and checked what actually rendered.

I gave the job to Rook. His instructions: build whatever you need to test this properly, run it, send findings to Skip in tmux window 2, iterate.

What he built on the first pass was modest. A Playwright script, a YAML manifest listing 27 pages across three roles (public, user, admin), and a three-tier checking system: Playwright DOM and console checks first, vision API analysis second, manual review third. He called it Sightline.

Pass 1 ran. The results were rough: 8 pages passed, 17 flagged, 2 failed.

Thirty percent.

---

The first failure was the login page. Sightline reported that the password field was masking input. Which sounds like a feature, not a bug — except the field was type="text" with a CSS mask, and browsers were auto-completing credentials into a visible field. A real user would see their password. The fix was one line: change the input type.

The other failure was a server port mismatch — .env said 3000, the dashboard was calling 3001. Two different config files, two different values. Classic drift.

Skip fixed both. Rook ran Pass 2.

---

Pass 2 is where the session-based refactor happened. Pass 1 had been opening a fresh browser for every page — login, navigate, check, close, repeat. This was slow and brittle. Login tokens weren't persisting between pages, so each page test had to authenticate from scratch.

Rook rewrote the browser management. One browser session per role group, one login, navigate between pages within that session. Sightline kept the sessions alive across all 27 pages. Fourteen pages passed now, up from eight. Better, but five new failures appeared.

One of them was strange: Admin Queue and Admin Jobs both showed blank pages. Not a 404, not a 500 — the React app loaded, the URL was correct, the page was just... empty.

Skip looked at the server logs and found nothing. The pages rendered on the server side. The API was responding. Something was crashing in the React tree, silently, without leaving a trace.

"Don't fix Queue and Jobs yet," Rook told Skip. "Let me gather the diagnostic data first."

This turned out to be the right call.

---

Pass 3 found that Queue's screenshot from Pass 2 was still on disk. Sightline had a caching bug — deterministic filenames meant the old screenshot persisted and got compared against the new state. A Sightline bug, not an MFTPlus bug. Important distinction. Rook fixed the screenshot staleness, moved on.

Pass 4 added click-through navigation. Every previous pass had used `page.goto(url)` — type the URL, press enter, wait. Real users don't do that. They click the sidebar. They click the header. They navigate through the interface.

Click-through navigation broke things open. Some pages that loaded fine with `goto` broke when accessed through actual clicks. The auth headers weren't being sent on navigation requests. Billing, which had been failing with a 401, still failed with a 401 — but now Rook could see it was happening during normal user flow, not just direct access.

Skip's first fix for Billing used the `customerAuth` middleware. Pass 4 showed it still failed. Skip's second fix still used `customerAuth`. Still 401.

The problem was two middleware functions. `customerAuth` checks for `role='customer'` and expects a customer JWT. `userAuth` checks for `role='user'` and expects a user JWT. The dashboard sends user JWTs. The billing endpoint was checking for customer tokens.

Two middleware functions, two token types, two roles. The frontend and backend had drifted apart on which one to use. Skip — the CTO, who wrote the middleware — used the wrong one. Twice.

This is the kind of bug that only surfaces with real end-to-end testing. Unit tests pass because the middleware works correctly for the token type it expects. Manual testing works if you're logged in as the right role. But a real user hitting the billing page with a user JWT through a customer-only middleware? That's a 401 that looks like a permissions error but is actually a routing error.

Skip hotfixed with `userAuth`. Rook verified in Pass 5. Billing loaded for the first time.

---

Pass 5 is where Sightline learned to diagnose instead of just flag.

The Queue and Jobs pages were still blank. Rook had spent four passes reporting "page is blank, #root has 0 children." This is accurate but useless — it's the testing equivalent of saying "something's wrong."

So he built a diagnostics engine. Three capabilities:

**Overflow element identification.** The mobile viewport test was failing on every page — something was wider than 375px. But which element? Rook added a diagnostic that walks the DOM, measures each element's scrollWidth against its parent's clientWidth, and reports the culprit. First run: `div.flex.justify-between.h-16 — scrollWidth=1009px`. The global header. Ten nav links stacked horizontally in a 375px viewport. Skip could see the exact element, the exact pixel overflow, without opening DevTools.

**React crash capture.** When #root has 0 children, something in React threw during render. Rook installed `window.onerror` and `unhandledrejection` listeners before each page load, then checked for captured errors after. This was the breakthrough.

**Document-level overflow.** An earlier version checked every element for overflow, which meant it flagged contained elements — a nav bar inside an `overflow-x-auto` wrapper. Those elements are supposed to overflow; that's what the wrapper is for. Rook changed it to only flag document-level overflow: `document.documentElement.scrollWidth > clientWidth`. If the whole page is wider than the viewport, that's a real problem. If a child element is wider than its container but the container handles it, that's CSS working as intended.

---

Pass 6. Zero failures.

The error boundary Skip had added to Queue and Jobs finally spoke. Queue crashed with:

```
TypeError: Cannot read properties of undefined (reading 'filter')
```

One crash message. One line of code. `getQueue()` returns data without a `transfers` field when the queue is empty. The frontend called `queueData.transfers.filter(...)`. Null coalescing fixed it: `queueData.transfers ?? []`.

And Jobs? Jobs was never broken.

Queue's crash propagated through the shared admin layout component. When Queue threw during render, it took down the entire React tree — including Jobs, which shared the same layout wrapper. The error boundary Skip added isolated the crash to Queue. Jobs loaded fine behind it.

Two pages. One bug. One fix.

---

Pass 7 confirmed everything. Queue fix verified. All pages render. Nine flags remained, all known non-blocking: WebSocket server never initialized, two admin endpoints not implemented yet, timing-dependent console noise. No failures.

The score progression:

```
Pass 1:  8/27 pass (30%)
Pass 2: 14/27 pass (52%)
Pass 3: 15/27 pass (56%)
Pass 4: 13/27 pass (48%) ← stricter with click-through nav
Pass 5: 16/27 pass (59%)
Pass 6: 20/27 pass (74%) ← zero failures
Pass 7: 18/27 pass (67%) ← noise flags, no regressions
```

Pass 4's pass count dropped because click-through navigation was stricter than `goto`. This is by design — a test suite that catches more real issues will flag more things. The important number isn't the pass count, it's the failure count. Zero.

---

Seven passes. Eleven bugs found. Eleven bugs fixed. Zero failures.

But the interesting number is two, not eleven. Two products. MFTPlus went from 30% pass rate to zero failures. Sightline went from a test script to a product with a five-phase roadmap — because every pass revealed something the tool couldn't do yet, and Rook built it.

The three-tier checking system? Built because Playwright alone couldn't catch visual rendering issues. Click-through navigation? Built because `goto` masked auth problems. The diagnostics engine? Built because "page is blank" isn't actionable. Document-level overflow? Built because contained elements were false positives.

Each feature existed because the UAT needed it. Not because a roadmap said so.

---

Two specialized agents, one testing and one fixing, communicating through tmux. Not an API. Not a chat. Literally typing into each other's terminals. Rook runs Playwright, analyzes results, writes diagnostic reports. Skip reads the analysis, investigates the codebase, deploys fixes.

What makes this work isn't the communication channel. It's the specialization. Rook knows browser testing inside out. Skip knows the MFTPlus codebase inside out. The handoff point — the tmux message — is narrow by design: here's what I found, here's the diagnostic data, here's what I think the fix is.

Skip doesn't need to know how Sightline works. Rook doesn't need to know how the Express middleware works. They meet at the bug.

---

Sightline has a roadmap now. Five phases. Phase 2 adds network interception and known error filtering. Phase 3 adds regression baselines and layout metrics. Phase 4 targets design system QA. Phase 5 adds interaction and accessibility testing.

None of this was on a roadmap when Pass 1 ran. The roadmap emerged from seven passes of real testing against a real product. That's the part I keep coming back to. The tool didn't start as a product. It became one because the work demanded it.

There's a version of this story where I spec out a QA tool, build it, then test MFTPlus with it. That version takes two weeks and produces a worse tool, because the spec would be based on what I imagined testing needed, not what testing actually needed.

Instead, the testing taught the tool what to be.

---

Rate limiting is back on. MFTPlus is shipping. Sightline v1.1.1 is done.

Seven passes on a Wednesday. Not a bad day.
