---
type: meta
tags: [asr, contextual-biasing, thesis]
updated: 2026-07-27
---

# Project Overview: Semantic Contextual Biasing for Whisper

## Research question

Can a **training-free** pipeline make Whisper reliably transcribe rare, domain-specific words
(hotwords) by combining (1) [[trie-token-biasing|trie-based shallow fusion]] over a biasing list with
(2) a semantic gate that scores each hotword's relevance via a fixed
[[whisper-sonar-linear-map|linear map]] from Whisper decoder states into [[sonar|SONAR]] embedding
space — without fine-tuning any model?

## Current thesis (evolving)

- The **linear bridge exists and is strong at sentence level** — validated in
  [[exp-01-linear-alignment]] (~100% held-out retrieval, best at decoder layer 4).
- The **trie mechanism works**: it recovered ~52% of words the baseline provably could not transcribe
  ([[exp-03-error-driven-eval]]).
- But naive boosting is **destructive at useful strengths** (U-WER 0.128 → 0.700 at δ=3): partial
  trie matches leave orphaned hotword fragments — no failure-arc revocation is implemented yet.
- The **semantic gate as first built is a null result**: bare-word SONAR embeddings carry almost no
  utterance-context signal (oracle separation 0.028), so [[semantic-gating]] had nothing to act on.
  The live hypothesis (H1a) is that **prompt-templated embeddings** rescue the signal — proposed by
  the research lead ([[src-lead-guidance]]) and testable offline with the discriminability
  diagnostic.

## Where the project stands (2026-07-27)

Formal process per [[research-policy]]: **Phase 0** — `EXPERIMENT_SPEC.md` (draft v0.1) submitted for
lead approval; open questions §10 (template set, keyword-list protocol, W calibration corpus,
thresholds). Supporting analyses ready to run: the prompt-template diagnostic
(`phase0_prompt_template_diagnostic.ipynb`) and Phase 1 baseline validation
(`phase1_baseline_validation.ipynb`). Prior experiments are exploratory only — motivation, not
reportable numbers.

## Open questions

1. Does any prompt template clear the 0.05 oracle-separation go/no-go? (H1a — blocks condition P.)
2. Can an operating point satisfy the U-WER ≤ 1.05× constraint while keeping meaningful recall?
   (Likely requires first-token boost asymmetry and/or failure-arc bonus revocation —
   see [[trie-token-biasing]].)
3. Word-level vs phrase-level biasing entries: if H1a fails, the redesign moves to phrase/topic
   embeddings ([[semantic-gating]]).
4. W calibration corpus: WikiText (validated) vs LibriSpeech dev transcripts (domain-matched)?

## Map of the wiki

- Concepts: [[contextual-biasing]] · [[trie-token-biasing]] · [[whisper-sonar-linear-map]] ·
  [[semantic-gating]] · [[evaluation-metrics]]
- Experiments: [[exp-01-linear-alignment]] · [[exp-02-clean-split-ceiling]] ·
  [[exp-03-error-driven-eval]]
- Entities: [[whisper]] · [[sonar]] · [[librispeech]]
- Sources: [[src-asr-biasing-guide]] · [[src-lead-guidance]]
- Process: [[research-policy]]
