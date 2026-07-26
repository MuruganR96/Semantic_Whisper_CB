---
type: concept
tags: [evaluation, wer, metrics]
updated: 2026-07-27
---

# Evaluation Metrics and Protocol

The metric suite exists because [[exp-03-error-driven-eval]] proved recall alone is misleading:
recall 0 → 0.52 looked like victory while U-WER 0.128 → 0.700 was the real story.

## Shared normalization (frozen, spec §6)

One function everywhere — mining, WER, recall: lowercase → keep only `[a-z' ]` → collapse
whitespace. [[librispeech]] refs are uppercase/unpunctuated; [[whisper]] outputs cased+punctuated;
without a shared normalizer every number is wrong.

## Definitions

- **Keyword recall (primary):** occurrence-level; an occurrence counts iff it lies inside an `equal`
  chunk of the jiwer word alignment. **Never substring matching** ("king" is not recalled by
  "kingsman"). Multi-word keywords: all words aligned contiguously.
- **B-WER / U-WER:** substitutions+deletions attributed by the *reference* word (biased-list →
  B-WER, else U-WER); insertions attributed by the *hypothesis* word. U-WER is the collateral-damage
  meter — the constraint metric for operating-point selection (U-WER ≤ 1.05× baseline, spec §7).
- **False alarms:** list words inserted, or substituted in where the reference has no such word.
  Requires a **distractor-loaded list** (≈50% words guaranteed absent from the audio) to be
  meaningful — distractors are what give the [[semantic-gating|gate]] something to prove.
- **Oracle gate separation:** mean cos(keyword, own-utterance ref embedding) − mean cos(distractor,
  utterance embeddings). Go/no-go ≥ 0.05.

## Protocol rules

- **Shard rule (frozen):** `sorted(ids)` → `random.Random(42).shuffle` → first 25% TUNE, rest
  DEVTEST — computed on *unfiltered* ID lists so text-only notebooks reproduce identical shards
  (SHA-256 printed and compared across notebooks).
- **Headroom check first:** report baseline recall and `target_occurrences` before interpreting
  anything — [[exp-02-clean-split-ceiling]] is the cautionary tale (baseline recall 0.983 ⇒ nothing
  to measure).
- **Error-driven target mining:** targets = rare reference words the baseline provably
  deleted/substituted in a prior baseline-only pass; guarantees headroom by construction.
- **Statistics:** differences involving < ~20 occurrences are noise; keep denominators visible.
- Tuning on TUNE only; DEVTEST for development reporting; test splits untouched (test-other is
  quarantined — [[librispeech]]).
