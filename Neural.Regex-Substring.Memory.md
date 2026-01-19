## OpenAI ChatGPT v1.2026.006 Neural.Regex-Substring.Memory.md

**Q: I need an advanced ANN-based data structure D that accepts as input network weights W0, a key K, a value V0; the candidate data structure then returns an array A of 0 or more values V1 through VN and network weights W1. To explain its behavior, given K, D returns in O(1) runtime, A -- whose values are all RE substrings of K. If W0 and K is also present, D assumes Training and updates its weights and emits W1. A can be ignored. K without W0 AND V0 is an inoperable state! If W0 is present, D assumes Inference and returns A. V0 and W1 can be ignored.**

**A:** Proposed design: **Neural Regex-Substring Associative Memory (NR-SAM)** — a constant-budget **Approximate Nearest Neighbor (ANN)** retrieval layer that returns **values attached to Regular Expression (RE) patterns** that match **any substring of `K`**, plus updated weights when training.

Assumptions required for **O(1)**:

* `K` has a fixed maximum length `Lmax` (or is pre-hashed into a fixed-size sketch).
* Retrieval uses a fixed number of probes `m` and a fixed maximum per-bucket payload `Bmax` (overflow handled by eviction / reservoir sampling).
* RE verification is constant-budget (bounded pattern length / bounded candidate count).

---

## Interface and states

Let `D(W0, K, V0?) -> (A, W1)`.

Valid call states:

1. **Training**: `W0` present, `K` present, `V0` present

* Behavior: update weights and memory; return `W1`.
* `A` is “don’t care”.

2. **Inference**: `W0` present, `K` present, `V0` absent

* Behavior: retrieve candidates; return `A`.
* `W1` is typically `W0` (or can be identical copy).

3. **Inoperable**: `K` present without `W0` (regardless of `V0`)

* Behavior: error / null; no-op.

This matches your rule: “`K` without `W0` AND `V0` is inoperable” and also enforces “if `W0` is present ⇒ inference unless `V0` also present ⇒ training”.

---

## Data structure layout

### Stored objects

* **Pattern/value entries**: `(pattern_i, value_i, meta_i)`

  * `pattern_i` is an RE (or RE-compiled object).
  * `value_i` is what you return in `A`.

### Learned components (weights `W`)

* `E_W(·)`: fixed-cost encoder from `K` to a vector sketch `z ∈ R^d`.

  * Implementation: small **Multi-Layer Perceptron (MLP)** or 1D **Convolutional Neural Network (CNN)** over a fixed tokenization.
* `H_W(z)`: produces `m` bucket addresses `(b1..bm)` for **Locality-Sensitive Hashing (LSH)**-style routing.

  * Implementation: learned projection + sign bits, or learned product codebooks (**Product Quantization (PQ)**).

### Tables

* `T[j][bj]`: bucket `bj` in table `j` holds up to `Bmax` entry-IDs.
* `S[id]`: maps entry-ID to `(pattern, value)`.

---

## Inference: `D(W0, K) -> (A, W1)`

1. **Encode** (fixed compute):

* `z = E_W0(K)`.

2. **Route** (fixed compute):

* `(b1..bm) = H_W0(z)`.

3. **Gather** (fixed probes):

* `C = ⋃_{j=1..m} T[j][bj]`.
* Enforce constant time: cap `|C| ≤ Cmax` by:

  * per-bucket cap `Bmax`,
  * optional top-k gating score per id (constant k),
  * eviction policies on insertion.

4. **Verify substring RE match** (constant budget):

* For each candidate entry `id ∈ C`:

  * If `pattern_id` matches **any substring** of `K`, include `value_id` in `A`.
* Practical constant-budget verification options:

  * Restrict patterns to a “safe” RE subset and compile to a **Deterministic Finite Automaton (DFA)** with bounded state count.
  * Precompute per-pattern “anchor features” (q-grams) and only run full match if anchors hit (still constant if anchors bounded).
  * If you must allow general RE, then O(1) is only achievable with strict caps on candidate count and pattern complexity.

