---
title: "Signum — OIDC Identity Provider"
date: 2026-04-01
link: "https://signum.gargworks.com"
summary: "Standards-compliant OIDC/OAuth 2.0 Identity Provider with SCIM provisioning and developer tools. Java 26 + Helidon SE, built from the RFCs."
---

**Live:** [signum.gargworks.com](https://signum.gargworks.com)

An OpenID Connect / OAuth 2.0 Identity Provider built from scratch against the RFCs. No Spring Security, no Keycloak, no auth libraries — just the specs, a lightweight HTTP server, and explicit wiring.

The public-facing side is a set of developer tools for working with identity protocols.

---

### What it does

**OIDC / OAuth 2.0 core:**

- OpenID Connect Discovery
- Authorization endpoint (PKCE required)
- Pushed Authorization Requests (PAR)
- Token endpoint (auth code + refresh grant)
- Token introspection and revocation
- UserInfo endpoint
- JWKS with key rotation

**SCIM 2.0 provisioning:**

- Full CRUD for Users and Groups (RFC 7643/7644)
- Bulk operations
- Schema discovery and ServiceProviderConfig

**Developer tools** (the public UI at signum.gargworks.com):

- **OIDC Playground** — step through a live authorization code + PKCE flow
- **JWT Debugger** — decode and inspect tokens, verify signatures
- **OIDC Checker** — validate any provider's discovery document
- **Token Generator** — generate secure tokens, keys, and secrets
- **Password Analyzer** — strength checking with real crypto math
- **SCIM Validator** — validate payloads against RFC 7643/7644 schemas
- **Post-Quantum Lab** — generate, sign, and verify ML-DSA (FIPS 204) JWTs

---

### How it's built

Three-layer architecture:

```
Layer 3: Interfaces (Web UI, Tools, Playground)
Layer 2: Use-Case Profiles (tenant-specific config)
Layer 1: Standards-Compliant IDP Core
Platform: Helidon SE 4.5 + DuckDB + JDK 26
```

The IDP core implements the protocol logic — discovery, authorization, token issuance, JWKS lifecycle, SCIM. Layer 2 maps that to specific tenants (clients, users, policies). Layer 3 is the public UI.

**Key decisions:**

- **Helidon SE** over Spring Boot — no DI container, no annotation magic. Explicit routing and wiring. The entire request path is traceable without framework knowledge.
- **JDK 26 preview features** — PEM APIs (JEP 524) for key material handling, Key Derivation Functions (JEP 510) for token derivation. No Bouncy Castle needed.
- **DuckDB** — embedded, zero-config. Stores users, clients, sessions, audit log. Good enough for a single-node IDP.
- **Built from RFCs** — RFC 6749, 7636, 9126, 7662, 7009, 7643, 7644, and OpenID Connect Core. Each endpoint maps directly to spec sections.
- **Post-quantum readiness** — ML-DSA (FIPS 204) signing available alongside Ed25519 and RS256. Mostly a learning exercise, but the PQC lab makes it interactive.

---

### Infrastructure

Deployed on AWS EC2 ARM64. JDK is jlinked to a minimal runtime (~45MB) shipped with the app.

```
Internet → Caddy (443) → signum (8086) → DuckDB
```

- **EC2 ARM64** — Graviton, systemd managed
- **Caddy** — auto-HTTPS, reverse proxy
- **jlink runtime** — custom JDK image, no full JDK on server
- **3-release rollback** — deploy script keeps previous versions

---

### What I learned building it

- Writing an OIDC provider from the RFCs gives you a much deeper understanding of the protocol than using a library. The specs are well-written but the interactions between them (PAR + PKCE + refresh rotation) have subtle edge cases.
- Helidon SE's lack of magic is both its strength and its cost. You wire everything yourself, but you also understand everything.
- JDK 26's PEM APIs eliminate the need for external crypto libraries for most key operations. The API is clean and well-designed.
- SCIM is underrated as a provisioning protocol. The schema is rigid enough to be useful but flexible enough to extend.

---

**Stack:** Java 26 · Helidon SE 4.5 · DuckDB · AWS EC2 ARM64 · Caddy · Cloudflare
