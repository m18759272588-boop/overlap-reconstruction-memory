# Overlap-Reconstruction Memory (ORM)

**A theoretical framework and reference implementation for intent-driven memory reconstruction in AI agents.**

> Memory systems should not retrieve. They should reconstruct.

---

## The Core Claim

Current AI memory systems treat memory as a retrieval problem: given a query, find the most similar stored fragments and return them.

ORM treats memory as a **reconstruction problem**: given a query and an intent, reconstruct the most coherent current cognitive state from incomplete, overlapping fragments — the way human memory actually works.

Three mechanisms drive this:

**Anchor** — High-confidence, decay-exempt memory units that serve as fixed coordinate points for reconstruction. Mathematically necessary: in compressed sensing terms, anchor nodes are the "certain rows" that make underdetermined reconstruction uniquely solvable.

**Overlap** — When multiple fragments point to the same fact independently, their mutual information raises reconstruction confidence. Optimal overlap rate ρ* ≈ 30–50%; below this the system is underdetermined, above it the measurement matrix loses rank.

**Reconstruction** — Rather than returning fragments ranked by similarity, ORM synthesizes a coherent cognitive state weighted by the current intent vector. The same memory pool returns different reconstructions depending on whether the active intent is "technical decision", "emotional context", or "project history".

---

## Why This Differs from Existing Systems

| System | Core Mechanism | What's Missing |
|--------|---------------|----------------|
| mem0 / OpenMemory | Vector similarity retrieval | No overlap confidence, no reconstruction |
| HippoRAG | Personalized PageRank over knowledge graph | Graph traversal, not intent-weighted reconstruction |
| MIRIX | Multi-agent routing + Active Retrieval (topic → retrieve) | Still retrieve, not reconstruct; no overlap validation |
| MemGAS | Multi-granularity + entropy router | Adaptive granularity, not intent-weighted |
| EverMemOS | MemCell self-contained extraction + compression | Extraction format only, retrieval layer unchanged |
| Zep | Temporal knowledge graph | Hours-latency writes, no overlap mechanism |

To our knowledge, ORM is the first framework to formally define:
- Overlap as a **confidence signal** (not just deduplication)
- Anchors as **mathematically necessary** nodes (not just "important memories")
- Reconstruction as **intent-weighted synthesis** (not similarity-ranked retrieval)

---

## Mathematical Foundation

ORM is grounded in **Compressed Sensing** theory.

Let the true cognitive state be an n-dimensional sparse vector **x** (unobservable directly). Each memory fragment is a measurement:

```
y_i = a_i · x + ε_i
```

**Recovery theorem**: If **x** is k-sparse and the measurement matrix satisfies the Restricted Isometry Property (RIP), exact recovery requires:

```
m ≥ C · k · log(n/k) fragments
```

For k=20 important dimensions and n=1000 conceptual axes:

```
m ≥ 20 × log(50) ≈ 78 fragments for guaranteed reconstruction
```

**Why Anchors are mathematically necessary**: Anchor nodes are zero-noise certain rows in the measurement matrix. Each anchor reduces the underdetermined solution space, making reconstruction unique. This is why anchor decay-exemption is not a preference — it is a formal requirement.

**Why Overlap raises confidence**: Fragment overlap = mutual information I(A;B). High mutual information between two fragments means independent verification of the same cognitive dimension → higher reconstruction confidence. Optimal overlap ρ* ≈ 30–50%.

**Reconstruction score**:

```
final_score(fragment, query, intent) =
    importance × similarity(fragment, query) × decay(age) × intent_weight(fragment.dimension, intent)

Anchor exemption:
    final_score = importance × similarity × intent_weight  [no decay term]
```

**Intent weighting** (T3):

```
context_score = Σ_d [ w_d(intent) × relevance_d(fragment) ]

where d ∈ {emotional, technical, stylistic, temporal, ...}
      w_d is dynamically derived from the current query's intent vector
```

---

## Architecture

```
Input (conversation / event)
        ↓
Fragment Extraction (MemCell format — self-contained context paragraphs)
        ↓
Write Gate (deterministic validation: schema → operation → L1 protection → confidence → commit)
        ↓
Storage (brainos / SQLite / Notion — user owns their data)
        ↓
Candidate Retrieval (vector similarity, top-k active fragments)
        ↓
Overlap Detection (mutual information estimation across candidates)
        ↓
Anchor Injection (forced inclusion, decay-exempt)
        ↓
Intent-Weighted Reconstruction (synthesize coherent cognitive state)
        ↓
Output (reconstructed context, not a ranked list)
```

