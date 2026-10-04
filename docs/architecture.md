# Architecture

Date: 2026-10-04  
Status: design proposal. The engine/UI approach (the contracts in §2 and §8–§9) was adopted on 2026-10-04 after the map spike and is recorded in [decisions.md](../memory/decisions.md); the items it lists as open stay open. None of the components described here is implemented. What exists is the scaffold status CLI, project memory and the `govdata/` catalogue. Scope, milestones and acceptance gates are in [PLAN.md](PLAN.md), which takes precedence where the two documents differ.

Labels follow the plan: **Decided** means Sushrut said it or `decisions.md` records it. **Adopted** marks the engine/UI design: decided on 2026-10-04 (`decisions.md`), with its listed open items still open. **Proposed** marks Claude's engineering proposals that still need confirmation. **Open** marks questions nobody has settled. Figures are facts from the linked scratch notes and hold as of the dates recorded there. None of them is a model output.

## 1. Scope and design choices

The application simulates the residents of a chosen map region through one year: where they live, with whom, what they earn and own, how they spend and travel each day, and which life events change their circumstances. It reports commute burden and compares simulated aggregates with official statistics without republishing them ([PLAN.md §1–3](PLAN.md)).

| Choice | Status | Basis |
|---|---|---|
| Households and persons are the core state; vehicles, trips and costs derive from them | Decided | Sushrut: "i want to model a person's life" |
| Desktop app for macOS and Windows only; no mobile or web | Decided | Sushrut, 2026-10-03 ([direction note](../memory/scratch/direction-2026-10-03.md)) |
| The region is any map snip | Decided | Sushrut: "the cities doesnt matter.. its like i get should add a snip of the map ..." |
| One simulated year, with aggregates compared with government figures | Decided | Sushrut: "basically we model an year with a particular population distribution ..." |
| All life events within the year, each a knob with a data-backed default; vehicle acquisitions from regional sales or registration data | Decided | [decisions.md](../memory/decisions.md), 2026-10-04 |
| Commute burden covers time, money, income share, unpredictability and "all the issues" | Decided | decisions.md, 2026-10-03 |
| A demand layer (households, persons, plans) kept separate from a movement layer (vehicles, transit), linked by person→vehicle and household→vehicle IDs | Proposed | Every person-based tool studied does this ([population-sim-facts](../memory/scratch/population-sim-facts.md)) |
| Movement fidelity rises by milestone: fixed baseline travel times (M4), queue or meso traffic with transit capacity (M6), microscopic focus areas (M8) | Proposed | No precedent was found that runs more than 100k concurrent vehicles with lateral movement and junction conflicts at real time on CPU, or lane-free lateral movement on any hardware; published SUMO city scenarios average 1k–21k vehicles on the road at once and relax junction logic ([city-scale-facts](../memory/scratch/city-scale-facts.md)) |
| A global open baseline for any snip, replaced by local data packs where official data exist | Proposed | [any-region-data-facts](../memory/scratch/any-region-data-facts.md) |
| Python engine on numpy columns with a multiprocessing pool; a runner process; a NiceGUI desktop UI; one MapLibre + deck.gl map component | Adopted 2026-10-04 | [engine-ui-architecture](../memory/scratch/engine-ui-architecture.md), [runs.md](../memory/runs.md) |
| A headless engine; the UI and the CLI are both clients of the same runner | Adopted | Every run can be reproduced from its `run.json` |
| Our own implementation; open-source projects are references only | Decided | [open-source-references.md](open-source-references.md) |
| Government files stay local; only manifests and our comparisons are tracked | Decided | [govdata/README.md](../govdata/README.md) |

Not claimed: microscopic lateral dynamics before M8, live data feeds, Google content in simulation inputs, or bit-identical results across macOS and Windows.

## 2. Data contracts

The UI and the engine share three contracts only: the scenario schema, the run-folder format and small progress messages (Adopted). The engine reads its inputs from region workspaces and from the `govdata/` manifests.

### 2.1 Region workspace (Proposed)

```text
data/regions/<name>/        # git-ignored (anchor the rule as /data/)
  region.json               # name; study_area (bbox first, polygon later); buffer (context buffer, size Open);
                            # WGS84 plus a local metric CRS; IANA timezone (Asia/Kolkata for Indian regions); created_at
  manifest.json             # one entry per layer, see below
  layers/*.parquet          # GeoParquet: buildings, places, land use, network
  osm/*.osm.pbf             # clipped extract
  basemap.pmtiles
  derived/                  # parameters derived from govdata/ (ward marginals, fitted rates)
  overrides/                # manual corrections, each with author, date and reason
```

The study area is what results describe. The buffer holds the context needed for trips that leave or enter it (External zones in §4; Region boundary in §6).

Each manifest entry records:

- the source, with its release ID or URL;
- retrieval time, bounds, row count and content hash;
- licence, attribution text, and share-alike and non-commercial flags;
- reference period and finest geography.

Overture deletes releases after about 60 days, so the release is pinned and the fetched files become the snapshot. Protomaps keeps its daily builds for about a week, so the basemap is copied, never hotlinked.

Local data enter only through `govdata/<dataset-id>/manifest.json`. A region stores derived parameters together with the dataset IDs and vintages they came from. It never holds copies of `raw/` files, and a file without a manifest entry must not feed a model (Decided rule).

