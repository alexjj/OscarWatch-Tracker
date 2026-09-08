# OscarWatch Performance Optimization Roadmap

## Goals (live tracking sessions)

- Cut unnecessary UI paints on the 4 Hz map / 1 Hz timeline paths
- Cut duplicate SGP4 work in the 250 ms live loop and 48 h pass scan
- Keep tracking accuracy identical
- Do **not** pursue snapshot or track-state pooling (see rejected PRs #112 / #118)

## Phase A: Live UI (highest continuous CPU)

| Item | Target | Status |
|------|--------|--------|
| World map: 1 px subpoint throttle; keep render caches across snapshot rebinds; greyline minute-only time invalidate | Fewer full map paints under load | Done in this change set |
| Timeline: dirty-flag / ≥1 px time throttle instead of blanket 1 Hz invalidate | Fewer timeline paints | Done in this change set |

## Phase B: Propagation CPU

| Item | Target | Status |
|------|--------|--------|
| Combined `GetLiveGeometry` (one `PositionEci` per sat per tick) | ~3× fewer SGP4 evals on the live path | Done in this change set |
| Pass predictor: one look-angle per 30 s coarse step | ~2× fewer SGP4 evals on 48 h scans | Done in this change set |
| Ground-track rebuild priority (above-horizon first) | Fresher tracks for visible sats within the 20 ms budget | Done in this change set |

## Phase C: Docs and measurement

| Item | Status |
|------|--------|
| Refresh `optimization-branches.md` / this roadmap against merged work | Done in this change set |
| Prefer equivalence + targeted alloc tests over large pooling frameworks | Ongoing |

## Explicitly not planned

- Snapshot array pooling / `ArraySegment` publish buffers
- `SatelliteTrackState` object pools
- StringBuilder pooling across CAT transports
- Precomputed trig tables for orbit maths

## Testing notes

- Property tests for world-map and sky-plot movement thresholds
- Timeline now-line / window 1 px threshold unit tests
- `GetLiveGeometry` equivalence vs separate look/subpoint/ECI; `SatellitePositionEciCount == 1` per combined call
- Pass predictor coarse-sample equivalence; existing horizon-mask pass tests
- Ground-track rebuild priority unit tests

## Success criteria

- No functional regression in az/el, footprints, pass AOS/LOS, or greyline
- Map and timeline stay responsive with large enabled-sat sets
- Live loop does one satellite SGP4 evaluation per sat per tick for look+subpoint+ECI
