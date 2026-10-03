# City-scale (about 4 lakh concurrent vehicles) facts

Status: in-flight scratch from a rubberduck session, 2026-10-03, macbookpro. Facts only; no architecture chosen. A workflow of 5 research agents and 5 adversarial verifiers produced it. Of the claims checked, 0 were refuted and 35 corrected; corrections are applied below. Probe scripts lived in the session scratchpad only.

## What exists at this scale

- **No precedent** runs more than 100k *concurrent* vehicles with continuous position, a step of 0.5 s or less, junction conflicts and lateral movement, at real time or faster, on CPU. None was found on any hardware with lane-free lateral movement either.
- **Most published "vehicles" figures are totals, not concurrency.** They count trips or insertions over a run (MOSS 2.46M, Heywood 512k, LuST 296k), not vehicles on the road at once.
- **SUMO:**
  - Its microsimulation is single-threaded; `--threads` is "not recommended" (#4767, #17057).
  - Its FAQ gives 80k–700k vehicle-updates/s on a desktop, and a 0.1 s step costs 10×.
  - Published city scenarios average 1k–21k vehicles on the road at once. BeST Berlin has 2.25M trips/day, averages about 21k on the road, and runs at 3.4× real time.
  - Large scenarios stay running by relaxing junction logic and teleporting vehicles: BeST logged 2,332 teleports, and MoST uses collision.action=none.
- **CPU multi-threaded microsimulators** (CityFlow, CBEngine) simplify junctions to fixed holding times. CBLab: 100k vehicles on the road at 0.261 s per 1 s step on 20 threads.
- **GPU microsimulators** (MOSS, LPSim/MANTA, FLAME GPU) are all NVIDIA CUDA; none runs on Apple silicon. They use lane-based IDM with 0.5–1 s steps and drop in-junction conflict/priority logic, partly because it deadlocks at scale. LPSim's abstract figures contradict its own tables.
- **Mesoscopic and queue models** handle this scale on CPU:
  - UXsim (Python, MIT): Chicago, 968k vehicles in 37 s, peaking at about 322k in the network at once. It uses 30-vehicle platoons and 30 s steps, so its workload is about 10⁴× smaller than micro at 0.1 s.
  - SUMO MESO: "up to 100× faster" than SUMO micro, with no sublane or opposite-direction driving.
  - MATSim QSim: single process at 82–586× real time on a 10% Ruhr sample.
  - POLARIS: 10M Chicago travellers in about 1.2 h on 16 cores.
  - All of these give up lateral position, lane-free filtering and junction conflicts.
- **Hybrid micro/meso** is an established technique: Aimsun, and Burghout's Mezzo+MITSIM (2004). The boundary is the hard part:
  - meso→micro: the boundary must generate a lane, speed and entry gap;
  - micro→meso: blocking and spillback must be passed back;
  - for lane-free traffic, no published way to generate a lateral position on entry was found.
  - Indian meso building blocks exist (MATSim seepage/passing queues, heterogeneous CTM, a 2D LWR model), but none has been coupled to a micro model.

## Making Python fast enough

- **Pure CPython:** the probe reached about 3.9–4.1M vehicle-updates/s for a bare single-file IDM update; real time for 4 lakh at 0.1 s needs 4.0M. NumPy-vectorised trivial IDM reached about 87M/s (system Python 3.9, verifier measurement).
- **Compiled CPU options** (current, Python 3.12, macOS arm64 + Windows x64): Numba 0.68, PyO3 0.29 + maturin + rayon, nanobind, Cython 3.3.
  - Numba `parallel=True` from pip on macOS arm64 gets only the workqueue layer with static chunks; TBB is unavailable there (inferred from docs).
  - Free-threaded CPython 3.14t is now official (PEP 779).
- **Cross-platform GPU (Mac Metal + Windows):** Quadrants (the Taichi fork; upstream Taichi has been dead since 2025-07), wgpu-py (WGSL, f32 only) and SlangPy (needs macOS 26+). Warp, MLX, JAX, FLAME GPU, MOSS and numba-cuda offer no Mac GPU path. Apple GPUs have no float64, and fast math is on by default.
- **Determinism:**
  - rayon reductions are unordered.
  - clang fuses multiply-add by default and MSVC doesn't.
  - Rust's basic float ops are exact IEEE with no auto-FMA (RFC 3514), so a Rust core can be bit-identical across macOS and Windows if it avoids libm transcendentals.
  - On GPUs, only run-to-run determinism on the same device is realistic.

## Routing and assignment

- **Bengaluru drivable OSM graph:** 199k nodes and 495k arcs (measured).
- **Routing 400k random OD pairs (measured):**
  - CH/CCH: pandana CH (AGPL) about 4 s on 14 threads; routingkit-cch (BSD-2) about 13–15 s single-threaded, with full re-weighting in about 27 ms.
  - Per-vehicle Dijkstra: 2–20 h (scipy, igraph, rustworkx, networkx). One tree per origin zone narrows the gap.
  - CCH rerouting of 5% of vehicles every minute costs about 0.75 s CPU per simulated minute.
- **Equilibrium (DUE)** needs tens to hundreds of full runs. SUMO duaIterate defaults to 50; MATSim Switzerland 10% takes 301 iterations, 83 h with QSim and 47 h with Hermes, with replanning about 22 h. Equilibrium cannot run inside a real-time run; precompute routes and reroute every 5–15 min instead.

## Indian city scale

- **Registrations** are cumulative: Bengaluru 1.26 crore (Mar 2026), Greater Mumbai 51.3 lakh, Hyderabad's three districts about 88 lakh. The in-use fleet is about 45–60% of registered (Goel et al., Delhi 2012 data).
- **Peak concurrent vehicles** (Little's law estimates, about 2× uncertainty): Bengaluru metro about 0.22–0.45M; Greater Mumbai about 0.15–0.45M; all of MMR about 0.25–0.6M; Hyderabad metro about 0.25–0.55M. 4 lakh fits a **metro region at peak**. Two-wheelers are about half of concurrent vehicles in Bengaluru and Hyderabad.
- **Peak-hour share of daily trips:** about 12% in Bengaluru and about 5.5–6.6% in Mumbai, so a whole day runs near 4 lakh for only a few hours.
- **OSM drive networks:** BBMP (717 km²) 158k nodes / 398k directed edges; Hyderabad city relation 142k / 367k; Greater Mumbai 32k / 72k. Only 2–9% of edges carry a `lanes` tag. Boundaries have changed (GBA, GHMC split, MMR enlarged in 2019 and 2024).
- **Data availability:**
  - No public OD matrices or machine-readable counts.
  - Count tables do exist in PDFs. The Bengaluru CMP has 45 screen-line counts (16 h video), turning counts at 89 junctions and 13 cordon points.
  - Uber Movement 2019 archive: CC BY-NC.
  - Akbar et al. (AER 2023): speed indices for 180 Indian cities, open replication data.
  - Mumbai's island city has almost no autos (they are restricted by area), so the model needs per-vehicle-class spatial restrictions.

## Rendering and recording

- **Drawing is not the bottleneck on this Mac (measured):** deck.gl held 120 Hz at 400k–2M points with binary attributes; pygfx/Metal drew 400k in about 6 ms. Windows and integrated GPUs are unmeasured.
- **The data path is the limit:** a 400k×3 float32 frame is 4.8 MB. Python→Chromium WebSocket reached about 270 MB/s (browser-side limit; Python→Python reached 2.7 GB/s). pywebview/QWebChannel bridges are JSON/Base64 only. An in-process native renderer (pygfx/wgpu) avoids IPC.
- **Level of detail in existing viewers:** zoomed out, show per-link aggregates or batched dots; zoomed in, fetch vehicles only for the viewport via a spatial index (A/B Street, SUMO-GUI).
- **Recording 400k at 10 Hz:** SUMO FCD XML would be about 2 TB/h. Quantised delta + zstd is about 20–45 GB/h (optimistic, synthetic). deck.gl TripsLayer can't hold a whole hour at this scale, so replay needs time-windowed chunks.
