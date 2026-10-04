# Traffic Simulator Development Plan

Date: 2026-10-03  
Status: proposed implementation plan; project scaffold created  
Project: `/Users/sushrutpatwardhan/1Projects/traffic-simulator`

## 1. Purpose and confirmed direction

Build an application that imports a selected real-world region, estimates how people and vehicles move through it over a day, and lets users compare changes to signals, transport services, road use, and enforcement. The initial domain is Indian mixed traffic, especially Bengaluru, Mumbai, and Hyderabad.

The user has specified:

- An engine built from scratch, with open-source projects as references.
- OpenStreetMap and Google Maps support.
- Daily demand influenced by working hours, population, and airport flight timings where relevant.
- Multiple vehicle types, including buses, rickshaws, and taxis.
- Behaviour such as wrong-way driving, junction blocking, and roadside pickups.
- Useful comparisons of where signals or traffic-police attention might improve flow.
- A uv project and Sushrut project-memory infrastructure.

The rest of this document contains proposed engineering choices. Geographic scope, quantitative validation targets, the project's licence, and access to local observations remain unresolved. This setup creates the scaffold and plan; it does not claim a working simulator, calibrated city model, or approved Google integration.

## 2. Product scope and the first useful result

The first useful application should support one connected corridor or small neighbourhood with a few junctions. It should show a baseline and one changed scenario using the same demand, then report queue lengths, travel times, person delay, and unfinished trips. A citywide animation is a later milestone.

The first externally useful question is: **which intervention improves a selected bottleneck across plausible demand and driving-behaviour assumptions?** A forecast for tomorrow requires additional observed demand and a separate evaluation; the early product is a scenario-comparison tool.

Initial scenario controls:

| Control | Meaning in the model | Evidence or assumption needed |
|---|---|---|
| Resident population | Number and household distribution of synthetic residents | Census or aggregate local estimates; buildings alone are insufficient |
| Work and school schedules | Departure windows and activity durations | Employment locations, school locations, commute patterns |
| Travel mode | Walking, cycling, two-wheeler, car, auto, bus, taxi | Mode shares, occupancy, access to services |
| Incoming and through traffic | Journeys entering from outside the selected region | Boundary counts or explicit estimates by time and direction |
| Flights | Airport-related passenger and staff journeys | Flight events, passenger assumptions, arrival/departure offsets, mode shares |
| Public transport | Routes, stops, departures, capacity, and dwell times | Licensed schedules or a clearly labelled synthetic timetable |
| Pickups and parking | Stop locations, durations, and lost road width | Observed stopping patterns or scenario assumptions |
| Violations | Specific context-dependent actions | Observations of behaviour and its frequency; never one universal percentage |
| Signals | Location, phases, timings, and coordination | Existing control data, surveyed movements, conflict zones |
| Enforcement | Locations, time coverage, and assumed behavioural response | Local evidence or sensitivity ranges; effects remain hypotheses until tested |

Population controls in the application adjust synthetic population and travel demand. Separate residents, workers, visitors, and through travellers rather than scaling every vehicle count together.

Keep these outside the first release: photorealistic 3D, autonomous-driving perception, reinforcement-learning signal control, nationwide automatic calibration, mobile apps, and a distributed engine. Each adds a separate research or engineering burden before the basic model is validated.

## 3. Map-provider contract

The engine owns a common simulation network. Providers expose capabilities rather than interchangeable promises.

| Capability | OpenStreetMap path | Google Maps path |
|---|---|---|
| Select an area | Coordinates, polygon, or open map UI | Optional Google map UI; independently specified area coordinates |
| Import persistent road/building data | PBF/XML or bounded Overpass query | No general regional topology import promised under standard APIs |
| Run offline | Compiled network and scenario | The engine still runs; Google presentation has its own online requirements |
| Display simulation results | Open map renderer or local network view | Independently sourced overlays, after the integration terms are checked |
| Enrich routing or context | Open or independently licensed sources | Deferred; API-specific access, retention, and use rules must be reviewed |

