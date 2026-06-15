# ORM Manifesto — User-Owned Cognition

---

> **Your AI should not own your memory.**
> **Memory should be portable. Intelligence should be replaceable.**

---

## The Problem

Every conversation you've had with an AI system has made it know you better.

Your preferences. Your decisions. Your failures. Your corrections. Your long-term projects. Your evolving thinking.

None of it belongs to you.

It belongs to the platform.

When the platform changes its terms, raises its prices, shuts down, or changes how memory is handled, retained, or used — you have no say. You cannot export it instantly. You cannot migrate it to another system. You cannot verify what they stored or how they used it.

This is not a technical limitation. It is a design choice.

**Your AI may understand you deeply.
Understanding ≠ Ownership.
Your memory should still be truly yours.**

---

## Why Now

AI models are becoming replaceable.

What is becoming scarce is continuity.

As users move across GPT, Claude, Gemini, open-source models, and agent systems, the missing layer is no longer intelligence. It is memory.

The future AI stack will not be:

```
model → user
```

It will be:

```
user-owned memory → interchangeable intelligence
```

The model is a temporary compute layer. It can be swapped. What cannot be swapped without loss is your accumulated cognitive history — your preferences, your decisions, your evolving understanding of yourself and your work.

This is the layer ORM is building.

---

## The Principle

ORM is built on a simple premise:

**Memory belongs to the user. Not to a platform. Not to a model.**

Your memory should move with you.

Today's AI landscape inverts this:

- The model is permanent (you stay on their platform)
- The memory is disposable (you lose it when you leave)

We believe the correct architecture is the opposite:

- **Memory is permanent** — it follows you, belongs to you, is verifiable by you
- **The model is replaceable** — swap it without losing your history

---

## The Rights

Users should be able to:

- **Inspect** their memory at any time
- **Export** it instantly, in full, in a portable format
- **Revoke** access from any system
- **Migrate** across models without losing continuity
- **Verify** what was stored and when
- **Consent** to how memory is used, shared, or retained
- **Delete** with guaranteed permanence

These are not features. They are rights.

---

## On Data and Evaluation

ORM's longitudinal datasets are used exclusively for evaluation:

- Retrieval quality
- Reconstruction fidelity
- Contradiction handling
- Longitudinal memory stability

**No model training. No fine-tuning. No distillation.**

All released datasets are anonymized before publication.

We believe your memory should not be used for any purpose — training, sharing, analysis — without your explicit, informed, revocable consent. This is a structural principle that ORM is built around.

If you are building on ORM: adopt this principle. Make it explicit. Make it verifiable.

---

## The Architecture That Follows

These rights are not achievable with current centralized memory architectures. They require:

**Storage that the user controls** — not a platform database, but user-owned storage (local, self-hosted, or user-designated). ORM's reference implementation supports Notion, PostgreSQL, and local SQLite as user-controlled backends.

**Processing that does not persist** — memory extraction and reconstruction happen at processing time. The system touches your data only in that moment, then releases it.

**Portability by design** — memory is stored in open formats (JSON, SQL). No proprietary lock-in. Any system that implements the ORM interface can read your memory.

**Anonymization before sharing** — any data used for evaluation or research is anonymized before it leaves your environment.

---

## The Structural Parallel

ORM's architecture shares a structural worldview with decentralized systems — not because we are building on blockchain, but because the problems we are solving are the same:

**Fragment = Distributed Partial Truth**
No single fragment holds the complete picture. Multiple fragments, overlapping and cross-validating, reconstruct the whole. This is distributed truth assembly, not centralized state read from a single database.

**Overlap = Proof of Coherence**
When multiple fragments independently point to the same fact, confidence rises. We call this Proof of Coherence: independent witnesses raising collective certainty. Traditional retrieval systems treat overlap as noise and suppress it. ORM treats it as signal.

**Anchor = Trusted Validator**
Anchor memories are high-confidence, decay-exempt fixed points — identity-defining beliefs and preferences that serve as coordinates for reconstruction. They function like a trusted validator set: their stability makes the rest of the reconstruction possible.

**Supersede ≠ Delete**
ORM never deletes history. Old memories transition through states: active → superseded → archived. The lineage is preserved. The audit trail is complete. You can always see what you believed, when you believed it, and what replaced it.

This is not crypto. But it is the same worldview applied to cognition:

*Distributed. Verifiable. User-owned. Portable.*

---

## What We Are Building

We are not building another AI memory product that stores your data on our servers.

We are building a **cognitive infrastructure protocol** — an open framework for how AI memory should work when memory belongs to the user.

The model is rented. The memory is yours.

---

## The One-Line Version

**Memory should be portable. Intelligence should be replaceable.**

---

*ORM — Overlap-Reconstruction Memory*
*github.com/m18759272588-boop/overlap-reconstruction-memory*
