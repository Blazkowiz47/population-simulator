# Learnings

Durable findings from this project. Keep this compact and useful for future work.

## Confirmed

- OSM provides vector snapshots for persistent network preparation. Google Maps' standard Roads APIs expose point/path operations rather than an equivalent regional road/building-network import. Exact capabilities and the applicable standard/EEA restrictions are documented with primary sources in [map-providers.md](../docs/map-providers.md).
- SUMO and OMoSim have recent public development. SiMTraM is a historical reference; its existence does not establish a maintained Indian city model. Current source/licence evidence is in [open-source-references.md](../docs/open-source-references.md).

- IDM free-road motion has exact references for δ=1 and δ=4 (from rest, u=v/v0: t=(v0/2a)[artanh u+arctan u], x=(v0²/4a)ln[(1+u²)/(1−u²)]). Ballistic update with in-step stop reproduces constant-deceleration stops exactly, and the IDM equilibrium gap exactly, at any dt. Verified numerically 2026-10-03; see [m1-context-brief](scratch/m1-context-brief.md) §4.
- Plain IDM settles exactly at s0 behind a stopped leader only if aT² ≥ 2·s0; otherwise it stops slightly inside s0. Its braking is unbounded near a sudden obstacle (placeholder car: 9.8 m/s² when first seen at 40 m), so the bounds and no-overlap criteria need safe insertion and blockage placement rules.
- Indian microscopic car-following evidence is concentrated on one Chennai corridor. The only open dataset with a clear licence is IIT Delhi/Noida drone data (Zenodo 10.5281/zenodo.17745347, CC BY 4.0). The 2014 Chennai Technion data states no licence.
- movsim/traffic-simulation-de is GPL-3.0 and movsim/movsim is GPL-3.0-or-later: reference only, never copy.

- NiceGUI 3.17.1 (MIT, Python 3.10–3.14) can host a JS map:
  - A custom component is a `ui.element` subclass with `component='x.js'` (a Vue component).
  - Python → JS: props plus `update()`, or `await run_method(...)`. JS → Python: `$emit` plus `.on(...)`.
  - npm packages need `esm=` plus a rollup build. The alternative is vendoring prebuilt deck.gl 9.4.0, `@deck.gl/json` and MapLibre 5 bundles, which needs no Node.
  - `run.cpu_bound` runs in a process pool (pickled functions); `run.io_bound` runs in a thread.
  - Verified against NiceGUI's examples on 2026-10-04; see [engine-ui-architecture](scratch/engine-ui-architecture.md).

## Likely But Needs Verification

- Mixed traffic, stopping obstruction, and junction blocking may change intervention rankings. This requires synthetic checks and local observations; no simulation evidence exists yet.

## Failed Approaches

- None recorded.

## Reusable Ideas

- Keep network provenance, demand assumptions, random streams, physics, and display separate so a scenario can be reproduced without a live map service.
