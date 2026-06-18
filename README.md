# MMDocIR-parsed

Parsed multimodal document artifacts for the [MMDocIR](https://github.com/MMDocRAG/MMDocIR) benchmark, produced by the [mas](https://github.com/PetrosVitalis/mas) ingestion pipeline (MinerU + table/vision enrichment).

Each `parsed_docs_<doc_stem>/` folder contains:

- `document.md` — layout-aware markdown with page markers
- `assets/` — figures, tables, equations (cropped images)
- `tables/` — extracted CSV tables
- `retrieval_index.json` — page/chunk/layout retrieval index
- optional vector sidecars (`retrieval_*_vectors.npz`, `encoded_*.pkl`)

**313 documents** from the MMDocIR evaluation corpus (~3 GB).

Used by `mmdocir_eval.py` in the main MAS repository via `thesis-multi-agent/MMDocIR-parsed` (symlink).
