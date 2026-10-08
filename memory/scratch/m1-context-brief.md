# M1 context brief

Status: in-flight scratch, gathered 2026-10-03 on macbookpro. Nothing here is adopted. `docs/` is unchanged. Source: a context-gathering workflow with 6 gatherers (plan audit, longitudinal model, engine semantics, Python tooling, Indian vehicle parameters, knowledge-base recall) and 4 adversarial verifiers. Of 146 externally sourced claims checked, 0 were refuted and 20 were corrected; corrections are applied below. Key analytic values were recomputed independently. The raw workflow output was session-only and was not kept.

## 1. Plan defects found (fix in docs once decided)

| # | Issue | Where | Proposed fix |
|---|---|---|---|
| 1 | The time-step rule conflicts. PLAN says *measure* sensitivity at a smaller motion step. Architecture says *neither* the 0.1 s motion step nor the 0.5 s decision interval is accepted until halving gives stable outcomes. | PLAN.md:141 vs architecture.md:56 | See D1 |
| 2 | M1 scope is stated four ways, and only their union is complete (snapshots + time-step test in §6; CLI, replay, metrics, benchmark in §10). | PLAN.md:139-141, :254-256; memory/index.md:13; note :39,:60 | Write one consolidated M1 definition of done in §6 and have §10 point to it |
| 3 | Fixture location: `tests/fixtures/` vs the CLI examples' `data/scenarios/`. The `.gitignore` rule `data/` is unanchored and also ignores `tests/fixtures/data/` and `src/traffic_simulator/data/` (verified with `git check-ignore`). | PLAN.md:125 vs :108,:229-232; .gitignore:16 | Fixture in `tests/fixtures/<name>/`; anchor rules to `/data/` and `/outputs/` |
| 4 | Benchmark sizes 100/1,000/**10,000** vs 100/1,000/**5,000**, with different metric lists. M9 owns benchmarking, but §10 asks M1 for "one benchmark". | PLAN.md:191 vs architecture.md:104 | M1: N = 100 and 1,000, union of metrics; decide the top size at M9 |
| 5 | The run manifest must record a source revision, but the repo has no commits by design. | PLAN.md:117, :135 | See D5 |
| 6 | The two run-manifest field lists differ. | PLAN.md:117 vs architecture.md:86 | Use the union, plus scenario sha256 and Python/platform |
| 7 | Acceptance terms are undefined: "configured bounds", "unexplained overlap", "stable outcomes", "queue", "clears", "replayable", "snapshot", "free-flow reference". If IDM output is clamped, the bounds check passes by construction. | PLAN.md:141, :256; architecture.md:54-60, :92 | Definitions in §3 below |
| 8 | movsim/traffic-simulation-de is cited without a licence. It is **GPL-3.0** (HEAD e1b2d5d4, 2026-09-21); movsim/movsim is GPL-3.0-or-later (HEAD 1f97c908). MATSim's licence is also unrecorded. | architecture.md:40, :76 | Add to open-source-references.md as "reference only, no copying" |
| 9 | "Initial model distributions" is listed as a decision, yet "does not block M1", while M1 needs two vehicle sizes. | PLAN.md:246 vs :250, :254 | Use labelled placeholder classes (§5) |
| 10 | M0 has no status marker. README:5,24, the `__init__.py` status text and PLAN §8 go stale after M1. `--locked` is used inconsistently. | PLAN.md:133-135, :225; README | Update them as part of M1 |
| 11 | `.hypothesis/` and `.benchmarks/` are not ignored. | .gitignore | Add them when those tools are adopted |

## 2. Decisions for Sushrut (recommended default first)

- **D1 Time-step policy.**
  - *Reference mode:* decision interval = motion step; convergence is tested by halving the motion step. Run 0.5 s held acceleration as a separate behavioural variant.
  - *Verified:* with held acceleration plus ballistic sub-steps, halving the motion step leaves states at decision instants bit-identical, so it only tests event handling.
  - *Decision-interval halving* converges at first order (Treiber & Kanagaraj standard set; final gaps 1.8748 / 1.8285 / 1.8014 / 1.7833 m → 1.7708 m). The architecture.md:56 gate is satisfiable with a declared tolerance, but it measures reaction-time sensitivity, not only numerical error.
  - SUMO reports rear-end collisions when tau < action step, so validate T ≥ Δt_dec for every class.
- **D2 Halving tolerance, fixed before results are seen.** Ladder: 0.2 / 0.1 / 0.05 / 0.025 s. Accept 0.1 s if its difference from 0.05 s is ≤ 0.1 s in every arrival time and ≤ 1 m in max queue, with errors shrinking monotonically. A successive-difference ratio near 2 is a diagnostic only: stops and insertion quantization lower the observed order.
- **D3 Meaning of "bounds".**
  - Keep IDM comfort parameters a and b separate from the per-class physical limits a_max, b_max and v_max, plus a road speed limit.
  - Any clamp is logged.
  - Compliant acceptance requires zero clamp activations, zero overlap and v ≥ 0.
- **D4 Runtime dependencies.**
  - **(A, recommended for M1) zero runtime dependencies.** stdlib `random.Random`, with per-stream seeds derived through `hashlib` from (root seed, stream, key), and only `.random()`-based transforms (the only cross-version guarantee). Frozen dataclasses with hand validation.
  - **(B) numpy + pydantic.**
    - numpy: `SeedSequence(root, spawn_key=(stream, key))` keyed streams. Generator outputs are not stable across numpy X.Y releases. Each key component must be < 2**32, otherwise keys alias.
    - pydantic v2: strict mode on a `json.loads` dict rejects lists for tuple fields, so use `model_validate_json`; `allow_inf_nan` defaults to True and must be turned off.
  - Why A: PLAN.md:68 says to add packages when a milestone needs them, and the M1 fixture has at most one stochastic element.
- **D5 Initial commit before M1?** In either case, record the src-tree sha256 and a dirty flag in run.json.
- **D6 Exit:** free outflow with exit accounting. Exit capacity waits for M3 receiving-space work.
- **D7 Scope:** M1 is longitudinal-only. y and width are in the schema but held constant.
- **D8 uv upgrade:** local uv 0.10.4 vs latest 0.12.22. The 0.12.0 changelog reports no build-backend config breaks; widen to `uv_build>=0.12.22,<0.13`. Do it as its own step with a relock, because lockfile compatibility is only guaranteed within a uv minor version. A stale managed x86_64 CPython 3.8 makes `uv python list` fail; that is machine-level cleanup for Sushrut. Settled 2026-10-04: uv 0.12.23, `uv_build>=0.12.23,<0.13.0`, relocked (`../decisions.md`).
- **D9 Python version:** SPEC 0 recommends dropping 3.12 in 2026 Q4. Stay on 3.12 for M1 and decide before M2. Settled 2026-10-04: Python 3.14 (`../decisions.md`).

## 3. Proposed M1 engine design

- **Units and coordinates.**
  - SI throughout, with unit-suffixed keys (`length_m`, `max_decel_mps2`).
  - x = front bumper along the road, measured from the upstream end (the SUMO FCD convention).
  - Gap `s = x_leader − L_leader − x_follower` uses the **leader's** length (stated explicitly in Treiber & Kanagaraj 2015 p.4; the 2000 paper's eq. 3 is ambiguous).
- **Clock.**
  - Integer ticks, with dt in integer µs (100000 allows five exact halvings). Never accumulate `t += dt`: 0.1+0.1+0.1 ≠ 0.3.
  - An event fires on the first tick with t ≥ event time.
  - Fixture event times are multiples of the coarsest dt tested.
- **Model.**
  - Plain IDM, δ = 4: `a[1 − (v/v0)^δ − (s*/s)²]` with `s* = s0 + max(0, vT + vΔv/(2√(ab)))`.
  - The guard is the 2017 author documentation's form; Treiber & Kanagaraj eq. 21 instead uses `max(s0+…, 0)`. Record a model ID such as `idm-2000+s0max`.
  - Defer IIDM/ACC: the IIDM book formulas were not readable (the sample chapter returns 404).
- **Integrator.**
  - Ballistic update (T&K eq. 14) with an in-step stop (eq. 15): if `v + a·h < 0`, then `x += −v²/(2a)` and `v = 0`. The same rule holds a stopped vehicle whose gap is below s0.
  - Ballistic is first order, with about 30% of Euler's error.
  - SUMO's default is Euler, which matters for any later cross-check. SUMO's IDM also sub-steps internally at 0.25 s.
- **Tick order.**
  1. Process due events.
  2. Observe the frozen state S_k (front position descending; ties broken by vehicle_id).
  3. Decide, as a pure function.
  4. Resolve.
  5. Integrate.
  6. Swept check.
  7. Exit.
  8. Insert. As in SUMO, an inserted vehicle first moves on the next tick.
  9. Invariants, then outputs.
- **Resolve caveat.** A projection that only keeps *this step's* gap ≥ 0 is unsound. Followers can drift into states where even b_max cannot prevent overlap, so a real safeguard needs a Gipps/Lücken stopping-distance condition. For compliant M1, treat any clamp activation as a test failure rather than masking it.
- **Swept check.**
  - Within a ballistic step the gap is piecewise quadratic, with breakpoints at the stop time −v/a and at the vmax cap.
  - Take the analytic minimum on each sub-interval, pairing vehicles by their pre-step order.
  - The chord-dip bound G·h²/8 must use the *effective*, post-clamp accelerations. Example: a 0.22 m dip at h = 0.5 s when the leader stops mid-step.
- **Blockage.**
  - A `control.json` object `{id, position_m, length_m, active_from_s, release_at_s}`. It is a leader with v = a = 0, is not a trip, and emits start/release events.
  - *Placement rule (simulated here, dt = 0.1 s).* With the §5 car at 16 m/s, a stopped obstacle first seen at 40 m demands 9.8 m/s², at 80 m 2.45 m/s² (1.63 b), and at ≥ 100 m about 1.03 b. With the bus, 40 m demands 5.8 m/s² and ≥ 100 m about 1.03 b.
  - So: make the blockage active from t = 0, place it ≥ 150 m from the entry, and validate `v²/(2(s − s0)) ≤ b` at activation.
- **Insertion.**
  - departPos "base": rear bumper at x = 0.
  - Strict FIFO by (scheduled_tick, trip_id), one attempt per gate per tick.
  - Require gap ≥ s0, IDM acceleration ≥ −b at the chosen speed, and insertion speed ≤ the class v0.
  - Never drop demand (SUMO's `--max-depart-delay` defaults to −1).
  - With base placement, gate capacity depends on dt, so the halving test must separate insertion-limited from dynamics-limited outcomes.
- **Exit.**
  - Arrival time is the in-step front-bumper crossing, taking the smallest root in [0, h] and handling a stop before the line.
  - The vehicle stays a leader until its rear clears the line. The road beyond L is free.
- **Trip states.**
  - SCHEDULED → PENDING → ACTIVE → COMPLETED. CANCELLED(reason) is possible only before insertion.
  - Engine faults (overlap, NaN, bound violation) abort a compliant run: write a diagnostic snapshot and exit non-zero. They are not trip outcomes.
  - Unfinished trips are reported as `unfinished_pending` / `unfinished_active`.
- **Invariants, checked every step.**
  - loaded = scheduled + pending + active + completed + cancelled.
  - inserted_cum = active + completed.
  - IDs are unique and never reused; timestamps are monotone; all state is finite.
  - 0 ≤ v ≤ min(v0, limit); vehicle order is preserved.
  - Swept minimum gap > 0 is the fault check. Gap < s0 is only a diagnostic, because ballistic IDM settles slightly inside s0.
- **Trip metrics** (a SUMO tripinfo analog).
  - Times: scheduled_depart_s, insert_time_s, depart_delay_s, arrival_time_s (interpolated), travel_time_s = arrival − insert, total_time_s.
  - route_length_m and free_flow_time_s = L / min(v0, limit).
  - time_loss_s = travel − free_flow. It equals the per-step sum using dx/dt when v_ref is constant.
  - waiting_time_s and waiting_count use v ≤ 0.1 m/s. SUMO also requires accel ≤ 0.5·a_max; declare whether that is adopted.
  - Also: min_gap_m, max_decel_mps2, override_count, state and reason.
  - Completed-trip statistics are always paired with unfinished counts.
  - Queue: the contiguous halted platoon (v ≤ 0.1 m/s) upstream of the blockage. Length in metres = blockage rear − rear of the last queued vehicle.
- **Outputs.**
  - `run.json`: resolved config with defaulted values marked, seed and stream registry, scenario sha256, uv.lock sha256, Python version, platform and src-tree hash, plus a volatile block (run_id, created_at, elapsed, hardware).
  - `trips.jsonl`, `events.jsonl`, `summary.jsonl`, `metrics.json`, `replay/frames.jsonl` (sampling period independent of dt).
  - Snapshots: the full restartable state, including the pending FIFO and RNG state.
  - Every output file carries its own `format_version`.
  - JSON gotchas: `getstate()` returns tuples, dict keys become strings, and `inf` is not JSON (must be avoided).
- **Determinism.**
  - Deterministic files are byte-identical on the same machine, Python version and uv.lock.
  - Test: two subprocess runs with PYTHONHASHSEED 0 and 12345.
  - Never iterate a set of str. Serialize with `json.dumps(allow_nan=False, sort_keys=True, separators=(',', ':'))`.
  - Use `math.fsum`; Python 3.12 changed `sum()` for floats.
  - The gzip header embeds filename and mtime, so hash the uncompressed payload.
  - No cross-platform identity claim, because libm pow/exp can differ.
- **CLI.** argparse subcommands `run`, `check-scenario`, `benchmark`; `main(argv) -> int`. The bare command prints status and help. `python -m traffic_simulator` would need a `__main__.py`.

## 4. Test catalogue (reference values recomputed 2026-10-03)

1. **Constant-deceleration stop.** Stops at exactly `x0 + v²/(2b)` for any h; error ≤ 1.2e-13 m observed.
2. **Free road, δ = 1.**
   - Continuum: `v = v0(1 − e^{−at/v0})`, `x = v0·t − (v0²/a)(1 − e^{−at/v0})`.
   - The ballistic discrete solution is exact. With `r = 1 − a·h/v0`: `v_n = v0(1 − rⁿ)` and `x_n = v0·t_n − (v0²/a − v0·h/2)(1 − rⁿ)`.
   - The continuum error halves under dt halving.
3. **Free road, δ = 4, from rest.**
   - With u = v/v0: `t(u) = (v0/2a)[artanh u + arctan u]` and `x(u) = (v0²/4a)·ln[(1+u²)/(1−u²)]`. Verified here by derivative identity and RK4. Invert by bisection; no SciPy needed.
   - Check: v0 = 15, a = 1, t = 10 s gives v = 9.636435428968 m/s, x = 49.373920075556 m.
   - The 2000 paper's set (a = 0.73, v0 = 120 km/h) reaches 0→100 km/h in 43.235 s; the paper says "within 45 s".
4. **Equilibrium following.** `s_e(v_l) = (s0 + v_l·T)/√(1 − (v_l/v0)^δ)` is an exact fixed point at any h. With v0 = 15, T = 1, s0 = 2, v_l = 10: 13.395751335635 m.
5. **Stopped leader.**
   - The rest gap is exactly s0 only when aT² ≥ 2·s0 (creep). Otherwise the follower overshoots and stops inside s0, held there by the stop rule.
   - (a, T, s0) = (1, 1, 2) gives 0.886·s0. Placeholder car: 1.454 m (s0 = 1.5). Placeholder bus: 1.932 m (s0 = 2.0).
   - Assert 0 < gap ≤ s0. Peak-deceleration bounds must be tied to a stored fixture: 1.2b was exceeded in benign cases.
6. **Halving.** The motion-step ladder and the decision-interval ladder are reported separately (D1, D2).
7. **Conservation invariants.** Optionally a Hypothesis `RuleBasedStateMachine`. The "place blockage" rule needs a precondition placing it beyond braking distance, or failures are spurious.
8. **Reproducibility.** Subprocess runs with different PYTHONHASHSEED, plus snapshot → resume equality and `serialize(load(s)) == s`.
9. **Bounds.** IDM acceleration is ≤ a by construction. 0 ≤ v ≤ v0 holds only if Δt_dec ≤ v0/(aδ); validate this at load. An RK4 reference at h = 1e-4 for multi-vehicle cases is about 10^8 evaluations in pure Python, so generate it offline as a stored fixture. It is valid only in reference mode.

## 5. Fixture vehicle classes: labelled estimates, not calibrated

Use a car and a standard city bus. Defer two-wheelers and autos to M4 (scope, plus lateral filtering behaviour).

| Class | L×W (m) | v0 (m/s) | T (s) | s0 (m) | a (m/s²) | b (m/s²) | b_max placeholder (m/s²) |
|---|---|---|---|---|---|---|---|
| car | 4.46×1.85 (Chennai 2023 drone bounding box; Indo-HCM alt 3.72×1.44) | 16.0 | 1.2 | 1.5 | 1.0 | 1.5 | 9 (SUMO emergencyDecel) |
| bus | 12.0×2.6 (Urban Bus Spec-II; Indo-HCM alt 10.10×2.43) | 12.5 | 1.5 | 2.0 | 0.8 | 1.0 | 7 (SUMO), assumption |

Caveats:
- T comes from gross s/V medians. IDM's T excludes s0, so the IDM-equivalent value is lower.
- The Kanagaraj & Treiber 2018 class values are for the **ACC** model and come from a GA fit of about 25 parameters to 5 aggregates, so they are weakly identified. Their 9 m/s² is a collision-state value, not a braking limit.
- No Indian bus braking data was found.
- Almost all Indian microscopic evidence comes from one Chennai corridor (Saidapet).
- Aerial bounding-box widths likely include mirrors.

Each parameter needs a provenance record: value, unit, source_id, confidence, status (estimate or calibrated), measurement_method.

## 6. Calibration datasets (M8; never commit)

- **IIT Delhi/Noida drone** (Zenodo concept 10.5281/zenodo.17745347): CC BY 4.0, 6 sites. Records say 0.1 s exports, while the paper says 30 fps; check. Only DEL2 spans free to congested flow. Buses are 1–2%.
- **Chennai 2014** (Technion; Kanagaraj et al., TRR 2491): 3,005 trajectories, 0.5 s, 245 m. No licence is stated, so fetch on demand with citation and never redistribute.
- **SPT Chennai drone** (chennaitrafficdata.com): announced as "will be made open access"; no licence yet.
- IDD, UVH-26, ITD and TRAF are image-only and do not suit longitudinal calibration.
- No open trajectory data was found for Bengaluru, Mumbai or Hyderabad.

## 7. Tooling facts (checked 2026-10-03)

- **Local environment:** uv 0.10.4, CPython 3.12.12, no dependencies. Hardware: Apple M3 Max, 10 performance + 4 efficiency cores, 36 GiB.
- **Latest versions:**
  - pytest 9.1.1: `[tool.pytest]` in pyproject, `--import-mode=importlib`, `strict = true`.
  - hypothesis 6.168.3 (MPL-2.0), ruff 0.16.10, mypy 2.4.0.
  - numpy 2.5.3, pydantic 2.13.5, msgspec 0.22.0, pytest-benchmark 5.3.0, scipy 1.18.1.
- **`uv add --dev`** writes the PEP 735 `[dependency-groups] dev` table with lower bounds. `uv run` includes it by default. It changes uv.lock, and therefore the lock hash recorded in run.json.
- **Benchmark:**
  - A CLI subcommand timed with `perf_counter`, plus tracemalloc in a fresh subprocess.
  - `ru_maxrss` is in bytes on macOS and KiB on Linux, and is a process-lifetime peak, so use one subprocess per size.
  - `platform.processor()` returns only `arm`; use `sysctl machdep.cpu.brand_string`, which is macOS-only, so add a fallback.
  - SUMO's analogous metrics are realTimeFactor and vehicleUpdatesPerSecond.

## 8. Knowledge-base context

- The KB `traffic-simulator` workstream is an init stub. Today's project note has not been aggregated (last sync 16:02:59; project memory written 16:05). `KB_ROOT` and `KB_NODE_NAME` are unset in this shell.
- **ruview-python** (reusable): light core dependencies; `uv run pytest -q` after every milestone commit; deterministic synthetic fixtures labelled uncalibrated; explicit seeds in recorded runs.
- **jpd-revisit:** one RNG stream per concern. Reproducing a run needs an identity check, not only the seed.
- **pad-candidates:** key outputs by config hash. A past regression came from stale cached results.
- There is no prior personal convention for schema versioning, seeding APIs, simulation run manifests, CPU benchmarks or CLI frameworks.

## 9. Primary sources

- Treiber, Hennecke & Helbing 2000, IDM: arXiv:cond-mat/0002177, eqs. 6–14, Table I.
- Treiber & Kanagaraj 2015, integration schemes: arXiv:1403.4881, eqs. 14–15, 21. Physica A 419:183–195.
- Kesting, Treiber & Helbing 2010, ACC/CAH: arXiv:0912.3613.
- Albeaik et al. 2022, IDM well-posedness: arXiv:2104.02583.
- Lücken 2019, Gipps collision resolution: arXiv:1902.04927.
- traffic-simulation.de/info/info_IDM.html; Traffic Flow Dynamics errata (traffic-flow-dynamics.org).
- SUMO docs: VehicleInsertion, TripInfo, Summary, StatisticOutput, Safety, Basic_Definition, SaveAndLoad. Source at tag v1_27_1 (MSNet.cpp:851-920 step order). Licence EPL-2.0 OR GPL-2.0-or-later.
- A/B Street sim crate at v0.3.49 / c0d3b737 (Apache-2.0): discrete-event scheduler, no teleport, keyed parking RNG.
- numpy v2.5.3 random compatibility policy; CPython 3.12 docs (random, json, hashing).
- Indian data: Indo-HCM 2017 Table 1.3 (seen only via an unofficial copy); Dhamaniya & Chandra TRB 2018; Kanagaraj et al. TRR 2491 (2015); Kanagaraj & Treiber 2018 (arXiv:1805.05076); Agrawal et al. arXiv:2608.00602; Urban Bus Specifications-II (2013).
