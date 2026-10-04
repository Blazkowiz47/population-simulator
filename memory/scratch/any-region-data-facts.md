# Any-region data facts (snip a map → synthetic people)

Status: in-flight scratch, 2026-10-03, macbookpro. Facts only. A workflow of 5 research agents and 5 adversarial verifiers produced it. Of the claims checked, 0 were refuted and 34 corrected; corrections are applied below. Probe scripts were kept in the session scratchpad only.

## Where people live

- **Building footprints** (fetchable by bbox):
  - **Overture buildings** (2026-09-23 release, ODbL). A measured 20×20 km box in central Bengaluru read about 97 MB and stored 61 MB, with 737,579 buildings: 71% from OSM, 24% Google, 5% Microsoft.
  - **Google Open Buildings v3**: CC BY 4.0 or ODbL (your choice). Taking it directly avoids ODbL share-alike.
  - **Microsoft**: CDLA-Permissive-2.0 since 2026-03-11. Files are split by quadkey (20–126 MB per city tile).
- **Building heights in Indian cities are nearly absent.** Overture height ~0.05% and floor count ~1% in central Bengaluru. Open Buildings 2.5D Temporal (2016–2023) has heights, but large errors for tall buildings. So floors per building, and therefore persons per building, is an uncertain modelled variable.
- **Population grids:**
  - **WorldPop Global2 R2025A** (alpha, CC BY 4.0, 100 m, age/sex, 2015–2030). For India its inputs are state-level only (35 units), so density *inside* a city is purely modelled. It comes as national files (not COG; no range reads), about 0.75 GB per layer.
  - **GHS-POP R2023A**: CC BY.
  - **Meta HRSL v1.5**: S3 COGs, range-readable. A 2025 India version spreads census totals onto Overture building counts, but carries a "not for commercial use" caveat.
  - **LandScan**: ambient (day/night blended) population.
- **Residential vs non-residential:** EMC-BUILT R2025A gives a 10 m non-residential layer for 2022 (CC BY; overall accuracy ~88%). Overture class tags are only ~2% filled.
- **Household defaults:** India's Census 2011 HH-01 gives sub-district size distributions. Others: NFHS via the DHS API (household size 4.4; urban 4.2), CORESIDENCE (includes India), and the UN 2026 national database.
- **Ready-made global synthetic households:**
  - **GLOPOP-S / GLOPOP-SG** (CC BY 4.0): ~1 km grid, reference year 2015. For India it gives DHS wealth quintiles, not income. India is ~819 MB compressed.
  - SPEW is dead and SynthPops unmaintained.

## Where people go

- **Overture Places**: CDLA-Permissive-2.0 (Foursquare-sourced part Apache-2.0), ~81.5M places worldwide.
  - Releases are deleted after ~60 days, so snapshot each region and pin the release ID.
  - In Bengaluru it has ~4.5× OSM's POI count (~2.5× with confidence ≥0.5). But 83% of records are Meta-sourced, median confidence ~0.54, and status is almost always empty.
- **Foursquare OS Places** (Apache-2.0, ~109M): needs registration since Oct 2025.
- **Google Places is unusable** for a persistent synthetic city (no storage, no display on a non-Google map).
- **OSM** completeness varies hugely (Patna is sparse). Merging OSM into a stored DB makes that DB ODbL.
- **Land cover:** ESA WorldCover (10 m, 2021, range-readable COGs, CC BY); Impact Observatory 10 m (2017–2023, no account). Dynamic World needs Earth Engine.
- **Jobs:** there is no open global employment grid. Proxies: non-residential building area, POI density, night lights.
  - India: the **6th Economic Census (2013)** has ward and enumeration-block codes plus worker counts (free login).
  - The 7th Economic Census was never published; the 8th is due 2027.
  - Schools: UDISE has lat/lon via India Data Portal (ODC-BY, 2021 vintage).