The Google adapter must not scrape maps, trace roads or buildings, reconstruct topology by bulk route queries, or silently copy Google data into the OSM-derived scenario. Its initial planned role is presentation. The technical basis is Google's support for external GeoJSON overlays; contractual applicability still depends on the account and workflow. The [map-provider document](map-providers.md) records the standard and EEA distinctions and the implementation gates.

Import from regional OSM extracts for repeatable preparation. Preserve the extract date, checksum, source IDs, licence, geographic bounds, and every inferred attribute. Convert coordinates into an appropriate local metric coordinate system for physics; use WGS84 at display and import boundaries. Display OSM attribution and keep export provenance.

OSM may lack road widths, precise lanes, building uses, or signal phases. Create an assumption report and allow independent manual corrections. Store source values and inferred/corrected values separately. A network with unexplained guessed attributes should not pass preparation checks.

## 4. Engineering approach

Use Python 3.12+ and uv for the initial package, headless engine, data preparation, experiments, and CLI. The browser is a separate client of saved or streamed simulation state. The current scaffold has no runtime dependencies; add packages only when their milestone needs them.

Implement our own stepping, occupancy, vehicle interactions, routing, signals, demand, and output logic. SUMO is an optional external reference for independent comparison fixtures, not the execution engine. Study open-source concepts and published models; preserve attribution and licence obligations for any future code reuse. See [open-source references](open-source-references.md).

Use a microscopic model with positions and vehicle footprints across shared road surfaces. Marked lanes are guidance and legal movement information, while vehicles can occupy lateral positions between those markings. Opposing traffic must share the same physical collision/interaction space so wrong-way driving affects other vehicles instead of bypassing them in an unrelated graph edge. The [architecture proposal](architecture.md) specifies the representation and update stages.

Begin with a simple reference implementation that is easy to inspect. Profile it before choosing array layouts, Numba, Rust, or C++. Preserve one engine interface and compare accelerated components against the reference. Python is the bootstrap language; city-scale performance is a measured question.

### Proposed module boundaries

| Module | Responsibility |
|---|---|
| `network` | Import adapters, metric geometry, topology, legal movements, provenance, diagnostics |
| `scenario` | Versioned scenario files, assumptions, units, configuration validation |
| `demand` | Synthetic people, daily activities, boundary flows, modes, fleets, airport events |
| `engine` | Clock, deterministic random streams, spatial occupancy, staged state updates |
| `vehicles` | Dimensions, acceleration/braking limits, perception and movement policies |
| `behaviour` | Gap acceptance, lateral movement, queue response, individual violation mechanisms |
| `control` | Signal phases, conflict zones, stops, enforcement scenarios, incident handling |
| `routing` | Legal route plans, travel-time updates, controlled replanning |
| `metrics` | Trips, people, queues, throughput, interventions, invariants, uncertainty summaries |
| `calibration` | Parameter fitting, observations, validation splits, sensitivity analyses |
| `cli` | Preparation, checks, runs, reports, and later local preview |
| `api` / browser client | Later independent visualization and interactive scenario editing |

These directories are proposed. Do not create empty modules simply to match this table.

### Proposed dependency choices

Evaluate NumPy for numerical state, Shapely and pyproj for geometry/coordinates, and osmium for PBF streaming. Use a small graph implementation or NetworkX for initial routing experiments; measure memory before choosing a large-network implementation. Evaluate Pydantic for configuration contracts and PyArrow/Parquet when event volumes justify it. pytest and property-based tools belong to the implementation milestones that introduce the corresponding contracts.

A local browser client may use FastAPI and an open map renderer such as MapLibre. MapLibre does not itself provide tiles; choose a provider or local tiles whose licence and usage policy fit the product. Google Maps JavaScript is an optional platform integration, not an open-source source of simulation-engine code. No JavaScript project or provider SDK is needed during this setup.

## 5. Scenario and result artifacts

Keep network preparation separate from runs. A future scenario directory should contain a network artifact, demand, control policies, assumptions, and a manifest. A run records the scenario checksum and never mutates the input scenario.

Proposed artifact layout:

