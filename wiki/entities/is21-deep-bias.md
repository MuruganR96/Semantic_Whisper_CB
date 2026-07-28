---
type: entity
tags: [dataset, benchmark, biasing-lists]
updated: 2026-07-28
---

# IS21 Deep-Biasing Benchmark (Le et al., Interspeech 2021)

The published LibriSpeech contextual-biasing benchmark (`fbai-speech/is21_deep_bias`), adopted
(pending lead approval, spec §10.2) as the **primary validation/evaluation protocol** — directed by
the user on 2026-07-28.

## What it provides

- **Per-utterance biasing lists** for LibriSpeech: rare words = outside the 5,000 most frequent
  training-corpus words, drawn from each utterance's reference.
- **Distractor variants** N ∈ {100, 500, 1000} rarity-matched distractors per list.
- The **B-WER / U-WER decomposition protocol** our [[evaluation-metrics]] already follows — adopting
  the lists makes our numbers directly comparable with published deep-biasing results.

## How the spec uses it

- **Development/tuning:** lists rebuilt on `dev-clean`/`dev-other` with the benchmark's own recipe
  (rare-word threshold + distractor sampling), on TUNE shards; N=100 default, {500, 1000} as
  robustness checks.
- **Phase 4 final reporting:** the officially released test-split lists, untouched until then.
- **Secondary protocol retained:** corpus-level error-driven lists ([[evaluation-metrics]]) as the
  headroom view — IS21 rare words are rarity-defined, not difficulty-defined, and
  [[exp-02-clean-split-ceiling]] showed rarity alone can leave no headroom on clean audio.

## Open items

- Lead decision §10.2: adoption + N default + whether IS21 `test-other` reporting is admissible
  given the exploratory quarantine ([[librispeech]]).
- Verify the released repo's exact file coverage (test-only vs dev lists) when wiring the loader;
  the dev-side recipe reconstruction is required either way.
