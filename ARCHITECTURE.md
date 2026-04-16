# System Architecture — phytochem-pubmed-rag

## Overview

```
Ethno-API Dataset (76,907 records)
        ↓
  [Compounds with CID]
        ↓
  PubChem PUG REST API
  (synonyms + PMIDs per compound)
        ↓
  Abstract Download
  (one .txt file per compound)
        ↓
  PGVectorRAG Index
  (PostgreSQL + pgvector)
        ↓
  Semantic Search
  (compound activity → matching abstracts)
        ↓
  Triple Relationship Table
  compound_id × activity × pmid
```

---

## Phase 1: PMID Collection

**Input:** Ethno-API Parquet file, compounds with non-null `pubchem_cid`
**Estimated scope:** ~57,757 compounds with CID

**Steps:**
1. For each compound: fetch all synonyms from PubChem (`/compound/cid/{cid}/synonyms/JSON`)
2. For each synonym: query PubMed E-utilities (`esearch.fcgi?db=pubmed&term={synonym}`)
3. Collect all unique PMIDs per compound
4. For each PMID: fetch abstract (`efetch.fcgi?db=pubmed&id={pmid}&rettype=abstract`)
5. Write one `.txt` file per compound containing all abstracts concatenated

**Rate limits:**
- PubChem: 5 req/s without API key, 100 req/s with (free registration)
- PubMed E-utilities: 10 req/s without API key, 100 req/s with (free NCBI key)

**Output format:** `data/abstracts/{cid}.txt` — one file per compound

---

## Phase 2: PGVectorRAG Semantic Search

**Tool:** PGVectorRAG (PostgreSQL 16 + pgvector extension)

**Steps:**
1. Load all `.txt` files into PGVectorRAG index (embedding per abstract chunk)
2. For each compound × activity pair from Ethno-API:
   a. Build semantic query from activity label + known synonym terms
   b. Search PGVectorRAG for top-k matching abstract chunks
   c. If relevance score > threshold: record PMID as match
3. Write matches to triple relationship table

**Embedding strategy:**
- All compound synonyms added to RAG database context
- Activity labels enriched with known semantic equivalents:
  - "antioxidant" ↔ "free radical scavenger", "oxidative stress reduction"
  - "anti-inflammatory" ↔ "NF-κB inhibition", "COX-2 suppression", "cytokine reduction"
  - (list to be expanded collaboratively)

---

## Output: Triple Relationship Table

```sql
CREATE TABLE compound_activity_evidence (
    id              SERIAL PRIMARY KEY,
    compound_id     INTEGER NOT NULL,        -- Ethno-API compound reference
    pubchem_cid     TEXT NOT NULL,           -- PubChem CID
    compound_name   TEXT NOT NULL,
    activity        TEXT NOT NULL,           -- Dr. Duke activity label
    pmid            INTEGER NOT NULL,
    article_type    TEXT,                    -- clinical_trial / in_vitro / in_vivo / review / meta_analysis / other
    title           TEXT,
    year            INTEGER,
    relevance_score FLOAT,                   -- PGVectorRAG similarity score
    validated       BOOLEAN DEFAULT FALSE,   -- manual validation flag
    created_at      TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_compound_activity ON compound_activity_evidence(pubchem_cid, activity);
CREATE INDEX idx_pmid ON compound_activity_evidence(pmid);
```

---

## Pilot Plan (5 Compounds)

Before scaling to 57K+ compounds, validate the system on 5 representative compounds:

| Compound   | CID     | Reason for selection            |
|------------|---------|----------------------------------|
| Quercetin  | 5280343 | High literature coverage, multi-activity |
| Curcumin   | 969516  | Well-documented, diverse activities |
| Resveratrol| 445154  | Extensive clinical trial record  |
| Capsaicin  | 1548943 | Distinct mechanism, clear naming |
| Berberine  | 2353    | Antimicrobial + metabolic activity |

Pilot success criteria: >80% of matched PMIDs are genuinely relevant upon manual review.

---

## Infrastructure

- **Development (POC):** Local machine with PGVectorRAG
- **Production scale:** Hetzner CCX33 (8 vCPU, 32 GB RAM, 240 GB NVMe)
  - PostgreSQL 16 + pgvector
  - Batch processing via Python scripts
  - Results exported to Parquet for Ethno-API integration
