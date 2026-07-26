---
type: experiment
tags: [exploratory, evaluation, negative-result]
updated: 2026-07-27
notebook: whisper_trie_sonar_biasing_pipeline.ipynb
status: exploratory — motivation only, not reportable (spec §9)
---

# Exp-02: First Pipeline Evaluation — Inconclusive by Ceiling

**Question.** Does the trie + SONAR-gated pipeline improve hotword recall on real audio
(LibriSpeech validation.clean)?

## Design

Full A/B/C harness built: [[trie-token-biasing|BPE trie]] with 4 surface forms, layer-4 state
capture hook, `TrieSonarBiasProcessor`, distractor-loaded biasing list, B-WER/U-WER/recall/false-
alarm metrics, two-stage δ→λ dev sweep. Targets mined by **text rarity** (document frequency ≤ 3).

## Result

All three conditions numerically identical (WER ≈ 0.030, recall 0.983, 0 transcripts differing
between B and C). **Baseline recall on the "rare" targets was 0.983** — 57/58 occurrences already
correct. The dev sweep rationally chose minimal δ=1, λ=2 because there was nothing to gain.

## Verdict and lessons

**Inconclusive — the test bench was empty, not the method wrong.**
1. Text rarity ≠ acoustic difficulty on clean read speech ([[librispeech]]): whisper-base already
   knows these words.
2. Evaluations must **verify headroom first** (baseline recall, occurrence counts) — now a protocol
   rule in [[evaluation-metrics]].
3. Fix implemented in [[exp-03-error-driven-eval]]: error-driven target mining + harder split.
4. Silver lining: biasing at low δ caused zero damage — the pipeline wiring was verified end-to-end
   (boost counters fired; U-WER flat).
