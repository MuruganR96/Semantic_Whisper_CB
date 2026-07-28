---
type: meta
updated: 2026-07-27
---

# Index

Catalog of every wiki page. Updated on every ingest. Start at [[overview]].

## Meta

- [[overview]] — research question, evolving thesis, project status, open questions.
- [[log]] — append-only chronology of ingests, queries, and lint passes.

## Concepts

- [[contextual-biasing]] — the problem and the method family (shallow fusion vs deep biasing).
- [[trie-token-biasing]] — BPE-token trie mechanics, surface forms, boost dynamics, failure modes.
- [[bpe-fusion-walkthrough]] — end-to-end walkthrough: how the BPE trie and word-level SONAR gating
  compose at decode time (incl. the free "group-up" property).
- [[whisper-sonar-linear-map]] — the fixed matrix W: calibration recipe, validation, gotchas.
- [[semantic-gating]] — gating boosts by SONAR similarity; the bare-word null result; H1a templates.
- [[evaluation-metrics]] — normalization, recall definition, B-WER/U-WER, false alarms, shard rule.

## Experiments (exploratory — motivation only, not reportable)

- [[exp-01-linear-alignment]] — Whisper→SONAR linear map holds (~100% held-out retrieval; layer 4). ✅
- [[exp-02-clean-split-ceiling]] — dev-clean-style eval had zero headroom (baseline recall 0.983);
  inconclusive by design flaw. ⚠️
- [[exp-03-error-driven-eval]] — trie recovers certified failures (recall 0 → 0.52) but δ=3
  vandalizes transcripts; semantic gate ≈ null, as its 0.028 diagnostic predicted. 🔬

## Entities

- [[whisper]] — model facts that bit us: BPE surface forms, layer indexing, KV-cache interplay.
- [[sonar]] — 1024-d sentence encoder; install gotchas; word-level weakness.
- [[librispeech]] — splits used, streaming pattern, test-other quarantine.
- [[is21-deep-bias]] — the Interspeech'21 biasing benchmark: per-utterance rare-word lists +
  distractors; adopted as primary eval protocol (pending §10.2 approval).

## Sources

- [[src-asr-biasing-guide]] — the original design document (docx in `raw/`); what survived scrutiny.
- [[src-lead-guidance]] — research lead's Slack guidance: prompt templates, evaluation-first process.

## Process

- [[research-policy]] — Rules 1–2, Phases 0–5, current phase status, artifact expectations.
