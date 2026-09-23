# NSW Planning Compliance Engine

An evidence-oriented engineering system for turning fragmented NSW planning and regulatory material into structured, address-aware development controls and reviewable assessment outputs.

## The problem this addresses

Planning feasibility is difficult to answer from a single document. A property can be affected by a combination of planning instruments, former-council DCP material, mapped constraints, development standards, overlays, exceptions, dates, and site-specific facts. The relevant rule is often not a simple keyword: it may be conditional, numeric, hierarchical, spatial, versioned, or applicable only to a particular development type.

That creates an underserved gap between:

- unstructured government documents and maps; and
- the structured, source-backed controls that a planner, designer, analyst, or software system needs in order to review a proposal.

This project explores that gap as a software-engineering problem. It is not presented as a legal-advice product or as a guarantee that a development is compliant.

## What is distinctive here

The project does not treat an LLM response as the compliance result. It uses AI-assisted extraction as one part of a wider pipeline, then adds deterministic structures and checks around it.

### 1. Regulatory documents become inspectable controls

The Python processing and extraction code represents rules with fields such as control type, numeric value, unit, source document, clause/section reference, extracted text, method, and confidence. The SQL migrations model regulatory provisions, development controls, document versions, applicability, review queues, and source relationships.

The intended result is not merely “the model says 6 metres.” It is a control that can be traced back to the material from which it was extracted and reviewed in context.

### 2. AI is bounded by deterministic logic

The repository contains explicit handling for:

- numeric and unit-aware control extraction;
- compound and conditional constraints;
- arithmetic checks and phased ledgers;
- control-type and schema validation;
- source-value and quote-gap checks;
- applicability and subject-mismatch controls;
- fail-closed behavior when required evidence or identity is unavailable.

This is the important engineering boundary: generative extraction can help interpret difficult regulatory language, but it is not allowed to silently substitute for deterministic validation or human review.

### 3. Provenance is part of the data model and UI

The compliance routes and enhanced property interface expose source document, section/clause information, extraction method, source text grounding, and review/confidence state where available. Tests cover provenance links, source values, citation fields, and quote gaps.

The design goal is that a reviewer can ask “which provision produced this result?” rather than accepting an opaque answer.

### 4. Property identity and spatial context matter

The system includes address identity checks, NSW Planning Portal and ArcGIS-oriented clients, property constraints, precinct/boundary migrations, environmental and spatial service integrations, and geospatial test surfaces.

That reflects a practical reality: a rule can be correct in isolation and still be wrong for the property if the parcel, council area, zone, instrument, or spatial overlay has been misidentified.

### 5. The repository treats regulatory change as an engineering concern

The schema and migrations include document/version tracking, effective dates, review queues, extraction hashes, data-source health checks, and audit-trail structures. The test suite includes fidelity gates, count-drop guards, schema gates, contract drift checks, golden fixtures, and regression-oriented controls.

The system therefore demonstrates work on the less visible part of regulatory AI: keeping structured outputs reviewable as source material and extraction logic change.

## Demonstrated scope

The current public source contains real implementation across:

- NSW planning document and DCP extraction;
- former Ashfield, Leichhardt, and Marrickville planning contexts in the application surfaces;
- LEP/DCP and development-control representations;
- setback, height, FSR, parking, open-space, and related control paths;
- conditional and compound constraints;
- address and property identity;
- ArcGIS and planning-data integration points;
- environmental, climate, flood, satellite, and other spatial service boundaries;
- compliance APIs and Next.js interfaces;
- PostgreSQL-oriented schema and migrations;
- AI/LLM extraction and semantic-processing integrations;
- Python tests, TypeScript application code, contract tests, golden fixtures, and data-quality gates.

These are code and test surfaces, not a claim that every integration is available from a clean public clone or that every workflow is production-deployed.

## Architecture

```text
Authoritative documents / spatial services / property data
                         |
                         v
        extraction, parsing, normalization, identity checks
                         |
                         v
     structured provisions, controls, applicability, provenance
                         |
              +----------+----------+
              |                     |
              v                     v
  deterministic calculations   AI-assisted interpretation
  and fail-closed gates        with source grounding
              |                     |
              +----------+----------+
                         v
          reviewable API and application responses
                         |
                         v
       tests, audit records, versioning, review queues
```

Repository areas:

```text
app/                    Root Next.js application and API routes
lib/                    TypeScript regulatory and application logic
services/               Python regulatory, data, geospatial, and API services
src/                    Python models and processing utilities
migrations/             SQL schema and regulatory-data migrations
tests/                  Unit, integration, contract, control, and golden tests
frontend-nextjs/        Separate larger Next.js frontend application
```

The root application and `frontend-nextjs` are separate Node applications.

## How AI is used

AI/semantic processing is used where regulatory language is difficult to reduce to a simple pattern—for example, relationships, conditional rules, and source-grounded extraction. The code contains integrations and processing paths involving LangExtract, AutoSchemaKG, RAG-style processing, and provider SDKs.

The surrounding system provides the more important safeguards:

1. identify the relevant property and regulatory context;
2. extract structured candidate provisions;
3. retain source text and document references;
4. validate schema, units, ranges, applicability, and relationships;
5. run deterministic arithmetic and rule checks;
6. expose method/confidence/review state;
7. fail closed or require review where required evidence is missing.

This reduces—but does not eliminate—the risk of extraction error or hallucination. No accuracy or hallucination-prevention guarantee is claimed.

## Testing and evaluation approach

The test suite is not only endpoint testing. It includes tests for:

- regulatory extraction fidelity and completeness;
- source and provenance integrity;
- control subject mismatches and quote gaps;
- numeric units and arithmetic constraints;
- applicability provenance and document identity;
- fail-closed behavior;
- schema and contract drift;
- golden scenarios and field-coverage ratchets;
- satellite/spatial integration boundaries;
- intelligence-brief and assessment contracts.

Some verification tests intentionally require a configured database or external data. They remain in the repository as evidence of the engineering approach, but a clean public checkout is not represented as a complete reproducible production environment.

## Practical local run

### Prerequisites

- Git
- Node.js 18 or newer
- Python 3.8 or newer
- A virtual-environment-capable Python installation

The following commands are for PowerShell. On macOS/Linux, use `source .venv/bin/activate` instead of the PowerShell activation command.

### Root application

```powershell
git clone https://github.com/lmcdo/nsw-planning-compliance-engine.git
Set-Location nsw-planning-compliance-engine

npm install
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt

npm run dev
```

Open <http://localhost:3005>.

If `py` is unavailable, use `python -m venv .venv`.

### Separate frontend

In another terminal:

```powershell
Set-Location nsw-planning-compliance-engine\frontend-nextjs
npm install
npm run dev
```

Open <http://localhost:3003>.

### Low-dependency verification

From the repository root:

```powershell
python -m compileall -q services src enrichment migrations tests
python -m pytest --collect-only -q
```

Selected tests can be run after installing Python dependencies:

```powershell
python -m pytest tests/test_constraint_arithmetic.py -q
python -m pytest tests/test_controls_provenance.py -q
```

## What full workflows require

The public checkout can start the application shells, but meaningful end-to-end workflows require route-specific infrastructure, usually including:

- PostgreSQL or Supabase with relevant migrations applied;
- regulatory documents and processed data;
- provider credentials for enabled AI/LLM services;
- mapping, property, NSW planning, ArcGIS, satellite, climate, or flood APIs as applicable;
- local configuration for storage, email, rate limiting, or other enabled services;
- appropriate Python and system dependencies for document/geospatial processing.

No credential-bearing environment file or private production dataset is included. Configure only the variables needed for the route being exercised, and never commit local secrets.

Example route probe, when the required data is available:

```powershell
curl "http://localhost:3005/api/compliance/setbacks?address=123%20Main%20St%20Ashfield%202131&semantic=true"
```

For POST routes, inspect the corresponding handler under `app/api/` before constructing a request. Payloads and dependencies are route-specific.

## What this project is—and is not

This repository demonstrates the design and implementation of a source-grounded regulatory information and decision-support system. It is especially relevant to engineering work involving document intelligence, structured extraction, AI control layers, provenance, geospatial data, and evaluation.

It is not:

- a substitute for a planning, legal, building, surveying, or other qualified professional;
- a guarantee that a proposal is compliant;
- proof that extracted data is current or complete;
- a claim of production scale, customer adoption, accuracy, or performance;
- a turnkey public SaaS deployment.

Regulatory content changes and must be checked against current authoritative sources.

## Publication boundary

This is a curated public subset of a larger private development repository. The private repository contains additional development history, operational material, data, and environment-specific work that is intentionally not published here.

See [SECURITY.md](SECURITY.md) for the publication boundary and handling expectations.

## License

This project is licensed under the MIT License; see [LICENSE](LICENSE).
