# phytochem-pubmed-rag

**PubMed Abstract RAG Pipeline for Phytochemical Activity Evidence**

This repository contains the pipeline for building a triple-relationship dataset
that connects phytochemical compounds, their biological activities, and published
PubMed literature as evidence.

---

## What This Does

The USDA Dr. Duke Phytochemical Database documents plant-compound-activity
relationships (e.g., "Quercetin has anti-inflammatory activity"), but without
any links to primary literature. This pipeline fills that gap by:

1. Collecting all PubMed article IDs (PMIDs) for each compound with a PubChem CID
2. Downloading the abstract for each PMID
3. Using a PGVectorRAG system to match abstracts to compound-activity pairs
4. Populating a triple relationship table: `compound_id → activity → pmid`

The result is a dataset where every activity annotation is backed by at least
one citable scientific source.

---

## Triple Relationship Schema

```
compound_id  |  activity           |  pmid      |  article_type  |  relevance_score
-------------|---------------------|------------|----------------|------------------
5280343      |  anti-inflammatory  |  38291847  |  clinical_trial|  0.87
5280343      |  anti-inflammatory  |  29104342  |  in_vitro      |  0.92
5280343      |  antioxidant        |  31882432  |  review        |  0.74
```

Article types: `clinical_trial`, `in_vitro`, `in_vivo`, `review`, `meta_analysis`, `other`

---

## Architecture

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full technical design.

---

## Pipeline Phases

### Phase 1 — PMID Collection (`src/phase1_collect/`)
- Input: Ethno-API dataset (compounds with PubChem CIDs)
- Process: PubMed E-utilities API → all PMIDs per compound + synonyms
- Output: One text file per compound containing all abstracts

### Phase 2 — RAG Processing (`src/phase2_rag/`)
- Input: Text files from Phase 1
- Process: PGVectorRAG semantic search → match abstracts to activities
- Output: Matches file mapping compound × activity → PMID list

---

## Data

No dataset files are committed to this repository.
See `data/sample/README.md` for instructions on obtaining the base dataset.

---

## Development Status

| Component         | Status      |
|-------------------|-------------|
| Phase 1 — Collect | In design   |
| Phase 2 — RAG     | Proof of concept (local testing) |
| Triple table      | Schema defined |
| Scale (24K+ compounds) | Planned after POC |

---

## Collaboration

This project is developed jointly between:
- **Alexander Wirth** — Data engineering, pipeline architecture, infrastructure
- **Dominic** — Cheminformatics domain expertise, activity ontology, manual validation

---

## Related

- [Ethno-API](https://ethno-api.com) — The base phytochemical dataset this enriches
- [USDA Dr. Duke's Database](https://phytochem.nal.usda.gov/) — Primary data source
- [PubChem PUG REST](https://pubchem.ncbi.nlm.nih.gov/docs/pug-rest) — PMID retrieval
- [pgvector](https://github.com/pgvector/pgvector) — PostgreSQL vector extension
