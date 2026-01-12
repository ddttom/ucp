# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains the **Universal Commerce Protocol (UCP)** specification - an open standard enabling interoperability between commerce entities (platforms, businesses, Payment Service Providers, Credential Providers) to facilitate seamless commerce integrations, especially for AI agents.

Key concepts:
- **Capabilities**: Core primitives like Checkout, Order, Identity Linking, Payment Token Exchange
- **Extensions**: Optional enhancements (Discounts, Fulfillment, AP2 Mandates, etc.)
- **Transport Agnostic**: Works via REST APIs, MCP (Model Context Protocol), or A2A
- **Schema-Driven**: JSON Schema definitions with custom UCP annotations

## Repository Structure

- **`source/`**: Authoritative schema definitions with UCP annotations
  - `source/schemas/shopping/`: Core type definitions and data structures
  - `source/services/shopping/`: Service/capability definitions (e.g., `embedded.json` for Embedded Protocol)
  - `source/handlers/tokenization/`: Payment handler schemas
  - `source/discovery/`: Profile schema for capability discovery

- **`spec/`**: Generated schemas (DO NOT EDIT DIRECTLY)
  - Mirror structure of `source/` with operation-specific variants (`.create_req.json`, `.update_req.json`, `_resp.json`)
  - Generated via `generate_schemas.py`

- **`docs/`**: Documentation source files (Markdown)
  - `docs/specification/`: Protocol specification pages
  - `docs/documentation/`: Conceptual guides and explanations

- **`generated/`**: Additional generated artifacts

- **`scripts/`**: CI/build scripts (e.g., `ci_check_models.sh`)

## Schema Generation Workflow

**Critical**: The `spec/` directory is generated from `source/`. Always edit `source/` files, never `spec/` directly.

### UCP Annotations

Source schemas use custom annotations to define operation-specific requirements:
- `ucp_request`: Field behavior in requests ("omit", "optional", "required")
- `ucp_response`: Field behavior in responses ("omit")
- `ucp_shared_request`: Shared between create and update requests

### Generation Process

1. Edit JSON schemas in `source/`
2. Run: `python generate_schemas.py`
3. This generates operation-specific schemas in `spec/`:
   - `type.create_req.json`: Fields for create operations
   - `type.update_req.json`: Fields for update operations
   - `type_resp.json`: Fields for responses
4. Verify outputs match expectations

### Validation

Run `python validate_specs.py` to validate JSON/YAML format and references in `spec/`.

## Documentation Development

Built with MkDocs Material.

**Setup**:
```bash
pip install -r requirements-docs.txt
```

**Local Development**:
```bash
mkdocs serve --watch spec
# Open http://127.0.0.1:8000
```

**Build Check** (CI uses strict mode):
```bash
mkdocs build --strict
```

This command fails on warnings (broken links, missing references, etc.). Always run before submitting PRs with documentation changes.

## SDK Model Generation

When JSON schemas in `spec/` change, regenerate SDK client libraries.

**Python Pydantic Models**:
The CI checks model generation compatibility via `scripts/ci_check_models.sh`, which uses `datamodel-code-generator` with uv:
```bash
bash scripts/ci_check_models.sh
```

This generates Pydantic v2 models from schemas to verify they can be generated successfully.

## Linting and Pre-commit

**Super-linter** runs on PRs (see `.github/workflows/linter.yaml`).

**Local Setup**:
```bash
pip install pre-commit
pre-commit install
```

This runs linters automatically on `git commit`.

**Linter Configuration**:
- Markdown: `.github/linters/.markdownlint.json`
- YAML: `.github/linters/.yamllint.yml`

## Conventional Commits

PR titles **must** follow Conventional Commits format: `type: description`

Breaking changes require `!`: `type!: description`

**Common types**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

## Architecture Notes

### Embedded Protocol (EP)

Defined in `source/services/shopping/embedded.json`. Uses JSON-RPC 2.0 via postMessage for merchant-to-host communication. Method prefixes indicate capability scope (e.g., `ec.*` for embedded checkout).

The generation script (`generate_schemas.py` Pass 3) aggregates methods from this file and extension schemas to produce the full OpenRPC spec.

### Schema References

When updating schemas, be aware that `$ref` pointers in source schemas are rewritten during generation to point to operation-specific variants. The generation script handles this automatically.

### Multi-Transport Support

The same capability can be exposed via different transports:
- HTTP/REST binding
- MCP (Model Context Protocol) binding
- A2A binding
- EP (Embedded Protocol) binding

Each binding has its own specification page in `docs/specification/`.

## Common Workflows

### Adding a New Field to a Schema

1. Edit the schema in `source/schemas/shopping/types/`
2. Add appropriate UCP annotations (`ucp_request`, `ucp_response`)
3. Run `python generate_schemas.py`
4. Run `python validate_specs.py`
5. Update relevant documentation in `docs/`
6. Run `mkdocs build --strict` to verify docs
7. Check that models can be generated: `bash scripts/ci_check_models.sh`

### Adding a New Extension

1. Define schema in `source/schemas/shopping/types/`
2. Update relevant service definitions if needed
3. Generate specs: `python generate_schemas.py`
4. Create documentation page in `docs/specification/`
5. Add to `mkdocs.yml` navigation
6. Build docs: `mkdocs build --strict`

### Updating Documentation Only

1. Edit Markdown files in `docs/`
2. Test locally: `mkdocs serve --watch spec`
3. Build with strict mode: `mkdocs build --strict`

## CI/CD

- **Linter**: Runs on PRs (`.github/workflows/linter.yaml`)
- **Docs Build**: Runs on changes to docs/spec (`.github/workflows/docs.yml`)
  - Validates YAML
  - Checks Python SDK model generation if `spec/` changes
  - Builds docs with `mkdocs build --strict`
  - Deploys to GitHub Pages from main branch
- **Conventional Commits**: Validates PR titles (`.github/workflows/conventional-commits.yml`)
- **Spellcheck**: Runs on PRs (`.github/workflows/spellcheck.yaml`)

## Virtual Environment (Recommended)

```bash
virtualenv .ucp  # or python3 -m venv .ucp
source .ucp/bin/activate
pip install -r requirements-docs.txt
# Work...
deactivate
```

Prefix with `.` to exclude from version control.

## Important Links

- Main site: https://ucp.dev
- Specification: https://ucp.dev/specification/overview
- Samples: https://github.com/Universal-Commerce-Protocol/samples
- SDKs: https://github.com/orgs/Universal-Commerce-Protocol/repositories
- Conformance Tests: https://github.com/Universal-Commerce-Protocol/conformance
- Discussions: https://github.com/Universal-Commerce-Protocol/ucp/discussions

## License

Apache License 2.0
