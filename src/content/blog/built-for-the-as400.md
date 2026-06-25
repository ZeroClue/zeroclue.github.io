---
title: Built for the AS400
description: I chose Java for nuAgent because of an AS400 requirement that never materialised. The agent ran on Windows, Linux, and macOS instead. The wrong constraint drove the right decision.
author: Armin Marxer
date: 2026-06-25
wordCount: 580
image: /og-image.png
draft: false
---

nuMotusDirect needed agents at client sites. The platform ran centrally: scheduled orchestration, configuration, policy. The actual file work happened at the edge. Agents on client infrastructure, running transfers, reporting back.

The question was what to build them with.

One potential client had an AS400. IBM's midrange workhorse, still running in production at financial institutions that had been running it since the eighties. Java was available on AS400. .NET was not. If we wanted that client, the agent had to be Java.

So I built the agent in Java.

The AS400 deployment never happened.

---

What happened instead: the Java agent ran on Windows. It ran on Linux. It ran on macOS. Every platform a client site might be running, without modification, without a separate build. The constraint I designed for turned out to be the wrong constraint — but the decision it forced was the right one.

That's a pattern worth noticing. The AS400 requirement didn't survive contact with the sales process. The cross-platform agent it produced has been the most broadly deployable thing I've built.

---

The other two decisions in nuAgent came from a different kind of constraint: enterprise networks.

If you've tried to deploy software that needs a persistent connection into a large financial institution, you know the problem. Inbound connections are a security review. A firewall exception request. A conversation with someone whose job is to say no. The deployment that should take a day turns into a month of waiting.

The agent only needed outbound SSH. That's almost always open. SSH outbound is how your staff connect to things; blocking it causes more problems than it solves. The RabbitMQ telemetry (status, heartbeat, transfer events back to the central platform) ran through that SSH tunnel. The agent phoned home. Nobody phoned the agent.

No inbound firewall rules. No security exception requests. The agent installs, opens its outbound connection, and the platform picks it up on the other end.

I designed that before the deployment problem existed, because I could see that it would. Enterprise security teams don't make exceptions easily, and they shouldn't. The answer isn't to fight that — it's to not need the exception.

---

The provisioning flow was the same thinking applied to identity.

The agent generates its own keypair on first run. It doesn't receive credentials from a human. It doesn't share a secret with the server. It creates its own identity and presents it. The central platform verifies and registers it. The credentials are stored in a way that only that specific agent instance can read them — tied to the machine, not portable.

Nobody has to provision it. Nobody can replay the credentials from another machine. If the agent is compromised, the blast radius stops at that installation.

It's zero-trust key exchange without anyone calling it that. The name came later. The problem it solves is as old as distributed systems.

---

The AS400 client never signed. The agent ran on everything else instead. The firewall constraint I anticipated turned out to be real on nearly every deployment. The provisioning design held for the life of the platform.

Sometimes the wrong reason gets you to the right place. Sometimes the constraint you designed for disappears and leaves a better one behind.
