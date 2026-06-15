# ORM Theory — Formal Framework

**Overlap-Reconstruction Memory: A Compressed Sensing Approach to Intent-Driven Cognitive State Reconstruction**

---

## 1. Problem Formulation

### 1.1 The Retrieval Failure Mode

Existing memory systems formulate the problem as:

```
retrieve(query) → ranked_list(fragments)
```

This fails because:

1. **Fragment isolation**: A fragment returned without context loses the associative network that made it meaningful.
2. **Static ranking**: Similarity-to-query is computed without reference to current intent. The same query about "the project status" should return different context depending on whether the agent's current intent is operational (what's next) or reflective (why did we decide this).
3. **No confidence aggregation**: If three fragments independently point to the same fact, this convergence carries evidential weight. Retrieval systems treat it as near-duplicate noise and suppress it.

### 1.2 The Reconstruction Formulation

ORM reformulates the problem as:

```
reconstruct(query, intent, fragment_pool) → cognitive_state
```

Where `cognitive_state` is a synthesized coherent representation of the agent's most current understanding — not a list of fragments, but a reconstructed epistemic position.

---

## 2. Compressed Sensing Foundation

### 2.1 Signal Model

Let the true cognitive state be **x** ∈ ℝⁿ, where n is the dimensionality of the latent conceptual space (unobservable directly).

**Sparsity assumption**: Human cognition is sparse. Of n possible conceptual dimensions, only k are meaningfully active at any given time (k << n). This is consistent with the principle "high density, not high volume": 100 fragments → 5 high-value cognitions.

Conceptually, each memory fragment can be interpreted as a partial noisy observation of this latent cognitive state. Let φ(f_i) denote the encoded representation of fragment f_i. We model this as:

```
z_i = φ(f_i) ≈ A_i · x + ε_i,    i = 1, ..., m
```

where:
- `φ` is a fragment encoder (e.g., embedding model)
- `A_i` is the fragment's projection onto the latent conceptual space
- `ε_i` is observation noise (imprecision in extraction, temporal degradation)
- `z_i` is the encoded fragment representation

This is a conceptual abstraction, not a claim that memory fragments are literal scalar measurements. The framework borrows the structural intuition of compressed sensing to reason about fragment diversity, anchor necessity, and reconstruction stability.

### 2.2 Recovery Conditions

**Theorem (Candès-Romberg-Tao, 2006)**: If **x** is k-sparse and **A** satisfies the Restricted Isometry Property (RIP) of order 2k, then **x** can be recovered from m measurements with:

```
m ≥ C · k · log(n/k)
```

**Applied to ORM as a theoretical lower-bound intuition**:

```
n = 1000  (conceptual dimensions, estimated)
k = 20    (important dimensions, based on L1 memory count)

m ≥ 20 × log(50) ≈ 78
```

This suggests a theoretical lower-bound intuition: roughly ~78 sufficiently diverse fragments may be required before reconstruction becomes stable. This is not a guarantee — it is a structural intuition derived from the compressed sensing analogy, pending empirical validation.

### 2.3 RIP and Memory Distribution

The RIP condition requires that no small set of fragment encodings is nearly linearly dependent. In memory terms: fragments should not be near-duplicates of each other.

**Implication for write gate**: The write gate filtering rule "do not store what duplicates existing memory" is not just about storage efficiency — it is the engineering approximation of the RIP condition. Highly redundant fragments reduce the effective rank of the observation set, making reconstruction less stable.

---

## 3. The Three Mechanisms

### 3.1 Anchor Nodes

**Definition**: An anchor is a memory fragment `f` with `is_anchor = true`, characterized by:
- `importance = 5` (maximum)
- `layer = L1`
- Decay exemption: not subject to recency scoring

**Formal role**: In the observation model, anchor fragments are treated as low-noise, high-confidence measurements:

```
z_anchor = φ(f_anchor) ≈ A_anchor · x + ε_anchor

where ε_anchor << ε_regular
```

This is distinct from zero-noise — anchors can still be wrong (anchor corruption is a documented failure mode). The claim is that anchors carry significantly less noise than ordinary fragments, making them the most reliable fixed points for reconstruction.

**Why anchors are structurally necessary**: Without high-confidence fixed measurements, the reconstruction problem:

```
min ||x||₁  subject to  ||Az - y||₂ ≤ ε
```