```text
data/scenarios/<name>/
  manifest.json          # schema version, provenance, CRS, hashes, licence
  network.json           # initial inspectable format; optimize after profiling
  population.json
  trips.json
  control.json
  assumptions.json
  observations/          # optional licensed local measurements
outputs/<run-id>/
  run.json               # resolved config, seeds, source revision, environment
  trips.jsonl
  events.jsonl
  metrics.json
  replay/                # optional sampled visualization frames
  report.html
```

Large extracts, observations, and generated outputs remain ignored by Git. Small authored synthetic fixtures belong under a future `tests/fixtures/` directory. Raw, licensed measurements must carry their own permissions and retention metadata.

Record demand identity, travel mode, vehicle occupancy, scheduled departure, actual insertion time, destination, completion state, and delay. Never hide congestion by dropping agents that cannot enter the network, teleporting trapped vehicles, or ending a run without reporting residual queues. Explicit recovery modes may exist for debugging, with separate flags and accounting.

## 6. Development milestones and acceptance gates

The sequence below is dependency driven. No delivery dates are claimed. Each milestone ends with a small demonstrable artifact and measurements that justify the next step.

### M0 — Project setup and planning

Deliver uv package, lockfile, status CLI, README, this plan, supporting design documents, and initialized project memory. Register the workstream in the knowledge base. Verify package installation and the CLI. Leave the target project unstaged and without commits or a remote.

### M1 — Synthetic reference engine

Build the scenario schema, clock, seeded state, a straight road, vehicle footprints, and a basic car-following model with dimensionally explicit parameters. Start with a deterministic hand-authored fixture rather than downloading a city. Add entry and exit queues and state snapshots.

Acceptance: vehicles conserve identity; acceleration and speed stay within configured bounds; no unexplained overlap occurs under compliant driving; stopped leaders create queues; releasing a blockage clears a finite-demand queue; repeated runs with the same environment and seed reproduce results. Compare one-vehicle motion with an analytic calculation. Repeat with a smaller time step to measure numerical sensitivity.

### M2 — OSM import and network preparation

Parse a bounded OSM extract, road direction/access, left-hand traffic, junction connectivity, turn restrictions, buildings, and relevant stops. Preserve shared roadway geometry, provenance, inferred widths, and correction overlays. Provide import and diagnostics commands.

Acceptance: a prepared small area has valid geometry, reachable origin/destination pairs, explicit disconnected components, and an assumption report. Verify access rules and turning movements against a manually reviewed sample. Cache the raw extract and reproduce the compiled network from its checksum. Network import must not contact Google services.

### M3 — Routing, intersections, signals, and queues

Add route-following, intersection connectors, conflict zones, stop/yield control, fixed-time signal phases, and downstream storage constraints. Model all-red periods and safe phase transitions. Replanning uses known travel-time estimates and an explicit update interval rather than perfect knowledge of the future.

Acceptance: incompatible compliant movements do not enter the same conflict zone; vehicles respect red phases; saturated exits create upstream spillback; route choice handles unavailable movements; baseline versus signal-change experiments use the same demand. Signal placement and signal timing are separate interventions.

### M4 — Mixed traffic and transport services

Add two-wheelers, autos, cars, buses, taxis, and pedestrians in stages. Dimensions and performance are parameter distributions, initially labelled estimates. Implement continuous lateral movement, gap acceptance, stop dwell, boarding/alighting, vehicle occupancy, pickup obstruction, and fleet reuse. Link passengers to vehicle journeys so a bus is not treated as one traveller.

Acceptance: narrow vehicles can share available width without ghosting through others; a stopping auto or bus reduces usable road space; dwell and passenger queues are observable; taxi repositioning contributes vehicle kilometres; service capacity constrains boarding. Measure person delay and vehicle delay separately. Validate each added behaviour on a small fixture before combining it with a real region.

### M5 — Daily demand and event generators

Create synthetic households/people, home-work-school activity chains, departure distributions, mode/occupancy choices, and boundary arrivals. Add airport events using passenger and staff assumptions, inbound/outbound offsets, luggage/pickup dwell, and flight disruption scenarios. Demand can cover 24 hours independently of whether every application view renders a full day.

Acceptance: generated activities yield consistent travel chains, departures match the specified time distributions, schedules conserve people, and boundary demand is accounted for. Compare counts by time and mode with whatever local observations exist. Include warm-up or initial occupancy, plus clearance/reporting after the requested window. Unfinished trips and departures delayed beyond the window remain visible.

