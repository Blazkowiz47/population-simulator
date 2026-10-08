# traffic-simulator Memory

## Context Card

Status: population-simulator plan and architecture written (2026-10-04); first region Bengaluru; project on Python 3.14.8 with uv 0.12.23 (2026-10-04); scaffold, memory and government data catalogue exist; no engine yet.
Domain: personal / transport simulation
Tags: traffic-simulation, india, mixed-traffic, openstreetmap
Project path: `/Users/sushrutpatwardhan/1Projects/traffic-simulator`
Main brain workstream: `wiki/workstreams/traffic-simulator/index.md`
Devices/servers: macbookpro (development)
Latest useful result: 2026-10-04 map spike works: a NiceGUI native window with MapLibre + deck.gl rendered central-Mumbai stores, buildings and 300k synthetic people from Python-written files, with Python↔JS events in both directions (`runs.md`). Earlier: scaffold CLI verified 2026-10-03.
Current blocker: None for the first synthetic engine. Bengaluru's study area, local observations, own software licence, and Google integration access remain unresolved.
Repository: https://github.com/Blazkowiz47/population-simulator (public, branch `master`; local dir/package still `traffic-simulator`; no licence declared).
Next action: Pre-M1 decisions settled (2026-10-04): Bengaluru first, Python 3.14, uv 0.12. Sushrut to confirm the proposed milestone order in `docs/PLAN.md` §6, then start the first implementation task (PLAN §9: `region create` / `region check`, smoke test on central Bengaluru). Earlier: confirm the milestone order and settle the pre-M1 decisions (first region, Python version and uv upgrade). Earlier note: rewrite `docs/PLAN.md` and `docs/architecture.md` for the population-simulator direction once the life-event and education research lands. Earlier: choose the first city and agree the realism and validation targets for commute burden (time, money, income share, unpredictability, crowding), then promote the household-first direction (`scratch/household-first-formulation.md`) into `decisions.md` and `docs/PLAN.md`. The older M1 items below assume the micro-corridor plan. Previously: Resolve M1 decisions D1–D9 in `scratch/m1-context-brief.md` and fix the plan defects it lists, then implement M1 in `docs/PLAN.md`: a deterministic straight-road fixture with motion, insertion/exit accounting, metrics, and physical checks.

## Active Threads

- Independent microscopic engine for Indian mixed traffic, using open-source projects as references.
- OSM network import and separate optional Google Maps presentation; see provider capabilities and terms in `docs/map-providers.md`.
- Daily person/vehicle demand, public transport, airport events, violations, and conditional enforcement comparisons.

## Recent Work

- 2026-10-08: Reviewed and prepared the remaining Python 3.14/Bengaluru changes for the user-requested Git checkpoint; lock check, CLI smoke test and diff whitespace check passed. See [daily note](notes/2026-10-08-macbookpro.md).
- 2026-10-04: Locked Bengaluru as the first region and upgraded to Python 3.14.8 and uv 0.12.23 (relocked; planned stack imports on 3.14). Updated PLAN §8/§9, architecture, README and `decisions.md`. See [today's note](notes/2026-10-04-macbookpro.md).
- 2026-10-03: Created the uv project, plan, architecture, provider/reference documents, and project-memory infrastructure. See [today's note](notes/2026-10-03-macbookpro.md).
- 2026-10-04: Rewrote `docs/PLAN.md` and `docs/architecture.md` for the population-simulator direction (drafted, reviewed by three independent agents, revised). Updated README, map-providers status note and open-source references. Adopted the engine/UI design and recorded two more decisions (product direction; no republishing). Re-labelled vehicle-registration flows in `govdata/catalog.yaml` as build.
- 2026-10-04: Proposed the engine/UI design (Python multiprocessing simulation, NiceGUI UI with one deck.gl/MapLibre component, runner connected by queue and files); see `scratch/engine-ui-architecture.md`. Captured life-event, education and timing research in scratch.
- 2026-10-04: Renamed the package and CLI to `population_simulator` / `population-simulator`. Added `govdata/` (government data catalogue, 104 datasets, no-republishing rules). Decided all life events are knobs with data-backed defaults. See [today's note](notes/2026-10-04-macbookpro.md).
- 2026-10-03: Rubberduck session: desktop-only, about 4 lakh vehicles, population-based, household-first direction (scratch notes). Published the initial commit to public GitHub repo https://github.com/Blazkowiz47/population-simulator on `master`.
- 2026-10-03: Gathered and verified M1 context (plan audit, IDM/integration, engine semantics, tooling, Indian parameters, KB recall). See [m1-context-brief](scratch/m1-context-brief.md).

## Recent Runs

- See `runs.md`; no traffic experiments have run.

## Durable Learnings

- See `learnings.md`.

## Decisions

- See `decisions.md`.

## Memory Skills

- Use project-local `remember`, `recall`, `scratch`, `organise-scratch`, and `check-initialisation` skills under `.agents/skills/` or `.claude/skills/`.

## Scratch

- See `scratch/index.md` for unresolved project-local captures and in-flight notes.

## Integrations

- See `integrations/index.md`. No external memory-publishing integrations are enabled.

## Links

- [Development plan](../docs/PLAN.md)
- [Architecture](../docs/architecture.md)
- [Map providers](../docs/map-providers.md)
- [Open-source references](../docs/open-source-references.md)
- Knowledge base: `/Users/sushrutpatwardhan/Library/Mobile Documents/iCloud~md~obsidian/Documents/sushrut`
