# Proposed architecture and validation

Status: design proposal, 2026-10-03. The project scaffold and memory exist; the components described here have not been implemented, benchmarked, calibrated, or validated. Milestones and acceptance gates belong in [PLAN.md](PLAN.md). All numerical settings below are starting candidates to test, not measured capabilities.

## Scope and design choices

Build an independently implemented traffic engine for experiments on Indian urban roads, starting with one observed junction or short corridor. Bengaluru, Mumbai, and Hyderabad are target settings; they do not yet have project datasets or calibrated presets. The first useful result is a reproducible comparison of a baseline and an intervention under declared assumptions. Full-city forecasting remains a later research and performance problem.

The proposed core is a headless Python package. Map ingestion, demand generation, simulation, analysis, and display communicate through versioned data contracts. SUMO is an open-source design reference, not the simulation runtime or a required dependency. Its documentation establishes useful precedents for sublane movement, junction occupancy, and stochastic runs; it does not validate this engine. Source inspiration should be recorded, and copying code requires checking that source's license separately. [SUMO documentation and license](https://sumo.dlr.de/docs/).

Prefer a readable implementation before optimization. Geometry and graph libraries are reasonable infrastructure dependencies; using them does not outsource vehicle dynamics. A web interface can later control jobs and replay results without defining the physics.

## Data contracts and provider boundary

Separate network-data providers from map-display providers. A basemap supplies a visual reference; it does not automatically supply the geometry, restrictions, buildings, or demand required by a simulation. OpenStreetMap is the proposed initial network source. Google Maps integration must use permitted provider capabilities and obey the provider policy in the development plan; the engine must not assume arbitrary Google road extraction or caching is available.

Proposed interfaces are:

| Contract | Responsibility |
| --- | --- |
| `NetworkSource.fetch(region, options)` | Return permitted raw features and their source metadata. Support local files before live services. |
| `NetworkCompiler.compile(features, overrides)` | Build a normalized network, preserving provenance and producing a quality report. |
| `DemandBuilder.build(network, assumptions, seed)` | Produce passenger activities, vehicle trips, service schedules, and boundary arrivals. |
| `Engine.load(network, scenario)` / `step()` | Execute the independently implemented model without a display or live provider connection. |
| `RunWriter` / `RunReader` | Persist manifests, summaries, events, and optional trajectories in versioned formats. |
| `MapRenderer` | Present a permitted basemap; transform engine coordinates for display. |

The proposed `NetworkSnapshot` contains a projected coordinate system in metres; road surfaces and centre lines; optional lane markings; physical barriers; grade or layer separation; directed legal connections; turn movements; junction areas and conflict zones; stop lines and signal groups; stops and pickup areas; buildings and activity zones; and boundary gates. Stable IDs must retain their source references. Each inferred width, speed, access rule, or building use should retain its assumption, confidence, and override history. Unconnected roads, ambiguous crossings, missing turn restrictions, and uncertain lane widths must appear in the quality report.

A `Scenario` references immutable network and demand snapshots, their content hashes, model versions, parameters with units, simulation intervals, service date, and timezone. Indian scenarios use `Asia/Kolkata`; UTC can identify run creation. Missing values must remain distinguishable from measured values and defaults. Overrides form a separate layer so that fetching new map data does not erase curation.

Proposed serialization starts with inspectable JSON or YAML configuration and tabular outputs. Large geometry or trajectory formats can follow measured needs. A schema version and explicit migration rule are required before persisted scenarios become public interfaces.

## Roads and mixed traffic

Use continuous longitudinal and lateral position on a drivable road corridor, with vehicle length, width, heading, and speed. Lane markings express legal guidance and route preferences; they do not make a motorcycle occupy an entire lane. A narrow-vehicle model must support filtering, side-by-side movement, and gradual encroachment, while a bus should occupy its actual width. SUMO's sublane model documents these phenomena and the computational cost of finer lateral decision resolution. [Sublane model](https://sumo.dlr.de/docs/Simulation/SublaneModel.html).

A proposed first implementation uses lateral strips to find neighbours and evaluate a limited set of candidate movements, while retaining continuous positions and footprints. Strip resolution must be tested against the narrowest simulated class. It should not create artificial gaps or prevent legal coexistence merely because centres fall in different strips. Exact footprint checks are a separate final check.

