---
title: Boring Is Better for Your Audit Trail
description: When I designed the MFTPlus audit chain, every instinct said Merkle tree. I used a hash chain instead. The boring choice was the right one, and the reason tells you something about how to pick data structures.
author: Armin Marxer
date: 2026-06-25
wordCount: 620
image: /og-image.png
draft: false
---

When I sat down to design the audit trail for MFTPlus, the answer felt obvious. Cryptographic audit chain. Tamper-evident. You want to prove that a log entry hasn't been modified after the fact. The data structure people reach for in that situation is a Merkle tree.

Bitcoin uses one. Certificate transparency logs use one. Every blockchain explainer leads with one. If you're building anything with "cryptographic audit" in the requirements, that's where your mind goes first.

I didn't use one.

---

Merkle trees solve a specific problem: proving a single leaf belongs to a large set without fetching the whole set. A Bitcoin light client can verify that one transaction is in a block without downloading the full megabyte. Certificate transparency lets a browser verify a certificate's inclusion without fetching the entire log. The optimization is real. It's valuable. It matters enormously for those use cases.

MFT audit is not those use cases.

When you verify an MFT transfer's audit trail, you fetch the full chain for that transfer. Not a proof path. Not a subset. The complete sequence of entries, start to finish. You need all of it to verify that none of it was tampered. There is no sparse verification problem. The Merkle tree's core optimization doesn't apply.

What you'd buy with a Merkle tree: tree construction, sibling proof generation, proof serialization, root hash management. In two languages, since the CLI verifier is Go and the server is TypeScript. If the implementations diverge on any detail of the proof format, verification breaks silently.

What you get with a hash chain: canonical field serialization, one SHA-256 call per entry, and each entry includes the previous entry's hash. It's ten lines of logic. SHA-256 behaves identically in Go and TypeScript. If the two implementations disagree on any entry, the mismatch is immediately visible.

---

The part that settled it was thinking about the output.

A Merkle proof tells an auditor: "inclusion confirmed." A hash chain tells an auditor: PROVEN or TAMPERED at sequence 47.

Not "proof verified against root." Not "inclusion confirmed." Tampered, and here's exactly where.

That's what an auditor actually needs. A financial institution running compliance review doesn't want to interpret a proof path. They want a yes or a no and, if it's a no, a location. The simpler data structure produces the more useful output.

---

There's a GDPR angle that reinforced the decision. The chain stores only opaque transfer references and content hashes. The PII — sender, recipient, filename, IP address — lives in a separate table that can be deleted cleanly for right-to-erasure requests. The audit record stays intact. The personal data goes. That separation is easier to reason about when the chain itself is simple.

---

I've noticed a pattern in my own thinking when I'm designing something: the interesting-sounding approach and the correct approach are often not the same thing. Merkle trees are interesting. They're elegant. Writing about them sounds impressive. Hash chains are a linked list with a hash function.

The test is always: what problem am I actually solving? Not what problem is the cool data structure designed to solve — what is my actual problem?

My problem was: did any of these 500 entries change after the fact? A hash chain answers that in O(n) and outputs a plain-English result.

Boring is better for your audit trail. The auditor who has to sign off on it will thank you.
