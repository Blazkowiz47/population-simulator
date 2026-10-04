# Population Simulator Development Plan

Date: 2026-10-04  
Status: canonical plan, rewritten for the population-simulator direction. Implemented so far: the uv scaffold and status CLI, project memory, and the `govdata/` catalogue. Everything else is planned.  
Project: `/Users/sushrutpatwardhan/1Projects/traffic-simulator` (local folder name kept; package `population_simulator`, CLI `population-simulator`)  
Repository: https://github.com/Blazkowiz47/population-simulator (public, branch `master`, no licence declared)

Labels used below: **Decided** means Sushrut said it or [decisions.md](../memory/decisions.md) records it. **Adopted** marks the engine/UI design: decided on 2026-10-04 after the map spike and recorded in `decisions.md`, with its listed open items still open. **Proposed** marks Claude's engineering proposals that still need confirmation. **Open** marks questions nobody has settled. Facts link to the scratch notes that hold their sources and caveats.

## 1. Purpose and confirmed direction

Build a desktop application that simulates the people of a chosen region living through one year (households, work, school, travel, money and life events) and checks the simulated numbers against official statistics. Sushrut's direction (Decided):

- **People first.** "i want to model a person's life.. i would rather model humans in various situations rather than anything else". A household owns a car, or its members use a rickshaw or bus; some people own and drive an auto or taxi for a living and park it near home.
- **Any region.** "the cities doesnt matter.. its like i get should add a snip of the map and then have multiple ways of visualising population distribution, income distribution, stores, etc..".
- **Desktop only:** macOS and Windows. Mobile and web are out of scope.
- **Knobs for everything in the UI, including time steps and behaviours** (Sushrut, 2026-10-03).
- **One year, cross-checked:** "basically we model an year with a particular population distribution and then we can see whether the simulation numbers and the govt numbers align..".
- **All life events, each a knob** with data-backed defaults (`decisions.md`; where no data exist, as for out-migration, the default is a marked assumption): vehicle purchase and sale; getting, losing and changing jobs; moving house; births and deaths; marriage and leaving home; education stages (primary, secondary, junior college, senior college); income changes; migration into and out of the region, including moves for jobs. Vehicle acquisitions follow regional sales or registration data where available ("if possible"). Every event can be switched off, which gives fixed-circumstances runs.
- **Commute burden, measured realistically:** time, money and income ("should measure all... time, money income"), including income share, unpredictability and "all the issues" ("yes it covers unpredictable trip too.. all the issues.. make it realistic"). `decisions.md` reads this as bringing reliability and crowding into scope. Ownership costs are inferred from "all the issues" and still need Sushrut's confirmation.
- **No republishing of government data:** "we won't be republishing.. we will state that our simulation numbers match".
- **An independent engine in Python**, with open-source projects as references only. Sushrut asked for "multiprocessing pool, queues and threading options".

Proposed (not yet confirmed): under the any-region direction, the 2026-10-03 India decision means Indian cities get the first local data packs; official sources have been surveyed for Bengaluru, Mumbai and Hyderabad.

**What changed from the 2026-10-03 plan.** That plan started with one corridor and a microscopic IDM car-following engine (old M1: a straight-road fixture), building towards a calibrated Indian corridor. Three things moved the project:

1. Sushrut's questions (§2) are about households. Claude's reading, which Sushrut liked: commute burden depends on city-wide congestion, transit capacity, fares and income far more than on lateral weaving or junction blocking.
2. Sushrut's scale is "more than 4 lakh vehicles at a time, each with its own route". No precedent was found that runs this concurrency microscopically with lateral movement, on any hardware. Queue and mesoscopic models handle that scale on a desktop ([city-scale-facts](../memory/scratch/city-scale-facts.md)).
3. The time span became a year with life events, and the region became any map snip.

The microscopic Indian mixed-traffic engine is not dropped. It becomes a later component for focus areas (M8), and its design brief, [m1-context-brief.md](../memory/scratch/m1-context-brief.md), remains valid for it. Sushrut liked this household-first ordering ("i like this"; [household-first-formulation](../memory/scratch/household-first-formulation.md)); it is not yet a `decisions.md` row.

## 2. Questions the tool answers and the first useful result

Sushrut's three questions (Decided), with Claude's proposed readings:

| Question | Model reading | What it needs |
|---|---|---|
| Household commute burden | Door-to-door time; money per trip and, inferred from "all the issues" and still to confirm, vehicle ownership costs; share of income; unpredictability; crowding | Incomes, trips costed by mode, travel times that vary over the year, transit loads |
| What changes when a family has a car | (a) *One family:* replay its days against baseline travel times, since one car does not change city congestion; cheap enough for every household. (b) *Many families*, e.g. ownership rising from 20% to 35%: needs a full city run | (a) M4; (b) M6 |
| What changes if everyone uses public transport | A transit-capacity question: roads empty while buses and trains overflow, with denied boarding and long waits. Bus fleet held fixed vs scaled up is a knob | Bus and metro capacity, GTFS timetables, walk access; M6 |

Outputs are distributions, not averages ([commute-burden-facts](../memory/scratch/commute-burden-facts.md)):

- by income fifth, gender, ward and household type (car, two-wheeler, none, livelihood auto or taxi);
- the share of people above each threshold, such as a one-way commute over 60 or 90 minutes, or transport spending above a stated share of income;
- winners and losers between scenarios.

