---
title: "Patscape — U.S. Patent Explorer"
date: 2026-03-01
link: "https://patscape.gargworks.com"
summary: "Search, explore, and analyze 9.4M+ U.S. patent applications from the USPTO Open Data Portal. Go + DuckDB, single binary."
---

**Live:** [patscape.gargworks.com](https://patscape.gargworks.com)

A tool for searching and analyzing U.S. patent data from the [USPTO Open Data Portal](https://data.uspto.gov). The ODP API is powerful but not built for interactive exploration — Patscape puts a searchable interface on top of the bulk data.

---

### What it does

Full-text search across 9.4 million patent applications with status, date, and type filters. Results include faceted summaries (status distribution, filing years, patent types, PTAB outcomes) displayed as inline strips.

Beyond search:

- **Patent detail** — prosecution timeline, family graph, assignments, associated PTAB activity. Prosecution highlights are algorithmically extracted (milestones, metrics, duration).
- **Portfolio analytics** — filing velocity, technology distribution, prosecution success rate by assignee.
- **Citation network** — recursive traversal (forward and backward) with configurable depth.
- **PTAB** — trial search and detail (IPR, PGR, CBM), decisions, cross-linking to challenged patents. Includes a heuristic outcome model based on tech center rate, patent age, and rejection count.
- **Appeals, interferences, petitions** — separate search tabs with their own decision detail views.

The search landing page has an animated tag river built from patent terms (Snowball stemming, configurable stop words) — click a term to search.

---

### How it's built

Hybrid data strategy: bulk-load bibliographic data into DuckDB for search, cache patent detail on demand from ODP, and proxy large data (XML full text, document PDFs) without storing locally.

```
fetch   →  paginate ODP API, write raw JSON to data/raw/
load    →  read JSON from disk, upsert into DuckDB (Appender API)
server  →  read-only queries + on-demand proxy to ODP
```

Each step is a separate Go binary. The server embeds all frontend assets via `embed.FS` — one binary, no external files needed at runtime.

In production, Caddy handles HTTPS with Let's Encrypt:

```
Internet → Caddy (443) → patscape (8070) → DuckDB
```

**Key decisions:**

- **DuckDB** — handles 9.4M records well for OLAP-style queries. Full-text search over patent titles/abstracts is fast enough without a separate search engine.
- **Bulk load via Appender API** — ~20x faster than row-by-row inserts. 15 normalized tables, upsert semantics for incremental updates.
- **On-demand caching** — patent detail is fetched from ODP on first access and cached in DuckDB. Avoids storing data that may never be requested.
- **Year-slicing** — ODP has offset caps on large result sets. The fetch tool automatically slices by year to work around this.
- **Vanilla JS + Web Components** — no framework, no build step. ECharts for timeline and graph visualizations.
- **API key auth + IP rate limiting** — protects against abuse without requiring user accounts.

---

### Infrastructure

Deployed on OCI Always Free tier (ARM64 instance). Cross-compiled from macOS via `make build-linux-arm64`.

- **OCI ARM64** — free tier, more than sufficient for the workload
- **Caddy** — reverse proxy, auto-HTTPS
- **DuckDB** — ~31GB database file (9.4M patents + PTAB + appeals + decisions)
- **Domain/DNS** — Cloudflare

---

### Datasets

| Dataset | Records |
|---------|--------:|
| Patent applications | ~9.4M |
| Appeal decisions | ~163K |
| PTAB proceedings | ~19K |
| PTAB decisions | ~20K |
| Petition decisions | ~7K |
| Interference decisions | ~2K |

---

**Stack:** Go 1.26 · DuckDB · OCI ARM64 · Caddy · ECharts · Cloudflare
