---
type: experiment
tags: [exploratory, alignment]
updated: 2026-07-27
notebook: whisper_sonar_alignment_experiment.ipynb
status: exploratory — motivation only, not reportable (spec §9)
---

# Exp-01: Does a Whisper→SONAR Linear Map Hold?

**Question.** Can a fixed matrix fitted by (regularized) least squares translate frozen
[[whisper]] decoder states into [[sonar]] space well enough for cosine similarity to be meaningful?
This was the riskiest assumption of [[src-asr-biasing-guide]] — tested before building anything on
it.

## Design

2,000 WikiText sentences → silence-conditioned pooled decoder states (every layer) × SONAR
embeddings; 80/10/10 train/val/test; OLS vs Ridge per layer; controls: shuffled-pairs regression
(floor), chance = 0.25%; prefix-state probe for the runtime distribution.

## Results (run on GPU machine, 2026-07-27)

- Held-out top-1 retrieval ≈ **100%** at every layer (pool of 400 — saturated); best layer **4**;
  shuffled control ≈ 0.
- True-pair cosine ≈ 0.5 vs random ≈ 0.15 — a wide *relative* gap, moderate absolute values ⇒
  downstream scoring must be relative, never absolute-thresholded.
- Prefix probe: top-1 0.29 (25% prefix) → 0.77 (50%) → 0.975 (75%) → 1.00 (100%) — graceful, so
  mid-decode use is plausible; early-word noise motivates keeping an unconditional δ floor in
  [[trie-token-biasing]].

## Verdict and product

**H1 (alignment) strongly supported.** Artifact `whisper_to_sonar_W.pt` ([[whisper-sonar-linear-map]])
consumed by all downstream pipelines. Caveats: ceiling effect masks layer differences (use prefix
numbers); lexical vs semantic content of the alignment untested at sentence level — the word-level
weakness surfaced later in [[exp-03-error-driven-eval]].
