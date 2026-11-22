# Week 3 V1 Data Pipeline Decisions & Traceability

## 1. Traceability to Topic 2 Requirements
- **Requirements Covered:** `FR1` (eligibility rules & CGPA/income limits), `FR2` (verifiable source references), `NFR4` (incremental index updates < 10 minutes), `RAI4` (100% citation presence & >= 95% source match).
- **Selected Pathways:** Dual-artefact ingestion from official JPA web scraping (`scrape_utm_scholarships.py`): **Pathway A (`PDF/TXT`)** on `jpa_scholarships.pdf` and **Pathway B (`CSV/XLSX`)** on `jpa_scholarships.xlsx`.

## 2. Part B Preparation Comparison & Justification (Step B3)

| Pathway / Approach A | Pathway / Approach B | Observed Evidence | Selected Approach & Trade-Off |
| :--- | :--- | :--- | :--- |
| **Pathway A:** Fixed-size slicing (`fixed-char-500`) vs. Paragraph-aware chunking (`paragraph-aware-char-500`) on `jpa_scholarships.pdf` | **Pathway B:** Raw Excel (`7 rows × 12 cols`) vs. Trimmed & deduplicated table (`prepared_table.csv`) on `jpa_scholarships.xlsx` | In `jpa_scholarships.pdf` (22 pages, 28,914 chars), both methods yielded `58` chunks because `FPDF` outputs single `\n` breaks; paragraph-aware splitting preserves logical separation where double breaks exist. In `jpa_scholarships.xlsx`, `0` duplicates were removed and whitespace was trimmed across all 12 columns while preserving the 7 JPA schemes (`PPN`, `LSPM`, `JKPJ`, `PIDN`, `DB40`, `PIL`, `PPF`). | **Selected:** `paragraph-aware-char-500` (`prepared_text.jsonl`) + `remove_duplicates=True, trim_text=True` (`prepared_table.csv`). **Accepted Limitations:** (1) Basic PDF splitter sets `page: None` instead of preserving page numbers; (2) `FPDF` `latin-1` encoding replaces Unicode symbols (`≤`/`≥`) with `?` and splits long URLs. |

## 3. Architectural Pipeline Rationale (Step C3)
- **ETL vs. ELT Decision:** **ETL (Extract-Transform-Load)** was selected so scraped HTML content from `penajaan.jpa.gov.my` is parsed, cleaned, and validated into structured tabular (`prepared_table.csv`) and chunked JSONL (`prepared_text.jsonl`) formats with SHA-256 checksums before loading into the vector store (`NFR2`, `RAI4`).
- **Batch vs. Streaming Rationale:** **Batch ingestion** was selected because JPA sponsorship schemes (PPN, LSPM, JKPJ, PIDN, DB40, PIL, PPF) are updated periodically per intake cycle rather than via real-time streams (`NFR4`).
- **Prepared Storage Locations:** `data/processed/prepared_text.jsonl`, `data/processed/prepared_table.csv`, and `data/week3_manifest.json` (Part B); `data/pipeline_v1/documents/prepared_text.jsonl` and `data/pipeline_v1/tabular/prepared_table.csv` (Part C).

## 4. Stage Contracts & Failure Handling

| Stage | Contract / Evidence | Failure Response |
| :--- | :--- | :--- |
| **Ingest** | `jpa_scholarships.pdf` and `jpa_scholarships.xlsx` exist under `data/raw` and load cleanly. | Raise `FileNotFoundError` / `ValueError` and stop execution. |
| **Inspect** | PDF yields `22` pages / `28,914` readable chars; Excel yields `7` rows × `12` columns with required keys (`programme_code`, `source_url`). | Stop or flag empty extraction / missing required columns. |
| **Prepare** | Paragraph-aware splitter produces `58` non-empty chunks; tabular cleaner trims strings without dropping valid rows. | Raise `ValueError` if output rows/chunks equal zero. |
| **Validate** | Output files exist, are non-empty, and SHA-256 fingerprints match across reruns (`Cell 9`). | Halt execution on missing file, empty output, or checksum mismatch. |
| **Record Lineage** | Source paths, `source_sha256`, `prepared_sha256`, and UTC timestamps written to `week3_manifest.json` and `pipeline_manifest.json`. | Stop and report missing source-to-output mapping. |