- **Destination choice:** OMoSim's gravity model fits well at 5 km but poorly at 500 m. So the realism of a synthetic city is at ward or zone scale, not at the level of an individual shop.

## Income and wealth

- **No open dataset gives an income distribution below state or district level for India.** The defensible recipe:
  1. Take the distribution *shape* from a survey: PLFS 2025 district earnings plus household non-labour income (unit data), or HCES state consumption Gini. Consumption and income Gini differ a lot (urban consumption ~0.29 vs income 0.38–0.56).
  2. Add a separate top tail.
  3. Use fine-scale proxies only to *rank* places within the region.
  4. Draw households from the calibrated distribution. The World Bank discourages assigning welfare from area covariates alone.
- **Proxies:**
  - Meta RWI: 2.4 km, CC BY-NC, India labels from NFHS-4 (2015-16). Its error (~0.5) is comparable to the spread within a city.
  - Census 2011 HL-14 ward asset shares: the most direct signal inside a city, but 15 years old, and open 2011 ward polygons are often missing.
  - Kummu 2025 GDP per capita (CC BY): constant within a district.
  - Night lights: mostly track density, not income.
- **Licence traps (non-commercial):** RWI, SHRUG (also share-alike), GDL, GEM exposure, India HRSL 2025. Keep a per-layer licence flag.

## Daily schedules and life course

- **No open tool** does snip → households → daily lives with income → years of life events.
- **OMoSim** (MIT, any region, OSM/Overture) is the closest daily-schedule precedent. Its defaults are German, but it already ships a Korea parameter set, so an India-TUS-derived set has precedent.
- **India TUS 2019/2024** (~450k persons; 48 half-hour slots; location type home / outside / non-fixed): gives schedules but no destinations or modes, and misses activities under 10 minutes.
  - Schedule transfer practice: condition on age band, sex, work/student status, vehicle access and weekday, with fallback cells for thin samples.
- **Life over years: SILO** (GPL-2.0, Java) is the best template. Annual events:
  - ageing, leaving home, marriage, births, divorce, death;
  - jobs, buying or selling cars;
  - moving house, constrained by housing cost, commute time and transport-cost share.

  It has Bangkok and Cape Town use cases. Alternatives: UrbanSim (BSD-3; adds and removes households, no ageing), OpenM++ (MIT), LIAM2 (GPL).
- **Good practice:**
  - run multiple seeds and show bands;
  - record per-attribute provenance (local / national / foreign donor);
  - include a validation and limitations report per run;
  - label all people and households as synthetic.

## Map snip, basemap and layers

- **Fetch paths for any bbox:**
  - Overture GeoParquet via overturemaps-py or DuckDB.
  - PBF extracts clipped with QuackOSM (Apache-2.0) or pyosmium. OSMfr publishes small daily India state PBFs (Karnataka 128 MB).
  - **Avoid** public Overpass (courtesy limits, overloaded) and tile.openstreetmap.org (no offline or prefetch use).
- **Offline basemap:** Protomaps PMTiles (BSD-3 code, CC0 styles, ODbL tiles). A Bengaluru extract at z0–15 is ~33 MB. Hotlinking the daily builds is discouraged and builds are kept ~1 week, so copy them to your own storage.
- **Layers:**
  - deck.gl 9.4. ScatterplotLayer is smooth to ~1M points, and a metro has 10M+ people, so switch by zoom: hex aggregates when zoomed out, sampled dots when zoomed in.
  - Zoomed-in synthetic dots imply false precision, so prefer aggregates and label them synthetic.
  - Use quantile classes, map rates rather than counts, and fix the HeatmapLayer colour domain.
  - WebGPU disables GPU aggregation.
- **India map law:** Criminal Law Amendment Act 1961 s.2(2) makes it an offence to publish a map of India not conforming to Survey of India boundaries. So omit international and disputed boundaries in city views, or use Survey of India-compliant data.
- **Licensing summary:** ODbL share-alike flows through OSM and Overture buildings into derived databases. Taking Google buildings directly under CC BY avoids it.