Independently implement an IDM-style longitudinal baseline for acceleration and braking, then add documented lateral candidate selection and gap acceptance. The IDM authors describe speed, distance, relative speed, and per-driver parameters as the longitudinal inputs. Their illustrative parameter values are not Indian urban defaults. Standard car-following alone does not resolve crossing or head-on interactions. [Author's model description](https://traffic-simulation.de/info/info_IDM.html), [open-source reference implementation](https://github.com/movsim/traffic-simulation-de).

Separate a shared physical road surface from its legal direction graph. A wrong-way vehicle and legal oncoming traffic must query and occupy the same physical space, even when represented by different directional routes. On divided roads, barriers and carriageway separation constrain movements; proximity in map coordinates must not imply a traversable median. Bridges crossing in plan view must not create junctions or collision candidates.

## Engine update and physical accounting

The proposed tick has a fixed sequence:

1. Process scheduled arrivals, service events, and controller changes due at the current time.
2. Read a frozen state and build local neighbour and junction observations.
3. Generate each agent's desired acceleration, lateral movement, stop action, and turn entry.
4. Resolve competing movements using the declared conflict and occupancy policy.
5. Integrate motion, check swept footprints, commit the new state, and write accounting events.

Every agent should perceive the same pre-update state. Mutating one agent while another is still deciding would make iteration order an unintended behavioural assumption. Simultaneous conflicts require a documented deterministic tie rule; evaluate a rotating priority or keyed tie-breaker for persistent bias. Bounded-speed and acceleration integration must handle reaching a stop within a tick without producing negative forward speed. Swept checks prevent fast vehicles passing through each other between snapshots.

Start testing with a candidate motion interval of 0.1 seconds and a separate candidate decision interval of 0.5 seconds. Neither is accepted until halving intervals produces stable outcomes for the relevant scenarios. Separating perception or reaction timing from numerical integration avoids turning a smaller tick into an implausibly faster driver response. SUMO documents this distinction through its action-step mechanism. [Safety and action-step timing](https://sumo.dlr.de/docs/Simulation/Safety.html).

Maintain explicit conservation accounts: every demanded trip is pending, active, completed, cancelled with a reason, or failed with a reason. Pending insertion queues remain counted when a boundary cannot admit vehicles. Every passenger is assigned to an activity, waiting, travelling, completed, or otherwise explicitly accounted for. Geometry penetration, duplicate IDs, NaN state, impossible transitions, and disappearance are engine faults.

Traffic-law violations are behavioural outcomes. They may relax a red-light, keep-clear, or legal-direction decision, but must not disable geometry accounting. Initially use collision-avoiding physical resolution and log attempted conflicts and overridden motions. Such safeguards can distort aggressive behaviour, so their activation rate must be visible. Later, an explicit incident mode may permit a collision outcome and its road obstruction or removal process. An accidental footprint overlap must never silently become a simulated crash, and the initial collision-avoiding mode cannot estimate crash rates.

## Junctions, stops, and controllers

Represent movement paths and conflict zones inside junctions. Vehicles take time to cross and can stop there. A node that instantaneously transfers vehicles between links cannot reproduce junction blocking. SUMO explicitly documents this limitation and its keep-clear mechanism. [Intersection dynamics](https://sumo.dlr.de/docs/Simulation/Intersections.html).

The proposed compliant model checks downstream receiving space before entry, applies signal or priority permissions, and checks conflicting trajectories. A blocking behaviour can ignore the receiving-space rule while still respecting the physical occupancy process. Once blocked, the vehicle occupies junction space and delays conflicting movements. Do not erase deadlocks through unreported teleportation; report gridlock and any explicit recovery policy.

Begin with fixed-time controllers containing phase groups, amber, all-red clearance, minimum greens, and compatible movements. Proposed interventions include adding a signal, changing splits or offsets, moving a stop or pickup area, changing turn access, and adding local enforcement. Adaptive control follows a validated baseline. Bus dwell, taxi pickup, and auto-rickshaw boarding should occupy a roadside position or bay for a sampled duration, then interact with re-entry traffic.

Violations need separate opportunity models: entering against red, entering a blocked junction, taking a wrong-way shortcut, queue cutting, or stopping outside a permitted area. Wrong-way generation should connect plausible origins and desired destinations through a physically possible movement; randomly reversing arbitrary vehicles is a debugging scenario, not a demand model. Enforcement is a hypothesis about detection, deterrence, and response over space and time. Its effect must be calibrated or varied over declared plausible ranges.

Use stable driver attributes plus context such as accumulated delay, local obstruction, and observed enforcement. If events are modelled as a time hazard, convert rate `lambda` to tick probability `1 - exp(-lambda * dt)`. For junction-entry opportunities, sample per opportunity instead. Reusing a fixed per-tick probability while changing `dt` changes the phenomenon being simulated.

## Daily demand, boundaries, and passengers

Buildings indicate possible activity locations; footprints alone do not identify population or journeys. Begin with explicit origin-destination and time-of-day demand, then add synthetic home, work, school, shopping, and other activity schedules constrained by local evidence. Keep residents, visitors, participation rates, household vehicle ownership, mode choice, occupancy, and through traffic distinct. A population slider should expose which quantities it changes. Activity-based person schedules have an open-source precedent in MATSim. [MATSim concepts](https://www.matsim.org/).

Passenger journeys and vehicle movement are separate objects. One bus carries many passengers; taxis and autos can cruise or reposition empty; a private vehicle may carry several people. Bus routes, stops, capacities, scheduled arrivals or headways, dwell, and passenger queues therefore belong in the scenario. Where a licensed usable feed exists, GTFS provides a schedule format; otherwise use documented manual schedules. No current city feed is assumed. [GTFS reference](https://gtfs.org/documentation/schedule/reference/).

Airport events require passenger counts or assumptions, mode shares, occupancy, pickup and processing delays, workers, and vehicle staging. A flight timestamp alone is insufficient. Start with manual synthetic events and label them; connect schedule providers only after their permitted use and coverage are known.

Simulate a buffer around the analysis region and define entry and exit gates for external and through trips. Boundary arrivals must follow time-dependent demand and receiving-space constraints. Congestion just outside the selected region may affect its exits; include downstream restrictions or enlarge the network when necessary. Report buffer-size sensitivity. A congested boundary must not remove unsatisfied demand or turn into an unlimited sink that exaggerates intervention benefits.

## Reproducibility, analysis, and display

Persist a run manifest containing code revision when available, dependency lock hash, schema and model versions, inputs and hashes, parameters, seed derivation, hardware, elapsed time, and warnings. Define reproducibility first on the same recorded software and platform; cross-platform floating-point identity needs separate testing.

Use independently derived RNG streams for population, departures, vehicle profiles, route choices, service dwell, and behavioural opportunities. Stable agent or event keys prevent unrelated draws from shifting when an intervention introduces extra events. Paired comparisons should reuse realized demand and latent driver profiles across treatments. SUMO's documentation provides a precedent for separate randomness sources and repeated seeded runs. [Randomness and reproducibility](https://sumo.dlr.de/docs/Simulation/Randomness.html).

Report ensembles with paired differences and uncertainty intervals, including parameter uncertainty separately from seed variation. Begin with a small declared replication count, then increase it according to an agreed precision or ranking-stability criterion. A stable mean under many seeds does not compensate for incorrect geometry or demand.

Proposed outputs include vehicle and passenger travel time, delay against a documented free-flow reference, throughput, served and unserved demand, queue length in metres, spillback duration, stopped time, bus reliability, pedestrian delay when pedestrians exist, blocking events, and violations by exposure opportunity. Completed-trip percentiles must be accompanied by unfinished trips; otherwise gridlock can look efficient because slow trips never finish. Near-miss surrogates such as time-to-collision require explicit definitions for closing or conflicting movements and are not validated accident probabilities.

The later UI should display provenance, uncertainty, baseline differences, and queued demand alongside animation. Rendering reads snapshots at its own rate and interpolates for display. It must not determine engine timing. Scenario edits create a versioned scenario and job rather than silently changing an ongoing comparison.

## Validation and scaling gates

First validate synthetic free-flow, stopping, following, merging, side-by-side movement, signal queues, spillback, wrong-way encounters, grade separation, and deadlock scenarios. Check conservation and numerical invariants on every development run. Compare integration steps, neighbour resolution, and conflict tie rules. Comparison against an open-source simulator may reveal disagreements, but agreement is not proof of real-world validity.

For the first observed Indian corridor, audit geometry and controls, estimate demand and mode composition, then calibrate dynamics and dwell against time-aligned counts, queues, speeds, travel times, and observed behaviour. Keep different days or periods for validation and preserve the split. Parameters that compensate for missing lanes, incorrect signal phases, or unknown boundary inflow should trigger a data-quality review. Several parameter sets may fit the same aggregate counts; retain that uncertainty instead of selecting one unexplained truth.

Publish validity by component, place, time, and measurement. Testing a Bengaluru junction does not validate Mumbai-wide forecasts. An intervention's simulated benefit remains conditional on its assumed compliance response until observations support that response. Prefer interventions whose ranking persists across plausible demand, geometry, behaviour, and boundary alternatives; otherwise report an inconclusive comparison.

Profile headless runs with candidate active populations of 100, 1,000, and 5,000 vehicles, recording simulated seconds per wall second, peak memory, neighbour-query cost, and output volume on named hardware. These are benchmark sizes, not promised capacities. Use local spatial indexes and array-oriented state before evaluating Numba or a native kernel. Any accelerated implementation must reproduce declared numerical tolerances and conservation outcomes. A 24-hour run at 0.1-second motion steps requires 864,000 global updates; rendering and trajectory retention need separate budgets. Whole-city or hybrid microscopic/mesoscopic scaling follows measured performance and validated transfer of flow at model boundaries.

Outstanding decisions are the first pilot location and observations, source coverage and licenses, public API stability, parameter priors, collision-mode scope, acceptable validation errors, reference hardware, and performance targets. The architecture gives each a place to be resolved; it does not claim they are settled.
