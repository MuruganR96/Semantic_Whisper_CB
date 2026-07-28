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

## Calibration question — answered ([[exp-04-audio-alignment]], 2026-07-28)

**Supersedes the earlier open question.** The transfer test settled it: the silence-fitted `W` keeps
only 0.28/0.31 top-1 on audio-conditioned states (paired cosine 0.48 → 0.13), while `W_audio`
(fitted on real-audio teacher-forced states, dev TUNE shards) achieves 1.000 on both splits.
**Calibration must be conditioning-matched:** proposed frozen artifact =
`whisper_to_sonar_W_audio.pt`, best layer **3** (note: layer choice shifts from 4 under silence to 3
under audio — hooks must follow). Early-prefix retrieval also improves (0.29 → 0.39/0.41 at 25%).
Lead approval pending (spec §10.3).

**2×2 grid complete ([[exp-05-text-ls-alignment]]):** corpus effect minor (W_wikitext 0.97/0.92 vs
W_text_ls 1.000 on LS text states); conditioning decisive and **asymmetric** — `W_audio` transfers
to text states at 0.97/0.99, silence-fitted matrices do not transfer to audio; text-only calibration
collapses in the early-prefix regime (0.06/0.13 at 25%). `W_audio` is best or near-best in every
cell.
