---
date: 2026-10-03
work_date: 2026-10-03
project: traffic-simulator
node: macbookpro
note_kind: daily
node_type: macos
device: macbookpro
server:
timezone: Europe/Oslo
repo_path: /Users/sushrutpatwardhan/1Projects/traffic-simulator
branch: master
commit:
sync_status: draft
source_format: markdown
tags: [traffic-simulation, india, mixed-traffic, openstreetmap]
---

# 2026-10-03

## Intent

- Create a uv project for a SUMO-like simulator built independently, using open-source references. Focus on Indian cities and support OSM plus a feasible Google Maps integration. Initialize project memory and write a detailed development plan.

## Work Done

- Created `/Users/sushrutpatwardhan/1Projects/traffic-simulator` with `uv init --app --package --python 3.12 --vcs git --build-backend uv --no-workspace --author-from none` and a project description.
- Added README, scaffold status CLI, and ignored local credentials/data/outputs. Created the local environment and `uv.lock`; no simulation dependencies or components were implemented.
- Wrote canonical [development plan](../../docs/PLAN.md), [architecture](../../docs/architecture.md), [map-provider design](../../docs/map-providers.md), and [open-source reference/licence notes](../../docs/open-source-references.md).
- Created one marked memory block in `AGENTS.md`, regular `CLAUDE.md` importing it, structured memory, scratch/integration indexes, and five matching memory skills under both `.agents/skills/` and `.claude/skills/`.
- Initialized knowledge-base workstream `traffic-simulator` and registered the macbookpro path. Project Git remains on unborn `main`, with no staged files, commits, or remote.
- Rubberduck session on UI stack, scale and scope. Sushrut set: desktop-only (macOS + Windows), about 4 lakh concurrent vehicles, population-based simulation, and household questions (commute burden, family gets a car, everyone uses public transport). He liked a household-first ordering. Captured in `../scratch/direction-2026-10-03.md` and `../scratch/household-first-formulation.md`, with evidence notes `ui-platform-facts.md`, `city-scale-facts.md` and `population-sim-facts.md`.
- Moved 36 third-party PDFs/HTML/text files that research agents had downloaded into the repo root out to the session scratchpad before publishing (copyright; not project files).
- At Sushrut's request: renamed the unborn branch to `master`, created the public GitHub repo https://github.com/Blazkowiz47/population-simulator (remote `origin`, SSH), and pushed the initial commit. The local directory, Python package and CLI are still named `traffic-simulator`. No software licence is declared yet.
- Gathered M1 context: plan audit plus verified research on IDM/integration, engine semantics (SUMO/A/B Street), Python tooling, Indian vehicle parameters and datasets, and KB recall. Read-only for code/docs. Summary in [m1-context-brief](../scratch/m1-context-brief.md).

## Experiments / Runs

- Command/config: `uv sync`; `uv run --locked traffic-simulator`.
- Dataset: None.
- Output path: `.venv`, `uv.lock`; CLI output only.
- Result: Both commands passed with CPython 3.12.12; the CLI explicitly reports that the engine is planned.
- Next action: Implement the M1 synthetic straight-road fixture and its numerical/accounting checks.

## Analysis Results

- OSM is suitable for persistent vector-network preparation. Google requires a distinct integration contract; its standard APIs do not supply an interchangeable regional road/building graph. The Google presentation path is planned and not configured.
- Provider/source evidence is preserved in the documentation. No API credentials, paid requests, map downloads, simulations, or calibrated city presets were created.
- M1 context: the plan has 11 fixable defects (time-step rule conflict PLAN.md:141 vs architecture.md:56; M1 scope split four ways; unanchored `data/` ignore; benchmark sizes 10,000 vs 5,000; undefined acceptance terms; unrecorded GPL-3.0 licence of movsim/traffic-simulation-de). With plain IDM and bounded braking, a stopped leader first seen within about 80 m violates bounds or overlaps; place blockages ≥150 m from entry and active from t=0. Details in the scratch brief.

## Learnings

- See `../learnings.md` and the source-linked design documents.

## Decisions

- Independent engine, Indian mixed-traffic focus, uv package, and provider separation are recorded in `../decisions.md`.

## Blockers

- None for M1. First pilot location, local measurements, own software licence, Google account terms/access, and validation thresholds are still open.

## Next

- Deliver a deterministic synthetic fixture with vehicle footprints, basic longitudinal motion, entry/exit queues, saved metrics, analytic checks, and a benchmark before importing real roads.
- First get Sushrut's choices on M1 decisions D1–D9 in `../scratch/m1-context-brief.md` (time-step policy, halving tolerance, bounds semantics, runtime deps, initial commit), then fix the plan defects in `docs/`.
