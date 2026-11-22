# Week 3 Data Inventory — AI-Finder-Scholarship

## Topic 2 Requirements Traceability (Step A1)
- **FR1 & FR2:** Require authoritative Malaysian scholarship eligibility rules (CGPA cutoffs, B40/M40 income limits, study levels, institutions, and fields of study) plus exact source URLs and document/page provenance.
- **RAI4:** Requires 100% citation presence and >= 95% source-citation match against official policy records.
- **NFR4:** Requires incremental updates when new scholarship circulars are published in under 10 minutes.

## Data Inventory Table (Step A4)

| Source ID | Filename / URL | Type and Purpose | Access / Rights | Version / Date / Owner | Inspection Evidence |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **S01** | jpa_scholarships.pdf (scraped from https://penajaan.jpa.gov.my/program-pelajar.html) | Unstructured PDF (22 pages, 28,914 chars); compiled official JPA Latihan Sebelum Perkhidmatan (LSP) guidelines for RAG text chunking (FR1, FR2, RAI4). | Public government portal data (Jabatan Perkhidmatan Awam Malaysia); permitted for educational research. | 2026 Intake Session / JPA (scraped Nov 2026) | Verified via pypdf: all 22 pages readable (28,914 chars). Observed FPDF latin-1 replacement (? for <=/>=) and mid-URL line wrapping across pages. |
| **S02** | jpa_scholarships.xlsx (sheet JPA_LSP_Scholarships, scraped from penajaan.jpa.gov.my) | Structured Excel table (7 rows x 12 columns); normalized metadata (programme_code, programme_name, bentuk_penajaan, syarat_umum, syarat_khusus, source_url) covering PPN, LSPM, JKPJ, PIDN, DB40, PIL, and PPF. | Public government portal data (JPA Malaysia); permitted for educational research. | 2026 Intake Session / JPA (scraped Nov 2026) | Verified via pandas: 7 programmes, 0 duplicates, 0 missing core fields (extra_sections null across all 7 rows). Observed truncated string "Pinjaman Boleh Ubah (PBU" on JKPJ. |