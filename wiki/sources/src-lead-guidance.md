---
type: source
tags: [guidance, slack]
updated: 2026-07-27
raw: "Slack thread (screenshots shared in session; originals not on disk)"
ingested: 2026-07-27
---

# Source: Research Lead Guidance (Taichi Nishimura, Slack)

Direction-setting messages from the research lead; reshaped both the method and the process.

## Technical guidance

1. **Prompt-templated keyword embeddings:** propose embedding keywords as
   `A speech that includes "{keyword}"` rather than bare words — explicitly flagged as unvalidated
   ("I am unsure this prompt is good or not"). This became **H1a**, and it lands precisely on our
   measured failure (0.028 bare-word separation, [[semantic-gating]]). Template choice is being
   settled by measurement (`phase0_prompt_template_diagnostic.ipynb`).
2. **Cost concern:** N keyword embeddings is costly at scale; suggested trie/radix **group-up** —
   embedding prefix-grouped keyword batches in one prompt ("This speech includes: 'te', 'team',
   'test'"). Assessment: yields one blurred group embedding, not per-keyword vectors; viable as a
   coarse two-stage gate; deferred (spec §10.5) since per-keyword embedding is one-off and cheap at
   current list sizes ([[sonar]]).
3. **BPE-level trie matching feasibility question:** answered affirmatively — implemented and
   working, with the 4-surface-form caveat ([[trie-token-biasing]]).

## Process guidance

- Start with the **naive approach** — easy to implement and validate — before optimizing.
- **Evaluation methodology before implementation**: define (1) embedding computation (prompt), and
  (2) recall scoring — correct-match definition and datasets (**dev-other and dev-clean**, not test
  splits) — so results are reproducible and interpretable. Formalized via [[research-policy]] into
  `EXPERIMENT_SPEC.md` (alignment-based recall definition in [[evaluation-metrics]]).

## Open with the lead

Spec §10 decision points: template candidate set, keyword-list protocol (per-utterance vs corpus),
W calibration corpus, thresholds, and reviewer assignment (Rule 2).
