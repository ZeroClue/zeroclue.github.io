---
title: "Boring Is a Feature"
date: 2026-06-12
description: "When building something other people have to trust and verify, complexity is a cost that falls on them, not just you. The boring algorithm is often the right one."
draft: false
---

The first instinct when designing a cryptographic audit system is to reach for something impressive.

When I started work on the MFTPlus audit chain, Merkle trees were the obvious move. They're used in Bitcoin, in certificate transparency logs, in distributed databases. They're elegant — a whole tree of hashes where changing any leaf invalidates everything above it. If you told an engineer "I used a Merkle tree for the audit chain," they'd nod approvingly.

I used a SHA-256 hash chain instead. Each entry links to the one before it. That's the whole structure.

The decision took about ten minutes once I was honest about the use case. Merkle trees solve one specific problem: proving a leaf is in a large set without fetching the whole set. Bitcoin light clients need this. Certificate transparency logs need this. An MFT audit chain, where you always fetch the full chain for a specific transfer, does not.

The tree would have been solving a constraint that doesn't exist.

But the deeper reason I chose the hash chain wasn't performance or correctness — it was that someone else has to verify this. The CLI verifier is in Go. The server is TypeScript. Both implement the same algorithm, and they have to stay in sync across languages and over time. A hash chain is canonical serialization plus SHA-256, which runs everywhere. Merkle tree construction with sibling proofs is a data structure that has to be implemented identically in both languages, tested to match, and maintained as both evolve.

Complexity in a system that needs to be independently verifiable is a cost you're imposing on everyone who verifies it.

An external auditor can run the Go CLI against a chain export and check whether the hashes line up. They don't need our server or access to our systems. SHA-256 is built into every runtime. The serialization format is a list of fields separated by pipes. If they want to write their own verifier from scratch, they can — the spec fits in half a page.

That wouldn't be true with a Merkle tree. The proof path format, tree construction, root hash verification: you'd need to read the spec carefully and trust that your implementation matches ours.

I notice this pattern in other places. The systems that hold up under scrutiny tend to be ones where the people who designed them prioritized the reviewer over the designer. The interesting move wasn't implementing the clever thing — it was stepping back and asking who's going to have to understand this three years from now, under pressure, during an audit.

Often the answer changes the design.

Boring code in a trust-critical system isn't a failure of imagination. It's a choice about who bears the complexity cost. The hash chain is more auditable because it's simpler, and it's simpler because being auditable was the goal — not being impressive.

The verifier outputs PROVEN or TAMPERED at seq 28. Not "proof verified against root." Not "inclusion confirmed." Just: tampered or not.

That's what an auditor needs.
