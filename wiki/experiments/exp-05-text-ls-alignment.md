---
type: experiment
tags: [alignment, text-to-text, spec-10.3]
updated: 2026-07-28
notebook: whisper_sonar_text_alignment_librispeech.ipynb
status: Phase 0 supporting analysis (spec §10.3) — run 2026-07-28 on GPU server, n_per_split=1000
---

# Exp-05: Text-to-Text LibriSpeech Alignment (silence-conditioned)

**Question.** The missing cell of the calibration 2×2 grid: LibriSpeech dev transcripts under
Exp-01's silence-conditioned recipe — isolating the **corpus effect** (vs [[exp-01-linear-alignment]],
conditioning held fixed) from the **conditioning effect** (vs [[exp-04-audio-alignment]], corpus held
fixed).

## Results (cross-matrix on text-conditioned test pools, 67/71 utterances)

| Matrix (eval layer) | dev-clean top-1 | dev-other top-1 | paired cos |
|---|---|---|---|
| **W_text_ls** (this run, layer 4) | **1.000** | **1.000** | 0.41 / 0.42 |
| W_wikitext (Exp-01, layer 4) | 0.970 | 0.916 | 0.34 / 0.33 |
| W_audio (Exp-04, its layer 3) | 0.970 | 0.986 | 0.39 / 0.45 |
| W_audio at layer 4 | 0.806 | 0.887 | 0.23 / 0.26 |

Prefix probe (W_text_ls, layer 4): 25% → **0.060 / 0.127**; 50% → 0.597 / 0.493; 75% → 0.970 /
0.916; 100% → 1.000. Best layer under silence conditioning stays **4** (as in Exp-01).

## Verdicts (the 2×2 decomposition)

1. **Corpus effect: minor** (~3–8 points): WikiText's matrix nearly matches the domain-fitted one on
   LS text states. Domain-matched *text* alone was never the issue.
2. **Conditioning effect: decisive and asymmetric.** `W_audio` transfers *to* text states almost
   perfectly (0.97/0.99 at its native layer 3), while silence-fitted matrices collapse *on* audio
   states (0.28/0.31, [[exp-04-audio-alignment]]). Audio-fitted calibration generalizes both ways;
   silence-fitted does not.
3. **Early-prefix collapse under text-only conditioning** (0.06/0.13 at 25% — far below WikiText's
   0.29 and audio's 0.39/0.41): early LibriSpeech utterance text is generic; without cross-attention
   supplying whole-utterance acoustic evidence, early states carry little identity. Reinforces that
   [[semantic-gating]] must run on audio-conditioned calibration.
4. **Net for spec §10.3:** `W_audio` (layer 3) is best or near-best in every grid cell — the robust
   frozen-artifact choice. Confirms and strengthens [[exp-04-audio-alignment]]'s proposal.

## Caveats

Same pool-saturation caveat as Exp-04 (67/71); the WikiText comparison also carries a normalization
difference (raw cased WikiText in Exp-01 vs shared-normalized text here) folded into the corpus axis.
