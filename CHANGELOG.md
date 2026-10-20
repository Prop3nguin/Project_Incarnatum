---
description: Release notes and version history for Acme, documenting new features, changes, and bug fixes across all releases.
---

# Changelog

All notable changes to Acme are documented here.

## v2.4.0 — 2026-02-15

### Added

- **Real-time CDC support** for PostgreSQL sources. See [[Guides/Real-time Streaming|Real-time Streaming Guide]].
- New `webhook` destination type for pushing events to HTTP endpoints.
- Pipeline-level environment variable overrides in `acme.yml`.

### Changed

- Improved error messages for connection failures. Errors now include the connector name and a suggested fix.
- The [[API Reference/Scheduler API|Scheduler API]] now returns `next_run_at` in ISO 8601 format.

### Fixed

- Fixed a race condition when running multiple pipelines targeting the same BigQuery table.
- Resolved memory leak in long-running streaming pipelines (>48h).

> [!note]
> This release requires a database migration. Run `acme migrate up` before starting the server.

---

## v2.3.0 — 2026-01-20

### Added

- [[Guides/Authentication|OAuth 2.0 authentication]] for API access.
- Support for Parquet file sources.
- `acme test` command for [[Guides/Testing Pipelines|pipeline validation]].

### Changed

- Default batch size increased from 1,000 to 5,000 rows.
- Connector health checks now run every 30 seconds (was 60).

### Fixed

- Fixed incorrect row counts in pipeline run summaries.
- S3 connector now handles eventual consistency properly.

---

## v2.2.0 — 2025-12-10

### Added

- Custom [[Core Concepts/Transforms|transform functions]] in Python.
- Pipeline dependency graphs with `depends_on`.
- Slack notification channel for pipeline failures.

### Fixed

- CSV parser now handles quoted newlines correctly.
- Fixed timezone handling in MySQL datetime columns.

---

## v2.1.0 — 2025-11-05

### Added

- Initial support for [[Core Concepts/Connectors|streaming connectors]] (Kafka, Redis Streams).
- Dark mode for the web dashboard.
- Export pipeline runs as CSV.
