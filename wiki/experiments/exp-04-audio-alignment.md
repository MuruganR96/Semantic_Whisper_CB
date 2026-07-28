---
type: experiment
tags: [alignment, audio-conditioned, spec-10.3]
updated: 2026-07-28
notebook: whisper_sonar_audio_alignment_experiment.ipynb
status: Phase 0 supporting analysis (spec §10.3) — run 2026-07-28 on GPU server, n_per_split=1000
---

# Exp-04: Audio-Conditioned Whisper→SONAR Alignment (LibriSpeech dev)

**Questions.** (H-audio) Does the linear map hold on the *deployment* state distribution —
decoder states conditioned on real audio? (H-transfer) Does Exp-01's silence-fitted `W` transfer to
those states? Directly decides spec §10.3 (calibration corpus/conditioning).

## Setup

dev-clean + dev-other **TUNE shards** (frozen shard rule), ≈1,385 (audio, reference) pairs after the
≤29 s filter (dev-clean 671: 537 train / 67 val / 67 test; dev-other 714: 572 / 71 / 71). Decoder
teacher-forced on the normalized reference over **real encoder outputs**; pooling identical to
Exp-01; layer sweep, Ridge α by val top-1; `W_audio` fitted on both splits' train sets jointly.

## Results

| Evaluated on audio-conditioned test states | dev-clean | dev-other |
|---|---|---|
| **W_audio** (this run, layer 3) — top-1 | **1.000** | **1.000** |
| … paired / random cosine | 0.482 / 0.167 | 0.511 / 0.190 |
| **W_silence** (Exp-01, native layer 4) — top-1 | 0.284 | 0.310 |
| … paired cosine | 0.131 | 0.136 |
| chance top-1 | 0.015 | 0.014 |

Prefix probe (top-1, `W_audio`): 25% → **0.388 / 0.409** (Exp-01 silence: 0.29); 50% → 0.836 /
0.775; 75% → 0.970 / 0.958; 100% → 1.000 / 1.000.

## Verdicts

1. **H-audio: strongly supported.** The linear bridge holds perfectly on deployment-distribution
   states, on the hard split too.
2. **H-transfer: refuted.** Conditioning mismatch is expensive — silence-fitted `W` keeps only
   ~28–31% top-1 on audio states (paired cosine collapses 0.48 → 0.13). **Calibration must be
   audio-conditioned**; `whisper_to_sonar_W_audio.pt` proposed as the frozen artifact (spec §4/§10.3
   updated; lead approval pending).
3. **Best layer shifts 4 → 3** under audio conditioning — the [[semantic-gating]] hook layer changes
   if adopted.
4. **Early-decode regime improves** (~+10 pts at 25% prefix): cross-attention gives the decoder
   whole-utterance acoustic evidence from step one — good news for gating where it matters most.

## Caveats

Test pools of 67/71 saturate full-sentence retrieval (all layers 94–100% in the sweep) — the
decisive evidence is the *contrast* between matrices on identical pools, not absolute percentages;
teacher-forced normalized text remains one step from Whisper's own cased runtime output.
