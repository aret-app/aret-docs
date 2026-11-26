# ADR-0015: Pre-built hiking routes from open data

- **Status:** Accepted
- **Date:** 2026-11-26
- **Deciders:** @balditommaso, @lorenzopieri99
- **Tracking:** aret-app/aret-docs#22 · implementation aret-app/aret-maps#4

## Context
Much of the value of apps like Komoot, Wikiloc or AllTrails is their library of ready-made routes. Those
libraries are **their proprietary databases**. Copying them, whether by scraping, bulk downloading or
re-importing their GPX exports, would breach their terms of service and the EU *sui generis* database right
(Directive 96/9/EC), regardless of any watermarks. Aret must build its route library only from sources
whose licence allows it, and for free.

The pilot region (ADR-0014) is the Alpi Apuane and the Appennino tosco-emiliano, where most marked trails
are maintained by the **Club Alpino Italiano (CAI)**.

## Options considered
1. **OpenStreetMap hiking route relations.** CAI has an official agreement with Wikimedia Italia (the Italian
   OpenStreetMap chapter), signed in 2016 and renewed in 2021 and 2024, under which its trail network is
   mapped and maintained in OSM as `type=route` + `route=hiking` relations: name, CAI number (`ref`),
   `cai_scale` difficulty, endpoints. Licence **ODbL**: free use with attribution; databases derived from it
   must stay ODbL. We already download and process this data in `aret-maps` (ADR-0005).
2. **Regional open data.** Emilia-Romagna publishes its official trail network (REER) with a **CC BY 4.0**
   licence in its INSPIRE metadata, with a WFS download service. Tuscany publishes most geographic data as
   CC-BY or CC-BY-SA, but the licence of its trail network (RET) is not confirmed yet.
3. **CAI sections and park authorities.** Many publish GPX files or PDF descriptions. These are protected by
   default: usable only with **written permission**.
4. **User-shared tours.** Aret users publish their own tours under an explicit open licence (Phase 3).
5. **Scraping or copying proprietary apps.** Rejected: illegal, and against the project's values.

## Decision
- **Start with OpenStreetMap route relations** (option 1) for the pilot region: they are complete enough,
  legally clean, already in our pipeline, and maintained by CAI itself.
- **Add regional open data per region only after checking its licence** (option 2): Emilia-Romagna first;
  Tuscany once the RET licence is confirmed in its catalogue.
- **Ask CAI sections and parks for permission** before using any of their files (option 3), case by case,
  and record each permission in this repository.
- **User-shared tours later** (option 4), with a licence chosen at publishing time.
- **Never** copy routes from proprietary apps (option 5).

## Consequences
- Route geometry comes from the member ways of each relation. **Gaps are never connected**: incomplete
  relations are kept as separate pieces and marked incomplete (the prototype's safety rule), and fixing them
  happens in OpenStreetMap, which benefits everyone.
- **Difficulty only from tags** (`cai_scale`, `sac_scale`); missing means "unknown", never guessed from
  length or climb.
- Ascent/descent and walking time need elevation data (a DEM) in the map pipeline.
- **Attribution:** "© OpenStreetMap contributors" wherever routes are shown or exported; regional data
  with its required credit (e.g. "Regione Emilia-Romagna, CC BY 4.0").
- **ODbL share-alike:** if Aret publishes the route catalogue as a downloadable database, it must be under
  ODbL. Maps and individual exported routes ("produced works") only need attribution.
- The catalogue is rebuilt with each OSM refresh, so trail fixes reach Aret automatically.

## Sources
- OSM wiki, CAI: https://wiki.openstreetmap.org/wiki/IT:CAI
- CAI at State of the Map 2020: https://pretalx.com/sotm2020/talk/CDES3T/
- Emilia-Romagna trail network metadata (INSPIRE): https://inspire-geoportal.ec.europa.eu/srv/api/records/r_emiro:2014-12-23T104300
- Tuscany RET specification: https://www.regione.toscana.it/documents/10180/492172/Specifiche_sentieristica_2.7.5_dicembre2013.pdf/478f8439-de27-4fb8-8117-e40f65fc1efc
