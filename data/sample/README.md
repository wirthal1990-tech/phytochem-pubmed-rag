# Data Directory

No dataset files are stored in this repository.

## Getting the Base Dataset

The pipeline uses the Ethno-API phytochemical dataset as input.

**Option 1 — Purchase:** Available at [ethno-api.com](https://ethno-api.com)
Delivered as Parquet + JSON, ready for Phase 1 input.

**Option 2 — Collaborators:** If you are a project collaborator with access to
the private collaboration repository (`phytochem-verification-collab`), the
dataset is available there.

## Generated Data

Phase 1 generates abstract text files into `data/abstracts/`.
This directory is excluded from version control (see `.gitignore`).

Phase 2 writes results to a PostgreSQL database (not file-based).
