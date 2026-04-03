# Progress NW Catalog API — Documentation Context

## Project Overview

This directory contains the **OpenAPI 3.0.3 specification** for the **Progress NW Catalog API** — a REST API for ventilation equipment catalog at [progress-nw.ru](https://progress-nw.ru/catalog).

The API is designed to support:
- **1C synchronization** — integration with accounting systems
- **Vector search (RAG)** — Retrieval Augmented Generation for AI-powered search
- **Technical filtering** — filtering by product specifications
- **Category metadata** — hierarchical category structure with spec schemas

## Key Files

| File | Description |
|------|-------------|
| `SPEC.md` | Техническое задание (Technical Specification) — detailed requirements for the OpenAPI spec, data schemas, endpoints, validation rules |
| `PR.md` | Pull Request description — summary of what was implemented, schemas, endpoints, security gates |
| `README.md` | Project overview with quick start guide and API parameters |

## API Summary

### Base URL
```
https://progress-nw.ru/api/v1
```

### Authentication
API Key via header: `X-API-Key`

### Rate Limiting
100 requests per minute

### Response Format
JSON, UTF-8

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/products` | Product list with filtering and pagination |
| GET | `/products/{product_id}` | Single product details |
| GET | `/categories` | Category tree |
| GET | `/specs/schema/{category_slug}` | Spec schema for a category |

## Key Data Schemas

### ProductForRAG
Main product model optimized for RAG vectorization:
- `id` — unique identifier (pattern: `^[a-zA-Z0-9\-_]+$`)
- `slug` — SEO-friendly URL identifier
- `name` — product name
- `category` — CategoryRef object
- `price` — price in rubles (0 to 10,000,000)
- `searchable_text` — text for RAG vectorization (max 4000 chars)
- `specifications` — array of SpecificationItem (0-100 items)
- `media` — images and documents
- `accessories` — related products (0-50 items)
- `status` — enum: `active`, `archived`
- `seo` — SEO metadata
- `updated_at` / `created_at` — timestamps

### CategoryRef
Category reference with hierarchy support:
- `id`, `name`, `slug`, `level` (nesting level 0-5)

### SpecificationItem
Product specification with filtering support:
- `key` — unified key (pattern: `^[a-z_]+$`)
- `label` — human-readable name
- `value` — string value
- `value_num` — numeric value for comparisons
- `unit` — measurement unit
- `group` — specification group
- `is_filterable` — available for filtering

## Security Quality Gates

All string fields must have `pattern` validation:

| Pattern | Applied To |
|---------|------------|
| `^[a-z0-9\-]+$` | slug, category_slug |
| `^[a-zA-Z0-9\-_]+$` | id, product_id |
| `^[a-z_]+$` | key, group |
| `^[^<>]+$` | name, label, value, search (XSS protection) |
| `^https?://` | url, spec_schema_ref |
| `^[A-Z]{3}$` | currency |
| `^[A-Z_]+$` | error code |
| `^[a-zA-Z0-9/\-°%]+$` | unit |
| `^[^:]+:[^:]+:[^<>]+$` | spec_filter |

## Development Workflow

### Validation
```bash
swagger-cli validate swagger.yaml
```

### Build JSON from YAML
```bash
swagger-cli bundle swagger.yaml -o swagger.json -t json
```

### Git Workflow
```bash
git add swagger.yaml swagger.json && git commit -m "..."
git push
```

### GitHub Pages
- **Source:** `main` branch, root directory
- **URL:** https://andreeme.github.io/progress-api-docs/
- **Renderer:** Redoc Standalone via CDN
- **Auto-update:** 1-2 minutes after push

## Repository Structure (Expected)

```
progress-api-docs/
├── swagger.yaml      # Source OpenAPI specification (YAML)
├── swagger.json      # Compiled specification (JSON)
├── index.html        # Documentation page with Redoc
├── README.md         # Project overview
├── SPEC.md           # Technical specification
└── PR.md             # Pull Request description
```

**Note:** The `swagger.yaml`, `swagger.json`, and `index.html` files are not present in the current directory. They may need to be created or are located elsewhere.

## Publishing Process

1. Make changes to `swagger.yaml`
2. Validate: `swagger-cli validate swagger.yaml`
3. Bundle to JSON: `swagger-cli bundle swagger.yaml -o swagger.json -t json`
4. Commit changes: `git add swagger.yaml swagger.json && git commit -m "..."`
5. Push: `git push`
6. GitHub Pages will auto-deploy in 1-2 minutes

## Versioning

- API version is specified in `info.version`
- Server URL contains version: `https://progress-nw.ru/api/v1`
- Major version changes require updating the server URL

## Contact Information

- **Email:** 1c@progress-nw.ru
- **Repository:** https://github.com/AndreeMe/progress-api-docs
- **Documentation:** https://andreeme.github.io/progress-api-docs/

## Important Notes

- This is a **documentation-only** project — no source code
- The specification follows **OpenAPI 3.0.3** standard
- All specifications are in **Russian** language
- The project uses **GitHub Pages** for documentation hosting
- **Redoc** is used for rendering API documentation
- The API supports **1C integration** via `updated_from` parameter for incremental sync
