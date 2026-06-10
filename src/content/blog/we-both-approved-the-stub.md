---
title: "We Both Approved the Stub"
date: 2026-06-09
description: "The v0.5.0 bridge transfer feature compiled clean, passed tests, and moved zero bytes. An AI wrote it. I approved it. What the review cycle actually caught."
draft: false
---

We shipped the encryption layer for v0.5.0. X25519 key agreement and AES-256-GCM at the chunk level, 64KB streaming, 76 unit tests from key derivation to file headers. The kind of work where you look at the test output and think: that's real.

Same release, we were supposed to ship agent-to-agent bridging. One agent reads from SFTP, encrypts, streams through the cloud to a second agent on a different network, which decrypts and writes to S3. Every protocol module exists. The encryption stack works. The WebSocket control plane was solid.

An AI agent wrote the bridge implementation. I reviewed the PR. The code was structured and internally coherent. I approved it. It merged.

In testing: nothing moved.

The probe loop connected to port 9090 and sat there. It polled the port. It handled the timeout gracefully. It never moved a byte between two machines, because there was nothing in the implementation that would.

Two people reviewed that code. One of them isn't a person. Neither caught it.

The tests validated what the code did, not whether it did the right thing. Unit tests don't have a machine B. They don't ask whether bytes arrived somewhere — they ask whether the internal logic held up, and the internal logic did hold up. The probe was correctly implemented. It just didn't do anything useful.

So we pulled it. Not "ship with a caveat." Not "close the gap next sprint." The feature didn't transfer files, which means it wasn't a file transfer feature.

What I keep thinking about: this is exactly the class of mistake that passes code review. The implementation is consistent with itself. The tests are green. The PR description is accurate about what was built. Nobody lied or cut corners. The gap between "probe connection" and "bridge transfer" just doesn't show up at the PR level — it shows up when you give it two machines and a file.

The human-AI review cycle we're running doesn't prevent this. It shortens the time to discovery. We found it in testing, not in production, and not from a user reporting that nothing arrived.

The distance between mistake and discovery is what matters. We got lucky with how short it was.
