# ADR-0007: iOS persistence — GRDB (SQLite)

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
The prototype stores all recorded activities in one `activities.json` file (rewritten on every change) plus one GPX file per hike. During recording, track points live **only in memory** until the user presses stop, so a crash, an iOS memory kill or an empty battery loses the whole hike. Planned features need more than files: tens of thousands of points per track, bbox queries for "tracks near here", offline map regions, and an outbox for server sync (spec §8). An Android app may come later (Phase 4).

## Options considered
1. **Keep JSON files, fix crash-safety** — smallest change, but every save rewrites everything, no queries, and sync would need a rewrite anyway.
2. **SwiftData** — Apple's framework, little code; weaker for large point series and spatial queries, harder to control for sync, and Apple-only with no reusable schema.
3. **GRDB (SQLite)** — full SQL, migrations, WAL mode, built-in R\*Tree spatial index, observable queries for SwiftUI.
4. **Shared cross-platform core now** (Rust + UniFFI, or Kotlin Multiplatform + SQLDelight) — reusable on Android, but a second build system and a slower MVP for two developers.

## Decision
Use **GRDB** behind a persistence protocol in its own module (`AretPersistence`). Keep the **schema as plain `.sql` migration files**, so a future Android app (Room or SQLDelight on SQLite) can reuse the same schema, and the sync protocol is shared through the server's OpenAPI spec. If Android arrives, moving storage and sync into a shared core can be reconsidered then.

## Consequences
- Recording appends points to the database every few seconds (WAL, small transactions): nothing is lost on a crash.
- A one-time migration imports existing `activities.json` and GPX files on first launch.
- The database file uses `FileProtectionType.completeUntilFirstUserAuthentication`, so background recording can write while the phone is locked (the prototype's `.complete` protection blocks that).
- GPX stays as the import/export format only.
