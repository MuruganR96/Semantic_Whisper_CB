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

## [2026-07-27] query | How does BPE-level trie fusion with SONAR gating work?

Walkthrough answer filed as [[bpe-fusion-walkthrough]]: BPE trie = WHERE (one token per edge, 4
surface forms, empty-suffix word starts), SONAR+W = HOW MUCH (word-level embeddings, per-step
projected-state cosines), joined by node `hids`; interior-node max-aggregation realizes the lead's
group-up idea without embedding subword fragments. Index updated.

## [2026-07-27] decision | V2 processor: first-token asymmetry + failure-arc revocation implemented

`TrieSonarBiasProcessorV2` added to the pipeline notebook (§7b, comparison harness §8c):
potential-based boost shaping — start η·δ, continue δ, complete δ+λ·cos (semantic term moved to
completion), abandon −E (exact revocation, EOS included via row broadcast). Marginal arithmetic
verified by standalone simulation (abandoned excursions net zero; completed chains locked).
Directly targets [[exp-03-error-driven-eval]]'s two damage mechanisms. Updated
[[trie-token-biasing]] improvements section. Scale evaluation pending (error-driven rerun with V2).

## [2026-07-28] decision | IS21 deep-biasing benchmark adopted for validation/evaluation

Per user direction: the IS21 benchmark (Le et al., Interspeech 2021) becomes the primary keyword-list
protocol — dev lists rebuilt via the benchmark recipe for tuning, official test lists reserved for
Phase 4; N=100 default. Error-driven lists retained as secondary headroom view. Spec §3/§5/§10.2 and
the Phase 0 review deck updated; entity page [[is21-deep-bias]] filed; index updated. Pending: lead
approval §10.2 incl. test-other admissibility.

## [2026-07-28] decision | Phase 0 review deck created

`phase0_spec_review.pptx` — 14 slides for the lead review: hypotheses, exploratory evidence,
baselines, IS21-based data protocol, frozen conditions, metrics, pre-registered criteria, ready
instruments, phase status, and the seven decisions requested (spec §10 + reviewer assignment).

## [2026-07-28] experiment | Exp-04: audio-conditioned alignment run — §10.3 answered

GPU run (n_per_split=1000, dev TUNE shards): `W_audio` = 1.000 held-out top-1 on both splits
(chance ≈1.5%); silence-fitted `W` transfers at only 0.28/0.31 on the same audio states (paired cos
0.48→0.13); best layer shifts 4→3; 25%-prefix retrieval improves 0.29→0.39/0.41. Filed
[[exp-04-audio-alignment]]; updated [[whisper-sonar-linear-map]] (supersession noted),
[[overview]], spec §4/§10.3, and the review deck (new results slide; decision 3 now cites the
measurement). Proposed frozen artifact: `whisper_to_sonar_W_audio.pt`.
