# Population Simulator

A desktop application (macOS and Windows) that simulates the people of a chosen map region living through one year: households, work and school, daily travel, money, and life events such as buying a vehicle, changing jobs, marriage, moving house and graduating. It reports household commute burden (time, money, share of income, unpredictability and crowding) and checks simulated totals against official statistics without republishing them. Indian cities (Bengaluru, Mumbai, Hyderabad) are the first focus; any map snip should work, with realism improving where local data exists.

**Current state:** a uv package scaffold with a status command, project memory, a catalogue of government datasets (`govdata/`), and the development plan. The simulation engine, data fetching and desktop app are planned, not implemented. A throwaway spike confirmed the desktop map approach (NiceGUI window with MapLibre and deck.gl layers); its code was deleted.

## Start here

- [Development plan](docs/PLAN.md): direction, scope, data strategy, milestones, acceptance gates, decisions and the first implementation task.
- [Architecture](docs/architecture.md): data contracts, population model, daily life, life events, movement, outcomes, execution, UI and map, validation.
- [Government data](govdata/README.md): rules for official datasets (downloads stay local; only manifests and our comparisons are tracked) and the [catalogue](govdata/catalog.yaml).
- [Map providers](docs/map-providers.md): OSM data, basemaps, and the Google Maps terms analysis.
- [Open-source references](docs/open-source-references.md): projects studied, candidate dependencies and their licences.
- [Project memory](memory/index.md): current status, decisions and next action.

## Local setup

Requires Python 3.12 or newer and uv. The package has no runtime dependencies yet.

```sh
uv sync --locked
uv run --locked population-simulator
```

The command reports the scaffold's status. The CLI commands in the plan are proposed interfaces.

## Data and licences

- Government data is never republished. Downloaded files stay in git-ignored `govdata/<dataset-id>/raw/`; the repository tracks only the catalogue, per-dataset manifests and our own simulated-vs-official comparisons.
- Map and place data come from open sources (OpenStreetMap, Overture Maps) and carry their attribution and licence flags. OSM-derived databases are ODbL (share-alike).
- Synthetic people and households are labelled as synthetic; they are not real persons.
- Maps of India must use Survey of India-conformant boundaries; city views omit national boundaries.

## Project conventions

- Use uv for Python commands and dependency management; add dependencies only when a milestone needs them, and record their licences.
- Keep the engine usable without the UI, a browser, an API key or a live map service; every run can be re-run headless from its manifest.
- Keep provenance, assumptions and "as of" dates with every region, scenario and result.
- Keep generated data and outputs outside Git; add small authored fixtures deliberately.
- Follow [AGENTS.md](AGENTS.md) for project memory. [CLAUDE.md](CLAUDE.md) imports the same instructions.

The project's own software licence is still to be selected, so default copyright applies. Licences of reference projects and data sources are recorded separately.
