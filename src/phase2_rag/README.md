# Phase 2 — PGVectorRAG Processing

This module processes the abstract text files from Phase 1 using PGVectorRAG
to semantically match abstracts to compound-activity pairs.

## Tool

PGVectorRAG — PostgreSQL 16 with pgvector extension.
See: https://github.com/pgvector/pgvector

## Process

For each compound in the dataset:
1. Load the compound's abstract text file (`data/abstracts/{cid}.txt`)
2. For each activity associated with this compound in Ethno-API:
   a. Build a semantic query using the activity label and known synonyms
   b. Search the PGVectorRAG index for semantically matching abstract chunks
   c. If relevance score exceeds threshold: record PMID as a match
3. Append matches to the triple relationship table

## Activity Synonym Expansion

Activity labels from Dr. Duke are not standardized. Known equivalents:
- antioxidant → free radical scavenger, oxidative stress reduction, ROS inhibition
- anti-inflammatory → NF-κB inhibition, COX-2 suppression, cytokine reduction
- antimicrobial → antibacterial, antifungal, bacteriostatic
- (list to be expanded during POC phase)

## Status

Under active development. POC testing in progress on 5 pilot compounds.