### Memory Layers

| Layer | Semantics | Decay | Anchor eligible |
|-------|-----------|-------|----------------|
| L1 | Stable identity / core preferences | Never | Yes (auto if importance=5) |
| L2 | Active state / ongoing decisions | Slow (180d half-life) | No |
| L3 | Episode summaries / compressed history | Fast (90d half-life) | No |

### Memory Operations

| Operation | Trigger | Effect on existing memory |
|-----------|---------|--------------------------|
| add | New fact, no prior record | — |
| update | Prior record still true, needs refinement | Preserved, content upgraded |
| supersede | Prior record is now false | status=superseded + pointer |
| archive | L2 loop closes, compress to L3 | status=archived, new L3 inserted |

---

## Test Routes

Four parallel evaluation strategies (T1 is the control baseline):

| Route | Description | Status |
|-------|-------------|--------|
| T1 | Existing mem0 + brainos system | ✅ Live (control) |
| T2 | MemCell extraction + recency decay + anchor | 🔨 In development |
| T3 | Multi-agent intent-weighted reconstruction | 📋 Designed |
| T4 | MAGMA-style graph structure upgrade | 📋 Future |

**Isolation principle**: T1 runs on production data. T2/T3 run on local snapshots. Zero cross-contamination.

---

## Dataset

**Malin-longitudinal-v1** (releasing with T2 results):

- Real multi-agent, multi-topic, long-horizon conversations
- Spans Brain OS / technical decisions / interpersonal context / corrections and memory conflicts
- Organized by Month1 / Month2 / Month3 for temporal evaluation
- Anonymized

This dataset is unique: no existing benchmark covers multi-agent + cross-topic + high-intensity long-term memory with real contradiction and supersede events. It cannot be synthesized.

---

## Evaluation Dimensions

| Dimension | Metric | What it measures |
|-----------|--------|-----------------|
| Recall Accuracy | top-1 / top-3 accuracy | Basic answer correctness |
| Context Fidelity | expected_context_nodes coverage | Did reconstruction pull relevant connected memories? |
| Storage Efficiency | gzip compression ratio as proxy | Information density per stored unit |
| Token Cost | $/month @ 100 users | Product viability |

Benchmarks: LongMemEval (arXiv:2410.10813), Malin-longitudinal-v1 (self-built).
Not using LoCoMo: 6.4% answer set errors documented.

---

## Related Work

- EverMemOS: arXiv:2601.02163 (ACL 2026)
- HippoRAG: arXiv:2405.14831 (NeurIPS 2024)
- MIRIX: arXiv:2507.07957
- MemGAS: arXiv:2505.19549 (ICLR 2026)
- MAGMA: arXiv:2601.03236 (ACL 2026)
- MemoryOS: arXiv:2506.06326
- mem0: arXiv:2504.19413 (ECAI 2025)
- LongMemEval: arXiv:2410.10813 (ICLR 2025)
- Memory in the Age of AI Agents survey: arXiv:2512.13564

---

## Status

- [x] Theory (this document + THEORY.md)
- [x] Reference implementation: Brain OS (production, private)
- [ ] `pip install orm-memory` — releasing with T2
- [ ] `npx orm-memory-mcp` — releasing with T2
- [ ] Malin-longitudinal-v1 dataset — releasing with T2 results
- [ ] arXiv paper — releasing with T3 results


## Falsification Conditions

ORM predicts failure under the following conditions. These are the tests we are actively running:

1. **overlap < 10%** — fragments near-orthogonal → reconstruction non-unique → Recall Accuracy collapse predicted
2. **overlap > 70%** — fragments near-duplicate → measurement matrix rank-deficient → Storage Efficiency degrades with no Accuracy gain
3. **anchor corruption** — wrong fact marked is_anchor → global reconstruction bias → Pollution Rate spike predicted
4. **high contradiction rate** — dense supersede events → reconstruction confidence oscillation → Context Fidelity degradation predicted
---

## License

MIT
