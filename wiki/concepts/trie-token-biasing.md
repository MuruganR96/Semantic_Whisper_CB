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

## Open improvements (ranked)

1. First-token asymmetry: reduced/zero boost on word-initial tokens, full δ only after ≥1 matched
   token ("reward continuation, not initiation").
2. Failure-arc bonus revocation on path abandonment.
3. Per-token δ scheduling by trie depth (deeper match → higher confidence → larger boost).
