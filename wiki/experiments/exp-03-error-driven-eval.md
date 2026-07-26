---
type: experiment
tags: [exploratory, evaluation, headroom]
updated: 2026-07-27
notebook: whisper_biasing_error_driven_eval.ipynb
status: exploratory — motivation only, not reportable (spec §9); used test-other (now quarantined)
---

# Exp-03: Error-Driven Evaluation — One Win, One Failure, One Mechanism Exposed

**Question.** With certified-failure targets (mined from baseline errors on LibriSpeech test.other),
does trie biasing recover them, and does the [[semantic-gating|SONAR gate]] beat trie-only on the
recall/false-alarm trade-off?

## Design

Phase-1 baseline pass over ~250 utterances → targets = rare words the baseline deleted/substituted
(baseline recall ≈ 0 *by construction*); distractor-loaded list; oracle gate-discriminability
diagnostic run *before* any biased decoding; A (cached baseline) / B (trie δ) / C (trie+gate δ, λ).

## Results (eval: 60 utts, 56 target occurrences; δ*=3, λ*=2)

| Condition | WER | B-WER | U-WER | Recall | False alarms |
|---|---|---|---|---|---|
| A baseline | 0.167 | 1.000 | 0.128 | 0.000 | 2 |
| B trie δ=3 | 0.690 | 0.482 | 0.700 | 0.518 | 5 |
| C +SONAR λ=2 | 0.806 | 0.464 | 0.822 | 0.536 | 6 |

Oracle gate separation: **0.028** (bare words) — flagged "weak" before decoding, and C ≈ B followed
exactly as predicted. Dev sweep (15 utts) had said δ=3 *improved* WER — eval exposed the tail risk.

## Verdict and lessons

1. **Win:** trie shallow fusion recovers half of certified-impossible words (recall 0 → 0.52,
   B-WER halved). The mechanism works.
2. **Failure:** at that δ the boost vandalizes unbiased words (U-WER 0.128 → 0.700) — damage
   mechanism documented in [[trie-token-biasing]] (orphaned fragments, word-start distortion field;
   e.g. "HUSBANDMEN" → "Wea Husspundman"). Operating points must be U-WER-constrained (spec §7).
3. **Null:** bare-word semantic gating adds nothing — root cause in [[sonar]] (word-level inputs out
   of distribution). Redirects effort to H1a prompt templates, not λ tuning.
4. **Method lesson:** tiny dev sets cannot estimate catastrophic-decode tail risk; spec now shards
   25% TUNE from full dev splits.
5. **Process cost:** mining burned test-other for final reporting ([[librispeech]] quarantine).
