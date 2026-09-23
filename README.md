# NSW Planning Compliance Engine

An engineering codebase for processing NSW planning and regulatory information into structured, source-linked data and application responses.

This is a curated public source release and engineering portfolio asset. It contains real development work across regulatory document processing, deterministic controls, AI-assisted extraction, geospatial analysis, APIs, database migrations, and automated QA. It is not legal advice and does not establish that a development is compliant.

## What is included

- Regulatory rule and document-processing code in Python and TypeScript.
- Next.js application surfaces and compliance API routes.
- Database migrations and service-layer integrations.
- Provenance, fail-closed, extraction, arithmetic, geospatial, and contract tests.
- AI/LLM integration points that require locally configured provider credentials.

The public repository intentionally does not include private credentials, production databases, private source documents, or the complete deployment environment.

## Architecture at a glance

```text
app/                    Root Next.js application and API routes
lib/                    TypeScript regulatory and application logic
services/               Python data, regulatory, geospatial, and API services
src/                    Python models and regulatory processing utilities
migrations/             SQL schema and data migrations
tests/                  Python unit, integration, contract, and control tests
frontend-nextjs/        Separate, larger Next.js frontend application
```

The root application and `frontend-nextjs` are separate Node applications. They should be installed and started independently.

## Practical local run

### 1. Prerequisites

- Git
- Node.js 18 or newer
- Python 3.8 or newer
- A virtual-environment-capable Python installation

The commands below are written for PowerShell. On macOS/Linux, replace the activation command with `source .venv/bin/activate`.

### 2. Install the root application

```powershell
git clone https://github.com/lmcdo/nsw-planning-compliance-engine.git
Set-Location nsw-planning-compliance-engine

npm install
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

If `py` is not available, use `python -m venv .venv` instead.

Start the root Next.js application:

```powershell
npm run dev
```

Open <http://localhost:3005>. The configured root development port is `3005`.

### 3. Run the separate frontend

In a second terminal:

```powershell
Set-Location nsw-planning-compliance-engine\frontend-nextjs
npm install
npm run dev
```

Open <http://localhost:3003>. The frontend expects some API, database, and service configuration depending on the page or route used.

### 4. Basic verification without private services

From the repository root, this check does not require production credentials or a live database:

```powershell
python -m compileall -q services src enrichment migrations tests
```

The repository also contains pytest tests, but the complete suite is not guaranteed to pass in a clean public checkout because many tests intentionally exercise database schemas, external APIs, regulatory datasets, or local service configuration.

To inspect the available tests:

```powershell
python -m pytest --collect-only -q
```

Run an individual test only after checking its dependencies, for example:

```powershell
python -m pytest tests/test_constraint_arithmetic.py -q
```

## What is required for full functionality

Starting a Next.js development server is possible with the public checkout, but meaningful end-to-end compliance workflows require additional services and data. The exact requirements vary by route:

- PostgreSQL or Supabase configured through environment variables, with the relevant migrations applied.
- Regulatory source documents and processed data appropriate to the workflow.
- Python packages from `requirements.txt` and, for selected workflows, geospatial/system dependencies.
- Provider credentials for any enabled AI/LLM, mapping, property, satellite, climate, email, storage, rate-limit, or payment integration.
- A local environment file created by the operator. No secret-bearing environment file is included in this public repository; do not commit one.

Common configuration names used by the code include database connection variables, `DATABASE_URL`, `PYTHON_API_URL`, `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`, and provider-specific credentials. Configure only the variables required by the route being exercised, and inspect the relevant source before enabling an integration.

There is no claim here that a fresh clone can reproduce the original private development environment or production deployment without those dependencies.

## Example API call

When the root application is running and the selected route has its required data available:

```powershell
curl "http://localhost:3005/api/compliance/setbacks?address=123%20Main%20St%20Ashfield%202131&semantic=true"
```

For POST routes, inspect the corresponding handler under `app/api/` before constructing a request. Payloads and required services are route-specific.

## Testing and engineering controls

The test tree includes coverage for regulatory extraction, provenance, source-value controls, fail-closed behavior, arithmetic constraints, data contracts, and integration boundaries. Tests that require private databases or external services are kept visible as engineering evidence but are not represented as guaranteed clean-checkout tests.

Useful commands:

```powershell
npm test
python -m pytest tests/test_controls_provenance.py -q
python -m pytest tests/test_constraint_arithmetic.py -q
```

Use `npm test` only after installing the Python dependencies; it delegates to the repository's pytest suite.

## Limitations and safety

- Regulatory content changes and must be checked against current authoritative sources.
- Extracted or classified content can be incomplete or wrong.
- External planning, property, spatial, and document sources vary in availability and quality.
- Automated results require appropriate human review and do not replace qualified planning, legal, building, surveying, or other professional advice.
- No accuracy, performance, customer, scale, or production-deployment claim is made without reproducible public evidence.

See [SECURITY.md](SECURITY.md) for the publication boundary and responsible handling expectations.

## License

This project is licensed under the MIT License; see [LICENSE](LICENSE).