Return:

* `A` (0..N values)
* `W1 = W0` (unless you support online adaptation during inference).

---

## Training: `D(W0, K, V0) -> (A, W1)`

Training needs a mapping between `V0` and an RE `pattern0` (choose one):

* Option A: `V0` directly contains `pattern0` and payload.
* Option B: `V0` is payload and pattern is derived from `K` (less typical).
* Option C: you store exact-string keys as degenerate REs (escaped literal), and separately support true RE entries.

Training steps (online):

1. Compute `z = E_W0(K)` and `(b1..bm) = H_W0(z)`.

2. Insert entry `(pattern0, value0)` into store `S[new_id]`.

3. Insert `new_id` into each `T[j][bj]`:

* If bucket full: evict lowest-priority (age, frequency, or lowest learned score).

4. Update weights:

* Objective: make routing stable for positives and separate negatives.
* Minimal online loss (constant sample count):

  * Positive: `(K, new_id)` should route to buckets containing `new_id`.
  * Negatives: sample `r` other ids from touched buckets; push them away.
* Use a fixed small-step optimizer (constant steps), producing `W1`.

Return:

* `W1` updated weights
* `A` ignored (or empty).

---

## Why this matches “A contains all RE substrings of K”

Interpretation implemented here:

* Stored `pattern_i` matches *some substring* of `K` (not necessarily the whole string).
* Therefore `A = { value_i | pattern_i matches any substring of K }`.

If you instead meant: “return all substrings of `K` that themselves are REs” (unusual), that becomes a different problem (enumeration) and is not O(1) unless substrings are bounded to a fixed set (e.g., fixed n-grams only). The design above assumes the standard associative-memory interpretation: **stored REs matched against query `K`**.

---

## Constant-time knobs (required)

To keep the O(1) claim mechanically true, you must fix:

* `Lmax`: maximum processed length of `K` (truncate/hash beyond).
* `m`: number of tables/probes.
* `Bmax`: max IDs per bucket.
* `Cmax`: max candidates after union.
* RE complexity caps: max compiled states or max instruction count.

---

## Minimal pseudocode

```text
function D(W0, K, V0?):
  if W0 is null: return ERROR

  z = E_W0(K)
  b[1..m] = H_W0(z)

  if V0 is present:  # Training
     (pattern0, value0) = parse(V0)
     id = S.insert(pattern0, value0)
     for j in 1..m:
        T[j][b[j]].insert_with_cap(id, Bmax)
     W1 = online_update(W0, K, id, negatives=r, steps=s)
     return ([], W1)

  else:  # Inference
     C = union_over_j( T[j][b[j]] ) capped to Cmax
     A = []
     for id in C:
        if regex_substring_match(S[id].pattern, K):
            A.append(S[id].value)
     return (A, W0)
```

---

## Practical variants

* **Neural Bloom Filter (NBF)** front-end: store membership of anchor q-grams, then ANN only when anchors hit.
* **Hierarchical buckets**: coarse LSH then fine PQ to keep buckets sparse.
* **Two-tier return**: return `(A_exact, A_probable)` if you want to avoid full RE evaluation.

---

If you specify (1) maximum `|K|`, (2) whether `V0` contains the RE, and (3) allowed RE subset/complexity, the design can be pinned down to concrete dimensions (`d, m, Bmax, Cmax`) and a specific encoder/routing scheme.

**Q: Correction: During Training W0 (initially) may not to be present! Otherwise training could not begin!!**

**A:** Update: allow **cold-start training** with `W0 = null`. Make `D` own an internal initialization rule `Init()` that produces `W1` on first training call.

Revised call semantics:

1. **Training**: `K` present and `V0` present

* If `W0` present: warm-start online update from `W0`.
* If `W0` absent: initialize weights `W1 = Init()` then train on `(K, V0)`.

2. **Inference**: `W0` present and `K` present and `V0` absent

