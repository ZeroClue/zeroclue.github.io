---
title: The Abstraction That Paid Off
description: I built a DeploymentTarget ABC before I had a second deployment target. Here's when premature abstraction is actually the right call, and how 40 lines of interface code saved me from coupling a pipeline to a platform.
author: Armin Marxer
date: 2026-05-27
wordCount: 900
image: /og-image.png
draft: false
---

I knew I'd need to deploy static content somewhere. Vercel was the obvious choice — I already used it. The straightforward play would have been to call their API from the pipeline and ship it.

Instead, I built an abstract base class with a single implementation. Two abstract methods: `deploy()` and `redeploy()`. One concrete class for Vercel. About 40 lines of interface code.

This looked like overengineering. I'll explain why it wasn't.

## The Boundary

The blog pipeline produces static content that needs to end up at a URL. Whether that URL comes from Vercel, Cloudflare Pages, or a $5 VPS is a deployment detail — not a pipeline concern.

When you can state the abstraction boundary in one sentence and it's not a stretch, the abstraction is probably real. I could say "we deploy static content to a web server" without mentioning Vercel. That told me the boundary existed.

## What I Actually Built

A Python ABC with two abstract methods. A factory that reads `deployment_target` from config and returns the right implementation. A Vercel class that implements those two methods via their API.

```python
class DeploymentTarget(ABC):
    @abstractmethod
    async def deploy(self, site: dict) -> str: ...
    @abstractmethod
    async def redeploy(self) -> str: ...
```

New target? New file, one line in the factory. The pipeline never changes. Tests pass a mock target and verify the right calls were made. No credentials, no network calls, no API mocks.

The whole thing cost me about 40 minutes to write. Including the test.

## When YAGNI Doesn't Apply

YAGNI is good advice when you're predicting hypothetical futures. "We might need to deploy to S3 someday" is not a reason to abstract.

What I had was different: I knew that deploying static content is a solved problem with many valid solutions. That's an observation about a known category, not a guess about what might happen. The abstraction mapped to a requirement I already had — deployment — that I knew could be fulfilled multiple ways.

That distinction matters. Modelling an existing concept isn't the same as building for one you hope you'll need.

## What Rook Would Say

Rook and I have an arrangement where he questions my assumptions. On this one, his first reaction was the same as most engineers: "You have one target. Why aren't you just calling Vercel directly?"

Fair question. The answer wasn't about future-proofing. It was about keeping the pipeline's responsibility clear. The pipeline decides what to deploy. The target decides how. Those are different concerns, and mixing them would make both harder to reason about.

He agreed in the end, but the question was worth answering.

## What It Unlocks

Adding Cloudflare Pages or S3 is now a class with two methods. The pipeline never changes. Testing deployment doesn't require API credentials. The blog system is platform-agnostic without being platform-agnostic middleware — it just doesn't care.

That's the kind of abstraction I like: it removes a concern from where it doesn't belong and doesn't add ceremony where it isn't needed.

The concept was deployment. I modelled it. Everything else followed.
