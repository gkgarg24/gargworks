---
title: "Endax — Energy Data Explorer"
date: 2026-02-01
link: "https://endax.gargworks.com"
summary: "U.S. energy data platform with 115 REST endpoints, AI chat, data stories, and analytics dashboards. Go + DuckDB, single binary."
---

**Live:** [endax.gargworks.com](https://endax.gargworks.com)

A platform for exploring U.S. energy data from the [EIA API v2](https://www.eia.gov/opendata/) (Energy Information Administration). The raw API is functional but not easy to browse interactively — Endax puts a searchable, visual interface on top of it.

---

### What it does

Browse and filter data across 10 energy categories — electricity, natural gas, petroleum, coal, CO2 emissions, nuclear, international, STEO forecasts, total energy, and SEDS. Each category has dropdown filters and full-text search across 5,188 data series using BM25 ranking.

**Analytics endpoints** — year-over-year changes, state rankings, multi-state comparisons, trend detection, anomaly detection (Z-score), change point detection (binary segmentation). CSV/JSON export on everything.

**Data Stories** — 8 guided narratives (29 steps) with interactive charts. Energy transition, emissions reckoning, forecast analysis, and more. Each story walks through a question with data-driven answers.

**Dashboards:**

- **Energy Transition** — fuel mix evolution, renewable penetration by state over time
- **Sector Analysis** — consumption breakdown by residential, commercial, industrial, transportation
- **Environmental Impact** — CO2 trends, sector breakdown, fuel analysis, state rankings
- **Forecast Analysis** — STEO energy forecasts and trend projections

**AI Chat** — floating widget on every page. Natural language queries against energy data via Gemini, with inline data tables, follow-up suggestions, and query caching.

**Semantic Search** — cosine similarity over pre-computed embeddings for glossary and series discovery.

**Auth** — JWT + bcrypt, refresh tokens, freemium gating (5/day anonymous, 100/day authenticated).

---

### How it's built

The data pipeline:

```
EIA API → fetch (JSON) → convert (Parquet) → load (DuckDB) → endax (REST API) → Web UI
```

Each step is a separate Go binary. In production:

```
Internet → Cloudflare (dual-stack) → Caddy (HTTPS 443) → endax (HTTP 8080) → DuckDB
```

The server is a single binary with DuckDB embedded — no separate database process. All frontend assets (25 pages) embedded via `embed.FS`. The DuckDB FTS extension is bundled so there's no internet access needed at runtime.

**Key decisions:**

- **DuckDB** — OLAP-optimized, native Parquet support, runs in ~80MB RAM. Handles 939K records across 10 category tables.
- **Parquet** as the intermediate format — ~10x smaller than JSON, fast bulk loading, reusable across schema changes.
- **Category-specific tables** — each energy category has different dimensions (state, sector, country), so separate tables give better query performance and type safety.
- **FTS extension** — BM25 ranking over 5,188 series across 10 tables.
- **Web Components** — 8 custom elements (`<category-page>`, `<trend-chart>`, `<story-navigator>`, `<ai-chat>`, etc.). ES modules throughout, Deno for lint/fmt/check.
- **ECharts** — all dashboards and story visualizations.
- **Dual AI providers** — GenKit and GenAI SDK behind a common `aiservice.Service` interface. Gemini 3.1 Flash-Lite.
- **IPv6-only EC2** — Cloudflare proxies IPv4 traffic. Saved $3.75/mo by dropping the public IPv4.

---

### Running costs

- Domain: ~$10/year (Cloudflare)
- EC2 t4g.micro: ~$3.50/month (IPv6-only)
- SSL, DNS, CDN: free (Cloudflare + Let's Encrypt via Caddy)
- AI: ~$0 (Gemini free tier covers current usage)

---

### Stats

- 115 REST endpoints
- 25 interactive pages
- 939,086 records across 10 categories
- 8 data stories (29 steps)
- 4 analytics dashboards
- 122 tests (32 unit + 90 DB)

---

**Stack:** Go 1.26 · DuckDB · Apache Parquet · AWS EC2 ARM64 · Caddy · Cloudflare · ECharts · Gemini AI
