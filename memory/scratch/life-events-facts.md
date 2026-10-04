# Life-event data facts: vehicles, demography, labour and income

Status: in-flight scratch, 2026-10-04, macbookpro. Defaults and data sources for the life-event knobs. A workflow of 3 research agents and 3 adversarial verifiers produced it. Of the claims checked, 0 were refuted and 22 corrected; corrections are applied below. Evidence files are in the session scratchpad (`lifeevents/`). One stray agent file (`ka59.json`) was moved out of the repo root.

## Vehicle acquisitions

- **VAHAN public dashboard** (analytics.parivahan.gov.in/analytics/publicdashboard/vahan):
  - **Filters:** state, RTO, class (76), fuel (34), maker, status (11, incl. scrap and deregistered), owner type (27, incl. INDIVIDUAL and FIRM), and an archive flag.
  - **The page loads its charts from GET JSON endpoints with no CAPTCHA**, so monthly RTO × class × owner-type flows are retrievable (verifier finding). Use them politely; they are undocumented.
  - **Telangana's historical records are on the dashboard** (TG CY2025: 1,031,169 registrations), so **Hyderabad 2025 can be calibrated.** This corrects earlier notes that said otherwise.
- **OpenCity mirrors:** CSVs of RTO × month × class × fuel × maker × transaction type for Bengaluru (10 RTOs) and Mumbai (4 RTOs), 2021 to Jan–Jun 2025. No owner type.
  - Bengaluru CY2024: 738,696 registrations, incl. 150,038 Motor Car and 23,695 Motor Cab.
  - Mumbai CY2024: 280,531, incl. 69,693 Motor Car and 17,594 Motor Cab.
- **Registration ≠ household acquisition:**
  - Commercial cabs are ~13–14% of new car-type registrations in Bengaluru and Hyderabad, and ~20–21% in Mumbai. These are owner-driver or fleet events.
  - Firm-owned share of Motor Car registrations (CY2024): ~12% Bengaluru, ~16% Mumbai.
  - So the household share of new car-type registrations is roughly 76% in Bengaluru and roughly 67% in Mumbai (derived).
- **First, replacement or additional purchase:**
  - Telangana's dealer-sales "secondVehicle" flag (tied to a 2% surcharge, abolished 23 Mar 2026) was set for 14–16% of cars and motorcycles in 2024.
  - 51–54% of Maruti buyers are first-time buyers.
- **Used cars:** ~1.4 used per new car nationally (5.9M in FY25). But registry ownership transfers are only 0.36–0.40 per new registration, so a used-car knob must state which of the two it targets.
- **Disposal and exit:**
  - Published stocks are cumulative, and formal scrapping is ~0.3% of flows.
  - De facto exit shows in VAHAN archive status: in KA-01, 25% of registered cars are in Temporary or Permanent Archive (non-compliant 1+ years). Use this as the active-fleet proxy.
  - The Bengaluru FY25-26 "kept for use" stock has an unexplained break, so don't validate against it.
- **Region apportioning:**
  - Mumbai: official PIN→RTO list (2017) joined to data.gov.in PIN polygons.
  - Hyderabad: mandal/locality→RTA lists.
  - Bengaluru: no jurisdiction list found.
  - The 2019 Motor Vehicles Act amendment (s.40) lets owners register at any RTO in the state, so RTO counts are not purely by residence.
- **Seasonality:** Bengaluru 2024 car registrations: January and October ~10.6% each, September ~6.4%. 2025 has two regimes because of the GST cut on 22 Sep 2025.
- **Methods:**
  - SILO: yearly 3-way logit (keep / add / remove) on changes in household size, income and licences, plus relocation.
  - ILUTE/STELARS and VOSim: add / dispose / replace / nothing, triggered by life events with hazard timing. STELARS calibrated to registered-stock attributes plus a household survey.
  - UK evidence: ownership changes are driven by household composition, licence changes, employment and income.
  - No Indian transaction or panel model exists. Registration-only calibration still needs an independent ownership survey (HCES / NFHS / Census 2027) to validate who owns.

## Demographic events

- **Births:** SRS Statistical Report 2024 (20 May 2026). Urban age-specific fertility rates by state, and **urban marital fertility rates** (25–29: KA 151.8, MH 152.9, TG 173.2 per 1,000 married women).
  - Fertility knob range ~1.3 (SRS 2024) to ~1.7 (NFHS-6 urban).
  - Sex ratio at birth: 885–926 girls per 1,000 boys.
