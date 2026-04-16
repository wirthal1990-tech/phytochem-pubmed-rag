# Phase 1 — PMID Collection

This module collects all PubMed article IDs (PMIDs) for each compound in the
Ethno-API dataset that has a PubChem CID.

## Inputs

- Ethno-API Parquet file: `ethno_api_v2.x.x.parquet`
- Compounds with non-null `pubchem_cid` (currently ~57,757 records)

## Process

For each compound with a CID:
1. Fetch all synonyms from PubChem PUG REST
2. For each synonym: search PubMed via E-utilities
3. Collect all unique PMIDs across all synonyms
4. Download abstract text for each unique PMID
5. Write one `.txt` file per compound: `data/abstracts/{cid}.txt`

## Rate Limiting

Both PubChem and PubMed enforce rate limits.
- PubChem: 5 req/s (no key) / 100 req/s (with key)
- PubMed: 10 req/s (no key) / 100 req/s (with NCBI API key)

Register for a free NCBI API key at:
https://www.ncbi.nlm.nih.gov/account/

## Status

Not yet implemented. POC validation on 5 pilot compounds first.
