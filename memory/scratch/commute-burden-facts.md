# Commute burden facts: income, costs, metrics

Status: in-flight scratch, 2026-10-03, macbookpro. Facts only. A workflow of 3 research agents and 3 adversarial verifiers produced it. Of the claims checked, 1 was refuted and 21 corrected; corrections are applied below. Prices change often, so treat every number as true only "as of" its date.

## Income: the Census has none

- **PLFS 2025 unit data** (released 27 Mar 2026, microdata.gov.in catalog 284) is the best current source.
  - Person-level labour earnings: regular wage (previous month), self-employment gross earnings (30 days), casual daily wages.
  - New from Jan 2025, household-level non-labour income: rent, interest, pension, remittances (`inc_tot`).
  - Identifiers: district code; each million-plus city is its own urban stratum.
  - Weights are mandatory: educated households are over-sampled in urban areas.
  - Self-employed earnings are gross, so owner-drivers' vehicle costs must be deducted separately.
- **HCES 2023-24** (household consumption survey) records no income. It has consumption per person (MPCE) plus 30-day spending by item (bus fare for commuting, taxi, auto, rail, petrol, diesel) and whether the household owns a car, two-wheeler or bicycle. It is the best target for the money side and for car vs no-car comparisons.
- **Access terms (both surveys):** MoSPI Category B — free registration, no redistribution. A public repo may hold only download scripts and derived parameters. MoSPI publishes an MIT-licensed official client: github.com/nso-india/mospi-unitdata.
- **Older and coarser sources:**
  - IHDS-II (2011-12): has true household income, but city samples of only about 350–520 households.
  - City mobility surveys: coarse income bands. Bengaluru (c. 2016) mean Rs 32,374/month with 7.6% spent on transport; Mumbai 2004 and 2019.
- **Coming later:**
  - The NSS 80th round included a National Household Travel Survey (fieldwork Jul 2025–Jun 2026); release pending.
  - A national household income survey is planned for 2026-27 (results mid-2027 per media only).
  - Keep the income module swappable.

## Money: cost per trip and per vehicle (as of about 3 Oct 2026)

- **Free buses for women:**
  - Karnataka **Shakti** (since 11 Jun 2023): women and transgender people domiciled in Karnataka ride non-AC buses free; Vajra/Vayu Vajra are excluded.
  - Telangana **Mahalakshmi** (since 9 Dec 2023): women and third-gender residents ride City Ordinary and Metro Express free; women's share of TGSRTC riders rose from 40% to over 67%.
  - Mumbai BEST has no equivalent (an election promise is unconfirmed). Some MMR operators give women 50% off.
- **Public transport fares:**
  - BMTC ordinary from Rs 6. The full stage table is unconfirmed, and a fare hike is pending (about 44% requested).
  - **Namma Metro Rs 10–90:** the Feb 2025 slabs still apply because the Feb 2026 rise is on hold.
  - Hyderabad Metro Rs 11–69. A combined Metro + bus pass costs Rs 8,500 per 30 days (Rs 7,000 metro + Rs 1,500 bus).
  - BEST Rs 10–60 non-AC. Mumbai suburban second-class monthly passes about Rs 130–315.
- **Auto and taxi meter fares:**
  - Bengaluru auto Rs 36 for the first 2 km, then Rs 18/km (meters are widely ignored, so treat the meter fare as a floor).
  - Hyderabad auto Rs 30 for 1.6 km, then Rs 14/km (GO of 2 Oct 2026).
  - Mumbai auto Rs 27 for 1.5 km, then Rs 18.22/km; Mumbai taxi Rs 33, then Rs 21.90/km (both from 1 Sep 2026).
  - Night surcharge: 1.5× in Bengaluru and Hyderabad, 1.25× in Mumbai.
  - Aggregator surge is capped at 2× (MoRTH 2025 guidelines).
- **Fuel prices:** petrol about Rs 111–116/L (Hyderabad highest); CNG Rs 89–112/kg.
- **Running costs per km:**
  - Small car: about Rs 5.1–5.8/km on fuel at 20–22 km/L. CEEW's 14 km/L figure is for an average Rs 9.5 lakh car, not a small car.
  - Two-wheeler: about Rs 1.9–2.5/km fuel plus about Rs 0.3/km maintenance.
- **Fixed ownership costs while a loan runs:**
  - Small car about Rs 7–9k+/month; two-wheeler Rs 2.5–3k/month.
  - Add comprehensive insurance and registration: on-road price is 15–21% above ex-showroom.
  - Road tax: Karnataka car 13–18%.
  - GST on small cars and two-wheelers cut from 28% to 18% (Sep 2025).
- **Best single citable source for per-km ownership cost:** CEEW (Jun 2025).
- **Owner-drivers:** treat as a small business model (fare revenue minus fuel, maintenance, insurance, and rent or EMI). Rent and idle-share data are weak (Khatua 2017 is for taxis), so use scenario ranges.
- **Design:** store versioned, dated fare tables and record the "as of" date on every result.

## How burden is measured

- **Affordability:** share of household income spent on transport.
  - Common limits are 10–15%.
  - UN SDG 11.2.1 metadata (2025): the poorest fifth should spend at most 5%.
  - Indian policy (NUTP 2014) sets no numeric target.
  - World Bank index (Carruthers et al. 2005): cost of 60 trips of 10 km a month.
- **Generalised cost** = money + value of time × time.
- **Extreme commute:** one-way over 60 or 90 minutes.
- **Access:** within 500 m of a bus stop, or 1 km of rail/metro (SDG 11.2.1).
- **Value of time in India:** the best urban evidence is Suri & Cropper, Mumbai 2019.
  - In-vehicle about Rs 46–49/h (about 40–42% of wage); out-of-vehicle about Rs 85–87/h.
  - Below-median earners about Rs 34/h; above-median about Rs 120/h.
  - The highway values in IRC SP:30 don't suit urban trips.
  - If value of time is tied to income, use an elasticity below 1 (World Bank 0.696). Also report with one average value, so richer people's time doesn't automatically dominate.
- **Indian benchmarks:**
  - HCES urban conveyance spending: 8.46% of consumption.
  - Bengaluru CMP: 7.6% of income.
  - Mumbai 2003–04: the poorest spent 16% of income, and 63% of the poorest workers walked.
  - TUS 2024: commuters spend 77 min (men) and 67 min (women) per day in total, both ways.
  - Bengaluru 2018: average one-way commute 42.5 min.
- **No framework** (MATSim, ActivitySim, BEAM) reports time / money / income-share burden as a built-in output; it is post-processing.
  - MATSim's dailyMoneyConstant is charged only on days the mode is used, so it cannot represent fixed vehicle costs.
- **Report distributions, not averages:**
  - by income fifth, gender, ward and household type (car / two-wheeler / none / livelihood auto-taxi);
  - the share of people above each threshold;
  - winners and losers between scenarios.
  - Low spending among the poor can mean walking or not travelling at all, so pair money with time, access and trip-participation measures.