- **Deaths:** SRS 2024 single-year urban death rates are noisy, so smooth them or use the 2020-24 life tables. Those include COVID deaths; the inflation is ~11–14% in MH and TG, ~0 in KA.
  - Don't use CRS district counts as city rates: they count by place of occurrence (Hyderabad district had 208,467 births).
- **Marriage:** derive from SRS never-married shares.
  - Women: ~11–18% a year at ages 20–34. Men: ~10–18% at 25–39 (TG 30–39 up to 18%).
  - Synthetic-cohort rates overstate marriage while marriage age is rising.
  - Marriage drives women's moves in (26% of Mumbai women's and 23% of BBMP women's recent migration) and household formation.
- **Joint households and splits:** Census 2011 HL-05: 10–14% of urban households have 2+ couples. IHDS: 8.5% of urban households split over 2004–2011/12 (≥1.2% a year). Knob range ~1–3%.
- **Migration into the city** (Census 2011 D-03, lived there under 1 year):
  - Greater Mumbai 1.44%.
  - BBMP 2.72% (adjusted ~2.7–3.5%).
  - GHMC 1.37%, ~2.35% adjusted for 42% missing duration.
  - Out-migration and return migration have no city-level rate; assume, and balance against population growth.
  - PLFS 2025 has no migration questions. A MoSPI Survey on Migration runs Jul 2026–Jun 2027. Census 2027 asks duration and reason.
- **Moving house within the city:** only a World Bank Mumbai 2019 survey (3,024 households): ≥2–3% of households a year as a floor, more like 2.9–4.0% from longer duration bins. Bengaluru is ~60% renters, so likely higher; no measurement exists.
- **School events:** June events using each state's age rule.
  - UDISE+ 2025-26: all-India rates. KA and TG state figures come only via a secondary reproduction; MH was not obtained.
  - KA secondary dropout 14–18% statewide is mostly rural, and may not hold for Bengaluru.
  - BBMP ward-level two-wheeler ownership ranges 14–64% (2011 HL-14).

## Labour and income events

- **PLFS 2025 panel:** every household is visited in 4 consecutive months.
  - Monthly revisit unit data: catalog 291 (Apr–Dec 2025) and quarterly catalog 292.
  - Gives transitions between employed (by type), unemployed and not in the labour force, plus industry, occupation, hours and earnings changes.
  - Does not show employer changes or the reason a job ended. The public file lacks membership-change codes, and visit-linking keys are unverified.
  - No official flow statistics exist. Fallback: Bhattacharya (2021), a quarterly all-urban matrix for 2017–19, which must be renormalised for attrition.
  - City samples are thin (Bengaluru ~1,392 households a year), so pool flows by state or across million-plus cities, by age and sex, then align to city stocks.
- **Formal job switches:** EPFO rejoiners ~1.8–2.3% a month (understated). Aon attrition 16–17% a year (≈1.4% a month). Both describe the formal sector only.
- **Dated wage events:**
  - Private pay rises usually in April (~9%, Aon 2025), some deferred (e.g. TCS in September).
  - Dearness allowance effective Jan/Jul (central 55% from Jan 2025, 58% from Jul 2025; KA 14.25% from Jul 2025), announced 3–8 months later with arrears.
  - Minimum-wage top-ups: KA 1 Apr; TG 1 Apr and 1 Oct; MH 1 Jan and 1 Jul.
- **City inflation (CPI-IW Dec 2024 → Dec 2025):** Bengaluru +5.1%, Mumbai +3.1%, Hyderabad +3.4%. Labour Bureau reuse requires permission.
- **Retirement:** central 60, KA 60, TG 61, MH 58 (60 for Group D), EPS pension 58. Central-government retirement is at the end of the birth month.
  - Informal workers: use the PLFS participation decline (urban men 88.6% at 55–59 → 58.0% at 60–64).
- **Leaving education:** urban men in education fall from 78.0% at 15–19 to 28.5% at 20–24 and 3.5% at 25–29. Ages 18–25 make up ~59% of new EPFO joiners; the June peak is unconfirmed beyond Mar–Jul.
- **Don't copy SILO's yearly IncomeAdjustment:** on master it applies zero change and silently freezes incomes.

## Access and terms

- MoSPI Category B unit data: free for non-commercial use including research. Commercial use is allowed but priced.
- **Foreign users need an institution recognised by the Government of India.** This Mac geolocates to Norway, which may matter for Sushrut's registration.
- Derived products count ("directly or indirectly").
