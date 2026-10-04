# Project Scratch

Use this directory for uncertain or in-flight project-only notes. Keep one topic per file; put daily activity in `memory/notes/` and the canonical development plan in `docs/PLAN.md`.

## Open Notes

- [m1-context-brief.md](m1-context-brief.md) (2026-10-03): M1 plan defects, decisions D1–D9 for Sushrut, proposed engine design, verified analytic test values, placeholder car/bus parameters, datasets/licences, tooling. Promote adopted items to `decisions.md` and `docs/PLAN.md`.
- [ui-platform-facts.md](ui-platform-facts.md) (2026-10-03): verified facts on Flutter maps, Python-in-Flutter (serious_python/Flet 1.0), Pyodide speed, traffic-sim UI precedents and schema-driven knobs, gathered during a rubberduck session on the UI stack. No stack chosen.
- [population-sim-facts.md](population-sim-facts.md) (2026-10-03): verified facts on person/household-based simulators (MATSim, BEAM, POLARIS, ActivitySim…), sampling, owner-driver fleets, Indian population/vehicle-ownership data, synthetic population methods. Scope and name undecided.
- [city-scale-facts.md](city-scale-facts.md) (2026-10-03): verified facts on simulating ~4 lakh concurrent vehicles: precedents, Python acceleration, hybrid micro/meso, routing, Indian city scale, rendering/recording.
- [direction-2026-10-03.md](direction-2026-10-03.md) (2026-10-03): Sushrut's evolving direction from the rubberduck session (desktop-only, 4 lakh, population-based, household questions, proposed rename). Not yet in PLAN/decisions.
- [household-first-formulation.md](household-first-formulation.md) (2026-10-03): current formulation, which Sushrut liked: household-first population simulator answering commute burden, family-gets-a-car, and everyone-on-PT questions; meso city traffic + transit capacity first, micro engine later for focus areas. Rename and commute-burden definition pending.
- [commute-burden-facts.md](commute-burden-facts.md) (2026-10-03): income sources (PLFS 2025, HCES 2023-24), per-mode trip and ownership costs as of Oct 2026 (incl. free-bus schemes for women), burden metric definitions, value of time, equity reporting.
- [any-region-data-facts.md](any-region-data-facts.md) (2026-10-03): open data for building synthetic people from any map snip (buildings, population grids, POIs, jobs, income proxies, schedules, life-course templates, basemaps, layer design, licence and India map-law traps).
- [year-validation-facts.md](year-validation-facts.md) (2026-10-03): which year to simulate (2025 aligns best), independent official check targets, validation without circularity, representing a year with weighted day types, Indian calendar and rain facts.
- [govdata-catalog.md](govdata-catalog.md) (2026-10-04): `govdata/` folder (no-republishing rules) and the official-portal survey behind `govdata/catalog.yaml` (104 datasets): reachability, terms, ward-level Census tables, manual steps for Sushrut, next actions.
- [education-and-timing-facts.md](education-and-timing-facts.md) (2026-10-04): state education stages, entry ages, 2025 board pass rates, AISHE GER and college directory, academic calendar, and the yearly timing of jobs, births, vehicle purchases and marriages.
- [life-events-facts.md](life-events-facts.md) (2026-10-04): defaults and sources for vehicle acquisition (VAHAN no-CAPTCHA endpoints; household share of registrations; archive-based active fleet), births, deaths, marriage, migration, moves, household splits, labour transitions, wage events, retirement; MoSPI terms for foreign users.
- [engine-ui-architecture.md](engine-ui-architecture.md) (2026-10-04): proposal: Python simulation (numpy arrays, multiprocessing, decide/resolve/commit days, keyed RNG), NiceGUI UI with one deck.gl map component, runner process connected by queues and run-folder files. Open: how the map works in NiceGUI.

## Routing

- Promote confirmed findings to `memory/learnings.md`, decisions to `memory/decisions.md`, notable runs to `memory/runs.md`, and status/next actions to `memory/index.md`.
- Keep scratch-only captures here until a result is promoted; use the local `scratch` and `organise-scratch` skills.
