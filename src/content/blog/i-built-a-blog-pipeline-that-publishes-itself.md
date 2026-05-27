---
title: I Built a Blog Pipeline That Publishes Itself
description: How I built a system that generates, queues, and deploys blog content on autopilot. The architecture, the things that broke, and why the abstraction layer saved me.
author: Armin Marxer
date: 2026-05-28
wordCount: 1100
image: /og-image.png
draft: false
---

I've been building a blog pipeline for the Kern platform. It generates content, queues it for approval, and deploys it to production — all on a schedule. The client approves or requests changes in a portal. They don't touch the infrastructure.

The project started as a straightforward feature request: clients want blog posts that publish regularly. It ended as a deeper lesson about what happens when you automate something that everyone agrees is valuable but nobody wants to do.

## The Flow

The pipeline has four stages:

1. **Schedule** — Celery beat fires hourly and checks which clients have posts due. Weekly, biweekly, and monthly schedules are supported. When a post is due, it triggers generation.

2. **Generation** — Content is generated through the ContentAgent API with the client's voice profile. Quality gate checks for repetition, specificity, and voice match before accepting the output.

3. **Review** — The generated post appears in the client's portal as a review item. They can approve, request changes, or skip. If they don't respond within 24 hours, they get a nudge email.

4. **Deploy** — Approved content gets injected into the client's generated site and deployed to their hosting target via the deployment abstraction layer.

The whole cycle takes about 3 minutes from schedule fire to review queue.

## What Broke

**Double deduction.** The first version deducted a quota credit in the worker loop and then deducted it again in the generation function. Clients started hitting quota limits after half their expected posts. Fix: moved quota deduction to a single point at the end of successful generation.

**Disappearing posts.** The initial publish flow only injected the current post into the site, not all previously published posts. First publish worked. Second publish overwrote the first one. Fix: changed the site builder to query all published posts per project.

**Stale n8n state.** An earlier version used n8n for orchestration. SplitInBatches stored loop position in staticData, which meant a single-item loop that already completed would skip the next item when re-run. Fix: remove SplitInBatches for single-item loops.

## The Abstraction That Saved Me

I wrote about this separately, but the deployment abstraction — a `DeploymentTarget` ABC with a factory and a Vercel implementation — was the single decision that kept this project from becoming a maintenance burden.

The pipeline doesn't know or care where content gets deployed. It calls `deploy()` and gets back a URL. That separation meant I could test the entire pipeline without deploying anything, and it means adding a new hosting target is a new file and one line in a factory.

## What Surprised Me

Clients don't care about the pipeline. They care that their site has fresh content. The technical architecture being invisible is the whole point.

What surprised me was how quickly the expectation shifted. A client who gets four blog posts per month for two months starts treating that as normal. When a post is late, they notice. The pipeline went from "nice to have" to "expected" in about six weeks.

Six weeks in, the pipeline isn't a feature anymore. It's just how the site works.
