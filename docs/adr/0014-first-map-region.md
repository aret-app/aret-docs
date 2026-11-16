# ADR-0014: First map region — Alpi Apuane + Appennino tosco-emiliano

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
The first tile build must be small enough for free storage tiers and fast to iterate on. The prototype's beta is already scoped to **Alpi Apuane** and the **Appennino tosco-emiliano** (two pilot bounding boxes in `ARETDiscoveryRegion.swift`), with 48 suggested summits and 21 sourced route descriptions there.

## Options considered
1. **Apuane + Tuscan-Emilian Apennines** — matches the prototype, the curated content and where we test in the field.
2. **Italian Alps** — larger audience, but a much bigger build and no curated content yet.
3. **Whole Alps** — several GB of tiles; too big for free tiers at this stage.

## Decision
Build the first tiles for **Alpi Apuane + Appennino tosco-emiliano**, from the Geofabrik extracts for **Toscana** and **Emilia-Romagna**, clipped to a bounding box around the two pilot areas.

## Consequences
- Small tile files (expected well under 1 GB), cheap to host and quick to rebuild.
- The bounding box is defined once in `aret-maps` and should match the app's pilot regions.
- Other regions come later, driven by users and field testing.