has a feasible set that may contain multiple valid solutions. Each anchor reduces uncertainty in the feasible set, collapsing reconstruction toward a more stable solution.

**Consequence**: Allowing anchors to decay introduces artificial noise into the most reliable observations, expanding reconstruction uncertainty. This is why anchor decay-exemption is a structural requirement, not a preference.

**Anchor auto-promotion rule**:
```
if importance == 5 AND layer == L1:
    is_anchor = True
```

### 3.2 Overlap as Confidence Signal

**Definition**: Two fragments `f_i` and `f_j` overlap if they independently encode information about the same cognitive dimension.

**Formal measure**: Mutual information between fragment encodings:

```
I(f_i; f_j) = H(f_i) + H(f_j) - H(f_i, f_j)
```

High `I(f_i; f_j)` means the two fragments carry independent evidence for the same fact.

**Confidence update**:

```
confidence(dimension d) = 1 - Π_i (1 - p_i(d))
```

where `p_i(d)` is fragment `f_i`'s probability of encoding dimension `d`. Multiple independent witnesses raise collective confidence even if each individual witness is uncertain.

**Optimal overlap rate**: Let ρ = average mutual information between fragment pairs.

- ρ → 0: Fragments are nearly orthogonal → underdetermined system → reconstruction unstable
- ρ → 1: Fragments are near-duplicates → observation set rank-deficient → no additional information gained
- Hypothesis: optimal overlap may fall around 0.3–0.5. Currently under empirical validation.

**Practical implication**: The write gate should not aggressively deduplicate. Two fragments covering similar ground from different angles (different extraction timing, different contextual framing) provide genuine evidential overlap — this is signal, not noise.

### 3.3 Intent-Weighted Reconstruction

**Definition**: The intent vector **w**(intent) assigns weights to cognitive dimensions based on the current query context.

**Reconstruction score**:

```
score(f, query, intent) = importance(f) × sim(f, query) × decay(f) × intent_weight(f, intent)
```

Where:
- `sim(f, query)` = cosine similarity between fragment and query embeddings
- `decay(f)` = 0.5^(age_days / half_life)  [Anchor: decay = 1.0]
- `intent_weight(f, intent)` = dot product between fragment's dimensional profile and intent vector

**Intent vector derivation** (T3):
```
intent = classify(query) → probability distribution over intent_types
intent_types = {technical_decision, emotional_context, project_history,
                relationship, stylistic_preference, factual_lookup}
```

**Reconstruction synthesis**:
```
cognitive_state = synthesis_operator(
    top_k_fragments_by_score,
    anchors (forced inclusion),
    query,
    intent
)
```

The synthesis operator is currently implemented as an LLM prompt — treated as an approximate reconstruction operator, not a formal optimization solver. The output is a synthesized paragraph, not a ranked list.

---

## 4. Memory Lifecycle Operations

### 4.1 Operation Semantics

| Operation | Formal trigger | Effect |
|-----------|---------------|--------|
| add | ∄ fragment encoding same fact | Insert new node, no edge |
| update | ∃ fragment encoding same fact, still true | Update content, create `updated_from` edge |
| supersede | ∃ fragment encoding fact that is now false | Old status → superseded, `superseded_by` pointer, create `supersedes` edge |
| archive | L2 loop closes (decision executed, state resolved) | Old status → archived, new L3 summary inserted, `archived_from` edge |

**update vs supersede decision rule**: "Is the old memory still true now?" Yes → update. No → supersede. This is a semantic judgment, not a similarity threshold.

### 4.2 Write Gate Policy

Five deterministic validation steps (zero LLM calls in gate):

```
1. SCHEMA      field types / enum membership / importance ∈ [1,5] / confidence ∈ [0,1] / reason non-null
2. OPERATION   add: target_id must be null
               update/supersede: target must be active + same user_id
               archive: target must be L2 active + new content must be L3
3. L1          any update/supersede of L1 → held_for_review
               importance=5 → held_for_review
4. CONFIDENCE  <0.70 → held_for_review
               supersede/archive <0.85 → held_for_review
5. COMMIT      → executor
```

States: committed / held_for_review / rejected. All three written to audit_log.

### 4.3 Decay Formula

```
recency_score(f) = 0.5 ^ (age_days(f) / half_life(f))

half_life by layer:
    L1: ∞  (Anchor, no decay)
    L2: 180 days
    L3: 90 days

final_score(f, q, i) = importance(f) × sim(f, q) × recency_score(f) × intent_weight(f, i)
```

