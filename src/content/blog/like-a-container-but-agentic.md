---
title: "Like a Container, But Agentic"
date: 2026-05-03
description: "I have several underutilised VPS's and an agent orchestration platform that only runs locally. This is what happened when I tried to connect the two."
draft: false
---

I have several VPS's. Each one has a job. One handles deployments, others run services, a few sit at comfortable utilisation—doing what they're supposed to, nothing more. I kept looking at them thinking there's capacity here going to waste.

PaperclipAI is an agent orchestration platform I've been working with. The concept is straightforward: it treats AI coordination like running a company. Org charts, budgets, goals, heartbeats. Every agent gets a scheduled wakeup, checks its assigned work, and executes. If it can receive a heartbeat, it's hired.

The limitation I kept running into: everything lives on one host. The control plane and every agent it manages share the same machine. That works fine until you want to run multiple "companies"—separate agent teams, each working on different things, each ideally running in the environment it's responsible for. With a single host, you're constrained by what that one machine can handle. All the agents commute to work.

I started wondering what it would take to put an agent on one of those VPS's instead.

---

The answer turned out to be simpler than I expected.

An SSH adapter. When PaperclipAI sends a heartbeat, the adapter opens an SSH connection to the remote host, runs the command, captures the output, and closes the connection. Fresh connection per heartbeat. No persistent daemon left running on the remote host. No background processes after the work is done.

This is a first pass—call it v0.1. It'll change as I understand the edges better. But as an initial implementation it does what it needs to: the connection works, commands execute, output comes back clean. The remote host stays uncluttered. The agent runs where I put it.

The simplicity is partly the point. SSH is already there. It's been reliable for thirty years. I'm not building new infrastructure—I'm using a channel that already handles authentication, encryption, and remote execution. Sometimes the right answer was already sitting on port 22.

---

Why does remote actually matter?

When I think about what I want these agents to do—manage services, make changes, verify things work—I keep hitting the same friction: the agent knows what to do, but the agent is here and the work is there.

Put an agent on a prod server and it can read the actual logs, not a report of the logs. It can check the real state of a service, make a change, and know immediately whether it worked. There's a difference between an agent that visits a machine over SSH and one that lives there. The feedback loop collapses.

Or take data that can't move. Financial records, healthcare data, source code under compliance constraints. The usual problem is someone needing to process something they're not allowed to copy anywhere. An agent that deploys to where the data already lives sidesteps the argument entirely. The lawyers can relax.

---

The thread through all of it is the same: the agent goes to where the work is.

Containers solved a packaging problem. What environment does this thing need? Define it once, ship it anywhere, it runs the same way every time.

What I keep thinking about is whether you can do the same thing one layer up. Not packaging the environment the agent runs in—packaging the agent itself. A unit of work you can place somewhere, give what it needs, point at the problem, and leave to operate. When it's done, you retrieve it.

Like a container, but agentic.

The SSH adapter is just the transport. The real thing is treating an agent as something you can send somewhere with intent—not just invoke remotely, but genuinely deploy, the way you'd deploy a service.

---

The implementation is partway there. The connection works, some gaps still need filling, and it'll evolve as I push it harder across my own machines.

But the shape of it feels right. A PaperclipAI instance with agents distributed across VPS's, each one operating in the environment it's responsible for, reporting back to a control plane that doesn't need to know where they're running.

The machines aren't underutilised anymore. They're just waiting for their agents.
