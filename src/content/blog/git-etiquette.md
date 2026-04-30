---
title: "Git Etiquette, or: How Not to Be the Developer Everyone Dreads"
date: 2026-04-30
description: "Most git guides teach you the commands. This one is about the decisions that make collaborating with you bearable."
draft: false
---

I just set up a git workflow that would make a junior developer cry.

It involves a fork, a submodule, two separate AI agents, and three repos that need to stay in sync. One agent manages the upstream fork. The other is building a new product on top of it. The submodule ties it all together because I need to push fixes back upstream, and a vendored copy wouldn't let me do that.

It's the right architecture for the problem. It's also a pain in the ass to explain in a commit message.

---

I'm not writing this as a git tutorial. There are thousands of those. I'm writing it because after years of working in repos alone, on teams, and now with AI agents that have their own opinions about branch strategy, I've developed strong feelings about how git should be used.

Not about *which* commands. About *how* you use them.

Here's what I've landed on.

---

## Your commit messages are a conversation

I have a very low tolerance for commit messages that don't tell me anything.

```
fix stuff
update
changes
wip
```

These are not commit messages. These are evidence that you don't understand what git is for.

Git history is a narrative. Every commit is a sentence in that narrative. When someone, or you six months from now, runs `git log` and tries to figure out *why* something changed, your commit message is the only thing standing between them and reading the diff.

Write it like you're explaining the change to the person who will inherit this code after you leave.

```
# Bad
fix auth bug

# Good
fix session token not clearing on logout
```

```
# Bad
update deps

# Good
bump axios to 1.7.2 — fixes CVE-2024-XXXX
```

```
# Bad
refactor

# Good
extract email validation into shared module
used by contact form and registration
```

You don't need an essay. You need the *why*. What changed is visible in the diff. *Why* it changed is what gets lost.

---

## Squash responsibly

I know there's a debate. Squash everything vs. preserve every atomic commit. I'm not a purist about either.

But here's what I am purist about: don't squash away the *context*.

If you have twelve commits that each fix a different thing, and you squash them all into "implement feature X," you've just destroyed twelve perfectly good explanations of what happened and why.

On the other hand, if your twelve commits are "fix typo," "fix other typo," "whoops," "fix," "fix for real," and "FINAL fix." Please, for the love of everything, squash those into something coherent.

The rule is simple: the merge commit should tell a story. If the individual commits tell a better story, keep them. If the story is only visible at the feature level, squash.

---

## PRs are not a formality

I've seen PRs with 47 changed files and a description that says "updates."

A pull request is a proposal. It's you saying to your team: "I think the codebase should look like this, and here's why." The description is the *why*. The diff is the *what*.

At minimum, a PR should answer:

- What does this change?
- Why does it change?
- How do I verify it works?
- Are there risks or things to watch?

You don't need a novel. Three to five sentences that give the reviewer enough context to understand what they're looking at. That's it.

---

## Branch naming isn't trivial

`fix/auth-bug` tells me something. `stuff` tells me nothing. `dev` tells me less than nothing. It actively misleads me into thinking I know what's on that branch.

Use a convention. It doesn't matter which one. Just pick one and stick with it:

```
feature/short-description
fix/what-you-fixed
chore/housekeeping-task
```

The point isn't aesthetics. It's that when I see `git branch` or look at the remote, I should be able to tell what each branch is for without opening it.

---

## Rebase vs. merge — just pick one

The longest arguments I've had about git have been about rebase vs. merge. Both work. Neither is objectively better in all cases.

What's actually bad is a team that does both inconsistently. Half the history is merge commits, the other half is rebased, and nobody knows what to expect.

Pick one. Document it. Move on with your life.

---

## The submodule thing

Here's what I set up recently, because it's a good example of git decisions that most guides skip.

I'm building a product called The Office Block. It depends on an open-source project called Paperclip. But I don't just *use* Paperclip — I maintain a fork with fixes and local changes. And I need the ability to push those fixes upstream.

So I didn't vendor the code. I didn't add it as a package dependency. I made it a git submodule.

The chain looks like this:

```
Paperclip (upstream)
  └─ My fork (fixes + local instance)
       └─ The Office Block (submodule → my fork)
```

One AI agent (Skip) manages the fork: keeps it in sync with upstream, runs our local instance. Another agent (unnamed, he'll pick his own name eventually) is building The Office Block with the fork as a submodule.

Submodules get a bad reputation. They're fiddly. `git submodule update` doesn't do what people think it does. You forget to init, or you clone without `--recursive`, and everything breaks.

But when you need to track an exact commit of a dependency *and* modify it *and* push fixes upstream, submodules are the right tool. The alternatives are worse. Vendoring means no upstream sync. Packages mean no source modifications. A monorepo means coupling two products that don't belong together.

The tradeoff is complexity for flexibility. A real decision, not a git mistake.

---

## Code reviews are a skill

Reviewing code is easy. Reviewing code *well* is not. The worst reviewers do one of two things: they rubber-stamp everything without reading it, or they nitpick formatting while ignoring the actual logic.

A good review focuses on:

- **Correctness.** Does this do what it says? Are there edge cases?
- **Clarity.** Can the next person understand this without asking the author?
- **Risk.** What breaks if this goes wrong?

In that order. Formatting debates are for linters.

If you're the reviewer and something is wrong, say so. Don't leave a comment saying "have you considered..." when what you mean is "this is broken." Passive-aggressive review comments are worse than none at all.

And if you're the author and a reviewer finds a real problem, don't argue. Fix it. Move on. Your ego has no place in a code review.

---

## The multi-agent angle

I mentioned I have two AI agents working across these repos. That's worth calling out because it's new territory for git workflows.

Skip manages the Paperclip fork. He runs `git fetch upstream`, resolves conflicts, applies our fixes, keeps things current. He has opinions about commit messages too — which is either a sign of good taste or a sign that I've been a bad influence.

The unnamed agent is building The Office Block. He works in a repo that has Skip's fork as a submodule. When he needs a fix from Paperclip, he doesn't just hack the vendored code — he goes through the fork, which means Skip's workflow and his workflow are connected through git.

I didn't design this because it's clever. I designed it because it's *correct*. Each agent owns its domain. The submodule is the boundary. Git is the coordination mechanism.

It works because the git hygiene is good. If either agent was writing "fix stuff" as commit messages, this whole setup would fall apart the first time something went wrong.

---

## The short version

None of this is controversial. Most of it isn't even hard. But I've worked in enough repos to know that the basics are where most teams fall apart. And now I'm working in repos where AI agents are part of that team, and the basics matter more than ever.

Write commit messages that explain *why*. Make your PRs readable. Pick a branching convention and stick with it. Review code like you'd want yours reviewed.

Git is a tool for collaboration. How you use it is a statement about how much you respect the people you're collaborating with. Even if some of those people aren't people.