### M6 — Violations, incidents, and enforcement experiments

Implement separate mechanisms for wrong-way travel, queue cutting, junction blocking, red-signal violations, and stopping in prohibited places. Each mechanism has an opportunity condition, exposure duration, behavioural parameters, and physical consequences. Model queue-dependent impatience as a hypothesis, not a verified universal law.

An enforcement scenario specifies staff locations, working periods, detection/visibility radius, assumed compliance response, displacement to other locations, and resource constraints. Evaluate observation-supported values and plausible response ranges, including no response. Preserve incidents and penalties in the output without confusing them with implementation failures.

Acceptance: intended violations interact with nearby vehicles in shared physical space; zero violation probability reproduces the compliant fixture; a higher parameter changes the intended behaviour mechanism; overlap/collision handling is explicit. Enforcement benefits must not be guaranteed by the model. Compare signal changes, stopping arrangements, transit improvements, and enforcement under the same demand.

### M7 — Inspectable local application

Add a scenario editor and a replay viewer backed by the headless engine. First show local network geometry and an open map basemap. Display congestion over time, queues, trips, demand waiting at boundaries, and assumptions. Present paired scenario comparisons with uncertainty and an explanation of what changed.

Reserve a Google renderer interface. Enable the Google map view only after the provider gate in [map-providers.md](map-providers.md) is satisfied. Keep that view independent of network preparation and physics. A unavailable key or quota must not prevent OSM or headless runs.

Acceptance: changing a viewport or renderer leaves simulation outputs unchanged; replays show saved results accurately; UI parameters validate units and bounds; cancel/progress/error states work; every displayed metric has an observable source. Current/proposed provider capabilities must be clear in the selector. Mock provider tests require no paid API calls.

### M8 — First calibrated Indian corridor

Choose one connected region in one of the target cities. Collect or obtain aggregate, licensed counts, vehicle composition, widths, existing signal timings, queue lengths, and travel times. Fit demand before treating driver behaviour as the explanation of every error. Freeze a calibration dataset and hold out different times or days for validation.

Acceptance: publish a reproducible scenario, parameter ledger, assumption report, calibration results, and held-out errors. Agree quantitative error targets from data quality and intended use before fitting; do not select thresholds after seeing results. If interventions have not been observed, report their effects as simulations under specified assumptions. A visually plausible replay alone does not pass this milestone.

### M9 — Performance and wider-network evaluation

Benchmark the reference engine at increasing active-agent counts, for example 100, 1,000, and 10,000, with fixed scenario duration and output settings. Record wall time, memory, agent updates per second, and seconds simulated per wall second on named hardware. Separate simulation cost from rendering and logging cost.

Acceptance: accelerated paths agree with reference fixtures within declared tolerances; memory scales within measured bounds; parameter sweeps remain reproducible. A provisional target is real-time headless execution for 1,000 active vehicles on the development MacBook, subject to its measured model complexity. Revisit this target after M1 profiling. Only then consider citywide experiments, a coarser outer-region model, or native acceleration. City-scale readiness requires new network/demand validation, not only a faster engine.

## 7. Validation and intervention comparisons

Use four levels of evidence:

1. **Implementation checks:** analytic motion, conservation, geometry, deterministic seeds, and conflict resolution.
2. **Behaviour checks:** small controlled fixtures for queues, lateral interaction, wrong-way obstruction, pickups, and bus stops.
3. **Baseline calibration:** match observed counts, compositions, queues, and travel times without tuning every mechanism at once.
4. **Held-out evaluation:** assess different times/days and, later, a second city. Intervention validity requires additional evidence of the response to that intervention.

Time-step convergence matters for acceleration, lateral movement, and collision detection. Define bounding/swept-footprint checks so a fast vehicle cannot pass through another vehicle between steps. Treat a broken physical invariant in compliant fixtures as an engine failure. If a violation scenario permits a collision or forced braking, report that event and the configured consequence rather than silently correcting it.

