---
title: "Gating on Value, Not Compute"
date: 2026-06-16
description: "Feature tiers should justify themselves by what they deliver, not by what they withhold. How we decided where to put the MFTPlus audit chain."
draft: false
---

When you're building a SaaS product, there's a gravitational pull toward locking your best features at the highest tiers. More features withheld at the top means more upgrade pressure. The logic feels clean.

There's a different frame: each tier should justify its price by the value it delivers, not by what it withholds.

We had a useful test case with MFTPlus. The SHA-256 hash chain is the feature that turns your audit log from a database table into something cryptographically verifiable. It needed a tier assignment. The instinct was Pro or Enterprise; save the good stuff for higher-paying customers.

But we asked a different question: at which tier does this feature justify the purchase?

If the hash chain lives at Enterprise, we're charging enterprise prices for something every compliance-conscious customer actually needs. And we're selling Starter and Pro customers audit logs that can be quietly altered. They might not know it yet, but that's what they're buying.

So it went at Starter, $150/month. SHA-256 chain plus `mftctl audit verify`. Proof that the record exists and hasn't changed, verifiable from a terminal in under a second, offline if needed. Pro adds HMAC-SHA256 keyed integrity and signed daily snapshots — infrastructure that genuinely costs more to run and that independent auditors need. Enterprise adds OpenTimestamps and Ed25519 signatures for regulated industries where vendor independence isn't optional.

Buyers have been through enough SaaS pricing cycles to recognize feature withholding when they see it. When a feature is gated at a tier that doesn't match its value, they assume you're extracting. When it lands where it actually belongs, the pricing page makes sense without a sales call to explain it.

The audit chain belongs at Starter because that's where it justifies $150/month. Not because that's where upgrade pressure peaks.

Customers who understand what they're paying for tend to stay. Customers who feel squeezed tend to churn and write blog posts about it. Gating on value is more honest — and it turns out more customers notice than you'd expect.