Burden dimensions are reported side by side rather than as one score (Proposed). Low spending by poor households can mean walking or not travelling, so money is always paired with time, access and trip participation. If value of time is tied to income, use an elasticity below 1 and also report with one average value, so richer people's time does not dominate. Every person and household is labelled synthetic, and every money figure carries the "as of" date of its fare and price tables.

**First useful result (Proposed).** For one snipped region: a synthetic population whose typical-weekday commute burden (time, money, income share) is reported by income fifth, gender and household type, plus the one-family car comparison. Every comparison figure is tagged `build`, `constraint` or `check` (§4). This is the M4 gate. Unpredictability needs the year (M5); crowding and the city-wide what-ifs need M6.

Open: which further issues count towards burden (candidates: access walk, heat and rain exposure, transfers, safety especially for women at night, seat availability); whether people adapt in what-if scenarios (fixed plans, rerouting only, mode re-choice, or full replanning); and what "realtime" meant, which Sushrut has not answered. Claude's reading: comparisons need batch runs faster than real time, and a live view is optional.

## 3. Scope

| | Items |
|---|---|
| **In scope** (Decided) | Synthetic households and persons for any map snip; incomes and vehicles, including owner-driver autos and taxis; daily routines over one year; all life events as knobs; commute burden and the three questions; cross-checks against government figures without republishing them; a desktop app for macOS and Windows with knobs and several map views (population, income, stores and more); a headless CLI (Adopted 2026-10-04 with the engine/UI design) |
| **In scope, later milestones** | City-scale traffic and transit capacity at Sushrut's stated scale, more than 4 lakh vehicles at a time, each with its own route (Decided scale; meso first, M6). The microscopic mixed-traffic engine for focus areas: signals, police, wrong-way driving, junction blocking, pickups (M8, from the household-first ordering Sushrut liked; when it returns, and whether junction-level questions stay, are Open) |
| **Deferred** | A Google Maps display adapter. The 2026-10-03 decision keeps it as a separate, optional adapter, but no supported desktop path has been found: Google ships no desktop Maps SDK, macOS WKWebView is not on Google's supported list for Maps JavaScript, and the key cannot be meaningfully origin-restricted ([ui-platform-facts](../memory/scratch/ui-platform-facts.md)). Whether it is wanted on desktop is Open (§8); the boundary and activation gates in [map-providers.md](map-providers.md) still apply |
| **Not re-confirmed** (from the original brief) | Airport-timed demand and enforcement experiments |
| **Out of scope** | Mobile and web apps (Decided); republishing government files (Decided); copying Google content into simulation inputs (Decided boundary); photorealistic 3D, nationwide automatic calibration and a distributed engine (carried over from the old plan) |

## 4. Data strategy

**Global baseline plus local data packs (Proposed).** Any snip gets a baseline from global open layers: buildings, population grids, places, land cover and a basemap. Where official data exists for an Indian city, a local data pack replaces baseline values. Every attribute records whether it came from local, national or foreign-donor data, so a run shows how local it is ([any-region-data-facts](../memory/scratch/any-region-data-facts.md)). This is the proposed answer to the open question of how "realistic" and "any map snip" fit together.

**Government records** follow [govdata/README.md](../govdata/README.md) (Decided). Downloaded files stay local in the git-ignored `govdata/<dataset-id>/raw/`. The repository tracks only `catalog.yaml` (104 datasets), per-dataset `manifest.json` files and our own `comparisons/`. Credentials stay with the user, and a file without a manifest entry must not feed a model.

- **Roles and the build vs check ledger.** Each dataset is marked `build` or `check`, and a dataset used to build cannot back a "matches" claim for the same quantity (Decided, govdata/README.md rule 5). Proposed: the ledger also works per quantity, with three roles, `build`, `constraint` and `check`. Reports call them "data used to build", "data used as constraints" and "independent validation data". At dataset level, govdata/README's `build` covers both `build` and `constraint`.
- **Vehicle checks.** Because vehicle acquisitions now follow registration data (Decided), VAHAN, MVD and FADA flows become build inputs. Vehicle checks then need a hold-out that contains no season or regime missing from the build data, or ownership surveys (HCES, NFHS, Census 2027). Census 2027 houselisting (Apr–Sep 2026) asked about vehicles, with bicycle grouped with two-wheeler; no table release date is known. The catalogue note's example hold-out (calibrate on January–June, check July–December) does not meet this for 2025: the second half contains the GST cut on 22 Sep 2025 and the festive window (about 20% of two-wheeler and 17% of car retail sales in 42 days). It works only if both are modelled explicitly, and the same applies to checking 2025 against rates built from 2021–2024. Otherwise choose another hold-out slice (Proposed: alternate months, or whole RTOs held out over the same period) and record the design in the ledger before fitting.
- **Catalogue work.** The catalogue's roles must be reassigned against the planned inputs, its verifier corrections merged into the main fields, and the ~60 "missed" datasets listed at the end of `catalog.yaml` triaged ([govdata-catalog](../memory/scratch/govdata-catalog.md)). M1 does this only for the datasets M2 uses.
- **Provenance.** Every source records its URL or release ID, retrieval date, checksum, reference period, finest geography and licence. Overture releases are deleted after about 60 days, so each region snapshots its layers and pins the release. FADA and EPFO revise earlier months, so each default stores its source vintage. Fares and prices live in versioned, dated tables.

**Licences and law** (facts; the handling is Proposed):