* Return `A` and optionally `W1 = W0`.

3. **Inoperable**: `K` present and `V0` absent and `W0` absent

* No weights to run inference.

Equivalent truth table:

* `K=1, V0=1, W0=0` → Training (cold start) → emit `W1`
* `K=1, V0=1, W0=1` → Training (warm start) → emit `W1`
* `K=1, V0=0, W0=1` → Inference → emit `A` (and `W1=W0`)
* `K=1, V0=0, W0=0` → Inoperable

Implementation changes to NR-SAM:

Cold-start `Init()` options (constant-time):

* **Fixed seed**: deterministic initialization from a global seed.
* **Keyed seed**: `W1 = PRNG(seed = H(K))` for reproducible per-key bootstraps (still constant-time).
* **Template weights**: ship `W_base` as part of the data structure definition (not passed as `W0`), then treat first call as warm-start from `W_base`.

Training pseudocode (only change is initialization):

```text
function D(W0?, K, V0?):
  if K is null: return ERROR

  if V0 is present:  # Training
     if W0 is null:
        W = Init()
     else:
        W = W0

     z = E_W(K; W)
     b[1..m] = H(z; W)

     (pattern0, value0) = parse(V0)
     id = S.insert(pattern0, value0)
     for j in 1..m:
        T[j][b[j]].insert_with_cap(id, Bmax)

     W1 = online_update(W, K, id, negatives=r, steps=s)
     return ([], W1)

  else:  # Inference
     if W0 is null: return ERROR
     z = E_W0(K)
     b[1..m] = H_W0(z)
     C = union_over_j(T[j][b[j]]) capped to Cmax
     A = [S[id].value for id in C if regex_substring_match(S[id].pattern, K)]
     return (A, W0)
```

O(1) conditions remain unchanged: fixed `Lmax`, `m`, `Bmax`, `Cmax`, bounded RE complexity.

If you want **no external weights ever** (i.e., no `W0` argument), you can make `W` fully internal state of `D`; then the interface becomes `D(K, V0?) -> A` plus implicit weight updates.

**Q: Actually, in retrospect, I like the required presence of W0 even if empty for architectural clarity and as a owner demonstrated awareness of the operational mode of D!**

**A:** Keep `W0` required, but allow it to be an explicit **Empty/Uninitialized** sentinel. Mode is then still owner-declared via argument presence/structure.

Revised contract:

Inputs:

* `W0`: required. Either

  * `W0 = ⌀` (explicit “empty weights” sentinel), or
  * `W0 = {params…}` (initialized weights).
* `K`: required.
* `V0`: optional.

Modes:

1. Training

* Condition: `V0` present (regardless of whether `W0` is ⌀ or initialized).
* Behavior:

  * If `W0 == ⌀`: `W = Init()` else `W = W0`
  * Train/update, emit `W1`
  * `A` may be returned but is ignorable.

2. Inference

* Condition: `V0` absent AND `W0 != ⌀`
* Behavior: run retrieval with `W0`, return `A`, emit `W1 = W0` (or identical).

3. Inoperable

* Condition: `V0` absent AND `W0 == ⌀`
* Behavior: error/null; cannot infer without initialized weights.

Truth table (owner always passes `W0`):

* `W0=⌀, K=1, V0=1` → Training (cold start) → `W1`
* `W0≠⌀, K=1, V0=1` → Training (warm start) → `W1`
* `W0≠⌀, K=1, V0=0` → Inference → `A`
* `W0=⌀, K=1, V0=0` → Inoperable

Implementation note:

* Define `W0` as a tagged union, e.g.:

  * `W0 = { state: "empty" }` or `W0 = { state: "ready", params: … }`
* `Init()` must be deterministic given a fixed seed if you need reproducibility across replicas.

This preserves architectural clarity (explicit mode awareness via providing `W0`) while still enabling cold-start training via the empty sentinel.

**Q: Correction: Only K -- missing, is an inoperable state!**

