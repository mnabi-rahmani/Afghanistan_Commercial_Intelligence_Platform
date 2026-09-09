# Afghanistan Commercial Intelligence Platform (ACIP)

A continuously updated commercial intelligence system for Afghanistan that discovers Afghan businesses, collects their public commercial activity, and answers practical market questions from an accumulated dataset.

## What it does

The platform turns scattered public commercial signals — Meta ads, Facebook pages, Instagram accounts, marketplaces, and business directories — into a persistent dataset of businesses, products, offers, prices, and source history.

Example questions it is designed to answer:

- Which businesses currently advertise Jack sewing machines in Afghanistan?
- What 5 kWh lithium batteries are being sold, at what prices, and by whom?
- What agricultural machines appeared for the first time in the last 90 days?
- Which product categories are growing fastest?

## Core design

The system is organized around **commercial entities**, not platforms:

```text
BUSINESS → social identities → ads/posts → PRODUCTS → OFFERS (price, seller, location, condition)
```

Facebook, Instagram, Meta Ad Library, marketplaces, and websites are sources — not the organizing principle.

## Architecture

```text
Discovery Sources → Afghan Business Registry → Social Identity Graph
    → Collectors (Meta, Facebook, Instagram, …) → Raw Archive
    → Normalization → Entity Extraction → Afghan Commerce Graph
    → SQL Analytics + Vector Search → AI Query Agent
```

Operational stack: **Carcer** (orchestration) + **Codex** (build/repair/test).

## Tech stack

- **Database:** PostgreSQL + pgvector
- **Object storage:** S3-compatible (R2/MinIO)
- **Collectors:** Self-hosted HTTP/Playwright first; commercial providers as fallback
- **Processing:** Local OCR, regex extraction; AI only for uncertain records
- **Orchestration:** Carcer job scheduler with retries, canaries, and health checks

## Getting started

### Prerequisites

- Docker and Docker Compose
- Python 3.12+

### Local development

```bash
cp .env.example .env
docker compose up -d
```

PostgreSQL will be available at `localhost:5432`.

## Development phases

| Phase | Focus |
|-------|-------|
| 0 | Reference dataset of known Afghan businesses |
| 1 | Core database + raw archive |
| 2 | Meta Ads collector |
| 3 | Facebook public page collector |
| 4 | Instagram collector |
| 5 | Afghan business registry + scoring |
| 6 | Advertiser discovery |
| 7 | Media intelligence (OCR, phones, prices) |
| 8 | Product + offer graph |
| 9–15 | Video, comments, feed sensors, trends, AI agent, ops, dashboard |

See [docs/plan-v4.md](docs/plan-v4.md) for the full architecture and development plan.

## Repository structure

```text
apps/           API, worker, dashboard, AI query
collectors/     Source adapters (meta_ads, facebook, instagram, …)
discovery/      Keywords, categories, business resolution
processing/     Media, OCR, extraction pipelines
analytics/      Trends, price history, coverage
database/       Migrations, models, repositories
orchestration/  Carcer jobs, health checks, canaries
jobs/           Collection and processing entry points
tests/          Unit, integration, and canary tests
docs/           Architecture and planning documents
```

## Status

Early development — Phase 0–2 (core schema + Meta Ads collector) in progress.

## License

Private — all rights reserved.
