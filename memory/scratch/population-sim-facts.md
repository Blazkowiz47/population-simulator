# Population-based simulation facts

Status: in-flight scratch from a rubberduck session, 2026-10-03, macbookpro. This note records facts only; the project scope and name are undecided. A workflow of 3 research agents and 3 adversarial verifiers produced it. Of the claims checked, 0 were refuted and 23 were corrected; corrections are applied below. The raw output was session-only. Microdata portals (microdata.gov.in, data.gov.in, DHS) were partly unreachable, so items marked (low) are unverified.

## How existing tools are built

- **Two layers everywhere.** Every person- or household-based tool separates a demand layer (households, people, plans) from a movement layer (vehicles on roads), linked by person→vehicle and household→vehicle IDs.
  - MATSim: Java, GPL-2.0+, release 2026.0.
  - BEAM: Scala, GPL-3.0+.
  - POLARIS: C++, licence-gated executables.
  - SimMobility: C++, non-commercial licence, dormant since 2022.
  - ActivitySim: Python, BSD-3, v1.6.0. Demand only; it hands trip matrices to an external traffic-assignment tool.
  - eqasim: GPL-2.0.
  - OMoSim: MIT. Demand only; calibrated to Germany.
- **Sampling is the norm.** MATSim runs 1/3/10/25% samples and scales flow and storage capacity by the same factor (Open Berlin: 10/3/1% samples; its data are CC BY 4.0). POLARIS has a traffic_scale_factor and ActivitySim uses expansion factors.
- **Sampling distorts on-demand fleets.** BEAM found pooled ride-hail efficiency plateaus only at about 40% samples (San Francisco). MATSim's FISS samples private cars only and keeps fleets and public transport at 100%. With many autos and taxis in Indian cities, a mixed scheme is worth considering.
- **Performance anchors:**
  - POLARIS (mesoscopic): 10M Chicago travellers in about 1.2 h on a 2×8-core desktop (2016); Chicago CBD, about 400k travellers, in 10.5 min on 6 cores.
  - MATSim QSim stops scaling at about 8 threads.
  - MATSim Rust prototype: single core runs at 560× real time for a 10% Metropole Ruhr sample (mobsim only, queue model).
  - No tool documents microscopic, city-wide, full-population simulation on a desktop, and none reports peak concurrent en-route vehicles.
- **Owner-driven taxis/autos parked at home: nobody models this.**
  - MATSim DVRP and BEAM drivers are fleet agents with start links and shifts.
  - BEAM removes human drivers at shift end and does not model their commute.
  - POLARIS's human-driver models are unreleased.
- **Household car sharing:** partial precedents only.
  - BEAM reserves the car once chosen and requires the same car for the return trip.
  - POLARIS has an intra-household AV assignment MIP (Gurobi).
  - MATSim socnetsim optimises allocation at tour level, but by default MATSim teleports a car that isn't where the driver is.
  - ActivitySim allows double-booking.
- **Parking near home:**
  - BEAM assumes dedicated home stalls.
  - POLARIS and MATSim's parkingproxy aggregate per zone or link, or apply penalties.
  - Explicit search is used only in small areas (Berlin about 60k agents; Zurich 0.1% sample) because it is expensive.
- **India and South Asia precedents:**
  - MATSim Patna: 10% sample; PCU-based mixed traffic with seepage; autos lumped into teleported "pt".
  - teg-iitr/matsim-iitr: Dehradun, Delhi, Jaipur, Chandigarh scenarios; no licence declared.
  - Dhaka MATSim (Transport Policy 2024, CC BY): CNG autos on the network as vehicles (2.7 m, PCE 1); 1% sample.
  - BharatSim: Mumbai synthetic population of about 12M people; IPU on an IHDS-II seed plus Census 2011 plus CTGAN. Its population-generator code has no licence.
  - IISc-TIFR epidemic simulator (Apache-2.0): synthetic households and workplaces for Mumbai and Bengaluru.