Weekly compression trigger:
```
if layer == L2 AND importance ≤ 2 AND days_since_accessed > 30:
    → synthesis operator → new L3 fragment
    → old fragment: status = archived
```

---

## 5. Applicability Boundary

ORM is not the right tool for every memory scenario. It is most valuable when:

- The user has rich, long-horizon, multi-topic memory (m > ~78 fragments)
- Memory contains contradictions, supersede events, and temporal drift
- Reconstruction quality depends on current intent (same query, different context = different answer needed)

ORM is likely overkill or unnecessary when:

- **Low-memory users** (m << threshold): insufficient fragments for stable reconstruction; standard retrieval is adequate
- **Purely factual systems** (FAQ bots, documentation search): RAG is sufficient, no intent-weighting needed
- **Low-contradiction domains**: if memory rarely conflicts or supersedes, the write gate overhead is not justified

---

## 6. Comparison with Existing Frameworks

| CS concept | ORM concept |
|-----------|-------------|
| Sparse signal x | Latent cognitive state (most dimensions inactive) |
| Encoded measurement z_i | φ(f_i), fragment encoding |
| Measurement matrix A | Projection from fragment space to latent space |
| RIP condition | Fragment diversity (write gate deduplication) |
| Anchor rows | is_anchor fragments (low-noise, high-confidence) |
| Approximate reconstruction | LLM synthesis operator |
| Sparsity k | Number of L1 memories (empirically ~20) |
| Recovery bound m | Lower-bound intuition for reconstruction stability (~78) |
| Noise ε | Temporal degradation, extraction imprecision |

### HippoRAG (arXiv:2405.14831)
Uses Personalized PageRank over a knowledge graph to traverse associative links. ORM's overlap mechanism has structural similarity, but ORM adds: (a) confidence aggregation via mutual information, (b) intent weighting, (c) Anchor as a structurally necessary node type.

### MIRIX (arXiv:2507.07957)
Active Retrieval generates a topic before querying six memory types — the closest existing work to T3 intent-conditioning. Key difference: MIRIX returns retrieved fragments; ORM synthesizes a reconstructed cognitive state. MIRIX does not define overlap confidence or anchor structural necessity.

### MemGAS (arXiv:2505.19549)
Multi-granularity storage with entropy-based router. This is adaptive granularity — a different axis than ORM's intent weighting. MemGAS does not address reconstruction or overlap confidence.

### EverMemOS (arXiv:2601.02163)
MemCell self-contained extraction and MemScene compression are the state of the art for extraction format. ORM's T2 adopts the MemCell format. ORM differs in the retrieval/reconstruction layer.

### mem0 (arXiv:2504.19413)
Production-grade hierarchical extraction with multi-signal retrieval. Engineering SOTA. ORM's implementation layer sits on top of mem0 as the reconstruction layer, not a replacement.

---

## 7. Open Questions

These are the empirical questions T2 and T3 are designed to answer:

1. What is the empirical value of k (active dimensions) for a real power user over 6 months?
2. Does measured optimal overlap ρ* fall in the hypothesized 0.3–0.5 range?
3. How much does intent-weighting (T3) improve Context Fidelity over pure similarity ranking (T2)?
4. What is the reconstruction quality degradation curve as fragment count drops below ~78?
5. Does the Anchor mechanism reduce "held_for_review" false positives in the write gate?
6. At what fragment count does ORM begin to outperform standard retrieval (the crossover point)?

---

## 8. Falsification Conditions

ORM predicts failure under the following conditions. These are the tests we are actively running:

1. **overlap < 10%** — fragments near-orthogonal → reconstruction unstable → Recall Accuracy collapse predicted
2. **overlap > 70%** — fragments near-duplicate → observation set rank-deficient → Storage Efficiency degrades with no Accuracy gain
3. **anchor corruption** — wrong fact marked is_anchor → reconstruction globally biased → Pollution Rate spike predicted
4. **high contradiction rate** — dense supersede events → reconstruction confidence oscillation → Context Fidelity degradation predicted

---

## Citation

If you use this framework or dataset, please cite:

```
@misc{orm2026,
  title={Overlap-Reconstruction Memory: Intent-Driven Cognitive State Reconstruction for AI Agents},
  author={Malin},
  year={2026},
  url={https://github.com/m18759272588-boop/overlap-reconstruction-memory}
}
```

arXiv preprint forthcoming.
