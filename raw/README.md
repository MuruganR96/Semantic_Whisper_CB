# raw/ — immutable sources

Curated source documents for the research wiki. **The LLM reads from this directory but never
modifies anything in it.** Drop new sources here (papers, articles, exported docs, transcripts,
images under `assets/`), then ask the agent to ingest — it will summarize into `wiki/sources/`,
update affected wiki pages, and append to `wiki/log.md`.

Current contents:

- `Comprehensive Guide ASR Contextual Biasing.docx` — the founding design document
  (ingested 2026-07-27 → `wiki/sources/src-asr-biasing-guide.md`).
