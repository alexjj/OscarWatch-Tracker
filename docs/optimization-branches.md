# OscarWatch Performance Optimization Branches

## Overview

Tracks performance work for live satellite tracking. Status here is the source of truth relative to main; older “Ready for implementation” wording was stale after merges through 2026-09.

## Skip (rejected)

| Approach | PRs | Why |
|----------|-----|-----|
| `SatelliteTrackState` object pooling | [#112](https://github.com/magicbug/OscarWatch-Tracker/pull/112) | Pool never successfully `Return`s while the UI holds the previous snapshot; measured slightly worse |
| Snapshot `ArraySegment` / mutable buffer reuse | [#118](https://github.com/magicbug/OscarWatch-Tracker/pull/118) | Worker vs UI race; boxing; mutates snapshots the UI still reads. `ToArray()` in `LiveTrackingService.PublishSnapshot` stays as the thread-safety boundary |
| StringBuilder pooling / radio command templates / trig precompute | (planned, not pursued) | CAT is I/O-bound; Flex/RigCtl Span work already shipped; SGP4 dwarfs `Math.PI` |

## Shipped

| Item | PR / notes |
|------|------------|
| `RemoveSatellite` in-place (no LINQ) | #103 |
| Sun position cache (~30 s) | #104 |
| Stale-tracks list reuse | #116 |
| World map footprint / ground-track key buffers | #117 |
| Pass elevation timeline render caches | #113 |
| Flex `ParseKeyValues` Span | #110 |
| Sky plot ≥1 px render throttle | #41 |
| Shared `HttpClient`, settings debounce, startup parallel I/O | #37–#39 |
| Site cache, enabled-sat cache, parallel pass prediction | earlier foundations |

## Remaining / in progress

### Live UI

1. **World map 1 px throttle + cache survival**  
   Stop clearing render/label caches on every `TrackStates` rebind. Invalidate only when a subpoint moves ≥1 px, or the greyline UTC minute rolls (not every 1 s `MapDisplayUtc` tick).

2. **Timeline smart invalidation**  
   Drop blanket 1 Hz `InvalidateVisual`. Redraw when window/live time drift reaches ≥1 px on the plot.

### Tracking / predictor CPU

3. **Combined live geometry (one SGP4 per sat per tick)**  
   `IOrbitPropagator.GetLiveGeometry` + `PublicOrbitToolsPropagator` override; `TrackingOrchestrator.GetLiveStates` and polar sampling use it.

4. **Pass predictor single look-angle per coarse step**  
   `BruteForcePassPredictor.EvaluateCoarseSample` replaces separate `IsVisible` + `ElAt` SGP4 pairs.

5. **Ground-track visibility priority**  
   Staggered non-focused rebuilds prefer above-horizon satellites (still max 2 / tick, 20 ms budget).

## Implementation Guidelines

- One focused PR per theme when splitting work for review
- Functional equivalence tests plus allocation/timing checks where practical
- Keep `SatelliteTrackState` snapshot copies immutable across the worker/UI boundary
- UK English in operator-facing strings; no em dashes in prose
