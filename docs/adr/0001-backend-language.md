# ADR-0001: Backend language — Rust

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
The backend serves small JSON APIs (auth, sync, route requests) and must be fast, secure and cheap to host on small or free machines.

## Options considered
1. **Rust** (axum, tokio, sqlx) — memory-safe, very low memory/CPU per service, small static container images, compile-time checked SQL.
2. **Go** — simpler and fast to write, garbage-collected, slightly larger runtime footprint.
3. **Kotlin/Java or Node.js** — large ecosystems, heavier runtimes for tiny services.

## Decision
Backend services are written in **Rust** with **axum** (HTTP), **tokio** (async runtime) and **sqlx** (PostgreSQL). Proven engines we don't write ourselves (e.g. the routing engine) run as separate containers.

## Consequences
- Slower to write than Go at first; compensated by fewer runtime bugs and tiny containers that fit free-tier VMs.
- Shared crates (config, errors, auth middleware, geo utilities) live in the backend workspace (ADR-0003).
