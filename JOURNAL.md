## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/149

**Issue title:** Structural chunker silently drops documents that contain no headings

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The `StructuralChunker.chunk()` method in `ingestion/chunking/structural_chunker.py` splits markdown documents by heading boundaries (h1–h3). When a document contains no markdown headings at all — for example, a plain-text resume or a brief README — the `_extract_sections()` helper never adds anything to the `sections` list because it only saves content when a heading has been seen. The result is that `chunk()` returns an empty list, and the document is silently excluded from the RAG index. A successful fix would make the chunker fall back to treating the entire document as a single chunk whenever no headings are detected, so no content is lost from the embedding pipeline.

**Branch name:** fix/149-structural-chunker-no-headings

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