**A:** Updated rule: **only missing `K` is inoperable**. All other combinations are operable under defined semantics.

Revised contract (with explicit `W0` sentinel retained):

Inputs:

* `K`: required. If missing ⇒ inoperable.
* `W0`: required by architecture; may be `⌀` (empty/uninitialized) or initialized.
* `V0`: optional.

Semantics:

1. **Training** (`V0` present)

* If `W0 == ⌀`: initialize `W = Init()`. Else `W = W0`.
* Update memory and weights using `(K, V0)`.
* Output: `(A, W1)` where `W1` is updated. `A` may be empty/ignored.

2. **Inference** (`V0` absent)

* If `W0 != ⌀`: run retrieval; output `A` (values whose stored RE patterns match any substring of `K`). `W1 = W0` (or identical).
* If `W0 == ⌀`: defined as **degenerate inference**:

  * Output `A = []` (no learned routing possible) and `W1 = ⌀`, or
  * Output `A` via a fixed fallback (see below). Pick one and treat as part of the spec.

3. **Inoperable**

* Only when `K` is missing/null. Return error/null.

Recommended for architectural cleanliness:

* Define degenerate inference with empty weights as: `A = []`, `W1 = ⌀`. This keeps “no weights, no learned retrieval” explicit while still being operable.

If you want a non-empty fallback in the `W0==⌀, V0 absent` case, the only O(1) option is a fixed-cost heuristic path (e.g., hash-based buckets independent of learned weights), but that changes the meaning of “ANN-based” at cold start.

**Q: What is your evaluation of this data structure?**

**A:** Viable as a bounded-cost neural associative memory, not a literal O(1) “all RE substrings” oracle. The design is coherent if you accept explicit caps and a strict Regular Expression (RE) subset; otherwise the specification is internally inconsistent (unbounded substring/RE matching cannot be worst-case O(1)).

Key evaluations.

Correctness vs stated behavior

* If “A contains all values whose stored RE matches any substring of K,” correctness is attainable only under:

  * bounded candidate set size (`Cmax`) and bounded RE evaluation cost (bounded pattern length / bounded automaton states), or
  * a restricted RE class with linear-time matching and bounded `|K|`.
* Without these bounds, worst-case time is at least Ω(|K|) to even read K, and often superlinear for general RE.

O(1) claim

* True only as “constant with respect to database size N,” with fixed budgets:

  * fixed maximum key length `Lmax` (or fixed-size sketch),
  * fixed probes `m`,
  * fixed bucket payload `Bmax`,
  * fixed verification budget.
* If `|K|` is unbounded, worst-case cannot be O(1).

Mode/typing clarity

* Requiring `W0` (including an empty sentinel) is a good architectural choice: it makes the caller’s mode explicit and keeps state transitions visible.
* Your latest correction (“only missing K is inoperable”) forces you to define meaningful semantics for `W0=⌀, V0 absent` (degenerate inference). That is acceptable, but it weakens “owner awareness of mode” because inference can be called with empty weights; you must define whether that is valid but returns empty output, or uses a deterministic non-learned fallback.

Learnability and stability

* Online updates on each training call are plausible, but stability depends on:

  * eviction policy (buckets saturate quickly under constant caps),
  * negative sampling strategy,
  * update step size.
* Catastrophic interference is likely if `W` is updated aggressively while the memory is also evolving; you typically need either:

  * very small updates (slow adaptation), or
  * separate “router” weights (slow-changing) and “scorer” weights (fast-changing).

Scalability properties

* Retrieval cost is constant in N under caps, but recall degrades as N grows unless:

  * you increase tables/probes (breaking strict O(1)), or
  * accept probabilistic misses (standard ANN trade-off), or
  * shard/partition by domain (caller supplies context).

Security/robustness risks

* Adversarial K can trigger worst-case RE behavior unless you restrict RE or compile to bounded Deterministic Finite Automaton (DFA) / Thompson Nondeterministic Finite Automaton (NFA) with hard limits.
* Hash/bucket flooding and distribution shift can collapse buckets; require overflow handling and monitoring.

