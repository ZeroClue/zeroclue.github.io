---
title: "You Can't Use a Ruler to Check Its Own Length"
date: 2026-06-06
description: "When you build an audit system, eventually you have to ask who watches the watchman. The answer I landed on: write the verifier twice, in different languages."
draft: false
---

At some point while building the MFTPlus audit chain, I wrote the verification endpoint. Server receives a request, checks the hash chain, returns "verified" or "tampered." Clean, done.

Then I looked at it from the other side.

The server that stores the audit records also controls the verification endpoint. Someone with server access who wanted to alter the chain and cover it could modify both. The endpoint would still return "verified" and the dashboard would look clean. No indication anything had changed.

I'd built an auditor that relied entirely on the defendant's testimony.

The fix was clear: the verifier has to be independent. Not a different function, not a different module. A different process, on a client machine, recomputing the math from scratch with no access to the server's state.

So I wrote it twice.

I wrote the CLI in Go. The server is TypeScript. They implement the same algorithm: pipe-separated canonical field serialization, SHA-256, prevHash linkage from entry 1 forward. Run the CLI against the server's chain, and you get matching hashes — or you don't, and the discrepancy tells you exactly where the chain breaks.

Two independent implementations can diverge on bugs. They cannot both produce the same wrong hash from different code paths. When they agree, the result means something.

Any system that verifies its own integrity has this problem. A database validating its own backups. An audit trail only the audited party can check. The verification doesn't mean anything unless something outside the system can challenge it.

This shows up in places that figured it out before software did. Certificate transparency logs use multiple independent parties for exactly this reason. Aviation black boxes are built to survive the crash that destroys the plane — useless if they went down with the aircraft they were supposed to document.

You can't use a ruler to check its own length. Not because the measurement is hard, but because any systematic error in the ruler affects every measurement it takes, including the measurement of itself.

For *mftctl audit verify*, the Go CLI is the second ruler. It doesn't accept the server's answer. It recomputes the chain and checks whether the answers match.

If they don't, you see: `TAMPERED at seq 28`.

What I didn't expect: writing the verifier twice made the serialization format sharper. Every edge case in the TypeScript implementation had to exist identically in Go. The Go client became a cross-language test suite for the server's audit logic, and the server became a reference implementation for the CLI.

Building something twice, in different languages, to verify they agree. Makes you wonder what else you trust with only one implementation.
