---
type: meta
updated: 2026-07-27
---

# Log

Append-only. Entry format: `## [YYYY-MM-DD] <op> | <title>` where `<op>` ∈ {ingest, query, lint,
seed, experiment, decision}. Newest entries at the bottom. `grep "^## \[" log.md | tail -5` shows
recent activity.

## [2026-07-27] ingest | Comprehensive Guide: ASR Contextual Biasing (docx)

Original design document ingested (now `raw/Comprehensive Guide ASR Contextual Biasing.docx`).
Summary and critique filed in [[src-asr-biasing-guide]]. Key claims: standard trie over radix for
ASR biasing; shallow-fusion logit bias via a least-squares Whisper→SONAR projection. Code in the
document was illustrative/broken; architecture worth testing.

## [2026-07-27] experiment | Exp-01: linear alignment holds

Whisper→SONAR linear map validated on held-out sentences (~100% top-1 retrieval, layer 4 best,
graceful prefix degradation). Artifact: `whisper_to_sonar_W.pt`. Filed [[exp-01-linear-alignment]];
updated [[whisper-sonar-linear-map]].

## [2026-07-27] experiment | Exp-02: clean-split evaluation hit ceiling

Trie+gate pipeline built and run on validation.clean — baseline already recognized 98.3% of mined
hotwords; all conditions identical. Inconclusive by lack of headroom, not a method verdict. Filed
[[exp-02-clean-split-ceiling]]; lesson recorded in [[librispeech]].

## [2026-07-27] experiment | Exp-03: error-driven evaluation on test.other

Certified-failure targets: baseline recall 0.000 → trie 0.518, but U-WER 0.128 → 0.700 at δ=3;
semantic gate ≈ trie-only, predicted by 0.028 oracle separation. Filed [[exp-03-error-driven-eval]];
updated [[trie-token-biasing]] (damage mechanism) and [[semantic-gating]] (null result).

## [2026-07-27] ingest | Research lead guidance (Slack) + research policy

Lead proposed prompt-templated keyword embeddings and required an evaluation plan before
implementation; company policy requires Phase 0 experiment specification. Filed [[src-lead-guidance]]
and [[research-policy]]. Consequence: EXPERIMENT_SPEC.md drafted; test-other quarantined; prior
experiments reclassified exploratory.

## [2026-07-27] seed | Wiki instantiated

Wiki created from accumulated session knowledge: 5 concept pages, 3 experiment pages, 3 entity
pages, 2 source pages, 1 process page, overview/index/log. Schema added to CLAUDE.md.

## [2026-07-27] decision | Audio-conditioned alignment experiment created (spec §10.3 instrument)

New notebook `whisper_sonar_audio_alignment_experiment.ipynb`: fits `W_audio` from real LibriSpeech
audio (teacher-forced reference over true encoder output, dev-clean/dev-other TUNE shards), layer
sweep + prefix probe under audio conditioning, and transfer evaluation of the silence-fitted `W` on
audio states. Directly informs the spec §10.3 calibration decision; results to be filed as exp-04.
Updated [[whisper-sonar-linear-map]] open-question section.

## [2026-07-27] decision | Text-to-text LibriSpeech alignment notebook completes the 2×2 grid

New notebook `whisper_sonar_text_alignment_librispeech.ipynb`: silence-conditioned states for
LibriSpeech dev-clean/dev-other TUNE transcripts → SONAR (Exp-01's recipe, domain-matched corpus).
Completes the calibration grid {WikiText, LibriSpeech} × {silence, audio}: vs Exp-01 isolates the
corpus effect, vs the audio notebook isolates the conditioning effect; cross-matrix section
evaluates all available W artifacts on text-conditioned states. Produces
`whisper_to_sonar_W_text_librispeech.pt`.
