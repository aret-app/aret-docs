# ADR-0002: Database — PostgreSQL + PostGIS

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
We store users, tracks (geometry + time + elevation), planned tours, POIs and mixed metadata, and need spatial queries ("tracks near here", bbox, length, simplification).

## Options considered
1. **PostgreSQL + PostGIS** — standard for geospatial data, JSONB for heterogeneous fields, mature replication and backups.
2. **MongoDB** — flexible documents and basic geo queries, weaker spatial functions and relational integrity.
3. **Separate stores per data type** (time-series DB, search engine…) — more systems to run for two people.

## Decision
**PostgreSQL with PostGIS** is the primary database. Large files (full-resolution tracks, photos, exports) go to **object storage**; **Valkey** handles caching and rate limits.

## Consequences
- One database technology to learn, back up and monitor.
- Each service owns its own schema and database role in a shared cluster at first (spec §7.2), so a schema can move to its own cluster later without code changes.
