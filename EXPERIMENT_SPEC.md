# Experiment Specification: Zero-Shot Contextual Biasing for Whisper via Trie Shallow Fusion with SONAR-Embedding Semantic Gating

| | |
|---|---|
| **Status** | DRAFT v0.1 — submitted for Phase 0 review |
| **Implementer** | Murugan Rajenthiran |
| **Reviewer** | TBD (assigned per Rule 2) |
| **Research lead approval** | Pending |
| **Prior work** | Exploratory notebooks 1–3 in this repository (pre-Phase-0; findings inform this spec, numbers not for reporting — see §9) |

---

## 1. Research hypothesis

**H1 (primary).** During Whisper beam-search decoding, gating a trie-based shallow-fusion keyword
boost with the cosine similarity between (a) the decoder's pooled hidden state projected into SONAR
space by a fixed, offline-fitted linear map `W`, and (b) the keyword's SONAR text embedding, improves
**keyword recall** over trie-only shallow fusion at equal or better unbiased-word error rate (U-WER)
and false-alarm count. No component of Whisper or SONAR is trained or fine-tuned.

**H1a (embedding side, testable offline before any decoding).** Prompt-templated keyword embeddings
(e.g., `"A speech that includes \"{keyword}\""`) separate in-context keywords from distractor
keywords better than bare-word embeddings, measured by the oracle separation diagnostic (§5.4).
*Motivation: exploratory measurement showed bare-word separation of only 0.028 — insufficient for any
gate to act on.*

**H0.** Semantically gated biasing is indistinguishable from trie-only biasing at matched operating
points (the gate adds no information).

## 2. Baseline methods

| ID | Method | Role |
|---|---|---|
| **B0** | `whisper-base`, beam search, no biasing | absolute reference; source of error-driven keyword mining |
| **B1** | Trie-based shallow fusion: BPE-token trie over the keyword list; additive logit boost δ for every token continuing a trie path (suffix matching per beam, per step) | established method the proposal must beat |
| **P** | B1 + semantic gate: per-token boost = δ + λ·max(0, cos(pooled-state @ W, keyword SONAR embedding)) | proposed method; **λ = 0 reduces P to B1 exactly** (clean ablation) |

## 3. Dataset and data split