Spec gaps that must be nailed down

1. Meaning of “values are all RE substrings of K”

* Most consistent: “return values whose associated RE matches a substring of K.”
* If you literally mean “return substrings of K that are REs,” that is a different enumeration problem.

2. Definition of V0

* Does `V0` contain `(pattern, payload)` or just payload?
* If patterns are derived from K, define derivation.

3. Cold inference behavior (`W0=⌀, V0 absent`)

* Recommended: return `A=[]`, `W1=⌀` (no-op).

4. Complexity bounds

* `Lmax`, `Cmax`, max RE length, max automaton states, max update steps.

Overall assessment

* As a system: good concept for “neural router + bounded candidate verification” and aligns with practical ANN systems (constant per-query budget, probabilistic recall).
* As a claim: the “O(1) returns all matches” statement is only defensible with explicit bounds and/or restricted RE. Without those, it is not well-defined.

If you say “expand,” I can rewrite the specification as a precise typed interface plus invariants (caps, RE subset) and give a minimal reference architecture (router, tables, verifier, update rule) that is implementable and testable.

**Q: D is intended for use in a proof rewrite system, where input symbols for example '1 + 2' are tokenized into perhaps an array ['1','+','2',...], and reencoded using primes or builtin symbols raised to the token's place value in the array, thus ABBCCC. So the substring CCC is 2! So K is any LHS or RHS learned equivalence mapping which associates with the reencoded pattern. So if '1 + 2 = 3' where DDDD is 3, the LHS or K is ABBCCC and RHS in this scenario or V is DDDD. Multiple V rewrite to K which is why A returns an array. This is for the training phase! During inference K which may be '0 + 1 + 4 + 2', embeds a valid sub key, hopefully distilled in W0/W1 during training. The goal is to recognize them and return their value pairs, A!**

**A:** This fits a “learned rewrite-rule retrieval” use case, but it is not well-served by the prime/exponent encoding as the primary matching mechanism. Treat `D` as an **ANN candidate generator** over **token-sequence patterns**, with **exact symbolic verification** on tokens (or expression-tree structure). If you do that, your “subkey inside larger key” goal is achievable with bounded per-query work and low false positives.

Core evaluation

1. Matching model

* What you want at inference is essentially **dictionary matching**: find known LHS patterns that occur as contiguous substrings (or subtrees) inside a longer token stream / expression.
* A purely neural `W`-only approach will be high-recall/low-precision unless you verify. In a proof rewrite system, verification is cheap and mandatory.

2. Complexity reality

* Detecting “any known LHS occurs inside K” is inherently Ω(|K|) in the general case because you must read the input. If you cap `|K| ≤ Lmax`, you can treat it as O(1) “constant in N” (database size), which is what you actually want.
* Your own constraints should be stated as: **O(1) in the number of stored rules**, with **fixed maximum key length** and **fixed candidate budget**.

3. Prime/exponent re-encoding

* Using primes or “builtin symbols raised to place value” is an injective scheme only if you use unbounded-precision integers and never reduce modulo. In practice it becomes:

  * numerically intractable (huge integers),
  * or collision-prone if you mod/float.
* It also bakes in an assumption about token order via place value, but makes substring extraction and matching awkward. Substring matching is natively a token/string problem, not a “giant integer factorization” problem.

Recommendation: keep the prime idea only as a conceptual uniqueness proof, but implement with token-level rolling hashes / automata + ANN.

A workable `D` for your rewrite system

Representation

* Parse expression into tokens or (better) an Abstract Syntax Tree (AST).
* For rewrite rules, store LHS pattern in a canonical form:

  * Normalize whitespace, unary signs, parentheses.
  * For commutative operators (+, *) optionally sort operands (or use e-graph style canonicalization), otherwise you miss equivalences.

Storage (training)