## Data for Indian cities

- **Census 2011, HH-14 (ward level):**
  - Household-size distribution and the share of households owning a car, a two-wheeler or a bicycle.
  - No joint car×2W table and no vehicles-per-household.
  - Values: BBMP car 18.9%, 2W 46.2%; Greater Mumbai car about 12.7%, 2W 15–17%; Hyderabad car 14.1%, 2W 49.9%.
  - Licence: GODL-India on data.gov.in; NADA terms are unclear.
- **Census 2027:** houselisting ran Apr–Sep 2026 and asked about vehicles, with bicycle grouped with 2W. No table release date is known.
- **Ward boundaries have changed since 2011.** Bengaluru: BBMP became the GBA, with 369 wards. OpenCity has a GBA 369-ward map with population. GHMC was split three ways in 2026. Mumbai's 97 Census ward units don't match BMC's wards.
- **NFHS-5 urban (2019):** car 11–14%, 2W 60–67% at state level. District figures need DHS microdata, which is restricted: no redistribution and no commercial use.
- **Comprehensive Mobility Plans (no public microdata):**
  - Bengaluru (2016 household survey, 10,167 households): 16% of households have no vehicle, about 20% have one car, about 60% have at least one 2W. 1.24 trips per person per day. Peak-hour motorised trips 12.6 lakh: public transport 47.8%, 2W 23.5%, car/taxi 21%, auto 7.7%. The report contradicts itself on trip lengths.
  - Mumbai (2015/16): autos 61% owner-driven and 39% hired; taxis 40% owned and 60% hired.
- **Time Use Survey 2024:** 139k households; 30-minute activity slots; records whether each activity is inside or outside the dwelling. No travel mode or destination. Needs a free MoSPI login.
- **PLFS CY2025:** 2.7 lakh households, a large recent seed, but no vehicle variables.
- **HCES 2023-24:** may record vehicle possession (low).
- **Fleets:**
  - Bengaluru: 3.6 lakh autos registered (2025) against a permit cap of 2.55 lakh; BMTC about 7,000 buses.
  - Mumbai: BEST about 2,740 buses; MMR more than 4.5 lakh autos.
  - Hyderabad: TGSRTC about 3,200 buses.
  - GTFS: the Hyderabad TGSRTC feed is a full timetable published by Open Data Telangana. The Bengaluru official feed states no licence; an unofficial feed is ODbL.
- **Placing households:**
  - Google Open Buildings v3 (CC BY 4.0 or ODbL), WorldPop (CC BY 4.0), GHSL (attribution), Microsoft buildings (CDLA-Permissive-2.0).
  - OSM building coverage in Indian cities is poor.
  - Global building heights are unreliable for Indian high-rises.
- **Scale check** (verifier's estimate via Little's law): Bengaluru's 2016 peak-hour trip starts imply about 1.8–2.8 lakh concurrent resident vehicles. Adding buses, goods vehicles, through traffic, empty autos/taxis and 10 years of growth makes 4 lakh a plausible order of magnitude for Bengaluru's peak. For Greater Mumbai alone, 4 lakh looks high.

## Synthetic-population methods

- **Seed-based synthesis** (IPF, IPU, PopulationSim BSD-3, PopGen3) needs household microdata. Census microdata is workstation-only. IHDS-II urban seeds are small (about 850–1,160 households per state). PLFS is large but has no vehicles.
- **Sample-free synthesis** (humanleague MIT, Gargiulo, Barthelemy-Toint) works from marginals only. It cannot build households containing persons without detailed conditional tables.
- **Activity schedules:** TUS for timing; OMoSim-style destination choice; mode choice from Indian MNL studies.
- **Validation layers:**
  - synthetic population vs ward marginals (SRMSE);
  - commute distance and mode vs Census B-28 (district level only);
  - activity participation vs TUS;
  - trip-length and mode-share targets;
  - link counts vs GEH and journey-time criteria;
  - city speed indices.
