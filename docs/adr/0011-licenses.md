# ADR-0011: Licenses and contributions — AGPL / MPL / DCO

- **Status:** Accepted
- **Date:** 2026-11-15
- **Deciders:** @balditommaso, @lorenzopieri99

## Context
Aret is open source but must stay sustainable: we don't want a company to run a closed copy of our hosted service, and the iOS app must be distributable on the App Store. The repositories were created with licenses from the spec, and the prototype (by @lorenzopieri99) is being imported into `aret-ios`.

## Options considered
1. **Permissive everywhere (MIT/Apache)** — maximum adoption, no protection against closed hosted clones.
2. **GPL for the app** — strong copyleft, but known conflicts with App Store terms.
3. **AGPL server + MPL app** — hosted modifications must be published; file-level copyleft on the app is App Store-friendly.

## Decision
| Repository | License |
|---|---|
| `aret-backend`, `aret-infra` | AGPL-3.0 |
| `aret-ios` (including the imported prototype) | MPL-2.0 |
| `aret-maps` | BSD-3-Clause (code); map data stays ODbL |
| `aret-docs` | CC-BY-4.0 |
| `.github` | MIT |

Contributions use the **Developer Certificate of Origin** (`git commit -s`). **No CLA.**

## Consequences
- Without a CLA, changing a license later needs the consent of every contributor; we accept that.
- OpenStreetMap attribution and ODbL share-alike apply to map data we publish.
