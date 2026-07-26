---
type: source
tags: [design-doc]
updated: 2026-07-27
raw: "raw/Comprehensive Guide ASR Contextual Biasing.docx"
ingested: 2026-07-27
---

# Source: "Comprehensive Guide — ASR Contextual Biasing using Tries and Frozen Embeddings"

The project's founding design document (docx, appears to be an exported AI-chat consolidation).
Proposed the full architecture this project tests.

## Claims and their fates

| Claim | Fate |
|---|---|
| Standard trie beats radix tree for ASR biasing (token-per-step match, per-step boosting, failure-pointer compatibility) | **Adopted** — consistent with literature; mechanics confirmed in implementation ([[trie-token-biasing]]) |
| A least-squares linear map can bridge Whisper decoder states → SONAR space, zero-shot | **Validated** — [[exp-01-linear-alignment]] (~100% held-out retrieval); the document's riskiest bet paid off |
| Shallow-fusion logit bias using that map enables training-free semantic biasing | **Partially refuted as specified** — bare-word embeddings carry no context signal ([[exp-03-error-driven-eval]]); redesign via prompt templates in progress ([[semantic-gating]]) |
| Cross-attention adapter as the trained alternative (Approach 1) | Documented escalation path if shallow fusion ceilings out |

## Defects found during scrutiny (kept for the record)

- Code was illustrative, not runnable: fabricated API (`_retrieve_init_tokens_for_generation`),
  mocked SONAR (random vectors), Script B truncated mid-class.
- Fed a raw zeros tensor as the spectrogram — silence must go through the feature extractor
  ([[whisper-sonar-linear-map]] recipe fixes this).
- Sentence-level fitting vs per-step runtime use unexamined — addressed by the prefix probe.
- Misleading artifact name (`_pca.pt` with no PCA involved).

## Where it lives

Immutable original: `raw/Comprehensive Guide ASR Contextual Biasing.docx`.
