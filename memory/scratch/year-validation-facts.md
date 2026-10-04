# Simulating a year and validating it against official statistics

Status: in-flight scratch, 2026-10-03, macbookpro. Facts only. A workflow of 3 research agents and 3 adversarial verifiers produced it. Of the claims checked, 0 were refuted and 26 corrected; corrections are applied below.

## Which year

- **Calendar 2025 lines up best:**
  - PLFS 2025 is the first January–December round. It has a **city report for 46 million-plus cities** (including Bengaluru, Greater Hyderabad and Greater Mumbai; 29 Jun 2026) and a **district snapshot** (18 Sep 2026; Bengaluru Urban unemployment 2.5%, NEET 15.0%). Both carry RSEs; "#" marks cells with fewer than 250 households.
  - Also covering 2025: ASUSE 2025 city enterprise estimates, the NSS Health 2025 survey, and MoRTH's 2025 accident report (expected about mid-2027).
- **National Household Travel Survey** (NSS 80th round): fieldwork Jul 2025–Jun 2026; not released, and not in the 2026-27 calendar.
  - It was designed for the Ministry of Railways, with district origin–destination as its key output, so it may not support checks *within* a city.
  - Its terms may be non-commercial.
- **Survey years don't line up:** HCES Aug 2023–Jul 2024, TUS 2024, NFHS-6 2023-24, Census 2011, and Census 2027 (enumeration Feb 2027; no release date). Either map each comparison to its survey's own period, or accept the gap explicitly.

## Independent check targets (if not used as inputs)

- **Ridership:** BMRCL, BMTC, TGSRTC, HMRL, BEST, suburban rail. Mostly via press, RTI or parliamentary answers, so record source and date and treat as approximate. Examples: Namma Metro record 11.2 lakh/day (10 Aug 2026); Mahalakshmi peak 37 lakh women's trips/day.
- **New vehicle registrations by RTO and month:** the VAHAN4 report view exports Excel. Compare flows, not stocks: the Bengaluru stock of 12.59M is "registered and kept for use" and has a series break. Telangana joined VAHAN around Mar 2026, but its historical records are on the dashboard too (see `life-events-facts.md`).
- **Fuel sales:** PPAC, state level.
- **Road accidents:** NCRB and MoRTH agree for Bengaluru and Hyderabad but not Mumbai (348 vs 2,604). Pick one source per city.
- **School enrolment:** public UDISE+ 2025-26 school-level files (pseudonymised) with ward, ULB, pincode and enrolment by class and gender, via OpenCity.
- **Health:** HMIS and the Health 2025 survey.
- **Jobs:** ASUSE city jobs by sector.
- **Car congestion by month:** TomTom 2025, "City" geography, car drivers only. Bengaluru is highest in Sep (80.7) and lowest in Feb/Mar (68.6); Mumbai highest in Jan/Sep and lowest in May; Hyderabad highest in Sep.
- **Long-distance trips:** the Domestic Tourism Expenditure Survey (NSS 80th round; microdata released 23 Sep 2026).
- **Coming later:** NHTS, the National Household Income Survey (2026-27; no confirmed fieldwork dates), Census 2027 houselisting (it asked about vehicles).
- **Geography crosswalks are needed for every target:** BBMP/GHMC/MCGM vs UA vs police commissionerate vs district vs RTO vs transit network vs education district.

## Validation method (avoiding circularity)

- **Calibration ≠ validation:** validation uses data not used in calibration (TAG M3.1 §3.3.1; FHWA 2010; Augusiak 2014).
  - Comparing against control tables is an *internal check*.
  - The TAG M3.1 §3.3.8 template reports three groups separately: data used to build, data used as constraints, and independent validation data.
- **Per-variable ledger of control vs hold-out:**
  - Census 2011 HL-14 already contains vehicle assets, and B-28 contains commute mode and distance. If either is used as a control, the matching outputs are internal checks.
  - A cheap quasi-hold-out: hold out some cross-tabs from the same census.
- **PLFS absolute totals** are ratios × MoHFW projections. If the simulation ages its population with the same projections, totals match by construction, so compare rates (LFPR, WPR, earnings distribution).
- **Tolerances:** use published RSEs or confidence intervals where they exist (PLFS city/district reports). Administrative counts and NFHS fact sheets have none, so set explicit bands *in advance*. GEH applies only to hourly counts; for daily counts use SQV.
- **Uncertainty:** runs that differ only in random seed understate it (38% coverage of a nominal 90% interval in an UrbanSim study). Also draw uncertain inputs. Choose the number of seeds by CV stability or confidence-interval half-width.
- **Pattern-oriented validation:** check several patterns at different scales (annual ratios, seasonality, distributions), with acceptance criteria fixed beforehand (ODD 2020 "Purpose and patterns"). Avoid "valibration", i.e. repeatedly tweaking until matched.
- **Indian precedents** (BharatSim, SynthPop++, IISc-TIFR, Patna MATSim) report mostly in-sample fit. A real hold-out design would go beyond them.

## Representing a year

- **Standard practice:** simulate a few weighted day types and annualise them.
  - UK TAG A1.3 annualisation; Transport Scotland TN013.
  - IRC:108-2015 seasonal factors (but these are freight-driven, so not an urban passenger profile).
  - Energy models' "typical days".
  - Avoid a typical meteorological year (TMY) for rain: precipitation is not used to select its months.
- **Week-long activity models:** actiTopp (GPL-3) and mobiTopp (MIT) are German-estimated. India has no multi-day diary (TUS covers one 24-hour day), so variation within one person has to be modelled.
- **Hybrid option:** run the cheap household/money layer for every day or month, and detailed travel only for representative days, then reweight.
- **Match each survey's estimator and period:**
  - PLFS: current-week status averaged over 2025, plus usual status.
  - HCES: 7/30/365-day recall, Aug 2023–Jul 2024.
  - TUS: one day, reported separately for normal days and off-days. State/all-India only; minutes are per participant; activities under 10 minutes are missed.
  - Bengaluru CMP: base year about 2015, AM peak hour 09:00–10:00, data reused from RITES 2016 and CTTS 2018.
- **Calendar facts:**
  - IMD 1991–2020 rainy days (≥2.5 mm) per year: Bengaluru city (station 43295) 62.6, Hyderabad 49.8, Mumbai Santacruz 78.6 (23.3 in July).
  - Normal monsoon onset/withdrawal: Mumbai 11 Jun / 8 Oct; Hyderabad 8 Jun / 14 Oct.
  - 2026 has 261 weekdays, about 242–243 of them not gazetted holidays. Private-sector holidays differ.
  - Real daily weather: IMD gridded rainfall at 0.25°, 1901–2025, downloadable from imdpune.gov.in.
- **Rain effects:** FHWA 2006 gives a flat 10–11% capacity drop for light and moderate rain, with free-flow speed −2 to −9%. Two Mumbai roads showed +8–140% travel time. Indian evidence is thin and site-specific.
