# Engine, UI and how they connect (proposal)

- Date: 2026-10-04
- Node: macbookpro
- Status: **adopted 2026-10-04** (see `../decisions.md`); `docs/PLAN.md` and `docs/architecture.md` were rewritten around it. Open items: Windows/WebView2, snip drawing, offline basemap, performance at millions of points, Python 3.14 upgrade.
- Sushrut's preferences:
  - Simulation in Python, using "multiprocessing pool, queues and threading options.. i would like that".
  - Asked what is better for the UI and how to connect both.

## Simulation in Python: feasible, with conditions

- **The workload changed.** Life events are a few discrete events per person per year. Daily routines run over representative day types (weighted to the year). City traffic uses a queue/mesoscopic model; UXsim (pure Python) handled ~3.2 lakh concurrent vehicles. The old target (4 lakh vehicles, microscopic, 0.1 s) was the part Python couldn't do. See `city-scale-facts.md`.
- **Store people as arrays (numpy columns), not per-person objects.** Measured on this M3 Max: pure-Python loop ~4M simple updates/s per core; numpy-vectorised trivial update ~87M/s. A full Bengaluru is ~1.3 crore persons.
- **Processes for CPU work.** Python 3.12's GIL limits CPU threads, so use a multiprocessing pool. Threads for I/O (downloads, file writes, UI). Free-threaded CPython 3.14 (PEP 779) is a reason to move off 3.12 later.
- **macOS and Windows use the spawn start method:**
  - top-level picklable worker functions;
  - an `if __name__ == "__main__":` guard;
  - `multiprocessing.freeze_support()` in packaged apps;
  - `multiprocessing.shared_memory` for large arrays.
- **Determinism:** random draws keyed by (run seed, household, day), not by worker; results merged in a fixed order. Output must not depend on worker count or scheduling.
- **Each simulated day runs in three steps:**
  1. Parallel **decide**: households read yesterday's frozen state.
  2. One deterministic **resolve** pass for competing claims (flats, jobs, school seats, bus seats, road space).
  3. **Commit.**
- **How to split work across processes:** household chunks within a day; day types; seeds and what-if scenarios (independent); map tiles for traffic.

## UI: NiceGUI (option B), Python-first

- **Facts checked 2026-10-04:**
  - NiceGUI 3.17.1, MIT licence, Python 3.10–3.14.
  - `run.cpu_bound` runs a function in a separate process (pickled; start method set via `run.process_pool_start_method`).
  - `run.io_bound` runs a function in a thread.
  - Desktop window via native mode (pywebview).
- **Knobs** are generated in Python from the Pydantic scenario schema. **Charts:** built-in ECharts and Plotly. **Map:** one custom deck.gl/MapLibre component (JavaScript); see the open question below.
- **Not using `run.cpu_bound` for the main simulation:** a simulated year needs a long-lived job with progress and cancel, so it gets its own runner process.
- **Windows caveats** (from `ui-platform-facts.md`): check that WebView2 is present (pywebview falls back to IE silently); GPU-less VMs have no WebGL; Windows ARM64 needs the x64 build.

## Connection

```text
NiceGUI app (main process, asyncio)
 ├─ knobs (from scenario schema) · map (deck.gl, reads run-folder layers) · charts/validation
 └─ JobManager ─ submit(scenario) ─► Runner process ─► multiprocessing pool
               ◄─ progress queue (%, day, events, "layer ready") ─
               ─ cancel event ─►
Runner writes outputs/<run-id>/: run.json, population, events, daily aggregates, map layers, validation
CLI uses the same Runner → headless, reproducible from run.json
```

- **The UI and engine share only three things:** the scenario schema (inputs), the run-folder format (outputs) and small progress messages.
- **Large data never goes through the queue.** It goes to files (Parquet/Arrow) or shared memory. The map reads pre-aggregated layers.
- **Switching to option A** (a TypeScript UI) later replaces only the UI layer.

## Open question (2026-10-04)

Sushrut: "how would the map work with NiceUI? because i see that the deck.gl map would be in javascript right?" Answer (2026-10-04), proposal: NiceGUI custom component mechanism, and the Python ↔ JS data path for the map.

- **The map is one NiceGUI custom component.** Checked against NiceGUI's own examples (`examples/custom_vue_component`, `examples/node_module_integration`, 2026-10-04): a Python class subclasses `ui.element` with `component='map.js'`, and `map.js` is a small Vue component.
- **Python → JS:** set props and call `update()`, or `await self.run_method('name', args)`, which can return a value.
- **JS → Python:** `this.$emit('snip', geojson)` in JS, handled by `self.on('snip', handler)` in Python. Examples: a drawn area, a clicked hexagon, a viewport change.
- **JS libraries without Node:** vendor the prebuilt bundles. Confirmed downloadable on 2026-10-04: `deck.gl@9.4.0/dist.min.js`, `@deck.gl/json@9.4.0/dist.min.js`, `maplibre-gl@5 dist`. NiceGUI's alternative, `esm={'pkg': 'dist'}`, needs an npm and rollup build.
- **Layers declared in Python as deck.gl JSON specs** (the format pydeck emits, rendered by `@deck.gl/json`). Adding a layer is then a Python change; the JS stays a generic renderer.
- **Large data goes by HTTP, not websocket messages.** The runner writes Arrow/binary layer files to the run folder. The local NiceGUI/FastAPI server serves them, and JS fetches them and passes binary attributes to deck.gl.
- **Basemap:** an offline PMTiles extract with the MapLibre pmtiles protocol, served locally.
- **Snip drawing:** a small rectangle/polygon tool on MapLibre that emits GeoJSON to Python. Library choice not checked.
- **Alternatives considered:** `ui.leaflet` (built in, no JS) sends one message per marker, so it only fits a first prototype. lonboard/pydeck notebook widgets need an anywidget host and don't embed in NiceGUI.

## Next action

Done 2026-10-04: the map spike proved the path on macOS (see `../runs.md` and `../learnings.md`). Remaining: Windows/WebView2, snip drawing, offline basemap, scale. Then rewrite `docs/PLAN.md` and `docs/architecture.md` around the decisions in this session. Needs Sushrut's go-ahead.
