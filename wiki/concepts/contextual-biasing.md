---
type: concept
tags: [asr, biasing]
updated: 2026-07-27
---

# Contextual Biasing

## The problem

ASR models mistranscribe rare words — names, drug names, product terms — because they are
underrepresented in training data. Contextual biasing supplies a **biasing list** of expected words
at inference time and nudges decoding toward them. The list is legitimate context (calendar entries,
contact names, domain vocabulary), so using reference-derived rare words in evaluation is the task
definition, not leakage.

## Method families

- **Shallow fusion (our approach):** add a bonus to the logits of tokens that continue a biasing-list
  word during beam search. Training-free; implemented via a [[trie-token-biasing|token trie]] walked
  per beam per step. Our variant adds a [[semantic-gating|semantic gate]] scaling the bonus by
  contextual relevance.
- **Deep biasing (contrast):** trained integration — e.g. TCPGen-style pointer networks or
  cross-attention adapters that consume the biasing list inside the model. Stronger, but requires
  training. This is the documented escalation path if shallow fusion's ceiling proves too low
  ([[exp-03-error-driven-eval]] verdict #4 territory).

## What success means

Not recall alone. The trade-off surface is recall vs **collateral damage**, measured as U-WER (error
rate on non-list words) and false alarms (list words hallucinated in) — see [[evaluation-metrics]].
Exp-03 made this concrete: recall 0 → 0.52 *and* U-WER 0.128 → 0.700 is a net loss.

## Key design tension (current frontier)

An unconditional boost strong enough to rescue hard words is strong enough to vandalize easy ones.
Two remedies on the table:
1. **Mechanical:** first-token boost asymmetry, failure-arc bonus revocation
   ([[trie-token-biasing]]).
2. **Semantic:** only boost words the utterance context supports ([[semantic-gating]]) — currently
   blocked on H1a (prompt-templated embeddings) after the bare-word null result.
