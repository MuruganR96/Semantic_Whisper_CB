---
type: process
tags: [policy, governance]
updated: 2026-07-27
---

# Research Policy — Rules and Phase Status

Company research policy (internal `research_policy.md`, shared 2026-07-27) governs all experiments.

## Rules

- **Rule 1 — Experiment specification before implementation:** hypothesis, baselines, dataset/split,
  training configuration, metrics, expected implementation differences, success/failure criteria.
  Conditions (decoding method, preprocessing, evaluation procedure, sample counts) are frozen on
  approval; changes require re-approval.
- **Rule 2 — Peer review:** every important implementation and experiment reviewed by someone other
  than the implementer. Each experiment names an implementer and a reviewer.

## Phases and current status

| Phase | Requirement | Status (2026-07-27) |
|---|---|---|
| 0 Design | Spec prepared; lead approves hypothesis/baseline/eval/conditions | **`EXPERIMENT_SPEC.md` draft v0.1 submitted**; §10 open questions pending; supporting H1a diagnostic notebook ready |
| 1 Baseline validation | Reproduce baseline; independent verification; gate to proceed | Notebook ready (`phase1_baseline_validation.ipynb`); reviewer unassigned |
| 2 Implementation | Proposed method + code/design review | blocked on 0/1 |
| 3 Smoke test | Small-scale pipeline verification | FAST mode built into notebooks |
| 4 Full experiment | Frozen approved configuration | — |
| 5 Result audit | Non-implementer verifies config/logs/results; only audited results reportable | artifact conventions in place (config JSONs, SHA-256, resumable JSONLs) |

## Standing consequences

- [[exp-01-linear-alignment]], [[exp-02-clean-split-ceiling]], [[exp-03-error-driven-eval]] are
  **pre-Phase-0 exploratory** — motivation only, never reportable.
- test-other is quarantined ([[librispeech]]); development on dev-clean/dev-other only.
- Frozen parity rules live in [[evaluation-metrics]] and spec §6 (notably `use_cache=False` for all
  conditions).
