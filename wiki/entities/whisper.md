---
type: entity
tags: [model, asr]
updated: 2026-07-27
---

# Whisper (openai/whisper-base)

Frozen ASR model under study. Multilingual base variant: d_model = 512, 6 decoder layers,
BPE vocabulary, 30 s log-mel input window.

## Facts that shaped the project

- **Context-sensitive BPE:** `word` / `Word` / `" word"` / `" Word"` are four different token
  sequences — the reason every biasing entry gets 4 trie surface forms
  ([[trie-token-biasing]]).
- **Layer indexing trap:** `decoder_hidden_states[k]` is the output of decoder *block* k =
  `model.model.decoder.layers[k-1]`; index 0 is the embedding layer. Layer 4 is the project's
  chosen representation ([[whisper-sonar-linear-map]]).
- **Decoding prompt:** `<|startoftranscript|><|en|><|transcribe|><|notimestamps|>` — 4 special
  tokens skipped by both trie matching and state pooling.
- **KV cache vs hidden states:** with the cache on, each step exposes only the new position's state;
  [[semantic-gating]] needs full-sequence states, so all frozen-config decoding runs
  `use_cache=False` (parity across conditions, spec §6). Cost: ~3–5× slower; production fix is
  incremental per-beam pooling.
- **Text-only states from a conditional model:** decoder states normally depend on audio; the
  silence-conditioning recipe (feature-extracted zeros, encoder output cached once) extracts usable
  text geometry anyway — validated by [[exp-01-linear-alignment]].
- **Baseline quality:** near-ceiling on clean read speech (the [[exp-02-clean-split-ceiling]]
  lesson); WER roughly 2–3× higher on `*-other` splits — where biasing has headroom.
- Boost-induced failure mode: forced token paths can leave hybrid garbage words; see
  [[trie-token-biasing]] damage mechanism.
