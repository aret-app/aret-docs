# ADR-0004: Map rendering on iOS — MapLibre Native

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
Aret has to show its own outdoor map (trails by difficulty, hiking routes, huts, contours, hillshade) and work offline in the mountains. The prototype already ships two map engines: Apple **MapKit** (`Map` in SwiftUI, used in Explore and in route/activity previews) and **MapLibre Native** 6.31 with the free OpenFreeMap "Liberty" style (`ARETTopoMapView`, the "ARET Sentieri (beta)" mode).

## Options considered
1. **MapKit only** — zero dependencies and familiar, but it can't render our own vector tiles or offline regions, and the cartography isn't made for hiking.
2. **MapLibre Native** — open source (BSD-2), renders our own PMTiles/vector tiles with our own style, offline packs, hillshade; already integrated in the prototype.
3. **Mapbox Maps SDK** — similar capabilities, but proprietary and paid beyond the free quota.

## Decision
**MapLibre Native** is the main map engine on iOS. The prototype's **MapKit fallback stays** until our own offline maps have been tested in the field; we remove it after that, in a separate decision.

## Consequences
- The style and tiles come from `aret-maps`; the app points at our CDN instead of `tiles.openfreemap.org` once the pilot-region tiles exist (ADR-0005, ADR-0014).
- Two map code paths exist for a while; new map features are built on MapLibre only.
- OpenStreetMap attribution must stay visible on every map.
