# Direction from the 2026-10-03 rubberduck session

Status: in-flight. These are Sushrut's statements during the session, in order. They are not yet reflected in `docs/PLAN.md` or `decisions.md`.

1. Wants UI knobs for everything, including time steps and behaviours. Asked about Python core + Flutter frontend, or pure Python with MP4 output plus an optional live view.
2. **Scope: desktop only, macOS + Windows.** Mobile and web are out of scope.
3. **Scale: "simulate realtime traffic", more than 4 lakh vehicles at a time, each with its own route.** Whether "realtime" means wall-clock speed or live data was asked but not answered.
4. **Population-based:** each home is a household with a car, or members use rickshaw/bus. Particular people own a taxi or rickshaw, drive it as a job and park it near home. "We simulate the population, not just the vehicles." Proposed renaming the project directory to `population-simulator`. Not yet done: Claude raised that the name may read as demography and listed what a rename touches (23 refs in 8 files, package/CLI names, `.venv`, KB `projects.yaml` + workstream folder, Claude project dir). There are no commits or remote yet.
5. **Questions the tool should answer:**
   - household commute burden;
   - what changes when a family has a car;
   - what changes if the entire population starts using public transport.

## Evidence notes

- [ui-platform-facts.md](ui-platform-facts.md)
- [city-scale-facts.md](city-scale-facts.md)
- [population-sim-facts.md](population-sim-facts.md)
- [m1-context-brief.md](m1-context-brief.md) — written before this direction; its M1 design assumes the original micro-corridor plan.