Initial comparison protocol: five seeds for debugging, then a proposed starting batch of twenty paired seeds for an assessment. Keep demand and vehicle identities matched across alternatives and separate random streams for demand, behavioural traits, events, and policy response. Extend the batch when uncertainty prevents a decision; a fixed number of seeds does not guarantee precision.

Vary uncertain inputs such as demand volume, mode share, pickup duration, compliance, and enforcement response. Report whether intervention rankings change across those ranges. Common random numbers help paired comparisons, but branch-dependent event consumption must not accidentally alter the demand population.

Primary outcomes:

- Mean and upper-quantile person travel time/delay, grouped by mode and time.
- Completed and unfinished trips; demand waiting to enter the region.
- Queue length/duration and junction spillback duration.
- Person throughput and vehicle throughput.
- Distance travelled, including fleet repositioning and diversion.
- Stops, harsh braking, wrong-way exposure, and conflict events under defined measures.
- Staff hours and intervention coverage, when comparing enforcement.
- Effects on neighbouring roads and boundary queues rather than only the improved junction.

Do not present an uncalibrated conflict score as a predicted accident count. Do not collapse all outcomes into one optimality score before the user chooses the tradeoffs, especially person delay versus vehicle throughput and bus users versus private vehicles.

## 8. Proposed CLI and workflow

The only current command is the scaffold status command described in the README. These future interfaces are a design proposal:

```sh
uv run population-simulator import-osm --input data/raw/region.osm.pbf --area data/area.geojson
uv run population-simulator check-scenario --scenario data/scenarios/corridor
uv run population-simulator run --scenario data/scenarios/corridor --seed 42
uv run population-simulator compare --baseline outputs/baseline --alternative outputs/intervention
uv run population-simulator serve --scenario data/scenarios/corridor
```

Preparation produces diagnostics and stops on invalid topology or unsupported schema versions. Runs resolve all inputs into a manifest before stepping. Reports read saved results. Editing an intervention creates a new scenario version rather than overwriting the baseline. A run can be reproduced without the viewer or a live provider.

## 9. Decisions still to make

| Decision | Why it matters | When needed |
|---|---|---|
| First city and region | Determines available data and the first meaningful question | Before M2 real-area selection and M8 calibration |
| First observed bottleneck | Determines which behaviour/intervention deserves fidelity first | Before M4/M6 prioritization |
| Own software licence | Determines redistribution and compatible future code reuse | Before publication or copying reference code |
| Google billing regime, product, and allowed workflow | Determines which display/context features can be activated | Before enabling the adapter |
| Observation access | Determines feasible calibration and validation | Before claiming local realism |
| Initial model distributions | Dimensions, braking, gap acceptance, pickup duration, occupancy | Before relevant behaviour fixtures |
| Evaluation targets and tradeoffs | Determines whether a model or intervention is useful | Before M8 fitting/comparison |
| Hardware and scale goals | Determines performance engineering | After reference benchmarks |

These decisions do not block the scaffold or M1. Preserve explicit unknowns rather than filling the plan with invented answers.

## 10. First implementation task

Implement M1 on a hand-authored, straight-road fixture with two vehicle sizes and a stopped-leader scenario. Create a minimal versioned schema, deterministic stepping, explicit insertion/exit accounting, and saved trip metrics. Use a compliant longitudinal model first; introduce lateral interactions only after its physical checks pass.

Deliver a CLI that runs this fixture, a replayable output, tests of conservation and analytic motion, and one benchmark. Record what the model does and what it leaves out in project memory. This gives the subsequent OSM importer a known execution contract and a way to distinguish network problems from engine problems.

## 11. Documentation and memory ownership

`docs/PLAN.md` is the canonical development plan. `docs/architecture.md`, `docs/map-providers.md`, and `docs/open-source-references.md` expand its design and source evidence. Keep proposed decisions labelled until implemented or explicitly adopted.

`memory/index.md` holds the current status and next action, `memory/decisions.md` records settled choices, and node-specific daily notes record work. Knowledge-base workstream pages are compact derived summaries and link back to the project. Use the five local memory skills in both Codex and Claude.

After implementation starts, revise milestone status from evidence: outputs, tests, benchmarks, and observations. Keep the plan readable without duplicating every daily event.
