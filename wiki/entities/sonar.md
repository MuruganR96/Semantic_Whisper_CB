---
type: entity
tags: [model, embeddings]
updated: 2026-07-27
---

# SONAR (text_sonar_basic_encoder)

Meta's multilingual **sentence** embedding model — 1024-d, language-tagged input (`eng_Latn`). The
target space of the [[whisper-sonar-linear-map]] and the scoring space of [[semantic-gating]].

## Findings

- **Sentence-level: strong.** As the target of the linear map, held-out retrieval ≈ 100%
  ([[exp-01-linear-alignment]]).
- **Word-level: weak for context.** Bare single-word embeddings barely separate in-context words
  from distractors against utterance embeddings (0.028 mean-cosine gap) — the root cause of the
  gate null result in [[exp-03-error-driven-eval]]. It is a *sentence* encoder; single words are
  out-of-distribution inputs. Hence H1a: prompt-templated keyword embeddings
  ([[src-lead-guidance]], [[semantic-gating]]).
- Embedding cost is one-off per biasing list (seconds–minutes for hundreds of entries) — the lead's
  trie group-up optimization is deferred until lists get large/dynamic.

## Operational notes

- Install: `pip install sonar-space` (depends on fairseq2); fairseq2 **hard-requires libsndfile at
  import** — `brew install libsndfile` on macOS. Ignore fairseq2's `psutil~=5.9` pin (works with 7).
- Run the pipeline on **CPU** (`device=torch.device("cpu")`) for compatibility — fairseq2's MPS
  support is patchy; it's a one-off cost per list/corpus.
- Always L2-normalize outputs; downstream code treats matmuls as cosines.
