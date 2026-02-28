---
title: "Endax — Energy Data Explorer"
date: 2026-02-01
link: "https://endax.gargworks.com"
summary: "A web app for exploring U.S. energy data from the EIA API, built with Go and DuckDB."
---

**Live:** [endax.gargworks.com](https://endax.gargworks.com)

A personal project I built to make U.S. energy data easier to explore. The data comes from the [EIA API v2](https://www.eia.gov/opendata/) (U.S. Energy Information Administration) — a public dataset covering electricity, natural gas, petroleum, coal, CO2 emissions, nuclear, and more. The raw API is functional but not easy to browse interactively, so I built a layer on top.

---

### What it does

The site lets you browse and filter data across 10 energy categories — electricity, natural gas, petroleum, coal, CO2 emissions, nuclear, international, and a few others. Each category has dropdown filters and a full-text search across ~5,200 data series using BM25 ranking.

Beyond basic browsing, there are analytics endpoints for year-over-year changes, state rankings, multi-state comparisons, and trend detection, plus CSV/JSON export. Two dashboards sit on top of that: one for energy transition (fuel mix over time, renewable penetration by state) and one for sector breakdown (residential, commercial, industrial, transportation). The database holds ~149,000 records covering 10 states across 11 years (2013–2023).

---

### How it's built

The data pipeline:

```
EIA API → fetch (JSON) → convert (Parquet) → load (DuckDB) → endax (REST API) → Web UI
```

Each step is a separate Go binary. In production, Caddy sits in front as a reverse proxy handling HTTPS, with Let's Encrypt for SSL:

```
Internet → Caddy (HTTPS 443) → endax (HTTP 8080) → DuckDB
```

The server (`endax`) is a single binary with DuckDB embedded — no separate database process, no external dependencies. The DuckDB FTS extension is bundled with the binary so there's no internet access needed at runtime.

**Key decisions:**

- **DuckDB** as the embedded database — OLAP-optimized, native Parquet support, runs in ~80MB RAM. Suitable for a t4g.micro instance.
- **Parquet** as the intermediate format between fetch and load — ~10x smaller than JSON, fast bulk loading, reusable across schema changes.
- **Category-specific tables** rather than a single generic table — each energy category has different dimensions (state, sector, country, etc.), so separate tables give better query performance and type safety.
- **FTS extension** for glossary search — BM25 ranking over 5,200 series is fast and gives better results than LIKE queries across 10 tables.
- **Vanilla JS** for the UI — no frameworks, no build step.

---

### What I learned building it

A few things that weren't obvious going in:

- DuckDB's FTS extension needs to be loaded on every connection, not just at schema init — took a while to track down.
- `scalar_subquery_error_on_multiple_rows` changed default behavior in DuckDB 1.4, which broke queries silently.
- Parquet as an intermediate format is genuinely useful — being able to re-load data without re-fetching from the API saved a lot of time during schema changes.
- Cross-compiling Go with CGO (needed for DuckDB) from macOS to Linux ARM64 has glibc version pitfalls — building on a Linux AMD64 host with an ARM64 cross-compiler is more reliable for production.

---

### Running costs

- Domain: ~$10/year (Cloudflare)
- EC2 t4g.micro: ~$7/month
- SSL, DNS, CDN: free (Cloudflare + Let's Encrypt via Caddy)

---

**Stack:** Go 1.23 · DuckDB 1.4 · Apache Parquet · AWS EC2 ARM64 · Caddy · Cloudflare
