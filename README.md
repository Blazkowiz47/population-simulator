# Population Simulator

> Direction update (2026-10-04): the project is being refocused from a traffic microsimulator to simulating people's lives (households, life events, commute burden) for any map region, validated against official statistics. The documents below still describe the earlier traffic-first plan until `docs/PLAN.md` is rewritten; see `memory/scratch/household-first-formulation.md` and `memory/decisions.md`.

A traffic simulator for Indian cities, with an independently implemented engine and open map data. The intended use is to explore daily congestion and compare traffic signals, public transport, pickup arrangements, and enforcement scenarios in places such as Bengaluru, Mumbai, and Hyderabad.

**Current state:** uv package scaffold, project memory, and development plan. The simulation engine and map integrations are planned; they are not implemented yet.

## Start here

- [Development plan](docs/PLAN.md): scope, milestones, acceptance checks, and the first implementation task.
- [Architecture](docs/architecture.md): proposed network, demand, movement, and validation design.
- [Map providers](docs/map-providers.md): OSM data import and the distinct Google Maps integration path.
- [Open-source references](docs/open-source-references.md): sources to study and licence boundaries.
- [Project memory](memory/index.md): current status, decisions, and next action.

## Local setup

Requires Python 3.12 or newer and uv. The project currently has no application dependencies.

```sh
uv sync --locked
uv run --locked population-simulator
```

The command reports the scaffold's status. It does not run a traffic simulation. Future CLI commands in the plan are proposed interfaces.

## Map support

OSM is the planned source for persistent road and building networks. Google Maps is planned as an optional display interface for independently generated simulation overlays, subject to the applicable account and service terms. The standard Google Maps APIs do not provide the same regional network import capability as OSM; see the [provider design](docs/map-providers.md).

## Project conventions

- Use uv for Python commands and dependency management.
- Keep the engine usable without a browser, API key, or live map service.
- Keep map attribution, data provenance, and assumptions with every scenario.
- Keep generated data and outputs outside Git. Add small synthetic fixtures deliberately when implementing the engine.
- Follow [AGENTS.md](AGENTS.md) for project memory. [CLAUDE.md](CLAUDE.md) imports the same instructions.

The project's software licence is still to be selected. Reference-project licences and map-data licences are recorded separately; this scaffold does not declare a licence for the new engine.
