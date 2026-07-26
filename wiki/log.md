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
