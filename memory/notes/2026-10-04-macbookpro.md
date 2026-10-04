---
date: 2026-10-04
work_date: 2026-10-04
project: traffic-simulator
node: macbookpro
note_kind: daily
node_type: macos
device: macbookpro
server:
timezone: Europe/Oslo
repo_path: /Users/sushrutpatwardhan/1Projects/traffic-simulator
branch: master
commit:
sync_status: draft
source_format: markdown
tags: [population-simulation, india, life-events, official-statistics]
---

# 2026-10-04

## Intent

- Continue the rubberduck session on the population-simulator direction. Set up government data storage. Rename the package. Choose the desktop UI.

## Work Done

- Created `govdata/` with no-republishing rules (`README.md`), a `.gitignore` rule for `govdata/**/raw/`, and `catalog.yaml`. The catalogue lists 104 datasets from a direct survey of official portals (Census, MoSPI, OGD/states, transport, other), each re-checked by a verifier. Details are in `../scratch/govdata-catalog.md`.
- Renamed the package `src/traffic_simulator` to `src/population_simulator`, and the distribution and CLI to `population-simulator`. Regenerated `uv.lock` and re-synced. Updated the README title and run command (with a direction-update banner), the PLAN §8 CLI examples, and `.gitignore` (`.DS_Store`). The local folder stays `traffic-simulator`.
- Decisions recorded in `../decisions.md`: life events happen within the year as knobs with data-backed defaults; vehicle acquisitions follow regional sales/registration data; all life events are in scope, including education stage transitions; package and CLI rename.
- Scratch evidence added: `commute-burden-facts.md`, `any-region-data-facts.md`, `year-validation-facts.md`, `govdata-catalog.md`, `education-and-timing-facts.md`, `life-events-facts.md`.
- Engine/UI proposal (`../scratch/engine-ui-architecture.md`), not yet adopted:
  - simulation in Python (numpy arrays, multiprocessing pool, decide/resolve/commit days, keyed RNG);
  - NiceGUI desktop UI;
  - a runner process connected by a progress/cancel queue and run-folder files.
- Map in NiceGUI, also a proposal: one custom component (Python class + small Vue/JS file) wrapping MapLibre and deck.gl. Python sends deck.gl JSON layer specs; large layer data is served as files over local HTTP; the deck.gl and MapLibre prebuilt bundles are vendored, so no Node toolchain.
- Moved a second stray agent download (`ka59.json`) out of the repo root.

## Experiments / Runs

- Command: `uv lock`; `uv sync --locked`; `uv run --locked population-simulator`.
- Result: the lock now lists `population-simulator` 0.1.0; the CLI prints the scaffold status. No simulation exists yet.

## Learnings

- This Mac geolocates to Norway, and many Indian government portals block it: data.gov.in, Karnataka state sites, data.telangana.gov.in and ABDM. censusindia.gov.in, microdata.gov.in, mospi.gov.in and the eSankhyiki API are reachable.
- Census 2011 ward-level PCA and HL-14 tables (households, size, vehicle ownership) are directly downloadable for BBMP, Greater Mumbai and the GHMC wards.

## Blockers

- Manual steps only Sushrut can do: MoSPI registration for unit data, VAHAN exports (CAPTCHA), NFHS-6 district PDFs (form), TGSRTC/HMRL GTFS (Google Form), the Survey of India boundary download checkbox.

## Next

- Sushrut to decide: approve a throwaway map spike (adds NiceGUI and numpy), and give the go-ahead to adopt the engine/UI proposal.
- When the life-event and education research completes, rewrite `docs/PLAN.md` and `docs/architecture.md` for the population-simulator direction.
