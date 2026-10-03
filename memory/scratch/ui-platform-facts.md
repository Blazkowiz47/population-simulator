# UI / cross-platform stack facts

Status: in-flight scratch from a rubberduck session, 2026-10-03, macbookpro. This note records facts only; no stack has been chosen. A workflow of 5 research agents and 5 adversarial verifiers produced it. Of the claims checked, 0 were refuted and 38 were corrected; corrections are applied below. The raw output was session-only.

## Maps in Flutter (pub.dev and GitHub, checked 2026-10-03)

- **google_maps_flutter 2.18.2** (2026-09-26) supports Android, iOS and web only.
  - Flutter closed the macOS request as *not planned* on 2026-05-25 ("no plans to support the plugin on platforms without an SDK"; flutter/flutter#77880).
  - Google ships no Windows, macOS or Linux Maps SDK.
  - The only abandoned desktop plugin found is google_map_windows 0.0.2 (2022).
- **Google basemap on all six Flutter targets:** the only compliant route is Map Tiles API 2D raster tiles drawn in flutter_map.
  - Google's Map Tiles policies explicitly cover third-party renderers and allowing overlays of non-Google data.
  - They also impose rules: attribution and logo; no pre-fetching or caching beyond the terms; respect Cache-Control; visualization only, with no geodata extraction and no offline use.
  - EEA-billed projects created after 2025-07-08 get no satellite 2D tiles.
  - flutter_map 8 caches tiles by default, so caching needs review.
- **flutter_map 8.3.2** is pure Dart and runs on all 6 platforms.
  - In practice it is raster-only: vector_map_tiles has no working web support.
  - Each marker is a widget, and the maintainers warn about performance with many of them.
- **maplibre_gl 0.27.1:** Android, iOS and web only.
  - Per-tick GeoJSON updates are a measured bottleneck: about 9 updates/s at 2,000 points on one Android phone, and about 29.5/s with an experimental FFI path.
- **maplibre 0.3.6** (josxha, pre-1.0): Windows and macOS only through a WebView (experimental); Linux is unsupported despite its pub.dev badge.
- **Animating thousands of vehicles:** no published Flutter benchmark exists. The likely route is a custom CustomPainter/drawAtlas layer in flutter_map.

## Maps in the browser

- MapLibre GL JS 6.11.2 and deck.gl 9.4.0 are mature for this. deck.gl's GoogleMapsOverlay and MapLibreOverlay share one layer stack.
- deck.gl's "1M points at 60 fps" figure is for panning static data on 2015 MacBook Pros. For data that changes every frame, the docs warn of stutter "even for layers with just a few thousand items" unless binary/typed-array attributes are used.
- MapLibre GL JS 6 requires WebGL2.

## Python inside Flutter

- **serious_python 5.0.0** (2026-09-26, Flet team) embeds CPython 3.12.14, 3.13.15 or 3.14.7 (default 3.14).
  - Platforms: Android, iOS, macOS, Windows, Linux. **No web runtime.**
  - Its in-process byte channel, `PythonBridge` (dart_bridge FFI), measured about 80 µs per small round trip and 4.5–7.2 GB/s throughput (M2 Pro, debug build).
  - numpy and others for mobile come from pypi.flet.dev (about 165 packages; no numba or rtree).
  - Crash fixes for numpy-heavy code on macOS landed in 4.7.1, about 2026-09-22.
- **Flet 1.0.0** (2026-09-14; latest 1.0.3) is Python UI rendered by Flutter.
  - `flet build`: apk, aab, ipa, macos, windows, linux, web.
  - Web modes: static (Pyodide in a module worker) or dynamic (Python on a server over WebSocket).
  - flet-map wraps flutter_map: raster only, no Google.
  - flet-webview has no Windows or Linux support.
  - RawImage (GPU texture streaming) and Canvas are candidate vehicle renderers; untested with the map.
  - In 1.0, event handlers run on the event loop, so a simulation loop must be moved off it.
- **iOS:** no subprocess or multiprocessing, so the engine must run in-process or on a remote server.
- **Android:** about 10 s after an app is cached its threads are frozen, unless it runs a foreground service.
- **Official CPython mobile support:** PEP 730/738 are Final (3.13) and Tier 3. python.org ships Android builds since 3.14.0, and iOS builds only as 3.15 pre-releases.
- **No mature JSON-Schema form renderer for Flutter:** the best candidate, json_schema_form_builder, is 0.1.0 and 10 days old. A Flutter UI would need its own schema-to-widget mapper.

## Python in the browser (Pyodide)

- Pyodide 314.0.7 (2026-09-14) runs CPython 3.14.2 with numpy 2.4.6, and ships pydantic 2.12.5, shapely and networkx.
  - Single-threaded: no threads, multiprocessing or subprocess.
  - Basic use needs no cross-origin isolation.
  - First load is about 6.2 MB (core) plus 2.9 MB (numpy).
- **Speed:** the docs say 3–5x slower than native, but that text dates from 2021. The agent's own runs on this M3 Max in Chromium 152:
  - pure-Python kernels 2.2–3.4x slower;
  - a toy 1,000-vehicle IDM step about 1.05 ms vs 0.31–0.38 ms native;
  - a 2D neighbour step about 5 ms vs 2.3 ms.
  - The real-time budget at 0.1 s steps is 100 ms. Sim-speed multipliers and sweeps shrink it; mobile browsers were not measured.
- **Knob updates while running:** step in chunks and yield between them, or use JSPI `run_sync` (Chrome 137, Firefox 153, Safari 27). Neither needs COOP/COEP.
- **Data transfer:** flatten arrays before `to_js` (1-D: about 2–6 µs; 2-D: about 0.25 ms).

## Precedents

- **Engines that run in the browser:** in traffic simulation, only JavaScript (traffic-simulation.de) or Rust compiled to WASM (the A/B Street family).
- **Python, C++ and Java engines** (SUMO, CityFlow, MATSim, MovSim) run natively. Their web UIs either stream from a server or replay saved files (SimWrapper, CityFlow).
- **A/B Street** shipped one Rust codebase as native + WASM (v0.3.49, 2024-01-09). The author's newer tools are web-only: Svelte + MapLibre + Rust-WASM. Their reason: "Web-only means the release process can be super simple".
- **No notable traffic simulator with a Flutter frontend** was found.
- **Distribution costs for native apps:**
  - Apple developer programme: $99/yr.
  - Windows signing: Azure Artifact Signing, whose eligibility for Norway is unclear, or an OV certificate at $150–300/yr.
  - Per-OS CI runners: no stack cross-compiles desktop builds, and iOS needs macOS.

## Knobs from one schema

- Pydantic 2.13.5 and msgspec 0.22.0 both emit JSON Schema 2020-12. Custom metadata must use the **`x-` prefix**, e.g. `x-unit`, `x-group`, `x-tier`, `x-live`, `x-soft-min`. The JSON Schema project adopted that prefix in a 2023 ADR, and the next core spec rejects other unknown keywords.
- Cross-field rules (dt ≤ tau, action step ≤ tau) must stay in the Python validators.
- **Web form renderers are mature:** RJSF v6.11.0 (2026-09-28) and JSON Forms v3.8.0 (basic/expert show/hide rules).
- **Slider requirements:** sliders need inclusive bounds and an explicit default. `Optional[float]` never becomes a slider.
- **Python-native auto-UIs:**
  - Panel/param maps parameters to widgets best, but its Pydantic support is pending.
  - Streamlit re-runs the whole script on each change.
  - ipyleaflet hangs past about 2,000 markers.
  - The deck.gl-based options (pydeck, lonboard) scale to many points.
- **Precedents separate three tiers:**
  - numerical settings, for experts (SUMO step-length, MATSim timeStepSize);
  - per-vehicle-type behaviour;
  - viewer pacing (sumo-gui delay, Mesa render interval), which must not change outputs.
- SUMO TraCI allows live behaviour setters (tau, accel, decel, actionStepLength…), but the step length is fixed after startup. That is a precedent for an `x-live` flag.

## Not yet researched / open

- MP4 export cost and the licensing of video basemaps. The plan is to draw our own OSM-derived basemap.
- The real engine's step cost at about 1,000 vehicles, native and in Pyodide.
- Flutter frame rates for animated vehicle layers.
- Whether Google ToS §3.2.3(e), "no use with or near a non-Google map in a Customer Application", allows an OSM/Google switch in one app. Under the standard terms it may not; the EEA terms differ. This needs legal review; see docs/map-providers.md.

## Desktop only (macOS arm64 + Windows), checked 2026-10-03

Same workflow pattern: 5 research and 5 verification agents. Of the claims checked, 1 was refuted and 31 corrected; corrections are applied below. "Measured" means run on this M3 Max with macOS 27.0.1. Windows was not measured.

### Python + HTML/JS in a native window

- **pywebview 6.2.1** (2026-04-15) uses WKWebView on macOS and WebView2 on Windows.
  - Measured: WebGL2 and WebGPU are available on macOS arm64, including inside a PyInstaller .app.
  - Measured transport: the JSON bridge takes about 0.2 ms per push for 1,000 vehicles; a loopback binary WebSocket sustained over 400 frames/s at 64 KB per frame.
  - Measured size: a macOS app is about 33 MB (16 MB zipped).
- **Windows caveats for pywebview:**
  - If the WebView2 runtime or .NET is missing, it silently falls back to IE11/MSHTML, which has no WebGL. The app must check for this itself and install the Bootstrapper.
  - WebGL no longer falls back to SwiftShader (Edge 144+), so GPU-less VMs and blocklisted drivers get no WebGL. Pass flags through `WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS`.
  - It does not start under native Windows ARM64 Python, so ARM64 means an x64 build under emulation until PR #1832 or the WinUI3 backend ships.
- **NiceGUI 3.17.1:** native mode uses pywebview.
  - Measured: 1,000–4,000 vehicles pushed as JSON arrived at 30–60 Hz, with about 4–10 ms p50 latency (transport only).
  - `ui.leaflet` sends one message per marker, so an animated map needs a custom deck.gl/MapLibre component.
  - Native app size is about 77 MB.
- **Tauri / Electron:** each adds a second toolchain (Rust or Node) plus sidecar lifecycle management.
- **Google Maps JS in a desktop webview:**
  - Google's error docs acknowledge WebView2. WKWebView on macOS is not on Google's supported list.
  - The API key cannot be meaningfully origin-restricted.
- **OSM tiles at runtime:** need an app User-Agent, a Referer and caching. Since about March 2026, requests without a Referer get HTTP 403. A bundled PMTiles extract avoids tile servers entirely.

### Qt (PySide6 6.11.2)

- **Licensing:** LGPLv3, except Qt Charts, Qt Graphs, Qt Quick 3D and Canvas Painter, which are GPL-only. Open-source users do not get LTS patch releases on PyPI, so expect to move to a new minor version about every 6 months.
- **QtWebEngine:**
  - Size: about 220–270 MB on Windows x64 and about 550 MB universal2 on macOS.
  - The Windows arm64 wheel has no QtWebEngine.
  - It needs extra macOS entitlements (allow-jit and others).
  - The PySide6 wheels also ship Qt WebView backends that use WebView2 and WKWebView without bundling Chromium.
- **Native maps are weak:**
  - Qt Location is still a technology preview in Qt 6.12. Its OSM plugin's defaults break OSMF policy (generic User-Agent, prefetching, Thunderforest tiles that show a watermark).
  - maplibre-native-qt cannot be used from Python without C++ work.
- **Knob forms:** no maintained library turns Pydantic or JSON Schema into Qt forms. magicgui's private builder is partial; pyqtgraph ParameterTree and guidata need glue code.
- **Drawing many vehicles:** candidates are `QPainter.drawPixmapFragments` fed from numpy, `QRhiWidget` (Metal/D3D11; shaders precompiled with qsb, a GPL tool), vispy, and fastplotlib/pygfx (alpha). There are no published benchmarks.

### Flutter / Flet desktop

- **WebViews:**
  - On Windows, every WebView2 plugin captures the page into a Flutter texture.
  - flutter_inappwebview's last stable release is from 2024; its beta is from Feb 2026, with no commits since and open crash bugs.
  - On macOS, Flutter's native-view embedding is "not fully functional".
- **Python inside the app:** serious_python and Flet are x64-only on Windows.
- **Flet 1.0.3** pins Flutter 3.44.8. Open issue #6874 says `flet build macos` fails on Xcode 27, which is the version on this Mac.
- **Vehicle layer:** it would have to be custom Dart (a flutter_map CustomPainter/drawRawAtlas layer), with no benchmarks published.

### Packaging and distribution

- **No cross-compiling:** every tool needs CI runners on macOS arm64 and Windows. GitHub's macos-14 runner image retires 2026-11-02.
- **macOS signing:** Developer ID plus notarization requires the Apple Developer Program ($99/yr). Since Sequoia, an unsigned app opens only via System Settings > Privacy & Security > Open Anyway.
- **Windows signing:**
  - Unsigned apps get a SmartScreen prompt with "Run anyway".
  - Smart App Control blocks unsigned .pyd/.dll files with no per-app exception. numpy's .pyd files are unsigned (numpy declined to sign them); PySide6's DLLs are signed by The Qt Company.
  - Options: Artifact Signing at about $10/month, organisations only; Microsoft's docs conflict on whether Norway is eligible. Or an OV certificate at $150–300/yr.
- **Tools:**
  - Briefcase is the most turnkey route: signed and notarized DMG plus an MSI. Set `universal_build=false` for Apple-silicon-only builds.
  - PyInstaller should be used in onedir mode; onefile combined with a macOS .app will be blocked in v7.
  - For auto-update, Velopack works with PyInstaller onedir.

### Rendering and MP4 export

- **Measured drawing cost** for 1,000 / 3,000 rotated rectangles at 1080p:
  - OpenCV fillPoly: 1–2 / 2–6 ms per frame.
  - Pillow: 3.5 / 8.9 ms.
  - matplotlib Agg: 2.8 / 7.8 ms.
  - End to end, OpenCV drawing piped to ffmpeg ran at 180–300 frames/s. matplotlib `FuncAnimation.save` with a vector basemap ran at about 20 frames/s.
- **Encoders:**
  - PyAV 19.0.1 has wheels for all targets, but bundles GPL x264/x265.
  - For non-GPL distribution, use an LGPL FFmpeg with VideoToolbox (macOS) or MediaFoundation/NVENC/QSV/AMF (Windows).
  - imageio-ffmpeg is stale and has no win_arm64 wheel.
- **Basemap in exported video:**
  - Render it ourselves from an OSM PBF or PMTiles extract. That is an ODbL Produced Work, so show "© OpenStreetMap contributors" in the frame. Never fetch OSMF tiles for export.
  - Do not bake Google tiles into exported MP4s. Maps Platform terms §3.2.3(a)/(b) forbid it, and the Map Tiles API allows video only as promotional clips of 30 s or less.
- **Precedents:** SUMO, Via and Mesa render frames deterministically offline and encode them with ffmpeg rather than screen-recording.
- **Live view without web tech:** pygame-ce is maintained and has wheels for all three targets. The GPU options (Arcade, pyglet, vispy, moderngl) rely on OpenGL, which is deprecated on macOS and limited on Windows arm64.
