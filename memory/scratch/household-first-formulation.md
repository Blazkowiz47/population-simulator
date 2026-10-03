# Household-first formulation

- Date: 2026-10-03
- Node: macbookpro
- Repo: `/Users/sushrutpatwardhan/1Projects/traffic-simulator` (still named `traffic-simulator`; no commits)
- Source: rubberduck session in Claude Code. Session chronology: [direction-2026-10-03.md](direction-2026-10-03.md).
- Status: in-flight. Sushrut responded "i like this" to the formulation below, which ended with Claude's lean towards household-first. The household-first ordering is liked but not yet formally recorded in `decisions.md` or `docs/PLAN.md`. The rename is still pending: Sushrut did not answer the rename question.

## The questions the tool should answer (Sushrut's words)

1. Household commute burden.
2. What changes when a family has a car.
3. What changes if the entire population starts using public transport.

## Current formulation

Labels: (S) Sushrut's statement; (F) researched fact, see evidence links; (I) Claude's inference.

- **(S) Population-based.** Households own a car, or their members use a rickshaw or bus. Some people own and drive a taxi or auto as their livelihood and park it near home.
- **(I) The questions are about people.** The population and activity layer is the core of the model, not an add-on to traffic.
- **(I) Fidelity shifts away from microscopic behaviour.** Commute burden is door-to-door time (walk, wait, ride, transfer, crowding) plus cost. That depends on city-wide congestion and transit capacity, not on lateral weaving, wrong-way driving or junction blocking.
  - (F) No tool runs about 4 lakh concurrent vehicles microscopically with lateral movement on any hardware.
  - (F) Queue and mesoscopic models handle this scale on a desktop: POLARIS; MATSim QSim; UXsim (Python), which peaked at about 3.2 lakh vehicles in its network.
- **(I) "A family has a car" splits into two questions:**
  - (a) *That family's day.* It doesn't change city congestion, so replay it against baseline travel times. Cheap, and can be done for every household.
  - (b) *Many families get cars* (e.g. ownership rising from 20% to 35%). This needs a full city run.
- **(I) "Everyone on public transport" is a transit-capacity question.** Roads empty while buses overflow: denied boarding and long waits.
  - Needs bus and metro capacity, GTFS timetables and walk access.
  - The bus fleet held fixed vs. scaled up becomes a knob.
  - (F) Bengaluru CMP: 12.6 lakh peak-hour motorised trips, 47.8% by public transport; BMTC has about 7,000 buses.
  - (F) GTFS: Hyderabad TGSRTC (Open Data Telangana); Bengaluru unofficial feed (ODbL).
- **(I) "Realtime" probably no longer matters.** Scenario comparison needs faster-than-real-time batch runs; a live view is optional viewing. Sushrut has not yet said whether "realtime" meant wall-clock speed or live data.
- **(I) Open: do people adapt?** In counterfactuals, do people change route, departure time or mode? (F) MATSim does this with about 300 replanning iterations; Switzerland at a 10% sample took 47–83 h.
- **(I) Proposed order (Claude's lean, which Sushrut liked):**
  1. Synthetic population and households (with vehicle ownership and owner-driver autos/taxis).
  2. City-wide mesoscopic or queue traffic.
  3. Bus and metro capacity.
  4. Household commute-burden outputs and counterfactual knobs.
  5. Later: the microscopic Indian mixed-traffic engine for focus areas (signals, police, wrong-way, junction blocking). The current M1 micro straight-road engine design becomes this later component.

## Still undecided

- ~~Rename~~ settled 2026-10-03: GitHub repo is `population-simulator`; local folder stays `traffic-simulator` (see `decisions.md`).
- (superseded) Rename to `population-simulator` or an alternative. A rename touches:
  - 23 references in 8 files;
  - the `traffic_simulator` package and `traffic-simulator` CLI;
  - `.venv` (needs `uv sync` after a move);
  - KB `system/registry/projects.yaml` (lines 101–105) and `wiki/workstreams/traffic-simulator/`;
  - the Claude project directory.

  It is cheapest now, before any commit.
- **Definition of "commute burden":** settled in part 2026-10-03. It measures time, money and income (Sushrut: "should measure all... time, money income"). Extended the same day: it also covers unpredictability and "all the issues", and should be realistic. Implied by "all the issues" but not yet confirmed: ownership costs count alongside per-trip costs. Proposed (Claude): report the dimensions side by side, per PLAN §7, rather than as one score. Other candidate issues not yet confirmed: access walk/heat/rain exposure, transfers, safety (especially women at night), seat availability.
- **Adaptation in counterfactuals:** fixed plans, rerouting only, mode re-choice, or full replanning.
- **First city.** Bengaluru has the richest open inputs: CMP with counts, GBA ward populations, unofficial GTFS. Hyderabad has the cleanest official bus GTFS.
- **Whether junction-level questions stay** in a later phase or drop out.
- **Desktop UI:** pywebview + web UI is the strongest evidence-backed option; not yet chosen.

## Evidence

- [population-sim-facts.md](population-sim-facts.md): MATSim/BEAM/POLARIS/ActivitySim; sampling; owner-driver fleets; Census HH-14; CMPs; GTFS; synthetic population methods.
- [city-scale-facts.md](city-scale-facts.md): concurrency precedents, meso/hybrid, routing (CCH), Indian city scale, rendering/recording.
- [ui-platform-facts.md](ui-platform-facts.md): desktop stack and packaging.
- [m1-context-brief.md](m1-context-brief.md): written before this direction; still valid for the later micro component.

## Next action

Choose the first city and agree what 'realistic' must match (validation targets before fitting, per PLAN M8). Then promote the household-first ordering to `memory/decisions.md` and revise `docs/PLAN.md` milestones (M1 becomes population + meso, micro moves later), and update `memory/index.md`'s next action.