Every synthesised attribute carries a provenance tier: **local** (official data for this region), **national** (state or all-India data) or **donor** (a global layer or another country's parameters). Run reports summarise how local each attribute is. Overrides are a separate layer, so refetching a source never erases manual corrections.

### 2.2 Scenario schema and knobs (Proposed details)

Sushrut asked for knobs for everything, including time steps and behaviours (Decided); the details below are Proposed. One Pydantic model is the source. It emits JSON Schema 2020-12, and the NiceGUI forms are generated from that schema in Python. Custom metadata uses the `x-` prefix, which the JSON Schema project adopted in a 2023 ADR; the next core spec rejects other unknown keywords ([ui-platform-facts](../memory/scratch/ui-platform-facts.md)).

| Keyword | Use |
|---|---|
| `minimum`, `maximum`, `default` | Inclusive bounds and the data-backed default. Sliders need all three, so an `Optional[float]` never becomes a slider |
| `x-unit` | Unit shown in the UI and written to `run.json` |
| `x-group` | Form grouping, e.g. `life_events.vehicles`, `costs.fares` |
| `x-tier` | `expert` (numerical settings), `behaviour` (behaviour and life-event rates), `viewer` (display pacing) |
| `x-source` | Dataset IDs and vintage behind the default |
| `x-live` | May change during a run; true only for viewer-tier knobs |
| `x-soft-min`, `x-soft-max` | Plausible range from the evidence. Values outside it are allowed and flagged |

- Cross-field rules stay in Python validators. Examples: event windows lie inside the simulated year; the representative-day set covers every day type with non-zero weight; a soft range lies inside its hard range.
- Validate with `model_validate_json`, because strict mode on a `json.loads` dict rejects lists for tuple fields. Turn off `allow_inf_nan`, which defaults to true ([m1-context-brief](../memory/scratch/m1-context-brief.md) D4).
- Each default records its value, unit, source, vintage, geography and status (`data`, `assumption` or `departed`). A knob moved away from its data-backed default is marked "departed from data" in the UI and in `run.json`.
- Defaults derived from MoSPI Category B unit data are not committed until the "derived products" question is settled ([PLAN.md §4](PLAN.md)). They live in the region's git-ignored `derived/` folder, and the schema refers to them by dataset ID.
- Every life event has an on/off switch (Decided). With all of them off, the year runs with fixed circumstances.
- Viewer-tier knobs must never change outputs; a test compares output hashes to check this. Changing any other knob creates a new scenario version and a new run.
- The scenario hash is the sha256 of its canonical JSON (sorted keys, no NaN).

Top-level sections:

- region and data pack;
- simulated year and calendar;
- population synthesis, income, and vehicles and owner-drivers;
- activities and modes;
- costs (fare and price tables by "as of" date);
- life events;
- movement (M6) and sampling;
- ensemble (seeds and uncertain inputs);
- outputs.

### 2.3 Run folder (Adopted contract; Proposed contents)

```text
outputs/<run-id>/           # git-ignored (anchor as /outputs/)
  run.json
  population/               # households, persons, dwellings, vehicles at start and end (Parquet)
  events.parquet            # life events: day, type, person/household IDs, cause
  daily/                    # aggregates per day and per day type
  trips/                    # person trips for representative days
  metrics/                  # burden tables and distributions
  layers/                   # binary map layers (Arrow) plus a layer index
  validation/               # comparison tables, validation and limitations report
  logs/                     # run logs; not part of the deterministic outputs
```

`run.json` holds a deterministic block and a volatile block:

- **Deterministic block:** the resolved scenario with defaulted values marked; the seed and random-stream registry; scenario and `uv.lock` hashes; Python version and platform; source revision (commit, dirty flag and source-tree hash); data-pack manifest hashes; and price "as of" dates.
- **Volatile block:** run ID, creation time, elapsed time, hardware, worker count and status (`running`, `complete`, `partial` or `failed`).

**Deterministic outputs** are every file in the run folder except `run.json`'s volatile block and log files. Tests compare their sha256 hashes.

Every output file carries its own `format_version`. Each file is written under a temporary name and renamed when complete. JSON uses sorted keys and `allow_nan=False`. Hashes cover uncompressed payloads, because gzip headers embed the file name and time.

### 2.4 Progress and cancel (Adopted)

The runner puts small messages on a `multiprocessing` queue, each with the run ID and a sequence number:

- `started`;
- `progress`, with day, percentage and event count;
- `layer_ready`, with the relative path of a new layer file;
- `warning`;
- one final `done`, `cancelled` or `failed`.

Cancel is a `multiprocessing.Event` that the runner checks between days and between household chunks (Proposed). The day in progress is discarded; the run keeps every committed day and is marked `partial`. Messages only inform the UI: the run folder is the source of truth, so a lost message never changes results. Large data never goes through the queue.

## 3. Population model

### 3.1 Entities (Proposed)

| Entity | Main attributes | Main sources |
|---|---|---|
| Building | Footprint, residential or not, modelled floors, ward or zone | Overture, Google Open Buildings v3, Microsoft; EMC-BUILT non-residential layer |
| Dwelling | Building, tenure (owned or rented), rooms | Census 2011 HL-14 |
| Household | Dwelling, size, composition, income, vehicles | HL-14, HL-05, PLFS, HCES; HH-01 (check) |
| Person | Household, age, sex, relationship, education stage, work status and type, earnings, workplace or school, driving licence (modelled; no local source yet) | Census PCA, PLFS, UDISE+, AISHE |
| Vehicle | Owner, class (car, two-wheeler, bicycle, auto, taxi), fuel, private or livelihood use, home parking | HL-14, VAHAN, CMPs |
| Job slot, school seat | Place, sector or stage, capacity | 6th Economic Census (2013), places, UDISE+, AISHE |

### 3.2 Homes in buildings

Heights are almost absent for Indian cities: in central Bengaluru, Overture has a height for about 0.05% of buildings and a floor count for about 1%. Floors per building, and with them persons per building, are therefore a modelled, uncertain variable drawn in ensembles. Residential use comes from the EMC-BUILT 10 m non-residential layer (2022, about 88% overall accuracy), because only about 2% of Overture buildings have a class tag. Proposed: allocate each ward's households to its residential buildings in proportion to footprint area × modelled floors.

Ward totals come from the Census 2011 PCA and HL-14 tables for BBMP, Greater Mumbai and GHMC. There are no open 2011 ward polygons, and ward boundaries have changed: GBA's 369 wards, the 2026 GHMC split, and Mumbai census wards that do not match BMC wards. Each region therefore needs a crosswalk (method Open). Scaling 2011 totals to the simulated year needs a population projection; growth within the city since 2011 is unknown and is treated as uncertainty.

Without a local pack, the fallbacks are:

- WorldPop or GHS-POP grids. WorldPop Global2 R2025A (alpha) uses state-level inputs only for India (35 units), so density inside a city is purely modelled; it comes as national files only, about 0.75 GB per layer;
- NFHS/DHS household sizes;
- GLOPOP-S as a global fallback (about 1 km resolution, reference year 2015, wealth quintiles rather than income).

### 3.3 Households and persons

Seed-based synthesis (IPF, IPU, PopulationSim) needs household microdata. Sample-free methods cannot build households containing persons without detailed conditional tables. The candidate seeds all have limits:

- Census microdata can be used only on a workstation.
- IHDS-II urban seeds are small (about 850–1,160 households per state).
- PLFS 2025 is large (2.7 lakh households) and has earnings, but no vehicles.

BharatSim's Mumbai population (IPU on IHDS-II, plus Census 2011 and CTGAN) is the nearest precedent ([population-sim-facts](../memory/scratch/population-sim-facts.md)).

Proposed: use PLFS 2025 households as the seed and fit them with IPU to ward marginals from the PCA and HL-14. HL-14 carries ward household-size bands (1 to 5, 6–8 and 9+), rooms, tenure, married couples per household and vehicle shares. HH-01 City household-size counts stay held out as a check, as `catalog.yaml` assigns. PLFS weights are mandatory, and each million-plus city is its own urban stratum. Joint households follow HL-05, where 10–14% of urban households have two or more couples. Until MoSPI access is settled, a labelled fallback seed is used (choice Open). The validation ledger (§10) records which marginals are controls.

### 3.4 Income (Proposed recipe)

No open dataset gives an income distribution for India below state or district level ([commute-burden-facts](../memory/scratch/commute-burden-facts.md), [any-region-data-facts](../memory/scratch/any-region-data-facts.md)). The proposed recipe:

1. Take the distribution's shape from PLFS 2025 unit data: person earnings (regular wage for the previous month, gross self-employment earnings for 30 days, casual daily wages) plus household non-labour income (`inc_tot`, new from January 2025). The fallback is HCES state consumption inequality, labelled as such, because the urban consumption Gini (about 0.29) is far below the income Gini (0.38–0.56).
2. Add a separate top tail.
3. Use fine-scale proxies, such as HL-14 asset shares, only to rank places within the region. Night lights mostly track density, not income, so use them only with care. Meta RWI is non-commercial, and its error is comparable to the spread within a city.
4. Draw households from the calibrated distribution. Never assign welfare from area covariates alone.

Self-employed earnings are gross, so owner-drivers' vehicle costs are deducted separately. Tail and ranking parameters are uncertain inputs in ensembles. The module sits behind one interface, so the NSS 80th-round National Household Travel Survey (fieldwork Jul 2025–Jun 2026; not released and not in the 2026-27 release calendar; designed for district origin–destination, so it may not support within-city checks; terms may be non-commercial) or the planned national household income survey (2026-27; no confirmed fieldwork dates; results mid-2027 per media only) can replace it later.

### 3.5 Vehicles and owner-drivers (Proposed)

HL-14 gives each ward's share of households owning a car, a two-wheeler or a bicycle. It has no joint car × two-wheeler table and no count of vehicles per household. Ownership is therefore modelled conditional on income, household size and number of workers, and fitted to the ward shares (build). HCES vehicle possession serves as a check only if it is not used to build. Census 2027 houselisting (Apr–Sep 2026) asked about vehicles, with bicycle grouped with two-wheeler; no table release date is known.

None of the frameworks studied models owner-driven autos and taxis parked at home. The Mumbai CMP reports autos as 61% owner-driven and 39% hired, and taxis as 40% owned and 60% hired. A livelihood vehicle is attached to a person with a shift schedule. Net earnings are fare revenue minus fuel, maintenance, insurance, and rent or loan payments. The evidence on rent and idle share is weak, so both are scenario ranges. Bengaluru's 3.6 lakh registered autos (2025) exceed its permit cap of 2.55 lakh, so the active fleet size is itself uncertain.

A household vehicle serves one user at a time and must be where its driver starts. Vehicles are allocated per tour in the decide step, with no double-booking (ActivitySim allows double-booking; MATSim teleports cars by default). Home parking is assumed. Explicit parking search is out of scope: it is expensive, which is why other models use it only for small areas.

## 4. Daily life and travel

**Roles and day types (Proposed).** Each day, a person's role follows from their attributes:

- worker: regular-wage, casual or self-employed (including owner-drivers);
- school or college student;
- not in the labour force;
- retired;
- young child.

Day types are working weekday, Saturday, Sunday or holiday, school-holiday weekday, and rain variants, weighted from the calendar (§5). TUS reports normal days and off-days separately, which maps onto working and non-working day types.

**Schedules.** India TUS 2019 and 2024 (about 450k persons, half-hour slots, location recorded as home, outside or non-fixed) give activity timing and out-of-home windows. They give no destinations or modes, cover one day per person, and miss activities under 10 minutes. Proposed: build schedule templates by cell (age band, sex, work or student status, vehicle access, day type), with fallback cells where samples are thin. OMoSim's Korea parameter set is a precedent for a national parameter set. India has no multi-day diary, so a person's day-to-day variation is modelled. TUS is MoSPI Category B unit data, so only derived parameters are stored.

**Destinations.** Workplaces and schools are fixed at assignment and change only through life events; the resolve step assigns job slots and school seats.

- **Jobs:** there is no open employment grid. Use 6th Economic Census (2013) worker counts by ward and enumeration block where available; otherwise use non-residential building area and place density.
- **Schools and colleges:** schools come from the UDISE+ school files (ward, pincode, enrolment by class and gender) and UDISE coordinates (2021 vintage). Colleges come from the AISHE directory, geocoded by address or PIN. Junior and PU colleges come from UDISE+ or board lists ([education-and-timing-facts](../memory/scratch/education-and-timing-facts.md)).
- **External zones (Proposed; buffer size and method Open):** destinations outside the study area are external zones with travel times, so residents who work or study outside the snip keep their commutes. People who live outside but work or study inside are boundary demand. The commute burden of a small snip depends on this treatment.
- **Other trips:** discretionary destinations use gravity choice over Overture Places and OSM. OMoSim's gravity model fits well at 5 km and poorly at 500 m, so realism is claimed at ward or zone scale, not for individual shops.

**Modes.** Walk, bicycle, two-wheeler, car (driver or passenger), auto, taxi or aggregator, bus, metro and suburban rail. Mode choice is a multinomial logit taken from Indian studies (which study is Open). It is constrained by:

- whether a household vehicle is available;
- driving licence;
- fares, including free-bus eligibility;
- per-class spatial restrictions (Mumbai's island city has almost no autos).

**Routing (Proposed).** Routes use OSM drive and walk networks. Only 2–9% of drive edges carry a `lanes` tag, so capacities are inferred and flagged. Customisable contraction hierarchies (CCH) are the proposed method:

- routingkit-cch (BSD-2) routed 400k random origin–destination pairs on Bengaluru's 199k-node, 495k-arc graph in about 13–15 s on one thread, with full re-weighting in about 27 ms;
- pandana took about 4 s on 14 threads but is AGPL;
- per-vehicle Dijkstra took 2–20 h and is ruled out.

**Transit.** GTFS is used where a usable feed exists. The TGSRTC and HMRL feeds are official: redistribution is allowed with attribution, but the download sits behind a Google Form. Bengaluru's official feed states no licence, and the unofficial feed is ODbL. Where no usable feed exists, a labelled synthetic timetable is used. M4 uses fixed baseline travel times by period of day, labelled as such. If a speed index sets those times, it becomes build data and cannot also check them.

**Costs (Proposed).** Fares and prices live in versioned tables. Each row records region, operator or mode, eligibility, effective-from date, source and "as of" date. The tables cover:

- public transport fare stages, slabs and passes;
- auto and taxi meter fares (treated as a floor), with night surcharges;
- aggregator surge (capped at 2×);
- fuel and maintenance per km by vehicle class;
- monthly ownership costs (loan, insurance, registration and road tax), with CEEW's per-km figure (Jun 2025) as the citable source. Its 14 km/L figure is for an average Rs 9.5 lakh car, not a small car (small cars about Rs 5.1–5.8/km on fuel at 20–22 km/L, as of about 3 Oct 2026).

Ownership costs are a household fixed cost, not a trip charge. MATSim's `dailyMoneyConstant` is charged only on days a mode is used, so it cannot represent them.

Free-bus rules are evaluated per person:

- **Karnataka Shakti** (since 11 June 2023): free for women and transgender people domiciled in Karnataka on non-AC buses; Vajra and Vayu Vajra are excluded.
- **Telangana Mahalakshmi** (since 9 December 2023): free for women and third-gender residents on City Ordinary and Metro Express buses.
- **Mumbai:** BEST has no equivalent scheme (an election promise is unconfirmed); some MMR operators give women 50% off.

Pending fare changes (BMTC, Namma Metro) are knobs.

## 5. The year and life events

**Decided** ([decisions.md](../memory/decisions.md), 2026-10-04): life events happen within the simulated year, and every event type is in scope: vehicle purchase and sale; getting, losing and changing jobs; moving house; births and deaths; marriage and leaving home; education stages (primary, secondary, junior college, senior college); income changes; migration into and out of the region, including moves for jobs. Each event is a knob with a data-backed default and can be switched off, which gives fixed-circumstances runs. Vehicle acquisitions follow regional sales or registration data where possible. The population is open at the region boundary, so accounting covers born, died, moved in and moved out. The mechanics in §5.1–5.3 are Proposed.

### 5.1 Calendar (Proposed)

The simulated year is Open. 2025 is suggested because PLFS 2025 is the first calendar-year round and has a report for 46 million-plus cities ([year-validation-facts](../memory/scratch/year-validation-facts.md)). Each of the 365 days gets a day type and calendar flags:

- gazetted and state holidays (private-sector holidays differ);
- state school terms and board result dates;
- festival windows;
- monsoon onset and withdrawal;
- daily rainfall from IMD's 0.25° gridded data.

Simulated days and times are local to the region's timezone; run timestamps are UTC.

A typical meteorological year is not used for rain. The rain effect is a knob with a wide range: FHWA gives a 10–11% capacity drop for light and moderate rain, two Mumbai roads showed travel times 8–140% higher, and Indian evidence is thin.

Proposed hybrid: the household and money layer runs every day, while detailed travel runs on representative days for each day type and is then reweighted. Unpredictability (§7) needs day-to-day variation, so the number of distinct travel days is a trade-off to be measured at M5.

### 5.2 Event engine (event scope Decided; mechanics Proposed)

Each event type defines its eligibility, an annual rate by attributes, a seasonal profile, its consequences, its dependencies and an on/off switch. An annual rate λ becomes a daily probability `1 − exp(−λ·Δt)` with Δt one day, scaled by a seasonal profile normalised to keep the annual total.

Dated events fire on their date:

- school transitions at each state's reopening date and age rule (2025: KA 29 May; TG 12 Jun, junior colleges 1 Jun; MH 16 Jun, Vidarbha 23 Jun);
- private-sector pay rises in April;
- dearness allowance in January and July;
- minimum-wage top-ups (KA 1 April; TG 1 April and 1 October; MH 1 January and 1 July);
- retirement at each state's retirement age.

Events that claim scarce resources (dwellings, job slots, school seats) go through the resolve step.

Events are chained by dependencies:

- Marriage forms households and moves people. It accounts for 26% of Mumbai women's and 23% of BBMP women's recent migration.
- A birth changes household size.
- A job change alters the commute and can trigger a move or a vehicle change.
- Vehicle changes follow household composition, licences, employment and income (UK evidence; SILO's keep/add/remove model and STELARS's add/dispose/replace model).
- Moves are constrained by housing cost, commute time and transport-cost share, as in SILO.

Do not copy SILO's yearly income adjustment: on its master branch it applies zero change, which silently freezes incomes.

| Event | Default source ([life-events-facts](../memory/scratch/life-events-facts.md)) | Timing | Main caveat |
|---|---|---|---|
| Vehicle purchase | VAHAN dashboard flows by RTO, month, class and owner type; OpenCity CSVs | Festive window (about 20% of two-wheeler and 17% of car retail sales in 42 days); January | A registration is not a household acquisition: households are about 76% of car-type registrations in Bengaluru and about 67% in Mumbai (derived). Owners may register at any RTO in the state. Apportioning RTO flows to a snip uses Mumbai's official PIN→RTO list (2017) joined to PIN polygons and Hyderabad's mandal/locality→RTA lists; no jurisdiction list was found for Bengaluru, so its apportioning is an uncertain modelled step. 2025 is split by the GST cut on 22 Sep. Telangana's history is on the VAHAN dashboard (TG CY2025: 1,031,169 registrations), so Hyderabad 2025 can be calibrated; this supersedes earlier notes saying Telangana joined VAHAN only in March 2026. FADA's 2025 figures exclude Telangana. OpenCity mirrors cover only Bengaluru (10 RTOs) and Mumbai (4 RTOs), 2021 to Jan–Jun 2025, without owner type |
| Vehicle sale or exit | Used-car and registry-transfer ratios; VAHAN archive status | — | 1.4 used cars per new car nationally vs 0.36–0.40 registry transfers per new registration: the knob must state which it targets |
| Job start, loss or change | PLFS 2025 panel, pooled by state or across million-plus cities; Bhattacharya (2021); EPFO; Aon | First jobs May–July; job switches April | No employer change or reason recorded; city samples are thin; EPFO and Aon cover the formal sector only |
| Income change | Aon, dearness allowance, minimum wages, CPI-IW | Dated | Reusing Labour Bureau data needs permission |
| Retirement | State retirement ages; PLFS participation by age for informal work | Central-government staff retire at the end of their birth month | — |
| Birth | SRS 2024 urban age-specific and marital fertility; fertility knob from about 1.3 (SRS 2024) to 1.7 (NFHS-6 urban) | Per-day rates; Bengaluru registrations 19% above uniform in December | Sex ratio at birth is 885–926 girls per 1,000 boys |
| Death | SRS 2024, smoothed, or the 2020–24 life tables | Modest seasonality | Life tables are inflated by COVID deaths (about 11–14% in MH and TG, about 0 in KA); CRS district counts are not city rates |
| Marriage | SRS never-married shares | Community calendars (Chaturmas, Shravana, Aashada, Ramadan, Muharram) | Synthetic-cohort rates overstate marriage while the marriage age is rising |
| Leaving home, household split | HL-05; IHDS (at least 1.2% of urban households a year) | Knob range about 1–3% a year | — |
| Moving house | World Bank Mumbai 2019 survey: 2–3% of households a year as a floor, 2.9–4.0% from longer duration bins | Families with school-age children April–early June (knob) | No Bengaluru measurement; about 60% of Bengaluru households rent |
| Education stage | State entry ages; 2025 board pass rates; AISHE; NSS out-of-state study | Reopening date by state (2025: KA 29 May, TG 12 Jun, MH 16 Jun); results April–May; supplementary exams June–July | Promotion rules at classes 5 and 8 not yet researched; stream mixes exclude CBSE/ICSE/IB; NSS out-of-state shares (KA ~15%, MH ~7%, TG ~5%) are flagged unreliable at state level (KA has 89 cases) |
| Moving in | Census 2011 D-03 (lived there under 1 year): Greater Mumbai 1.44%, BBMP 2.72%, GHMC 1.37% (higher after adjusting for missing durations) | — | 2011 vintage; PLFS 2025 has no migration questions |
| Moving out | No city-level rate: assumed, and balanced against population growth | — | Includes moves out for jobs. A MoSPI migration survey runs July 2026–June 2027 |

### 5.3 Open-population accounting (Decided requirement; method Proposed)

Every day, for persons and households: present = previous + born − died + moved in − moved out. Vehicles balance in the same way across bought, sold, scrapped, moved in and moved out. IDs are never reused, and people who leave stay in the outputs with their exit day and reason. In-migrants are synthesised from a donor distribution by reason, such as marriage or a job (Proposed). M4's typical days use the same day keys as the year's representative days. With all events switched off, household and person state is unchanged across the year, and each representative day's outputs are byte-identical to the M4 run of that day type.

## 6. Movement

**M4: fixed travel times.** Door-to-door times come from routed baseline times by period of day. One family's car changes nothing city-wide. The one-family comparison therefore replays that household's days with and without a car, against the same travel times and with the same keyed random draws.

**M6: city traffic and transit capacity (Proposed).** The road model is a queue or mesoscopic model (which one is Open). Precedents at this scale:

- UXsim (Python, MIT) ran 968k Chicago vehicles in 37 s, with a peak of about 322k in the network at once, using 30-vehicle platoons and 30 s steps.
- MATSim QSim, POLARIS and SUMO MESO also handle this scale.

All of them give up lateral position, lane-free filtering and junction conflicts. Indian building blocks for meso models exist: PCU-based capacities with seepage (as in MATSim Patna), a heterogeneous cell transmission model (CTM) and a 2D LWR model. Two-wheelers are about half of concurrent vehicles in Bengaluru and Hyderabad, so how they are treated matters.

- **Routing:** routes are precomputed with CCH, and a share of vehicles is rerouted periodically. Rerouting 5% of vehicles every minute cost about 0.75 s of CPU per simulated minute. Equilibrium assignment needs tens to hundreds of full runs, so it is not done inside a run.
- **Transit:** vehicles carry a capacity, follow timetables and queue passengers. Denied boardings and extra waits are recorded. Fleet size is a knob (held fixed or scaled). Defaults come from operator figures: BMTC about 7,000 buses, BEST about 2,740, TGSRTC about 3,200.
- **Autos and taxis:** they run owner-driver shifts, including empty and cruising trips, under per-class spatial restrictions.
- **Sampling (Proposed):** sample private vehicles and keep fleets and public transport at 100%, as MATSim's FISS does; scale flow and storage capacity by the sample fraction. BEAM found that pooled ride-hail efficiency levels off only at about 40% samples.
- **Accounting:** no vehicle or passenger is dropped or teleported without being counted. Gridlock is reported, not hidden; for comparison, SUMO's BeST scenario logged 2,332 teleports.
- **Region boundary:** the region needs a buffer and gates for through and outbound trips, building on the buffer and external zones used from M2 (§2.1, §4). Arrivals at the boundary respect the space available to receive them, and sensitivity to buffer size is reported.

**M8: microscopic focus areas.** The design in [m1-context-brief.md](../memory/scratch/m1-context-brief.md) is the starting proposal: IDM car-following, ballistic update, a decide/resolve/integrate tick, swept collision checks and conservation accounting. Decisions D1–D3, D6 and D7 remain open for M8; D4 is settled by the adopted engine design (numpy and Pydantic); D5 is moot because the repository has commits; D8 (uv) and D9 (Python version) are project-wide and tracked in [PLAN.md §8](PLAN.md). The 2026-10-03 version of this document described the rest of that engine. It is in Git history (commit `931da1a`) and will be revised when M8 starts. It covered:

- continuous lateral position;
- one physical road surface shared by legal and wrong-way movement;
- junction conflict zones;
- violations modelled as opportunities.

Hybrid micro/meso simulation is an established technique (Aimsun; Mezzo with MITSIM, 2004), but the boundary between the two models is hard:

- meso→micro: the boundary must generate a lane, speed and entry gap;
- micro→meso: blocking and spillback must be passed back;
- for lane-free traffic, no published method was found that generates a vehicle's lateral position on entry.

## 7. Outcomes: commute burden

The burden dimensions are Decided: time, money, income share, unpredictability and "all the issues". Ownership costs are inferred from "all the issues" and still need confirmation. The definitions below are Proposed. The reference points are candidate checks from [commute-burden-facts](../memory/scratch/commute-burden-facts.md); any of them used to build the model becomes an internal check.

| Dimension | Proposed definition | Reference points |
|---|---|---|
| Time | Door-to-door, per trip and per day: access walk, wait, in-vehicle, transfer and egress time; out-of-vehicle time shown separately | TUS 2024: commuters spend 77 min (men) and 67 min (women) per day, both ways; Bengaluru 2018: 42.5 min one way |
| Extreme commute | Share of people with a one-way trip over 60 or 90 minutes | — |
| Money | Fares and running costs per trip, plus monthly ownership costs. Owner-drivers' business costs reduce their earnings instead | HCES: urban conveyance is 8.46% of consumption |
| Income share | Monthly household transport spending ÷ household income | Common limits 10–15%; SDG 11.2.1: at most 5% for the poorest fifth; NUTP 2014 sets no target; Bengaluru CMP: 7.6% |
| Generalised cost | Money + value of time × time, with an income elasticity below 1 (World Bank: 0.696); also reported with one average value of time | Mumbai 2019: in-vehicle Rs 46–49/h, out-of-vehicle Rs 85–87/h |
| Unpredictability | Spread of a person's door-to-door time for the same trip across the year, e.g. a high percentile minus the median (percentile Open) | Needs M5 |
| Crowding | Load factor of the vehicles boarded, standing time, denied boardings, extra wait | Needs M6 |
| Access and participation | Living within 500 m of a bus stop or 1 km of rail or metro; trips per person per day; share not travelling | SDG 11.2.1; Bengaluru CMP: 1.24 trips per person per day |

Results are distributions, not one score. They are broken down by income fifth, gender, ward and household type (car, two-wheeler, none, livelihood auto or taxi), with the share of people above each threshold and the winners and losers between scenarios run on the same population. Money is always reported alongside time, access and participation, because low spending can mean walking or not travelling. Every figure is labelled synthetic and carries the "as of" dates of its prices. Further issues, such as heat and rain exposure, safety at night and seat availability, are Open.

## 8. Execution (Adopted; details Proposed)

**State.** People and households are stored as numpy columns, one per attribute, not as per-person Python objects:

- persons are sorted by household, with offset arrays mapping each household to its persons;
- categories are small integer codes;
- IDs are stable integers.

Measured on this M3 Max: about 4M simple pure-Python updates per second per core, against about 87M with numpy (a trivial vectorised update; verifier measurement with system Python 3.9). A full Bengaluru is about 1.3 crore persons. On disk, state is stored as Parquet and Arrow.

**Processes.** The NiceGUI app runs in the main process. Each run gets its own runner process, which owns a multiprocessing pool. The runner is a non-daemonic process, because a daemonic process cannot start a pool; the job manager terminates it explicitly on cancel timeout and on app exit. macOS and Windows use the spawn start method, which requires:

- top-level, picklable worker functions;
- an `if __name__ == "__main__":` guard;
- `multiprocessing.freeze_support()` in packaged builds;
- large arrays in `multiprocessing.shared_memory`, passed by name, shape and dtype rather than pickled.

Threads handle I/O only (file writes, fetches), since Python 3.12's GIL limits CPU work in threads. Work is split by household chunks within a day, by day types, by seeds and scenarios (as independent runs), and later by map tiles for traffic. Free-threaded CPython 3.14 (PEP 779) is a later option (Open). Compiled kernels (Numba, PyO3 with rayon, Cython) come only after profiling. Two caveats: rayon reductions are unordered, and Numba's parallel mode installed from pip on macOS arm64 gets only the workqueue layer (inferred from Numba's documentation).

**The day.**

1. **Decide** (in parallel): each chunk reads yesterday's frozen state and today's calendar. It returns proposals: plans, mode and vehicle use, event draws and claims on scarce resources.
2. **Resolve** (one deterministic pass): competing claims for dwellings, job slots, school seats, bus seats and, from M6, road space are settled in a fixed order with keyed tie-breaks, never by the order in which workers finish.
3. **Commit**: apply accepted changes, update the accounts, write daily aggregates and check invariants: the population balances, nobody is in two places, each vehicle has at most one user, no capacity is negative, and all values are finite.

**Randomness and reductions.**

- **Keyed streams:** draws come from numpy `SeedSequence(root_seed, spawn_key=(stream, household, day))`, and the streams are registered in `run.json`. Each key component must be below 2**32, or keys alias.
- **Version pinning:** numpy Generator output is not stable across numpy X.Y releases, so the `uv.lock` hash is part of a run's identity.
- **Scenario comparisons:** scenarios use the same synthetic population and keyed draws, with one stream per decision or event type and keys built from stable IDs. People created during a run get IDs keyed by (day, event type, ordinal), not a global counter. Unchanged decisions then draw the same numbers. Where a scenario changes which events occur, draws can diverge, and comparisons report this.
- **Fixed-order reductions:** chunk results merge in chunk order. Float sums use `math.fsum`, because Python 3.12 changed `sum()` for floats.
- **Determinism target:** byte-identical deterministic outputs (§2.3) on the same machine, Python version and `uv.lock`, for any worker count. This is tested with one worker and with several, and with different `PYTHONHASHSEED` values.
- **No cross-platform identity claim:** libm functions differ, and clang fuses multiply-add by default while MSVC does not.

## 9. UI and map (Adopted; open items listed)

```text
NiceGUI app (main process, asyncio)
 ├─ knob forms (from the scenario schema) · map component · charts and validation views
 └─ JobManager ─ submit(scenario) ─► Runner process ─► multiprocessing pool
               ◄─ progress queue (%, day, events, "layer ready") ─
               ─ cancel event ─►
Runner writes outputs/<run-id>/; the CLI uses the same Runner
```

NiceGUI 3.17.1 (MIT, Python 3.10–3.14) runs in native mode through pywebview, which uses WKWebView on macOS and WebView2 on Windows. The app binds to `127.0.0.1`. Runs do not use `run.cpu_bound`, because a simulated year needs progress reporting and cancel. Charts use NiceGUI's ECharts or Plotly elements.

**Map component.** The map is one `ui.element` subclass with `component='map.js'`, a small Vue file wrapping MapLibre and deck.gl.

- **Python → JS:** set props and call `update()`, or `await run_method(...)`, which can return a value.
- **JS → Python:** `$emit` in JS, handled by `.on(...)` in Python. Used for a drawn snip, a clicked feature or a viewport change.
- **Layers:** declared in Python as deck.gl JSON specs (the pydeck format) and rendered by `@deck.gl/json`, so adding a layer is a Python change.
- **Layer data:** binary files in the run folder, served by the local server and handed to deck.gl as binary attributes. The websocket is not used for bulk data: Python→Chromium WebSocket reached about 270 MB/s, and the pywebview bridge carries only JSON or Base64.
- **No Node toolchain:** the prebuilt deck.gl 9.4.0, `@deck.gl/json` 9.4.0 and MapLibre 5 bundles are vendored.

On macOS, the spike rendered 1,717 stores in 41 ms, 27,986 building footprints in 0.4 s and 300k points in 1.0 s ([runs.md](../memory/runs.md)). Its lessons are requirements for the real component ([learnings.md](../memory/learnings.md)):

1. Call `map.resize()` from a ResizeObserver.
2. Set `pickingRadius` to about 5 px.
3. Bind to `127.0.0.1`.
4. With `_normalize: false`, give binary polygons closed rings with a consistent (clockwise) winding order.

Development note: restart the app after editing the component's JavaScript.

Not yet tested: Windows with WebView2, snip drawing (library not chosen), the offline basemap, and millions of points. Windows needs three safeguards:

- If the WebView2 runtime or .NET is missing, pywebview silently falls back to IE11/MSHTML, which has no WebGL; the app must check for this and install the WebView2 Bootstrapper.
- GPU-less VMs and blocklisted drivers get no WebGL, since Edge 144+ no longer falls back to SwiftShader.
- Windows ARM64 needs the x64 build.

pywebview's licence is not yet recorded.

**Layers and level of detail (Proposed).** Planned layers: population, income, places and stores, buildings, jobs, schools, transit, burden and events. A ScatterplotLayer stays smooth up to about 1M points (for static data; layers that change every frame need binary attributes or they stutter at a few thousand items), and a metro has more than 10M people. The map therefore shows hexagon aggregates when zoomed out (h3-py is a candidate) and sampled dots for the viewport when zoomed in. On this Mac, deck.gl held 120 Hz at 400k–2M binary points; the limit is the data path, not drawing. Windows and integrated GPUs are unmeasured.

- Layers map rates rather than counts, with quantile classes and fixed colour domains.
- WebGPU disables deck.gl's GPU aggregation.
- People layers and tooltips say "synthetic". Aggregates are preferred, because zoomed-in dots imply false precision.
- Movement replay in M6 loads time-windowed chunks.

**Basemap and boundaries.**

- **Offline basemap:** each region gets a Protomaps PMTiles extract, read through MapLibre's pmtiles protocol and copied to our own storage. A Bengaluru extract at zoom 0–15 is about 33 MB. "© OpenStreetMap contributors" stays visible.
- **Prototypes:** may use online OpenFreeMap ([map-providers.md](map-providers.md)). `tile.openstreetmap.org` is not used.
- **India boundaries:** publishing a map of India that does not conform to Survey of India boundaries is an offence (Criminal Law Amendment Act 1961, s.2(2)). City views omit international and disputed boundaries (Proposed: the basemap style drops those layers). Any India-wide view uses the Survey of India Administrative Boundary Database.
- **Google:** the 2026-10-03 decision keeps Google Maps as a separate, optional display adapter. No supported desktop path has been found: Google ships no desktop Maps SDK, macOS WKWebView is not on Google's supported list for Maps JavaScript, and the API key cannot be meaningfully origin-restricted. It is therefore deferred, and whether it is wanted on desktop is Open (§12). The activation gates are in [map-providers.md](map-providers.md).
- **Video export:** Open. Sushrut asked about MP4 output on 2026-10-03; nothing has been decided. If it is added, it renders our own OSM basemap with attribution and never uses Google tiles.

## 10. Validation and uncertainty (rules Decided where marked; methods Proposed)

Methods and sources are in [year-validation-facts](../memory/scratch/year-validation-facts.md).

- **Build vs check (Decided, [govdata/README.md](../govdata/README.md) rule 5).** A dataset used to build the population cannot back a "matches" claim for the same quantity. Proposed extension: keep the ledger per variable, not only per dataset. Each quantity records its role (build, constraint or check), dataset ID, vintage and geography, and constraint quantities cannot back a "matches" claim either. Reports list the three groups separately (TAG M3.1 §3.3.8). [PLAN.md §4](PLAN.md) maps these roles onto the catalogue's dataset roles.
- **Comparison records** live in `govdata/<id>/comparisons/`. Each holds our value; the official value with its table, version and URL; the official margin of error, if published; and whether that dataset was used to build the model.
- **Like with like.** Match each survey's estimator, period and geography. Compare PLFS rates, not totals, because PLFS totals are ratios multiplied by population projections. Every target's geography needs a crosswalk (wards, districts, RTOs, police and transit areas).
- **Tolerances before results**, per layer, committed before that layer is fitted or compared. Use published RSEs or confidence intervals where they exist, and set explicit bands for administrative counts and fact sheets. Use GEH for hourly counts and SQV for daily counts.
- **Uncertainty.** Ensembles vary seeds and uncertain inputs (floors per building, income tail, out-migration, rain effect, ownership costs). Seed-only ensembles understate uncertainty: one UrbanSim study found 38% coverage for a nominal 90% interval. The number of seeds follows result stability or interval half-width.
- **Pattern-oriented checks** at several scales, with criteria set in advance and no tweaking until the numbers match.

| Layer | Candidate independent checks (only if not used to build) |
|---|---|
| Population | Held-out Census 2011 cross-tabs (SRMSE); HH-01 City household-size counts |
| Daily life | TUS participation; Census B-28 commute mode and distance (district level only); trip rates |
| Money | HCES conveyance share and vehicle possession |
| Year | PLFS 2025 city and district rates only for quantities not taken from the PLFS seed or panel (record which in the ledger); UDISE+ enrolment only if school seats are built from another source; VAHAN flows for a hold-out that contains no season or regime missing from the build data (see the GST and festive-window caveat in [PLAN.md §4](PLAN.md)). SRS is a build input for births and deaths, so SRS comparisons are internal checks |
| Movement | Akbar et al. speed indices (180 Indian cities); TomTom 2025 monthly congestion (car drivers only); ridership (approximate); CMP screen-line counts |

Each run writes a validation and limitations report. "Matches" is used only for independent data within the tolerance set in advance.

## 11. Performance and scaling gates (Proposed)

Scale anchors ([city-scale-facts](../memory/scratch/city-scale-facts.md)):

- A full Bengaluru is about 1.3 crore persons.
- Sushrut's stated scale is more than 4 lakh vehicles at a time, each with its own route.
- Peak concurrent vehicles, estimated with Little's law (uncertain by about 2×): Bengaluru metro 0.22–0.45M, Greater Mumbai 0.15–0.45M, all of MMR 0.25–0.6M, Hyderabad metro 0.25–0.55M.
- The peak hour carries about 12% of daily trips in Bengaluru and 5.5–6.6% in Mumbai.

Sizes are declared before measuring: the test fixture, the first region, and a synthetic scale test at full-Bengaluru order (about 1.3 crore persons). Map layers are measured at 1M points and at the first region's full synthetic population. Targets are set after the first measurement.

| Milestone | Measure on named hardware | Gate |
|---|---|---|
| M1 | Fetch time and size per region (Overture: about 97 MB read and 61 MB stored for 20×20 km of central Bengaluru) | A rebuild reproduces the content hashes |
| M2 | Persons per second, peak memory, wall time, at the declared sizes | Identical deterministic outputs with one worker and with N workers; targets set after the first measurement |
| M3 | Layer load time and frame rate at 1M points and at the first region's full synthetic population; Windows | Clear errors when WebView2 or WebGL is missing |
| M4–M5 | Trips per second; routing time; wall time for a year; travel days needed for stable unpredictability | M4's typical days use the same day keys as the year's representative days; with all events off, household and person state is unchanged across the year, and each representative day's outputs are byte-identical to the M4 run of that day type |
| M6 | Simulated time vs wall time at peak concurrency; rerouting cost; memory | The road model runs at least 4 lakh concurrent vehicles, each with its own route, with wall time vs simulated time recorded (the speed target waits on the meaning of "realtime"); for a whole-metro run, peak concurrency lies within the Little's-law range for that metro, widened by its stated uncertainty of about 2× |

Benchmark practice:

- Run each size in a fresh subprocess. `ru_maxrss` is a lifetime peak, in bytes on macOS and KiB on Linux.
- Record hardware explicitly, because `platform.processor()` returns only `arm`. The reference machine is an M3 Max with 10 performance and 4 efficiency cores and 36 GiB.
- Do not store full trajectories. 400k vehicles at 10 Hz would be about 2 TB/h as SUMO FCD XML, and about 20–45 GB/h quantised (delta + zstd; an optimistic estimate from synthetic data). Runs keep aggregates, sampled trajectories and windowed replay instead.
- Escalate in this order: vectorise, then sample, then compiled kernels, then free-threaded Python. Each step must reproduce the declared outputs.

## 12. Outstanding decisions

Outstanding decisions are listed in [PLAN.md §8](PLAN.md); sections affected here:

| Decision | Sections |
|---|---|
| First region and first local data pack; how the 2026-10-03 India decision applies | §2.1, §3 |
| Python version; uv upgrade | §8 |
| Trips leaving or entering the snip (buffer, external zones) | §2.1, §4, §6 |
| Simulated year | §5.1; price tables in §4 |
| 2011-ward crosswalk; population projection | §3.2 |
| Fallback seed and fitting method; MoSPI access and derived defaults | §2.2, §3.3–3.4 |
| Validation targets and tolerances | §10, §11 |
| Snip-drawing library; Windows/WebView2 results; millions of points | §9, §11 |
| Remaining burden dimensions, ownership costs, unpredictability percentile | §7 |
| Mode-choice source; household vehicle allocation rule | §3.5, §4 |
| Tracking the fare tables we compile | §4 |
| Representative travel days vs daily travel | §5.1, §11 |
| Queue vs meso model; sampling scheme; adaptation in what-if scenarios; meaning of "realtime" | §6, §11 |
| Building source and region packs under ODbL | §2.1 |
| Project software licence | All |
| Google Maps on desktop; video export | §9 |
| When the microscopic engine returns; M1-brief decisions D1–D3, D6, D7 | §6 |