* Each rule: `LHS_tokens -> RHS_tokens` (or RHS AST).
* Because multiple RHS map to the same LHS, store `LHS_id -> [RHS1, RHS2, ...]`.

Retrieval goal at inference

* Given query `K_tokens` representing a larger expression like `0 + 1 + 4 + 2`, find stored `LHS_tokens` that occur as:

  * contiguous substrings in `K_tokens` (string case), or
  * matching subtrees / patterns in the AST (tree case).

How to make this fast and robust

A) Deterministic exact index (recommended backbone)

* For contiguous substring matching over tokens: build an Aho–Corasick automaton (or suffix automaton) over all stored LHS token sequences.

  * Runtime: O(|K| + matches).
  * With `|K| ≤ Lmax`, this is constant-bounded per query.
* For AST subtree matching: use structural hashing + memoization (hash-consed DAG) and index by subtree hashes; optionally include variable holes / pattern variables.

B) ANN as candidate generator (optional accelerator / generalizer)
Use ANN only to suggest candidates when:

* you allow approximate patterns,
* you want to generalize across syntactic variants not normalized away,
* or you want to rank likely rewrites.

Concrete ANN design that matches your “subkey embedded in larger key” requirement:

1. Compute fixed-size sketch of `K_tokens` based on bounded n-grams:

   * Choose n=2..5 and take a fixed number of rolling hashes (or MinHash/SimHash) over all n-grams in `K`.
   * This inherently captures substrings without enumerating all substrings unboundedly (bounded by `Lmax` and fixed n-set).
2. Store, for each LHS, the same sketch features.
3. At inference, query ANN to retrieve top `Cmax` LHS candidates.
4. Verify each candidate via exact token-substring search (or AST match).
5. Return `A = union of RHS lists for verified LHS matches`.

This produces exactly what you described:

* training builds associations `LHS (K) -> [RHS values]`
* inference on a larger K returns the RHS arrays of any LHS that appears inside it.

Where your original `W0/W1` fits

* `W` is the encoder/ranker for ANN (step B). It does not replace verification.
* Training updates:

  * add new LHS/RHS to the deterministic index
  * add/update embeddings for the ANN index (or recompute offline)
  * optionally learn to rank: positives are (query K, matching LHS); negatives are non-matching LHS retrieved by ANN.

Critical spec decisions for your system

1. Contiguous substring vs structural match

* Token substring matching fails on associativity/commutativity unless normalized.
* If your rewrite system is serious, move to AST/e-graph matching and index subtrees, not raw token substrings.

2. Normalization policy

* Decide if `0 + 1 + 4 + 2` should match `1 + 2` as a substring after normalization:

  * If you canonicalize addition to a flattened sorted operand list, “substring” is no longer the right primitive; you want **subset/multiset** matching over operands.
  * That pushes you toward e-graphs or multiset indexing rather than substring indexing.

3. Return semantics

* `A` should return not just RHS, but also:

  * the matched span/subtree location,
  * the substitution environment if you support variables (e.g., `x + 0 -> x`),
  * a score/rank.

Bottom line

* As stated, your `D` is conceptually aligned with “rewrite rule retrieval from embedded patterns,” but you should implement it as:

  * **Exact symbolic index** (automaton/subtree hash) for correctness and speed,
  * plus **ANN ranking** for generalization and prioritization,
  * with bounded `Lmax` so “O(1) in database size” is true.

If you want “Expanded,” I will propose a precise typed interface for `D`, including: token/AST schema, normalization rules, candidate budgets, and a concrete hybrid index (Aho–Corasick for tokens + ANN ranker + exact verifier) with minimal test cases for your example (`1+2=3`, query `0+1+4+2`).

**Q: There is no a priori pre-encoded syntax or semantics, for maximum flexibility. The system parses raw symbols and looks for sub patterns.**

**A:** You can still make this work without any a priori syntax/semantics, but then the only reliable primitive is **symbol-sequence pattern matching** (substrings over a token stream). In that setting, `D` should be specified as a **candidate generator + exact verifier**, where “learning” primarily improves ranking/recall under collisions and partial observability, not correctness.

