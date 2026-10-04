# Government data folder and portal survey

- Date: 2026-10-03 to 2026-10-04
- Node: macbookpro
- Repo: `/Users/sushrutpatwardhan/1Projects/traffic-simulator` (GitHub `population-simulator`)
- Source paths:
  - [govdata/README.md](../../govdata/README.md): rules.
  - [govdata/catalog.yaml](../../govdata/catalog.yaml): 104 datasets, parses as valid YAML.
  - `.gitignore` rule `govdata/**/raw/`.
- Status: in-flight. `govdata/`, the `.gitignore` change and the scratch notes from 2026-10-03/04 are **uncommitted**.

## What was set up (at Sushrut's request)

- **Folder for government records, with Sushrut's rule:** "we won't be republishing.. we will state that our simulation numbers match".
  - Downloaded files go in `govdata/<dataset-id>/raw/`, which is git-ignored.
  - Tracked: `catalog.yaml`, per-dataset `manifest.json` (URL, date, checksum, licence), and `comparisons/` (our value vs the official value, with citation, margin of error, and whether the dataset was used to build the model).
  - A dataset used to build the model can't back a "matches" claim for the same quantity.

## Portal survey (5 groups, each re-checked by a verifier)

Verifier results by group: Census 8 confirmed / 7 corrected / 3 unverifiable; MoSPI 13/11/0; OGD and states 11/6/5; transport 15/3/5; other 6/9/2. The verifiers also listed about 60 datasets the survey missed; they are at the end of `catalog.yaml`. Verifier corrections are stored per entry under `verification` and have **not** been merged into the main fields yet.

- **Reachability:** this machine geolocates to Norway, and many Indian government hosts block it.
  - **Unreachable:** data.gov.in and its state instances (503, 403 or connection refused), Karnataka state sites (DULT, BBMP, KGIS, Transport, DES), data.telangana.gov.in, ABDM HFR, the VAHAN legacy dashboard, schoolgis.
  - **Reachable:** censusindia.gov.in, microdata.gov.in, mospi.gov.in, esankhyiki/api.mospi.gov.in, analytics.parivahan.gov.in, MoRTH, nfhsiips.in, Survey of India, Labour Bureau, RBI, Maharashtra MVD.
  - Retry blocked portals from an Indian network.
- **Census 2011 (censusindia.gov.in, no login, direct XLS/XLSX):**
  - **Ward-level tables for all three cities:**
    - PCA Town/Village/Ward: households, population, ages 0–6, SC/ST, literacy, workers. BBMP 198 wards (84.4 lakh people); Greater Mumbai 97 census wards; GHMC wards spread across the 2011 Hyderabad, Rangareddy and Medak files.
    - HL-14: household size, rooms, ownership, condition, and bicycle, two-wheeler and car ownership. Its ward codes match the PCA.
  - B-28 commute tables are district-level only.
  - No open ward boundaries exist (the geographic page returns 404).
  - **Terms:** reproduction requires permission (copyright policy, node/286); NADA pages say all rights reserved. This is compatible with not republishing.
  - **Census 2027:** the population questionnaire adds "travel to place of work" and "driving licence". No data published yet.
- **MoSPI (MoSPI's dissemination guidelines, GSDD 2026, notified 27 Apr 2026):**
  - **Category A:** reports, factsheets, press notes and eSankhyiki aggregates are reusable with citation, commercially or not.
  - **Category B (unit data):** free registration, non-commercial only, Annex-I undertaking, "shall not be shared … without prior approval". Publishing derived aggregates with a citation is fine. Whether a synthetic population fitted to unit records counts as "sharing" is unclear.
  - PLFS 2025 public file `mplus_identifier.xlsx` maps survey sample areas to 46 million-plus cities: Bengaluru 116, Greater Mumbai 160, Greater Hyderabad 94.
  - Million-plus city labour report published 30 Jun 2026; district figures 18 Sep 2026.
  - **The eSankhyiki API answered without a key:** HCES state Gini and conveyance shares via `/api/hces/getHcesRecords`.
  - **NHTS (NSS 80th round):** fieldwork Jul 2025–Jun 2026 in about 24,752 sample areas. Only the instructions and questionnaire are out; no release date.
  - Domestic Tourism Expenditure Survey: report 597 plus unit data (catalog 307).
- **OGD and states:**
  - GODL-India (gazette of 10 Feb 2017) allows commercial reuse with attribution. data.gov.in API keys need a logged-in account.
  - TGSRTC and HMRL GTFS: redistribution allowed with attribution, but the download sits behind a Google Form.
  - BMC Census-2011 FAQ PDF gives figures for all 24 BMC wards.
  - Maharashtra MVD vehicle population by RTO.
- **Transport:**
  - MoRTH Road Accidents in India 2024 (Jun 2026; tables for 50 million-plus cities).
  - Maharashtra MVD Statistics 2024-25: Greater Mumbai's 4 RTOs hold 51.3 lakh vehicles, with 2.95 lakh new registrations in FY24-25.
  - VAHAN analytics dashboard: no login, but a CAPTCHA per export, so it is a manual pull. The dashboard's chart JSON endpoints need no CAPTCHA (undocumented; see `life-events-facts.md`). Telangana joined VAHAN in March 2026, but its historical records are also on the dashboard (TG CY2025: 1,031,169), so Hyderabad 2025 can be calibrated.
  - PPAC state fuel sales xlsx (reproduction needs permission).
  - Telangana state vehicle totals only (no Hyderabad breakdown).
  - L&T Hyderabad Metro FY25-26 annual report: about 4.17 lakh riders/day (a corporate filing).
- **Other:**
  - NFHS moved to nfhsiips.in. NFHS-6 district fact sheets are PDF-only behind a form; no unit data.
  - UDISE+: no official bulk school-location download (the public school-level files via OpenCity remain the route).
  - **Survey of India Administrative Boundary Database:** free 1:1M state/district/taluk shapefiles; a checkbox is required before download. This is the conformant boundary source for India maps.
  - Bhuvan LULC: view-only; downloads limited to government users.
  - Labour Bureau CPI-IW monthly for Bengaluru, Mumbai and Hyderabad (Sep 2020–Aug 2026; reproduction needs permission).
  - RBI Handbook of Statistics on Indian States 2024-25 (state income).

## Manual steps that only Sushrut can do

Claude must not log in, register, accept terms, solve CAPTCHAs or enter credentials. These need Sushrut:
- MoSPI registration for unit data (PLFS, HCES, TUS, DTES);
- VAHAN exports (CAPTCHA);
- NFHS-6 district PDFs (form);
- TGSRTC/HMRL GTFS (Google Form);
- the Survey of India boundary checkbox;
- data.gov.in API key;
- Census reproduction permission, only if raw tables ever need republishing (not planned).

## Open question (answered 2026-10-04)

Life events happen within the year, each with a knob. Vehicle purchases follow regional sales or registration data; see `decisions.md`. Consequence for this catalogue: VAHAN, MVD and FADA registration flows become **build** inputs, so vehicle checks must use a hold-out (e.g. calibrate on Jan–Jun, check Jul–Dec) or other sources (HCES/NFHS/Census 2027 ownership).

## Next actions

1. Commit and push `govdata/`, `.gitignore` and the scratch notes when Sushrut asks.
2. Merge verifier corrections into the main catalogue fields, and triage the ~60 "missed" datasets.
3. For each catalogue entry, assign build vs check against the planned model inputs (see [year-validation-facts.md](year-validation-facts.md)).
4. Retry blocked portals from an Indian network or VPN, if available.
