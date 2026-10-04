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

6. Decisions made the same day (recorded in `decisions.md`): GitHub repo is `population-simulator`; the local folder stays `traffic-simulator`; commute burden covers time, money, income share, unpredictability and "all the issues", and should be "realistic".
7. **Centre on people:** "i want to model a person's life.. i would rather model humans in various situations rather than anything else".
8. **City-agnostic:** "the cities doesnt matter.. its like i get should add a snip of the map and then have multiple ways of visualising population distribution, income distribution, stores, etc..". This supersedes the per-city `cities/` folder idea. Open: what timescale "a person's life" means (a day vs years), and how "realistic" and "any map snip" are reconciled when local data is missing.
9. **Timescale and validation:** "yes government surveys can be a good to cross verify with.. basically we model an year with a particular population distribution and then we can see whether the simulation numbers and the govt numbers align..". Claude's reading, not yet confirmed: a fixed population lives through one year (day-to-day and seasonal variation), and aggregate outputs are compared with official survey statistics. Open: whether life events happen within the year; which surveys build the model vs which check it, to avoid circular validation.
10. **Life events with knobs (2026-10-04):** "we shall add knobs for all of those.. since we are trying to mimic the population realistic, we can use the data of cars' sold or increase of registered vehicles in that region if possible". Recorded in `decisions.md`.
11. **All life events (2026-10-04):** "all of these should be included.. even the children graduating primary, secondary, jr. college, sr. college.. marriages, move-ins, move-outs for job, etc". Recorded in `decisions.md`.

## Evidence notes

- [ui-platform-facts.md](ui-platform-facts.md)
- [city-scale-facts.md](city-scale-facts.md)
- [population-sim-facts.md](population-sim-facts.md)
- [m1-context-brief.md](m1-context-brief.md) — written before this direction; its M1 design assumes the original micro-corridor plan.
