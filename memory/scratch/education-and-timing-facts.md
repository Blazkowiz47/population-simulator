# Education pathways and event timing facts

Status: in-flight scratch, 2026-10-04, macbookpro. Facts for life-event defaults. A workflow of 2 research agents and 2 adversarial verifiers produced it. Of the claims checked, 0 were refuted and 18 corrected; corrections are applied below.

## Education stages by state (2025 simulation year)

- **NEP 2020:**
  - School is 5+3+3+4: foundational (pre-school + classes 1–2), preparatory (3–5), middle (6–8), secondary (9–12).
  - UG degrees are 3 or 4 years with exits. The Ministry of Education asked for class 1 entry at age 6+ from 2024-25.
- **Class 1 entry age, per state and year:**
  - **Karnataka (KA):** 6 by 1 June. Relaxed to 5 years 5 months in 2025-26; a 60-day relaxation in 2026-27.
  - **Maharashtra (MH):** 6 by 31 December, so June entrants are about 5.5–6.5 years old.
  - **Telangana (TG):** 5+ in 2025-26.
  - So KA entrants are the oldest, by up to about a year.
- **Junior college (classes 11–12):**
  - **KA:** PUC I and II. Only II PUC is a board exam (KSEAB, which also runs SSLC).
  - **TG:** Intermediate. Both years are public exams (TGBIE), with groups MPC, BiPC, CEC, MEC, HEC; 20% internal assessment from 2026-27.
  - **MH:** FYJC/SYJC. Only HSC is a board exam (MSBSHSE).
  - Junior and PU colleges are not in AISHE; take them from UDISE+ or board lists.
- **Degree length:**
  - KA general UG: 3 years since 2024-25.
  - MH: 4-year honours under NEP, but Mumbai University still offers 3-year programmes, so default to 3 years with a small year-4 probability.
  - TG: 3 years (DOST admissions).
  - BE/BTech 4 years; MBBS 5.5 years.
- **Promotion at classes 5 and 8** depends on each state's no-detention or re-exam policy (not yet researched). Don't assume promotion is automatic.

## 2025 board pass rates (defaults for fail/repeat)

- **Class 10:**
  - KA SSLC exam 1: 62.34% (girls 74%, boys 58%; urban 67%). About 74% pass after all three attempts (exam 2/3 figures conflict between sources).
  - TG SSC: 92.78%.
  - MH SSC: 94.10%.
- **Class 12:**
  - KA II PUC exam 1: 73.45%.
  - TG Inter 1st year: 66.89%. TG 2nd year: about 65.5–65.7%; the 71.37% headline doesn't reconcile.
  - MH HSC: 91.88%.
- **2026 changed sharply** (KA SSLC 94.1%, II PUC 86.5%) after the pass mark dropped to 33%. Use 2025 values for a 2025 run.
- **Stream mixes** come from state-board candidates only and exclude CBSE/ICSE/IB, which are common in these cities.

## Higher education

- **AISHE 2022-23 and 2023-24** (released together 8 Jul 2026). GER for ages 18–23: KA 41.9, MH 36.4, TG 46.6 (India 30.0).
  - Use regular-mode enrolment (Table 6a) for commute purposes: distance learning is about 12% of MH enrolment.
  - GER numerators include students from other states.
- **Studying out of state** (NSS 75th round, 2017-18, Report 585 Table 10): KA ~15%, MH ~7%, TG ~5%. MoSPI flags state-level figures as unreliable (KA has only 89 cases), so treat these as weak defaults.
  - Bengaluru, Pune and Hyderabad are net destinations for students.
- **College locations:** the AISHE directory (dashboard.aishe.gov.in/hedirectory) has name, address, district and management, with Excel export. Coordinates are probably not public, so geocode by address/PIN. AISHE terms restrict reuse; fine under our no-republishing rule.

## Calendar (2025)

- **School reopening:** KA 29 May; TG 12 Jun (junior colleges 1 Jun); MH 16 Jun (Vidarbha 23 Jun).
  - KA 2025-26 Dasara break for government schools ran to 18 Oct.
  - TG Dasara 21 Sep–3 Oct; Sankranti 11–15 Jan.
- **Board results:** TG Inter 22 Apr; KA II PUC 8 Apr; TG SSC 30 Apr; KA SSLC 2 May; MH HSC 5 May; MH SSC 13 May.
  - Supplementary and second attempts run Jun–Jul, so not everyone transitions on result day.
- **UG classes start in June** for state colleges (KA 9 Jun, Mumbai University 13 Jun, TG DOST 30 Jun). MH FYJC confirmations ran to 7 Jul.

## Timing of other events

- **First jobs:** EPFO new subscribers aged 18–21 peak in May–July (index ~1.12–1.15) and are lowest in October (~0.77).
  - Job switches (EPFO rejoiners) peak in April (~1.11).
  - First EPFO estimates are revised upward, so use revised vintages.
  - EPFO state tables give net payroll by age band, not monthly entry curves by state.
- **Births:** Bengaluru registrations are near-uniform, except December at +19% above the uniform share. Use per-day rates (February's low is a calendar artifact).
  - Deaths: modest seasonality; April is lowest in 2023–24 but not in 2022.
  - Source: Karnataka Civil Registration System (CRS) reports, month-of-occurrence tables.
- **Vehicles:**
  - FADA festive window (42 days, Navratri→Diwali): about 20% of annual two-wheeler and 17% of car retail sales. October and November dominate.
  - 2025 is distorted by the GST cut on 22 Sep 2025.
  - FADA monthly bases mix coverage: May–Aug include Telangana in later releases.
  - FADA's 2025 figures exclude Telangana. But **Telangana's historical records are on the VAHAN public dashboard** (see `life-events-facts.md`), so Hyderabad 2025 can be calibrated from VAHAN.
  - Smaller bumps: Ugadi/Gudi Padwa, Akshaya Tritiya, January.
- **Marriages:** calendars differ by community.
  - Generic North-Indian, Marathi and Kannada muhurat lists give zero in Jul–Oct (Chaturmas). Telugu practice has heavy Shravana (August) and Sep–Nov weddings; Aashada is avoided in the south.
  - Muslim weddings avoid Ramadan and Muharram.
  - Muhurat date counts vary by source, and CAIT's "46 lakh weddings" figure has no published method.
  - Registration is compulsory in MH (within 90 days) and TG, but no monthly counts were found.
- **Rental moving peaks:** only weak (commercial) evidence. Default family moves with school-age children to Apr–early June, as a knob.
- **Store the source vintage with every default:** FADA and EPFO both revise earlier months.

## Access notes

- Blocked from this machine: FADA (Cloudflare), EPFO (403), state board sites, data.gov.in (503), HMIS.
- Reachable: MoSPI, AISHE, CRS (dc.crsorgi.gov.in), OpenCity.
- PIB works via curl. The Wayback Machine works as a route around geo-blocks.
- One research agent reported that the FADA Cloudflare block "also hit the user's Chrome". That suggests it used the Claude in Chrome browser tool.
