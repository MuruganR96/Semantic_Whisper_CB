---
type: concept
tags: [semantic-gate, sonar, embeddings]
updated: 2026-07-27
---

# Semantic Gating

The project's novel claim: scale each hotword's [[trie-token-biasing|trie boost]] by its contextual
relevance, computed as `cos(pooled decoder state @ W, hotword SONAR embedding)` — so relevant words
get boosted hard while irrelevant list entries (distractors) are suppressed. Formula per token:
`boost = δ + λ·max(0, cos)`; λ=0 reduces exactly to trie-only (clean ablation).

## Mechanics

- A forward hook on decoder layer 4 (module index 3 — see [[whisper-sonar-linear-map]]) captures
  full-sequence hidden states each step; requires `use_cache=False` for all conditions (parity, spec
  §6).
- Pool text-token states → normalize → `@ W` → normalize → one matmul against all hotword embeddings
  gives every beam × every hotword cosine per step.
- Production port (not yet built): incremental per-beam pooled states reindexed by `beam_idx`, KV
  cache back on.

## Status: bare-word gating is a null result — H1a is the live hypothesis

- **Oracle discriminability diagnostic** (the crucial cheap test): with *reference-text* utterance
  embeddings — perfect context — bare-word target embeddings separated from distractors by only
  **0.028** mean cosine. No gate can amplify a signal that isn't there.
- [[exp-03-error-driven-eval]] confirmed the prediction end-to-end: C ≈ B on recall (0.536 vs 0.518),
  no false-alarm advantage.
- **H1a (from [[src-lead-guidance]]):** prompt-templated embeddings — e.g.
  `A speech that includes "{keyword}"` — move keywords into SONAR's sentence-like input distribution
  and should raise separation. Template selection is by measurement
  (`phase0_prompt_template_diagnostic.ipynb`): 5 candidates scored by separation gap and ROC-AUC on
  dev TUNE shards; go/no-go at gap ≥ 0.05 (spec §7).

## Interpretation discipline

The diagnostic is an **upper bound**: runtime context is the noisier `state @ W` projection, not the
reference embedding. Template fails diagnostic ⇒ gate cannot work (skip decoding entirely). Template
passes ⇒ gate *may* work; expect a smaller runtime effect.

## If H1a fails

Escalation paths, in order: phrase/topic-level biasing entries (embed the list as topic sentences);
softmax-normalized similarity across the list (exploit relative structure despite small gaps);
non-semantic **acoustic-evidence gating** (boost only when the hotword's first token already carries
probability mass) — attacks [[trie-token-biasing]]'s damage mechanism directly without SONAR.
