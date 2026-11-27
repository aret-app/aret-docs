# ADR-0018: Weather radar overlay and rain along the route

- **Status:** Proposed
- **Date:** 2026-11-27
- **Deciders:** @balditommaso, @lorenzopieri99
- **Tracking:** aret-app/aret-docs#31 · implementation aret-app/aret-backend#26, aret-app/aret-backend#27,
  aret-app/aret-ios#19

## Context
In the mountains, the weather decides the day. Hikers want to see **where it's raining** and how the rain
has been moving, on the same map as their route. More useful still: **will I meet rain on this route, at
my pace?** For example: "rain likely from km 4, around 14:30".

Sites like meteoeradar.it show exactly this kind of radar map. That one belongs to WetterOnline (the
"Weather & Radar" app) and is served from their own servers: a commercial product, not data we may reuse.

Two pieces of data are needed:
1. **Radar:** observed rain every 5 minutes over the last hours, for the overlay and its animation.
2. **Forecast:** expected rain in short steps (15 min) for the next hours, at the places and times the
   hiker will reach.

## Options considered
**Radar**
1. **Protezione Civile national radar mosaic** (radar.protezionecivile.it): the official Italian product,
   every 5 minutes, from the national radar network.
   - Open data, but its licence and the way to download it legitimately must be confirmed. Its API refuses
     non-browser clients (HTTP 403); the department publishes a downloader that uses an official websocket
     (`pcm-dpc/DPC-Radar-data-downloader`).
2. **RainViewer API:** worldwide radar tiles every 5 minutes, the last 2 hours. Free, no key, credit
   "Weather data by RainViewer" required.
   - Its terms describe it as for "personal, educational, and small-scale community use", with no uptime
     guarantee.
   - Its nowcast frames were empty when checked (2026-11-27).
3. **Commercial providers** (WetterOnline, Meteomatics, Tomorrow.io…): paid. Rejected for now.

**Forecast**
1. **Open-Meteo:** precipitation per point in 15-minute steps.
   - Free for **non-commercial** use under 10,000 calls a day; commercial use needs a paid plan.
   - The model used for Italy, and the credit, are to be confirmed.
2. **Our own nowcast from radar frames** (extrapolating the rain's movement, e.g. with the open-source
   pysteps): good for 30–60 minutes ahead, but real work. Possible later.

## Decision
- **Free sources only for now:**
  - **Radar:** the **Protezione Civile mosaic** if its licence and access check out, otherwise
    **RainViewer**. Compare both over the Apuane first: radar beams are blocked by mountain ridges, so
    coverage there matters more than anywhere.
  - **Forecast:** **Open-Meteo**. Aret is free and open source, so the non-commercial tier fits; if Aret
    ever earns money, revisit this ADR.
- **Fetched by the backend on a schedule, never by each phone**, as in ADR-0017:
  - **weather-svc** pulls radar every 5 minutes and a forecast grid over the region every 15–60 minutes,
    caches them, and serves the app. External services see one fetch per interval, however many users.
- **Rain along the route** (aret-app/aret-backend#27):
  - Arrival time at points along the route comes from the route's walking time (aret-app/aret-maps#5),
    scaled by the hiker's **pace factor**.
  - The pace factor is computed **on the phone** from the user's own tracks, so the API only sees the
    factor, never the tracks.
- **Wording is a safety rule:** "rain likely", "storm risk", with the forecast's time; **never "you'll stay
  dry"**. Summer storms in the Apennines form within minutes, and no forecast can rule them out.
- **Credit** to every source, shown wherever its data appears.

## Open items (aret-app/aret-backend#26)
- Protezione Civile radar: licence (CC BY-SA 4.0?), legitimate download method, format and projection.
- RainViewer: whether caching and serving its tiles through our CDN fits its terms; whether nowcast
  returns.
- Open-Meteo: model used for Italy, data licence and credit, expected calls per day.
- Radar quality over the Apuane and Garfagnana for both radars, on a few rainy days.

## Consequences
- A new backend service, weather-svc, with a fetcher on a schedule and a cache, like the map pipeline.
- The app gets a radar overlay and a route rain warning (aret-app/aret-ios#19).
- No cost for the alpha. A cost appears only if Aret becomes commercial (Open-Meteo's paid plan) or if we
  need an uptime guarantee.
- Weather is real-time data, so unlike routes and places (ADR-0017) it can't be built weekly. The cache
  must be fresh, and the app must show how old the data is.