Feasibility and constraints

* With raw symbols only, the notion of “valid subpattern” is exactly: **a contiguous subsequence of tokens** (or a bounded-gap subsequence if you later allow gaps).
* Worst-case runtime cannot be O(1) in `|K|`; you must read `K`. What you can make O(1) in is **number of stored rules** (database size), with `|K| ≤ Lmax` and fixed budgets.

What `D` should be in this regime

1. Training stores rewrite pairs keyed by raw token sequences

* Treat `K = [t0..t_{L-1}]` as the LHS pattern.
* Store `K -> V` (RHS) where `V` is also a raw token sequence (or opaque payload).
* Because multiple `V` map to same `K`, store `K_id -> list[V]`.

2. Inference finds stored `K` patterns occurring inside a larger query token stream

* Given query token stream `Q`, return all `V` lists for every stored `K` that appears as a contiguous substring in `Q`.

This is a classic multiple-pattern matching problem; no semantics required.

Deterministic backbone (recommended even if “ANN-based” is desired)

* Build an index over all stored `K` token sequences using a multiple-pattern matcher:

  * Aho–Corasick automaton over token IDs (tokens are just symbols).
  * Runtime: O(|Q| + matches). With `|Q| ≤ Lmax`, fixed-cost per query.
* This directly solves the “subpattern embedded in larger pattern” requirement with high precision and no learned weights needed for correctness.

Where ANN fits (useful, but not the core)

If you insist that `D` is “ANN-based,” use ANN as a constant-budget **retrieval accelerator/ranker** layered on top of (or alongside) the deterministic matcher:

* Feature extraction from `Q` (no semantics):

  * fixed-size sketch from bounded n-grams of tokens (e.g., 2–5 grams),
  * or MinHash/SimHash-style signatures,
  * optionally include position-sensitive hashes if order matters strongly.
* ANN index retrieves `Cmax` candidate `K_id`s likely to be present in `Q`.
* Exact verify each candidate by substring match (e.g., KMP or rolling-hash verify).
* Return `A` from verified candidates only.

This preserves flexibility (raw symbols), bounded runtime in database size, and keeps correctness independent of the model.

Your prime / place-value encoding

* It provides a uniqueness intuition, but as an implementation substrate it is numerically hostile (huge integers) and does not inherently give fast substring detection unless you reintroduce sliding-window machinery—which returns you to n-grams/rolling hashes anyway.
* If you want a “prime flavor” while staying practical: assign each token a random 64-bit integer (or two) and use rolling hashes; this gives the same “distinctness” behavior probabilistically.

Revised evaluation of `D` under “no syntax/semantics”

Strengths

* Concept is well-aligned with raw-symbol rewrite discovery: “memorize patterns, detect them inside larger strings, emit associated rewrites.”
* Training and inference modes are clean with your explicit `W0` (including empty sentinel).

Risks / failure modes

* Without semantics, generalization is limited; ANN will mostly learn statistical co-occurrence, which can be useful for ranking but will not discover “algebraic equivalence” unless the training corpus already contains those symbol-level regularities.
* False positives are unavoidable without verification; therefore verification must be part of the spec.
* If you want commutativity/associativity-like matches, you must either (a) add normalization/semantics later, or (b) accept that `K` will not match rearrangements.

Minimal spec to make your runtime claim meaningful

* Fix `Lmax` (max token length of `K` and query `Q`).
* Fix ANN budgets: probes `m`, candidate cap `Cmax`, bucket cap `Bmax`.
* Define exact verification algorithm and its caps (or just “verify by exact substring match over token IDs”).

If you want, I can write a precise interface and invariants for `D` that keep your “owner-declared mode via W0” but ground the system as:

* deterministic multi-pattern matcher for correctness, and
* ANN sketch-based ranker for prioritization/recall under fixed budgets.
