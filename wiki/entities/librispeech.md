---
type: entity
tags: [dataset]
updated: 2026-07-27
---

# LibriSpeech

Read-audiobook English ASR corpus; the project's evaluation data. Accessed exclusively via
HuggingFace **streaming** (`openslr/librispeech_asr`, configs `clean`/`other`) — full splits are
never downloaded.

## Split usage and status

| Split | Role | Status |
|---|---|---|
| dev-clean, dev-other | all development and tuning (TUNE/DEVTEST shards, seed 42) | active per spec §3 |
| test-clean | final reporting only | frozen, untouched |
| test-other | final reporting only | **quarantined** — exploratory Exp-03 mined targets from it |

## Practical notes

- References are uppercase/unpunctuated → shared normalization mandatory
  ([[evaluation-metrics]]).
- Utterances > 29 s are excluded (Whisper's 30 s window would truncate; exclusion counts reported).
- Audio decoding across `datasets` versions varies (dict-with-array vs torchcodec `AudioDecoder` vs
  path) — notebooks carry a three-way `get_audio()` fallback. Text-only analyses strip the audio
  column (`remove_columns(["audio"])`) to avoid decoding entirely.
- **Headroom pattern:** `whisper-base` is near-ceiling on clean splits (rarity-mined words 98%
  recognized — [[exp-02-clean-split-ceiling]]); `*-other` splits error 2–3× more, and error-driven
  mining on them yields certified-failure targets ([[exp-03-error-driven-eval]]).
