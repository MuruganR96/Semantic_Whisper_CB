---
type: concept
tags: [trie, bpe, shallow-fusion]
updated: 2026-07-27
---

# Trie-Based Token Biasing

## Why a standard trie (not a radix tree)

One BPE token per edge matches beam search's one-token-per-step rhythm exactly: partial-match state
is "which trie node is this beam at," boosts apply at every step, and failure/fallback logic assumes
uniform single-token transitions. A radix tree's collapsed multi-token edges break all three. (Claim
from [[src-asr-biasing-guide]], consistent with the deep-biasing literature; our implementation
confirmed the mechanics.)

## Implementation facts (hard-won)

- **Surface forms:** [[whisper]]'s BPE is context-sensitive — `word`, `Word`, `" word"`, `" Word"`
  tokenize differently. Every biasing word is inserted under all 4 forms sharing one hotword id.
  Miss this and mid-sentence occurrences silently never match.
- **Stateless suffix matching:** rather than carrying per-beam trie state (fragile under beam
  reordering), each step re-walks every suffix of the generated tokens up to `max_depth`. The empty
  suffix maps to the root — that is what lets a new biasing word *start*; leading-space surface forms
  confine it to word boundaries.
- **Boost:** `δ` per continuing token (+ optional [[semantic-gating|semantic term]]); overlapping
  suffix matches take the max, never stack.

## Failure modes observed in [[exp-03-error-driven-eval]]

- **Orphaned fragments:** boost pulls the decoder into a hotword path; acoustics disagree mid-word;
  the beam keeps the earned bonus and emits a mangled hybrid ("HUSBANDMEN" → "Wea Husspundman").
  Root cause: **no failure-arc revocation** — the classical fix is Aho-Corasick failure pointers
  with score rollback on path abandonment.
- **Word-start distortion field:** the root's children (first tokens of *all* list words) get +δ at
  every word boundary of every beam. At δ=3 with ~120 words this measurably corrupts unbiased words
  (U-WER 0.128 → 0.700).

## Improvements — status

1. **First-token asymmetry — implemented (V2):** word-initial tokens earn η·δ (default η=0.25), full
   δ only for continuations. `TrieSonarBiasProcessorV2`, pipeline notebook §7b.
2. **Failure-arc bonus revocation — implemented (V2):** potential-based shaping — per-candidate
   marginal Φ(hyp+token) − Φ(hyp), where Φ = locked completed-keyword bonuses + current partial
   credit. Abandoning a partial (incl. EOS) subtracts exactly the earned credit; any finished
   hypothesis's net shaping = completed keywords only. Verified by unit simulation (start η·δ /
   continue δ / complete δ+λcos / abandon-net-zero). Design changes vs V1: semantic term λ·cos now
   granted **once at completion** (unrevocable spend removed from mid-word tokens); greedy
   longest-suffix automaton with completion-reset (overlapping-keyword continuations dropped —
   documented simplification); revocation applied as a whole-row broadcast, so EOS-mid-word is
   covered for free. **Not yet evaluated at scale** — needs the error-driven eval rerun with V2
   under the spec's U-WER-constrained sweep.
3. Open: per-token δ scheduling by depth; small revocable per-token semantic advance (η_sem·λ·cos)
   if sweeps show rare targets dying mid-word under completion-only semantics.
