# Bhoomi Evidence Commons — Product Record

## Original problem statement
Build a national land governance research and policy evidence platform for SIH 2025 so researchers and policymakers can discover Indian land governance evidence, inspect citations, understand dispute patterns, and model transparent policy scenarios without pretending to predict the future.

## Architecture decisions
- React frontend with FastAPI backend and MongoDB persistence, using the provided environment configuration.
- Curated public-source evidence is the initial corpus; analytics are explicitly labeled illustrative until live LARR/DILRMP feeds are connected.
- Role-based demo login uses JWT with Public, Researcher, Official, and Admin roles.
- Simulation uses deterministic transparent formulas rather than a black-box ML model.
- AI integration is configured around GPT 5.4 with the Emergent LLM key; grounded deterministic fallback keeps the college demo reliable.

## User personas
- Researcher: asks sourced questions, opens passages, and builds a literature map.
- Official: scans dispute hotspots and models reform levers.
- Public explorer: reads the evidence corpus.
- Admin: manages the demo corpus and access model.

## Core requirements
- Citation-grounded search and document viewer.
- Dispute analytics with district detail and trend chart.
- Honest policy simulation with assumptions and source papers.
- Workspace with pinned documents, debates, and data sources.
- Responsive, simple, dynamic SIH demo experience.

## Implemented (2025-02-06)
- Built the Bhoomi evidence search, synthesis, citation drawer, analytics, simulation, workspace, and role login flows.
- Added five curated evidence records, district analytics sample records, quarterly trend data, and demo credentials.
- Added API routes for auth, search, synthesis, documents, analytics, simulation, and workspace persistence.

## Prioritized backlog
- P0: Connect verified LARR/DILRMP data source and ingest 100+ documents.
- P1: Wire GPT 5.4 streaming synthesis to the evidence chunks and add PDF export.
- P1: Add researcher annotations, saved searches, and workspace invitations.
- P2: Add real India district GeoJSON and map layers.

## P0/P1/P2 remaining
- P0: Replace illustrative analytics with validated live data before judging claims.
- P1: Add backend ingestion pipeline, semantic chunk search, and production auth hardening.
- P2: Expand workspace collaboration and public innovation portal.
