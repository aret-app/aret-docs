# ADR-0003: Service granularity — Cargo workspace monorepo

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
We want independently deployable services (spec §6) but are two developers; one repository per service would multiply CI, versioning and review work.

## Options considered
1. **One repository per microservice** — strong boundaries, high overhead.
2. **Cargo workspace monorepo** — one repository, one binary and one container per service, shared crates.
3. **Single monolith binary** — simplest, but harder to split later.

## Decision
`aret-backend` is a **Cargo workspace**: `services/*` (one binary and container each) and `crates/*` (shared code). We start with **identity**, **tracks** and a **worker**; planner, places, media and social arrive with their roadmap phase.

## Consequences
- One CI pipeline and atomic changes across services and shared crates.
- Services deploy and scale separately, as containers.
- A service can still move to its own repository later if a team forms around it.
