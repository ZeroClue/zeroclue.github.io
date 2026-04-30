---
title: "Building the Same Thing Twice (On Purpose)"
date: 2026-04-29
description: "MFTPlus v1 had nine containers and multiple compose files. Then I closed the laptop and sat on the couch. This is what changed."
draft: false
---

At some point last year, I typed a docker compose command that wrapped twice across the terminal.

```
docker compose -f docker-compose.yml -f docker-compose.staging.yml \
  -f docker-compose.fail2ban.yml -f docker-compose.what-the-hell-am-i-thinking.yml
```

I looked at it for a moment. Closed the laptop. Walked over to my wife, gave her a kiss, and sat down on the couch.

Didn't think about it for a week.

---

That was MFTPlus v1. An automated, secure transport mechanism for any kind of data — the thing you build when you're tired of the script that worked fine until it didn't. Scripts fall through the cracks when known things break. Then there's chaos, panic, and it generally ends in confusion and pain. MFTPlus was supposed to fix that.

Nine containers later, it had become a statement about my ability to do things *correctly*.

The thing is — I loved building it.

That's the part that's hard to explain. The container sprawl wasn't something that happened to me. I made it. Enthusiastically. Every new service felt justified at the time. Every abstraction had a reason.

Which is, I now understand, one of the most dangerous sentences in software development.

---

V1 is also where I learned to build with AI. Or thought I did.

I came in with an idea and used models to move faster — get components built, get patterns right, skip the friction of figuring everything out from first principles. What I didn't expect was the learning curve. Models, agentic coding, prompt engineering, context management — I got completely absorbed. It was *fun*. I learned an unfathomable amount, and I didn't notice what was accumulating in the background.

What I didn't account for is that when you ask an AI to help you build something, it helps you build *more* of it. More abstractions. More services. More config files with increasingly unhinged names.

I thought I was steering. Turns out I was wrong.

The AI wasn't doing anything wrong. It was very good at answering the question I kept asking, and I kept asking slightly the wrong question. *How should I handle this?* tends to produce more architecture than *should I handle this at all?*

By the time I typed that compose command, I had a Go codebase I was genuinely proud of, wrapped in infrastructure I could no longer fully explain.

V1 was the classroom. I just didn't know it yet.

---

The week on the couch was useful.

I came back to v1 with one decision already made: I'm keeping it. There are good ideas in there. But I'm not continuing it — I'm starting over.

New folder. Empty project. One rule written at the top of a note I still look at:

*Think less.*

Not think worse. Not stop thinking. Less of it in the code. Less abstraction, less infrastructure, less *what if we need to scale this later*. Developer-first, not enterprise-first. Build for the person who wants to move files reliably — not for the compliance committee that might exist one day.

---

V2 is leaner. Not because I became a minimalist, but because I built a system to argue me out of my own worst instincts.

Something else changed too. The way I talk to models is completely different now — not just the prompts, but the whole approach. Giving them context, persona, something to push against. V1 I was asking for help. V2 I'm having a conversation.

The one thing I've figured out about working with AI — the thing that took longest to learn — is that how you communicate with it can never stay the same. The models keep evolving. The techniques that worked six months ago are already going stale. The skill isn't learning how to talk to AI. It's accepting that you'll have to keep relearning it.

I call it the devil's advocate approach, though it's less a methodology and more a habit born from necessity.

When I have a feature idea now, I don't build it. I argue about it first.

I take the idea to a model — doesn't matter which one — and ask what it thinks. I pull on it. Ask for problems, improvements, reasons it's a bad idea. I work it until it stops being a sentence and starts being a structured thought.

Then I open a new session, different model, and paste that thought in cold. No context, no history. Just: *what do you think of this? What's wrong with it?*

Then I relay each session's output to the other. Back and forth, a few rounds, adjusting the context each time. The two models have no shared memory. They can't just agree with each other. What I'm looking for is either convergence — both land on roughly the same shape — or a collision that reveals something I hadn't seen.

What comes out the other end isn't a feature. It's a PRD that's already survived an argument.

The ideas that don't survive don't get dropped outright. They go to a third conversation with different context, and I watch what happens. What I've found is that the multi-model, multi-context approach strips out an enormous amount of noise. The features that make it through aren't necessarily the best ideas I had. They're the ones that held up.

V1 had no filter. Every idea that sounded good became a container.

V2 has a gauntlet.

---

I'm still building v2. It's not done.

I've probably already added more features than I should — the devil's advocate agent keeps telling me so, and I keep listening to it less than I should. Old habits.

But the architecture is different. The thinking is different. And somewhere in that folder I haven't deleted, v1 is still sitting there — nine containers, multiple compose files, and everything I learned getting here.

I'm not sure it's completely dead.

Maybe that's the third iteration talking.