- **ODbL share-alike** flows from OSM and Overture buildings into derived databases. Taking Google Open Buildings directly under CC BY 4.0 avoids it. Region packs are not distributed until this is decided.
- **Non-commercial traps:** Meta RWI, SHRUG (also share-alike), GDL, GEM exposure, the India HRSL 2025 release and the Uber Movement archive. Each layer carries a licence flag, and the run report shows it.
- **MoSPI Category B unit data** (PLFS, HCES, TUS, DTES): free registration and an Annex-I undertaking. Unit data "shall not be shared … without prior approval", directly or indirectly. Publishing derived aggregates with a citation is allowed, and a public repository may hold download scripts and derived parameters. The notes differ on commercial use: the GSDD 2026 survey says non-commercial only, the life-events note says commercial use is priced. Foreign users need an institution recognised by the Government of India, which may matter for Sushrut's registration. Whether a synthetic population fitted to unit records counts as "sharing" is unclear (Open); until it is settled, defaults derived from unit data are not committed (§5). Category A reports and aggregates are reusable with citation.
- **Reproduction permission** is needed for Census 2011 tables, Labour Bureau CPI-IW and PPAC fuel data; AISHE restricts reuse. All of this is compatible with not republishing.
- **India map law:** publishing a map of India that does not conform to Survey of India boundaries is an offence (Criminal Law Amendment Act 1961, s.2(2)). City views omit international and disputed boundaries; any India-wide view uses the Survey of India Administrative Boundary Database.

