---
type: concept
tags: [trie, bpe, shallow-fusion, sonar, walkthrough]
updated: 2026-07-27
---

# How BPE-Level Trie Fusion with SONAR Gating Works (Walkthrough)

Companion to [[trie-token-biasing]] and [[semantic-gating]]: the end-to-end mechanism in one page.
Filed from a query answer (2026-07-27).

## Division of labor

- **BPE trie — WHERE to boost:** exact prefix matching over token IDs, one token per edge, one edge
  per beam-search step.
- **SONAR + W — HOW MUCH:** one word/phrase-level embedding per keyword; per-step cosine between the
  beam's projected decoder state ([[whisper-sonar-linear-map]]) and every keyword.
- **Joined by `hids`:** every trie node records which keyword IDs pass through it; a node's boost
  looks up those keywords' similarities. SONAR is **never** applied at BPE granularity — subword
  fragments are out-of-distribution for a sentence encoder and semantically meaningless.

## Offline (once per list)

1. Each keyword → 4 surface forms (±capitalization × ±leading space; [[whisper]] BPE is
   context-sensitive) → token paths inserted into the trie, all sharing one hotword ID; `hids`
   recorded on every node along each path. Phrases = longer paths.
2. Each keyword → one SONAR embedding (bare or H1a-templated), L2-normalized, stacked into
   `E [H×1024]`. Cost: seconds, once — the answer to the embedding-cost concern in
   [[src-lead-guidance]].

## Runtime, per decoding step

1. **Semantic pass (vectorized over beams):** hook on decoder layer 4 captures full-sequence states
   (`use_cache=False`, spec §6) → mean-pool text tokens → normalize → `@ W` → normalize →
   `sims = ŝ @ Eᵀ` per beam.
2. **Trie pass (per beam):** try every suffix of the generated tokens up to `max_depth−1` **plus the
   empty suffix** (= root = a new keyword may start; leading-space forms confine this to word
   boundaries). Each matched node's children are boostable next tokens with
   `bonus = δ + λ·max(0, max(sims[child.hids]))`; overlapping matches take the max, never stack.
3. `scores[beam, token] += bonus`; beam search proceeds. A correct keyword accumulates
   ≈ `L·(δ + λ·cos)` advantage over its L tokens; a distractor gets ≈ `L·δ` — the gap **is** the
   gate's value, and exists only if the embedding side has signal (H1a diagnostic).

## Group-up for free

Interior nodes aggregate `max(sims)` over their keyword subtree ("could anything through this prefix
be relevant?"), sharpening to the exact keyword at terminal nodes — the hierarchical group-up idea
from [[src-lead-guidance]], realized at scoring time without embedding fragments.

## Costs and failure modes

Per-step compute is two small matmuls + O(beams·depth²) lookups — negligible. The measured risks
([[exp-03-error-driven-eval]]): word-start distortion field (root children of all keywords boosted at
every word boundary) and orphaned fragments (no failure-arc revocation). Fixes queued in
[[trie-token-biasing]]. Reference implementation: `whisper_trie_sonar_biasing_pipeline.ipynb` §5–7.
