# ADR-0005: Tile format and hosting — tile folder on Cloudflare Pages

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
We build our own vector tiles for the pilot region (ADR-0014) and need to serve them to the app, online and for offline downloads, at **€0 and without putting a payment card on any account**. The spec proposed one PMTiles file on Cloudflare R2, but R2 requires a card even on its free tier.

## Options considered
1. **Cloudflare R2 + PMTiles** — one file, 10 GB free, no traffic fees; needs a card on file.
2. **GitHub Pages + PMTiles** — no card; max 100 MB per file, ~1 GB per site, soft 100 GB/month, and the terms discourage using Pages as an app's data CDN.
3. **Cloudflare Pages + folder of tiles (`z/x/y.pbf`)** — no card, unlimited bandwidth on a global CDN; limits of 20,000 files and 25 MB per file per deployment.
4. **Stay on OpenFreeMap** — free public tiles, but no hiking-specific data.

## Decision
Publish the pilot-region tiles as a **folder of vector tiles on Cloudflare Pages**, together with the style, fonts (glyphs) and sprites. Planetiler produces an MBTiles/PMTiles file; a script in `aret-maps` exports it to `z/x/y.pbf` and deploys it with Wrangler from GitHub Actions.

## Consequences
- The pilot region at zoom 0–14/15 is a few thousand to ~10,000 tiles, inside the 20,000-file limit. **When a region no longer fits, we revisit this ADR** (split across several Pages projects, or move to PMTiles on R2).
- MapLibre uses a normal XYZ vector source; offline packs work the same way.
- Tile compression and `Content-Type` headers must be checked (Pages `_headers` file) so MapLibre receives valid tiles.
- The app keeps using OpenFreeMap until our tiles are live.