**Access.** This Mac geolocates to Norway, and many Indian portals block it (data.gov.in, Karnataka state sites, data.telangana.gov.in). Steps only Sushrut can do: MoSPI registration, 6th Economic Census unit files (free login), VAHAN exports (CAPTCHA; the dashboard's chart data endpoints need none but are undocumented), NFHS-6 district PDFs (form), TGSRTC/HMRL GTFS (Google Form), the Survey of India download checkbox, and a data.gov.in API key. Claude does not log in, register, accept terms or solve CAPTCHAs.

**Key sources by layer** (dates, figures and caveats are in the linked notes):

| Layer | Main sources | Main caveat | Evidence |
|---|---|---|---|
| Region and basemap | Overture GeoParquet (overturemaps-py or DuckDB); OSM PBF clipped with QuackOSM or pyosmium; Protomaps PMTiles basemap | Avoid public Overpass and tile.openstreetmap.org; copy PMTiles builds to own storage | [any-region](../memory/scratch/any-region-data-facts.md) |
| Homes | Overture buildings, Google Open Buildings v3, Microsoft buildings; EMC-BUILT non-residential layer | Heights are almost absent in Indian cities, so persons per building is an uncertain modelled variable | [any-region](../memory/scratch/any-region-data-facts.md) |
| People and households | Census 2011 ward PCA and HL-14 (BBMP, Greater Mumbai, GHMC); HH-01 city household sizes (held out as a check); WorldPop and GHS-POP grids; GLOPOP-S as a global fallback | 2011 vintage; ward boundaries have changed; no open 2011 ward polygons; WorldPop Global2 R2025A (alpha) uses state-level inputs only for India (35 units), so density inside a city is purely modelled; national files only, about 0.75 GB per layer | [govdata-catalog](../memory/scratch/govdata-catalog.md), [population-sim](../memory/scratch/population-sim-facts.md) |
| Income | PLFS 2025 unit data (earnings plus household non-labour income); HCES 2023-24 (consumption and transport spending) | No open income distribution below state or district level: take the shape from surveys, add a top tail, use proxies only to rank places; keep the module swappable | [commute-burden](../memory/scratch/commute-burden-facts.md) |
| Places, jobs, schools | Overture Places and OSM; 6th Economic Census (2013); UDISE+ school files; AISHE college directory | No open employment grid; realism holds at ward or zone scale, not for individual shops; Google Places is unusable | [any-region](../memory/scratch/any-region-data-facts.md), [education](../memory/scratch/education-and-timing-facts.md) |
| Daily schedules | India TUS 2019 and 2024; OMoSim as the precedent | No destinations or modes; one day per person; activities under 10 minutes are missed | [year-validation](../memory/scratch/year-validation-facts.md) |
| Life events | VAHAN dashboard and OpenCity mirrors; SRS 2024; Census 2011 D-03 migration; PLFS 2025 panel; EPFO; board results; AISHE | Registration is not household acquisition; out-migration has no city-level rate; PLFS city samples are thin; OpenCity covers Bengaluru and Mumbai to mid-2025 only; Telangana's history is on VAHAN | [life-events](../memory/scratch/life-events-facts.md), [education](../memory/scratch/education-and-timing-facts.md) |
| Transport and money | OSM networks; GTFS (TGSRTC and HMRL official, redistribution with attribution, behind a Google Form; Bengaluru's official feed states no licence, the unofficial feed is ODbL); dated fare tables; CEEW (Jun 2025) per-km ownership cost | Prices change often; meter fares are a floor; women ride some buses free in Karnataka and Telangana; CEEW's 14 km/L figure is for an average Rs 9.5 lakh car, not a small car (small cars about Rs 5.1–5.8/km on fuel at 20–22 km/L, as of about 3 Oct 2026) | [commute-burden](../memory/scratch/commute-burden-facts.md) |
| Calendar and weather | IMD gridded rainfall; state school calendars; holiday lists | Indian evidence on rain effects is thin and site-specific | [year-validation](../memory/scratch/year-validation-facts.md) |

## 5. Engineering approach

The design in [engine-ui-architecture.md](../memory/scratch/engine-ui-architecture.md) is **Adopted (2026-10-04)** and recorded in `decisions.md`: Sushrut liked the map spike ("I love this") and asked for the plan to be updated. Its open items stay open: Windows/WebView2, snip drawing, the offline basemap, performance at millions of points and the Python 3.14 upgrade. [architecture.md](architecture.md) holds the detailed design; this section summarises it.

**Engine (Adopted; details in [architecture §8](architecture.md)).** People and households are stored as numpy columns, not per-person objects. CPU work runs in a multiprocessing pool, with threads only for I/O; macOS and Windows use the spawn start method. Each simulated day runs **decide, resolve, commit**: households decide in parallel from yesterday's frozen state, one deterministic pass resolves competing claims (flats, jobs, school seats, bus seats, road space), and then the day is committed. Random draws are keyed by run seed, household and day, not by worker, and results merge in a fixed order, so outputs do not depend on worker count or scheduling. Bit-identical results across macOS and Windows are not claimed. The year uses weighted day types; a cheap household and money layer can run every day or month, with detailed travel only on representative days (Proposed).

**Runner and contract (Adopted; [architecture §2.3–2.4](architecture.md)).** A simulated year is a long job with progress and cancel, so it runs in its own runner process rather than NiceGUI's `run.cpu_bound`. The runner reports small progress messages on a queue and stops on a cancel event. Large data never goes through the queue; it goes to Parquet/Arrow files in the run folder or to shared memory. The UI and engine share only three things: the **scenario schema**, the **run-folder format** and **small progress messages**. The CLI uses the same runner, so every run is headless and can be re-run from its `run.json`. Replacing the UI later would touch only the UI layer.

**Desktop UI (Adopted; [architecture §9](architecture.md)).** NiceGUI 3.17.1 (MIT, Python 3.10–3.14) in native mode through pywebview, with knob forms generated from the scenario schema and charts from NiceGUI's ECharts or Plotly elements. The map is **one custom component** wrapping MapLibre and deck.gl. Python declares layers as deck.gl JSON specs; the runner writes binary layer files, which the local server serves over HTTP; the prebuilt deck.gl, `@deck.gl/json` and MapLibre bundles are vendored, so no Node toolchain is needed. The [map spike](../memory/runs.md) confirmed this path on macOS: 300k points in about 1 s and 28k building polygons in about 0.4 s, with events working in both directions. Its lessons ([learnings.md](../memory/learnings.md)) and the Windows caveats are requirements, listed in architecture §9. Proposed: map layers switch by zoom and are labelled synthetic.

**Knobs (requirement Decided; details Proposed; [architecture §2.2](architecture.md)).** Sushrut asked for knobs for everything, including time steps and behaviours. A Pydantic scenario schema emits JSON Schema with UI metadata under the `x-` prefix, and the forms are generated from it. Every knob has a default recorded with its source and vintage, and an inclusive range. Life-event and behaviour defaults are data-backed (Decided for life events); where no data exist, as for out-migration or owner-driver rent, the default is marked `assumption`. Expert and viewer knobs carry documented defaults. Knobs come in three tiers: expert numerical settings, behaviour and life-event rates, and viewer pacing, which must never change outputs. A knob moved away from its data-backed default is marked "departed from data" in the UI and in `run.json`. Each life event can be switched off (Decided), which gives fixed-circumstances runs. Defaults derived from MoSPI Category B unit data are not committed until the "derived products" question is settled (§4, §8): they live in the git-ignored region `derived/` folder, and the schema refers to them by dataset ID.

**Proposed module boundaries** (create modules only when a milestone needs them):

| Module | Responsibility |
|---|---|
| `scenario` | Schema, knob metadata, defaults with sources, validation |
| `regions` | Snip geometry and buffer, fetching and clipping open layers, region manifests, licence flags |
| `govdata` | Catalogue and manifest loading, build/constraint/check roles, comparison records |
| `population` | Households, persons, homes, income, vehicles, owner-drivers, job and school assignment |
| `calendar` | Simulated year, day types and weights, holidays, school terms, weather days |
| `life_events` | Event rates and timing, open-population accounting |
| `activities` | Daily schedules, destinations, modes, trip costs |
| `mobility` | Networks, routing and fixed baseline travel times (M4); meso traffic and transit capacity (M6) |
| `engine` | Keyed random streams, decide/resolve/commit, process pool, shared memory |
| `runner` | Jobs, progress, cancel, run-folder writer |
| `metrics` | Commute burden, distributions, winners and losers, people accounting |
| `validation` | Tolerances, comparisons, three-way reports |
| `ui` | NiceGUI app, map component, knob forms, charts |
| `cli` | Region, check, run and compare commands |
| `focus` | Microscopic focus-area engine (M8) |

**Proposed repository layout:**

```text
src/population_simulator/   # package; modules above, added per milestone
  ui/static/                # vendored deck.gl and MapLibre bundles, map component
govdata/                    # catalogue, manifests, comparisons (raw/ git-ignored)
data/regions/<name>/        # git-ignored: snip and buffer, fetched layers, basemap.pmtiles, derived parameters, manifests
outputs/<run-id>/           # git-ignored: run.json, population, events, daily aggregates, map layers, validation
tests/fixtures/<name>/      # small authored synthetic fixtures (tracked)
docs/  memory/
```

The `.gitignore` rules `data/` and `outputs/` are unanchored, so they also ignore nested directories such as `tests/fixtures/data/`; anchor them as `/data/` and `/outputs/`. `run.json` records the resolved scenario with defaulted values marked, the seed, scenario and `uv.lock` hashes, Python version, platform, source revision, data-pack manifests and price "as of" dates. Every output file carries its own format version.

**Proposed CLI** (only the status command exists today):

```sh
uv run --locked population-simulator region create --name <name> --bbox <west,south,east,north>
uv run --locked population-simulator region check --name <name>
uv run --locked population-simulator run --scenario <file> --seed 42
uv run --locked population-simulator run --rerun outputs/<run-id>/run.json
uv run --locked population-simulator compare --baseline outputs/<a> --alternative outputs/<b>
uv run --locked population-simulator app
```

`run --rerun` re-runs the resolved scenario, seed and data-pack manifests recorded in a `run.json`, and warns if the `uv.lock` hash or source revision differs.

**Dependencies.** The package has none today; the spike used NiceGUI, pywebview, pyarrow, shapely and numpy. Add each when its milestone needs it, and record its licence in [open-source-references.md](open-source-references.md) when adding it. Python stays on 3.12 until the version decision (§8); free-threaded CPython 3.14 (PEP 779) is a reason to move later.

## 6. Milestones and acceptance gates

The milestones, their order and their acceptance gates are Proposed ([decisions.md](../memory/decisions.md) notes); only the scope they deliver is Decided. The sequence is dependency-driven; no delivery dates are claimed. Each milestone ends with something runnable and measurements that justify the next step. The order keeps the household-first ordering Sushrut liked, with these proposed changes: a region workspace (M1) and a desktop shell (M3) are added; commute burden on fixed travel times (M4) and the year with life events (M5; its scope was decided on 2026-10-04) come before city traffic and transit (M6), so the first useful result arrives sooner; and ensembles and validation reporting get their own milestone (M7). Each layer's validation targets and tolerances are committed before that layer is fitted or compared (§7, §8).

M3's map component can start alongside M1, using region layers; its runner integration starts once M2 has written run-folder format v0. M6 depends on M4. Its unpredictability output also needs M5's calendar, so if M6 comes first, unpredictability waits for M5.

### M0 — Scaffold, research and direction (done)

Evidence: the uv package, lockfile and status CLI (`uv run --locked population-simulator`, verified 2026-10-04); project memory and skills; the public repository; `govdata/` with its rules and a 104-dataset catalogue; verified fact notes in `memory/scratch/`; the package rename; and the map spike recorded in [runs.md](../memory/runs.md). No simulation exists.

### M1 — Region workspace and data foundation

Snip a region (bounding box first, polygon later) into `data/regions/<name>/`. A region is a study area plus a context buffer; `region.json` records both geometries (buffer size Open, §8). Fetch and clip Overture buildings and places (pinned release), an OSM extract and a PMTiles basemap; name the PMTiles extract tool and record its licence. Write manifests with licence and attribution. Load `govdata/` manifests. Define the per-variable ledger file and the `govdata/<id>/comparisons/` record format, and draft the ledger for the first region's inputs. Reconcile catalogue roles and verifier corrections for the datasets M2 uses for the first region; merging the rest of the catalogue and triaging the ~60 missed datasets is a separate task.

Acceptance: a region rebuilt from its manifests reproduces the recorded content hashes (defined in §9), or reports which upstream source changed; every layer carries a licence flag, with share-alike and non-commercial layers marked; tests confirm that nothing under `data/`, `outputs/` or `govdata/**/raw/` is tracked; once fetched, everything works offline.

### M2 — Synthetic population v1, the headless runner and first map layers

Engine and contracts: scenario schema v0 (region, population and income sections); keyed random streams and the household-chunk process pool; the headless runner and the `run` CLI command, including `--rerun`; run-folder format v0 (`run.json`, population Parquet, map layers).

Population: households and persons placed in residential buildings, with age, sex, household size, income, worker and student status, vehicle ownership, and owner-driver autos and taxis. Jobs and schools are assigned at zone scale. Destinations outside the study area are external zones with travel times; in-commuters are boundary demand (Proposed; buffer size and method Open). Seed: PLFS 2025 households fitted with IPU to ward marginals once MoSPI access exists (Proposed, architecture §3.3); otherwise a labelled fallback seed (Open). This requires a crosswalk from 2011 wards to current units and a projection to the simulated year (architecture §3.2). Income takes its shape from PLFS 2025 unit data once access is settled; until then from HCES state consumption inequality, labelled as such because consumption and income inequality differ. Map layers show population, income, stores and buildings.

Acceptance: population tolerances are committed before fitting; build marginals are reproduced within them and reported as internal checks; at least one held-out cross-tab is reported separately; every attribute has provenance (local, national or donor); runs with one worker and with several give identical deterministic outputs (architecture §2.3); wall time, memory and persons per second are recorded on named hardware at the sizes declared in architecture §11, with targets set after the first measurement.

### M3 — Desktop app shell

The UI job manager on top of the M2 runner (progress queue, cancel event, `partial` status); knob forms from the schema; a NiceGUI native window with the map component, built with the spike lessons; snip drawing; offline basemap display; tests on Windows with WebView2.

Acceptance: a cancelled run's folder has status `partial`, contains every committed day and nothing from the discarded day, and passes the same reader checks as a complete run; a run started from the UI and the same `run.json` re-run from the CLI give identical deterministic outputs; viewer settings never change outputs; a missing WebView2 runtime or WebGL produces a clear error; the local server binds to 127.0.0.1 only. A minimal packaged build of the shell (Briefcase or PyInstaller onedir) starts on clean macOS and Windows machines, and its behaviour with Windows Smart App Control on is recorded (Smart App Control blocks unsigned `.pyd` files, including numpy's, with no per-app exception). If the Windows/WebView2 or packaging checks fail, revisit the engine/UI decision ([decisions.md](../memory/decisions.md)).

### M4 — Daily life and commute burden on typical days

Day types; schedules derived from TUS; destination and mode choice; door-to-door times on fixed baseline travel times, labelled as such; per-trip and ownership costs from dated tables; burden metrics and their distributions; the one-family car comparison. This milestone delivers the first useful result (§2).

Acceptance: people are conserved through every activity chain; trip costs reproduce their fare tables; burden tolerances are committed before comparison; burden is reported by income fifth, gender and household type, with threshold shares; each comparison figure is tagged `build`, `constraint` or `check`; checks use quantities not used to build (candidates include the HCES conveyance share and published commute times; the choice is recorded in the ledger).

### M5 — The year

The calendar of the simulated year (2025 suggested); weighted day types; the life-events engine with every event type and its seasonal timing (school stages at each state's reopening date and age rule, April pay rises, festive vehicle purchases, marriage calendars, family moves); the resolve step for flats, jobs and school seats; open-population accounting.

Acceptance: births, deaths, in-migrants and out-migrants close the population balance every day; every event can be switched off; M4's typical days use the same day keys as the year's representative days, and with all events off, household and person state is unchanged across the year and each representative day's outputs are byte-identical to the M4 run of that day type; event counts lie within a sampling-error band (Poisson or binomial) declared in the test; vehicle flows are checked against a hold-out that contains no season or regime missing from the build data, designed and recorded in the ledger before fitting (§4); PLFS comparisons use rates, not totals.

### M6 — City-scale traffic, transit capacity and the what-if scenarios

A queue or mesoscopic road model; precomputed routes with periodic rerouting; transit with capacity, denied boarding and the fleet knob; a sampling scheme (Proposed: sample private vehicles, keep fleets and public transport at 100%); the "many families get cars" and "everyone on public transport" scenarios; unpredictability and crowding added to burden.

Acceptance: no vehicle or passenger is dropped or teleported without being counted; vehicle and passenger capacities are respected; the road model runs at least 4 lakh concurrent vehicles, each with its own route, on named hardware, with wall time vs simulated time recorded (the speed target waits on the meaning of "realtime"); for a whole-metro run, peak concurrent vehicles lie within the Little's-law range for that metro, widened by its stated uncertainty of about 2×; movement tolerances are committed before comparison; travel-time patterns and ridership are compared with independent sources (speed indices, monthly congestion patterns, reported ridership, which is approximate); scenarios are compared on the same synthetic population.

### M7 — Ensembles and reporting

Multi-seed runs that also vary uncertain inputs; a validation and limitations report for every run; comparison records in `govdata/<id>/comparisons/`, in the format defined at M1; scenario comparison reports (§7).

Acceptance: reports separate data used to build, data used as constraints and independent validation data; no "matches" claim rests on build or constraint data; seed counts are justified by result stability or interval width.

### M8 — Microscopic focus-area engine

The former M1: a deterministic straight-road fixture, then junctions, signals, wrong-way driving, junction blocking, pickups and enforcement for chosen focus areas, coupled to the meso model. Its decisions, plan defects, test values and placeholder parameters are in [m1-context-brief.md](../memory/scratch/m1-context-brief.md); line references in that brief point to the docs at commit `ca8246a`. D4 (runtime dependencies) is settled by the adopted engine design, and D5 (initial commit) by the existing commits. D8 (uv upgrade) and D9 (Python version) apply now (§8, §9). Only D1–D3, D6 and D7 wait for M8.

Acceptance: the analytic, conservation, determinism and step-halving checks in that brief; vehicles crossing the meso/micro boundary are conserved and spillback passes back. No published method was found that generates a lateral entry position for lane-free traffic, so this boundary is research work.

### M9 — Packaging and distribution

Installers for macOS (arm64; Intel Macs not yet considered) and Windows, built on per-OS CI runners, since no tool cross-compiles (GitHub's macos-14 image retires 2026-11-02). Candidates: Briefcase (signed, notarised DMG and MSI) or PyInstaller in onedir mode. M3 has already checked a minimal packaged build; M9 adds signing, installers and CI. Costs and gates: the Apple Developer Program ($99/yr) for notarisation; a Windows OV certificate ($150–300/yr) or Artifact Signing (about $10/month, organisations only; Microsoft's docs conflict on whether Norway is eligible); Smart App Control blocks unsigned `.pyd` files, including numpy's; the WebView2 bootstrapper.

Acceptance: a bundled small region installs and runs offline on clean macOS and Windows machines.

## 7. Validation and scenario comparison

The principles below are Proposed except where marked. Methods, sources and candidate checks per layer are in [architecture §10](architecture.md) and [year-validation-facts](../memory/scratch/year-validation-facts.md).

- **Calibration is not validation.** Comparing outputs with the tables used to build them is an internal check. Reports follow the TAG M3.1 template and list three groups separately: data used to build, data used as constraints, and independent validation data.
- **Build vs check** (Decided, [govdata/README.md](../govdata/README.md) rule 5), kept as a ledger per variable, not only per dataset, with the roles defined in §4. If Census 2011 HL-14 vehicle shares or B-28 commute tables are used as controls, the matching outputs are internal checks. Holding out some cross-tabs of the same census gives a cheap quasi-hold-out.
- **Like with like.** Match each survey's estimator, period and geography; compare PLFS rates, not totals; every target needs a geography crosswalk.
- **Tolerances fixed in advance**, per layer, before that layer is fitted or compared. Use published RSEs or confidence intervals where they exist, and set explicit bands for administrative counts and fact sheets. No repeated tweaking until the numbers match.
- **Uncertainty.** Seed-only ensembles understate it (one UrbanSim study found 38% coverage for a nominal 90% interval), so ensembles also draw uncertain inputs.
- **Scenario comparisons** use the same synthetic population and keyed draws, with one stream per decision or event type and keys built from stable IDs. People created during a run get IDs keyed by (day, event type, ordinal), not a global counter. Unchanged decisions then draw the same numbers. Where a scenario changes which events occur, draws can diverge, and comparisons report this. Reports show distributions, threshold shares, and winners and losers, not one optimality score.
- **Wording.** "Matches" is used only for independent data within the pre-set tolerance. Everything else is called an internal check, or a simulation under stated assumptions. Indian precedents mostly report in-sample fit; a real hold-out design goes beyond them.

## 8. Decisions

**Made** (Sushrut's statements and `decisions.md` rows; reasons and consequences are in [decisions.md](../memory/decisions.md) where a row exists, otherwise in the linked source):

| Date | Decision |
|---|---|
| 2026-10-03 | Independent engine; open-source projects are references only |
| 2026-10-03 | Focus on Indian mixed traffic, including Bengaluru, Mumbai and Hyderabad; pilot open. How this applies under the any-region direction is still to make (below). Proposed reading: these cities get the first local data packs, and mixed-traffic behaviour moves to M8 |
| 2026-10-03 | uv-packaged Python 3.12+ application |
| 2026-10-03 | OSM network import; any Google Maps integration is a separate, optional display adapter |
| 2026-10-03 / 2026-10-04 | Public repository `population-simulator` on `master` (2026-10-03); local folder stays `traffic-simulator` (2026-10-04) |
| 2026-10-03 | Desktop only (macOS, Windows); knobs for everything, including time steps and behaviours; population-based; model people's lives; any map snip; one year cross-checked with government figures (Sushrut's statements in the [direction note](../memory/scratch/direction-2026-10-03.md)) |
| 2026-10-03 | Commute burden covers time, money, income share, unpredictability and "all the issues", realistically; reliability and crowding are in scope; ownership costs are inferred and still to confirm |
| 2026-10-04 | Life events happen within the year, each a knob with data-backed default rates and an off switch (fixed-circumstances runs); vehicle acquisitions follow regional sales or registration data where possible |
| 2026-10-04 | All life events are in scope, including education stages (primary to senior college), marriages, and moving in and out of the region for jobs; the population is open at the region boundary |
| 2026-10-04 | Package `population_simulator`; distribution and CLI `population-simulator` |
| 2026-10-04 | Government data is never republished; we state only whether our numbers match (Sushrut: "we won't be republishing.. we will state that our simulation numbers match"; rules in [govdata/README.md](../govdata/README.md)) |
| 2026-10-04 | Engine/UI design adopted: Python engine on numpy arrays with a multiprocessing pool, decide/resolve/commit days and keyed random draws; a runner process; a NiceGUI desktop UI; one MapLibre + deck.gl map component; connected only by the scenario schema, run-folder files and progress/cancel messages. Open items: Windows/WebView2, snip drawing, offline basemap, performance at millions of points, Python 3.14 upgrade. Revisit if Windows testing or scale tests fail |

**Still to make:**

| Decision | Why it matters | When needed |
|---|---|---|
| First region and first local data pack | Sets what is fetched and checked first; Bengaluru has the richest open inputs, Hyderabad the cleanest official bus GTFS; Hyderabad 2025 registrations are on VAHAN | Before M1 acceptance |
| How the 2026-10-03 Indian-mixed-traffic decision applies under the any-region direction | Sets where local data packs come first | With the first-region choice |
| Python version (3.12 vs 3.13/3.14, including free-threaded builds; SPEC 0 recommends dropping 3.12 in 2026 Q4) | Parallelism and dependency support; the Python 3.14 upgrade is an open item of the engine/UI decision | Before §9 adds the first runtime dependency |
| uv upgrade and the `uv_build` bound (M1 brief D8; local uv 0.10.4, `uv_build>=0.10.4,<0.11.0`) | Lockfile compatibility holds only within a uv minor version | Before §9 adds dependencies |
| Treatment of trips leaving or entering the snip (buffer, external zones) | The commute burden of a small snip depends on it | Before M2 job assignment |
| Simulated year (2025 suggested: PLFS 2025 is the first calendar-year round, with city figures) | Fixes data vintages, prices and the calendar | Before M2 |
| Crosswalk from 2011 wards to current units; population projection to the simulated year | Ward marginals feed the population, and ward boundaries have changed since 2011 | Before M2 |
| Fallback seed and fitting method | M2 needs a seed before MoSPI access is settled | Before M2 |
| MoSPI unit-data access; whether a fitted synthetic population counts as "sharing"; whether defaults derived from unit data may be committed | PLFS, HCES and TUS are the main income and schedule sources | Before M2 uses unit data |
| Validation targets and tolerances | Prevents choosing thresholds after seeing results | Per layer, committed before that layer is fitted or compared (M2 population, M4 burden, M5 events, M6 movement) |
| Snip-drawing library; Windows/WebView2 results; performance at millions of points | Open items of the engine/UI decision | M3 |
| Remaining burden dimensions, including ownership costs; unpredictability percentile | Defines the main output | Before M4 |
| Source study for mode choice; household vehicle allocation rule | Sets mode shares and vehicle use | M4 |
| Whether the fare tables we compile are tracked in the repository, given the no-republishing rule | Repository contents and citations | M4 |
| Representative travel days vs daily travel | Run cost vs day-to-day variation for unpredictability | M5 |
| Queue vs meso model; sampling scheme | Fidelity and run cost of city traffic | Before M6 |
| Adaptation in what-if scenarios | Changes results and run cost (MATSim-style replanning takes hundreds of iterations) | Before M6 |
| Meaning of "realtime"; live-view needs | Sets performance targets | Before M6 |
| Building source and distribution of region packs under ODbL | Share-alike obligations | Before sharing any region data |
| Project software licence | The repository is public without one, so default copyright applies | Before distribution or outside contributions |
| Google Maps on desktop | No supported desktop path found; terms need review | Only if wanted; gates in [map-providers.md](map-providers.md) |
| MP4/video export | Packaging, licensing of encoders and basemap | Before M9 |
| When the microscopic engine returns; whether junction-level questions stay | Scope of M8, then M1-brief decisions D1–D3, D6 and D7 | After M6 |
| Code signing and certificates | Cost and install friction | Before M9 |

## 9. First implementation task

A small, verifiable first slice of M1 (Proposed). Before step 3 adds the first runtime dependencies, settle the Python version and the uv upgrade (§8).

1. Anchor the `.gitignore` rules as `/data/` and `/outputs/`, and add pytest with a `tests/` folder. Add `.hypothesis/` and `.benchmarks/` to `.gitignore` when those tools are adopted.
2. Add the CLI subcommands `region create` and `region check`, using `main(argv) -> int`; the bare command still prints the status.
3. `region create --name <name> --bbox …` writes `data/regions/<name>/region.json` (study area, context buffer, CRS, IANA timezone, creation time), fetches Overture buildings and places for the buffered box from a pinned release, and stores them as GeoParquet with a `manifest.json` (release, URL, retrieval time, row counts, content hash, writer library versions, licence, attribution, share-alike and non-commercial flags). Pin the latest Overture release at fetch time (the facts note used 2026-09-23) and record it. The content hash is the sha256 of rows sorted by Overture `id` with a fixed column order. The buffer width is a parameter until its method is decided (§8).
4. `region check` verifies manifests against files, lists licence flags, and fails if any region file or anything under `govdata/**/raw/` is tracked by Git.
5. Tests run offline on a tiny authored fixture in `tests/fixtures/`: `region create --from-dir <path>` reads pre-fetched files, so tests exercise it without the network. As a manual smoke test, reuse the spike's central-Mumbai box (72.820,19.010,72.860,19.065) without treating the spike's counts as expected values. This does not choose the first region.

Done when the tests pass, a second `region create` for the same release reproduces the row counts and content hashes, and the run is recorded in `memory/runs.md`. This adds the first runtime dependencies (overturemaps-py or DuckDB, plus pyarrow); record each new dependency's licence in [open-source-references.md](open-source-references.md) when adding it. In parallel, M3's map component can start by rebuilding the spike's component under `ui/` on region layers, with the spike lessons (architecture §9) as checks; its runner integration waits for M2's run-folder format v0.

## 10. Documentation and memory ownership

| Location | Holds |
|---|---|
| `docs/PLAN.md` | This canonical plan: direction, scope, milestones, gates, decisions |
| [architecture.md](architecture.md) | Detailed design: data model, daily engine loop, runner, run-folder format, UI and map component; later the focus-area engine |
| [map-providers.md](map-providers.md) | OSM and Google provider facts, terms and activation gates. Its body predates the redesign. Where it suggests a pilot corridor, small public Overpass queries or the `NetworkSource`/`NetworkCompiler`/`MapRenderer` interfaces, this plan takes precedence: public Overpass is not used, network import belongs to `regions` and `mobility`, and display belongs to `ui`. Its 2026-10-04 status note records the desktop findings; under the 2026-10-03 decision the Google adapter stays optional and deferred, and whether it is wanted on desktop is Open (§3, §8) |
| [open-source-references.md](open-source-references.md) | Reference projects and their licences (studied, never copied); candidate dependencies and their licences |
| [govdata/README.md](../govdata/README.md), `govdata/catalog.yaml` | Government data rules and the dataset catalogue |
| `README.md` | Entry point: what the project is, where to start, local setup and conventions |
| `memory/index.md` | Current status, blocker and next action |
| `memory/decisions.md` | Settled decisions and their reasons |
| `memory/runs.md`, `memory/learnings.md` | Runs (including the spike) and durable findings |
| `memory/notes/` | Daily notes per node |
| `memory/scratch/` | Fact notes with sources and caveats; promote results from here |

Keep the labels (Decided, Adopted, Proposed, Open) until an item is implemented or recorded in `decisions.md`. Update milestone status from evidence (outputs, tests, benchmarks and comparisons), not from intent.
