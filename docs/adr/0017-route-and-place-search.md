# ADR-0017: Route and place search from our own OpenStreetMap-derived copy

- **Status:** Proposed
- **Date:** 2026-11-26
- **Deciders:** @balditommaso, @lorenzopieri99
- **Tracking:** aret-app/aret-docs#27 · implementation aret-app/aret-maps#4, aret-app/aret-maps#5,
  aret-app/aret-maps#6, aret-app/aret-backend#24, aret-app/aret-ios#18

## Context
The core way to discover routes in Aret: the user types a place, e.g. *"Pania della Croce"*, and sees the
routes that reach it. Each route has its own colour on the map, plus a short comparison: length, ascent and
descent, highest point, time, difficulty, and how it differs from the others.

The route data comes from OpenStreetMap hiking relations, mostly CAI trails (ADR-0015). CAI has no public
API for its trail network; that data reaches us through OpenStreetMap.

The iOS prototype searches **live, from every phone**:
- `OnlineRouteSearchService` and `ARETPeakDiscoveryService` query the public **Overpass API**
- place names go through Apple's `MKLocalSearch`

That doesn't hold up in production:
- **Usage limits:** Overpass is a volunteer-run service, and its usage policy expects fewer than about
  10,000 queries a day. With many users we would be throttled or blocked, and it would look like abuse.
- **No comparison data:** nothing in those answers gives ascent, time or a comparison. Each phone would
  redo the route assembly, including the "never bridge a gap" safety rule.
- **Not ours to control:** results depend on services that may change, slow down or disappear. Apple's
  search doesn't know most peaks, passes or huts.

## Options considered
1. **Live third-party APIs from the app** (the prototype). Rejected for the reasons above.
2. **Proxy those APIs through our backend, with a cache.** Lowers the load, but every cache miss still hits
   Overpass, we still compute nothing ourselves, and we still depend on it at request time.
3. **Our own copy:** build routes, places and elevation from the OpenStreetMap extract on a schedule, load
   them into our database, and answer every search ourselves. **Chosen.**
4. **Ship the whole catalogue to the app.** Works offline, but it's large, hard to update, and search
   still needs a places index. Kept as a later addition: offline **region packs** built from the same data.

## Decision
- **Never call third-party services per user request**, from the app or the backend: no Overpass, Nominatim,
  Apple search or CAI websites. Their data reaches Aret only through scheduled imports: **one download per
  rebuild**, however many users we have.
- **Build pipeline in `aret-maps`**, from the daily Geofabrik Italy extract, refreshed **weekly**:
  - route catalogue: aret-app/aret-maps#4
  - elevation from a free DEM (ascent, descent, highest point, time estimate): aret-app/aret-maps#5
  - places index (peaks, passes, huts, springs, trailheads): aret-app/aret-maps#6
- **Served by places-svc** (spec §6.2, pulled forward from phase 2): places and routes in schema `places`.
  They come from the same source, share one refresh cycle, and are read-only. Each import loads new tables
  and swaps them in one transaction. aret-app/aret-backend#24.
  - `GET /v1/places?q=`: name search, tolerant of accents and typos
  - `GET /v1/routes?near=<place>`: routes that pass within ~200 m of the place first, then routes starting
    within a few km, each with the comparison fields
  - `GET /v1/routes/{osm_id}` and `…/gpx`: full geometry, elevation profile, GPX download
- **The app** shows the list and the colours, and writes the short difference line from the numbers
  (aret-app/aret-ios#18). Routes keep their OpenStreetMap relation id as a stable id.
- **Safety rules stay server-side** (ADR-0015): gaps are never bridged, difficulty comes only from tags,
  names are never invented, and time is always labelled as an estimate.

## Licensing (ODbL)
- Keeping a private copy and processing it is allowed, commercially too.
- **Produced works** (maps, route lists, a single route's GPX) need the credit **"© OpenStreetMap
  contributors"** wherever they are shown or exported.
- Offering the catalogue itself **as data**, through the API or as files, is public use of a derivative
  database: it stays **ODbL**, and we say so in the responses and the README. That is fine for an
  open-source project.
- **User-shared tours** (phase 3) are kept in separate datasets under the licence their authors choose.
  Merging them into the OpenStreetMap-derived database would make them ODbL too.
- Elevation data carries its own credit: Copernicus or INGV, depending on aret-app/aret-maps#5.

(This is our reading of the ODbL, not legal advice; to be reviewed before a public launch.)

## Consequences
- **Data freshness:** at most a week old. A fix in OpenStreetMap, such as closing a route's gap, reaches
  Aret at the next rebuild.
- **Load:** stays on our servers, and search is easy to cache. External services see one download a week.
- **Plan change:** places-svc moves into the current phase. The app removes its live Overpass and
  `MKLocalSearch` calls once the API exists.
- **Combining routes** ("126 to the pass, then 7 to the summit") is route planning with the routing engine
  (ADR-0006), later.
- **Offline:** region packs for the app can be cut from the same build later, with no new data source.