- **Corpora:** LibriSpeech `dev-clean` and `dev-other` for all development and tuning (per research
  lead's instruction). `test-clean` / `test-other` are frozen and untouched until a Phase 4 run with
  the approved configuration; exploratory use of `test-other` in prior notebooks disqualifies those
  numbers from reporting (§9).
- **Shards:** each dev split is partitioned by fixed seed (42) into **TUNE (25%)** — hyperparameter
  selection (δ, λ, prompt template) — and **DEVTEST (75%)** — development-time measurement. Utterance
  IDs for every shard are recorded in the run artifacts.
- **Keyword lists (decision point for lead — §10.2):**
  - *Proposed primary:* per-utterance biasing lists in the style of the published deep-biasing
    literature — rare words from the utterance's own reference plus **N = 100 rarity-matched
    distractors** absent from that reference; comparable with prior published work.
  - *Secondary (headroom view):* corpus-level error-driven list — words `whisper-base` (B0)
    substituted/deleted on the shard, rarity-filtered, plus equal-count distractors mined outside the
    shard. Guarantees measurable headroom (exploratory finding: text-rarity alone yields lists the
    baseline already recognizes at 0.98 recall).
- **Audio:** 16 kHz mono; utterances longer than 29 s excluded (Whisper 30 s window; exclusion count
  reported).

## 4. Training configuration

No training. All models frozen:

- **ASR:** `openai/whisper-base` (HF revision pinned in artifacts).
- **Text embedder:** SONAR `text_sonar_basic_encoder` (package version pinned).
- **Calibration of `W` (the only fitted artifact):** ridge regression (α selected on a held-out
  validation shard by top-1 retrieval) from Whisper decoder layer-4 mean-pooled, L2-normalized text
  states to unit-norm SONAR sentence embeddings; calibration corpus is a decision point (§10.3):
  WikiText sentences (exploratory default, top-1 retrieval ≈ 100% held-out) vs. LibriSpeech dev
  transcripts (domain-matched). `W` is fitted once, checksummed, and frozen before Phase 3; it is
  never refitted after approval.

## 5. Evaluation metrics

All computed after one shared text normalization (lowercase; strip all characters except `[a-z' ]`;
collapse whitespace), applied identically to references and hypotheses.

1. **Keyword recall (primary).** Occurrence-level. A reference occurrence of a keyword counts as
   recalled **iff** it lies inside an `equal` chunk of the jiwer word alignment between normalized
   reference and hypothesis. No substring credit (e.g., "king" is NOT recalled by "kingsman");
   multi-word keywords require all their words aligned contiguously. Recall = recalled occurrences /
   total occurrences; occurrence counts reported alongside.
2. **WER** (overall, jiwer), and **B-WER / U-WER** decomposition: substitutions and deletions
   attributed by the reference word (biased-list word → B-WER, else U-WER); insertions attributed by
   the hypothesis word.
3. **False alarms:** biasing-list words inserted, or substituted in where the reference word is not
   that keyword, summed over the corpus.
4. **Oracle gate separation (offline, H1a):** mean cos(keyword embedding, SONAR embedding of its own
   utterance's reference) minus mean cos(distractor embedding, same utterance embeddings). Computed
   per candidate prompt template on TUNE references only; used to select the template and as a
   go/no-go pre-check for P (§7).
5. Every metric reported separately for `dev-clean` and `dev-other`, with utterance and occurrence
   counts.

## 6. Expected implementation differences between baseline and proposed method

The **only** permitted difference between B0, B1, and P is the additive bias term inside a single
`LogitsProcessor`:

| Condition | Frozen value (all of B0 / B1 / P) |
|---|---|
| Model checkpoint | `openai/whisper-base`, pinned revision |
| Decoding | beam search, num_beams = 5, deterministic (no sampling), language = en, task = transcribe, no timestamps |
| KV cache | **disabled for all conditions** (P requires full-sequence states; B0/B1 match to eliminate the confound) |
| Preprocessing | identical feature extraction, 16 kHz, same utterance filter |
| Evaluation | identical normalization, jiwer version pinned, same alignment attribution rules |
| Seeds | 42 everywhere; shard membership recorded |

Allowed to differ: δ (B1, P), λ (P only), keyword list contents per protocol (§3). Any other
difference requires re-approval under Rule 1.

## 7. Success and failure criteria

**Pre-registered operating-point rule** (TUNE shard only): select (δ, λ) maximizing keyword recall
subject to **U-WER ≤ 1.05 × B0 U-WER** and **false alarms ≤ B1's at the same δ**.
*Motivation: exploratory run at unconstrained δ = 3 reached recall 0.52 but U-WER 0.128 → 0.700 —
recall alone is not success.*

Evaluated on DEVTEST:

- **Success (H1):** P exceeds B1 keyword recall by ≥ 5 points absolute at satisfied constraints, on
  both dev splits; and B1 exceeds B0 recall by ≥ 10 points (sanity that the trie mechanism engages).
- **Partial / negative (documented, not failed):** B1 ≫ B0 but P − B1 < 5 points → semantic gate
  ineffective at word granularity; report as negative result with the H1a diagnostic as explanation.
- **Failure:** no (δ, λ) satisfies the U-WER constraint → shallow fusion unsafe at these scales;
  escalate to boost-mechanics work (first-token asymmetry, failure-arc bonus revocation) as a new
  Phase 0 spec before further gating research.
- **Go/no-go pre-check for P:** if the best prompt template's oracle separation (§5.4) is below
  **0.05** (threshold subject to lead approval), P is not run; H1a is reported as refuted and the
  embedding-side redesign becomes the next spec.

## 8. Phase mapping (per research policy)

| Policy phase | Concrete deliverable |
|---|---|
| **0 — Design** | this document, approved |
| **1 — Baseline validation** | B0 WER on `dev-clean`/`dev-other` reproduced and cross-checked against published Whisper results; independently re-run by the reviewer from the committed config |
| **2 — Implementation** | trie + processor + metrics module with unit tests (casing/leading-space tokenization, alignment-based recall, B/U-WER attribution); reviewer code review |
| **3 — Smoke test** | FAST-mode run (≈ 20 utterances/split) verifying pipeline, boost counters firing, metric outputs |
| **4 — Full experiment** | frozen-config run on both dev splits; TUNE→DEVTEST protocol; artifacts: config JSON, shard IDs, seeds, per-utterance transcripts, all CSVs, `W` checksum, code commit hash |
| **5 — Result audit** | reviewer reproduces headline numbers from artifacts alone |

## 9. Prior exploratory work and known deviations (disclosed)

Notebooks 1–3 (this repository) predate this spec:

1. **Alignment validation (kept as motivation):** a fixed linear Whisper→SONAR map achieves ~100%
   held-out sentence retrieval (layer 4 best); prefix states degrade gracefully (top-1 0.29 at 25%
   prefix → 1.00 at full sentence).
2. **Deviations that this spec corrects:** target mining used `test-other` (now quarantined); KV
   cache differed across conditions (now frozen off for all); tuning used 15 utterances with a
   WER-only constraint (now TUNE shards with a U-WER constraint); bare-word keyword embeddings (now
   template selection under H1a).
3. Exploratory numbers appear nowhere outside §9 motivation notes and are not for reporting,
   publication, or patent discussion.

## 10. Open questions for research lead (blocking approval)

1. **Prompt template candidate set** for H1a — proposed: bare word; `A speech that includes
   "{keyword}"`; `The speaker mentions {keyword}`; template selection strictly by §5.4 diagnostic on
   TUNE. Additions welcome.
2. **Keyword list protocol** — per-utterance lists with N=100 distractors (literature-comparable,
   proposed primary) vs. corpus-level error-driven lists (headroom view, proposed secondary): report
   both, or one only?
3. **`W` calibration corpus** — WikiText (validated in exploration) vs. LibriSpeech dev transcripts
   (domain-matched; would need a small held-out check). Proposed: fit both once, select by held-out
   retrieval on TUNE references, freeze the winner.
4. **Thresholds** — 5-point recall margin, 1.05× U-WER budget, 0.05 separation go/no-go: acceptable?
5. **Trie group-up for embedding cost** (your suggestion): proposed to defer to a follow-up spec —
   at current list sizes (≤ a few hundred keywords) per-keyword embedding is one-off and cheap; the
   grouped-prompt design changes embedding semantics (one centroid per group) and deserves its own
   evaluation once the naive approach is validated, consistent with "start simple, validate first."
