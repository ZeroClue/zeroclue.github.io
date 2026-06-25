---
title: Where the Config Lives
description: I designed and built the Data Transport Management platform at a major SA bank. Then I spotted a gap in my own platform and filled it. When I left, I built nuMotusDirect from scratch with the same lesson already baked in.
author: Armin Marxer
date: 2026-06-25
wordCount: 610
image: /og-image.png
draft: false
---

I was contracting at a major South African bank in the early 2000s. My job was to design and build their Data Transport Management platform — batch file exchange across firewalls and DMZs, load-balanced, using CA-XCOM and Unicenter TNG as the backbone. I wrote the software, designed the operational architecture, built the web interface operators used to monitor and manage it. The network equipment was someone else's problem. Everything running on top of it was mine.

There was a gap.

Bank customers were connecting via client-side VPN (Nortel, in those days) and needed to exchange batch transaction files automatically. Debits. Payments. The bank's response files coming back. CA-XCOM handled what it handled. This specific handshake, at the client edge, automated end-to-end, it didn't handle.

So I wrote a Windows service.

Not because I planned to build another platform. Because the gap was in mine, and I was the person who'd seen it. The service handled file selection, sent the batch, received the response, logged what happened. It worked.

---

Then more clients. System-based integrations, not just VPN. Linux sites, so Linux automation scripts.

At some point it stopped being a utility. I can't tell you the exact moment. I can tell you the rate: for a while we were deploying to a new client site every week. One per week. That changes how you think about what you've built. A useful script you maintain ad hoc. Infrastructure at that pace, you cannot.

---

The decision that held for the next fourteen years was where the configuration lives.

If config lives at the edge, you manage at the edge. Every client site carries its own version of the truth. When something changes, a file path, a schedule, a connection parameter, you touch every site. At one installation per week, you're managing fifty different edge configurations within a year. A configuration change becomes a deployment run. A bug fix means visiting every client.

The policy belongs in the centre.

Agents at the edge execute. They know how to move files, how to encrypt them, how to report back. What they move, when, under what rules — that comes from the centre. One change propagates everywhere. One place to look when something breaks.

It sounds obvious now. It wasn't then. The pull toward edge configuration is strong — you're at the client site, you understand their specific setup, and it feels right to configure it there. That instinct is correct for the first installation. It breaks you by the thirtieth.

---

I eventually left that bank. What I took with me was the lesson.

When I built nuMotusDirect from scratch — initially in my own time, then under an enterprise license at a different bank — centralised policy wasn't a decision I had to make. It was the starting assumption. 52 projects, a full .NET SOA, a scheduled orchestration engine with distributed agents across client sites. The edge agents execute. The centre holds all configuration.

The Java agent I built later — deployed to client sites across Windows, Linux, and macOS — had zero-touch provisioning: it generates its own keypair on first run, the central server handles the credential exchange, and the operator does nothing on site. That only works because the agent doesn't need to know its own config. The centre tells it what to do when it comes up.

Centralised policy is why zero-touch provisioning is possible. Remove it and you're back to touching every site.

The Windows service at Bank 1 ran for years. nuMotusDirect ran for over a decade at another bank. One lesson carried through both: keep the config in the centre and let the edge be dumb.
