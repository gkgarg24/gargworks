---
title: "Automation & Operations Tools"
date: 2023-06-01
work: true
summary: "A collection of Go and Java tools built for identity platform operations — migrations, data analysis, provisioning, and directory sync."
---

**Charter Communications** · Developer · Go, Java

A set of tools built to support day-to-day operations on the identity platform. Each one addresses a specific operational need:

- **UID REST Client** — Go CLI for interacting with the Identity and Billing platform APIs. Handles everything from quick one-off account lookups for customer support to bulk operations across millions of records for data analysis and migrations. Uses an adaptive worker pool to scale concurrency based on the workload.

- **Voice Provisioning Tool** — Go tool for managing voice service provisioning. Includes a local DuckDB store for analyzing data sync issues between the provisioning system and the identity platform.

- **RSD Sync Tool** — Java tool that synchronizes roles for identity records between the Roles Service Directory and the Enterprise Service Directory. Uses Java virtual threads to process over 85 million records and keep the two systems in sync.
