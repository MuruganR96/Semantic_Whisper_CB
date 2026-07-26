---
type: concept
tags: [embedding-alignment, linear-map, sonar, whisper]
updated: 2026-07-27
---

# The Whisper→SONAR Linear Map (W)

A fixed matrix `W ∈ R^{512×1024}` translating [[whisper]] decoder hidden states into [[sonar]]
embedding space, fitted **once, offline, on frozen models** — the load-bearing artifact
(`whisper_to_sonar_W.pt`) that makes training-free [[semantic-gating]] possible.

## Calibration recipe (validated in [[exp-01-linear-alignment]])

1. Parallel sentences through both frozen models (exploratory: 1,600 WikiText sentences).
2. Whisper side: teacher-force the text over a **silence** spectrogram (30 s of zeros through the
   *feature extractor* — never a raw zeros tensor), cache the encoder output once, mean-pool decoder
   hidden states over real text-token positions only (skip the 4 prefix specials and `<|endoftext|>`),
   L2-normalize.
3. SONAR side: embed the same sentence, L2-normalize.
4. Ridge regression (α by validation top-1 retrieval), no intercept — runtime must be a bare matmul.

## Evidence

- Held-out sentence retrieval ≈ **100% top-1** (chance 0.25%); true-pair cosine ≈ 0.5 vs random
  ≈ 0.15 — retrieval works on the *gap*, not absolute closeness. Never use absolute cosine
  thresholds downstream; use relative/normalized scores.
- **Layer 4** of the decoder aligned best; all layers saturated on a 400-pool, so layer choice should
  use prefix-probe numbers, not full-sentence retrieval.
- **Prefix degradation is graceful:** top-1 ≈ 0.29 at 25% prefix → 0.77 at 50% → 1.00 full — the
  property that makes mid-beam-search use plausible.

## Gotchas

- Indexing: `decoder_hidden_states[4]` = output of `model.model.decoder.layers[3]` (index 0 is the
  embedding layer). Hooks use `best_layer − 1`.
- Runtime pooling must mirror calibration pooling (mean over text tokens, L2-norm) — the prefix probe
  is the evidence this transfer works.
- Full-sequence states at decode time require `use_cache=False` (see [[whisper]]).

## Open question

Calibration corpus: WikiText (validated) vs LibriSpeech dev transcripts (domain-matched) — spec §10.3;
proposed resolution: fit both, select by held-out retrieval on TUNE references, freeze the winner.
