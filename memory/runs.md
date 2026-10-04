# Runs

Track experiments, long-running jobs, evaluations, and important development runs.

| Date | Device/server | Branch/commit | Command/config | Dataset | Output path | Result | Next |
|---|---|---|---|---|---|---|---|
| 2026-10-04 | macbookpro | n/a: throwaway spike outside the repo (repo untouched at ca8246a) | `uv run --no-project --python 3.12 --with nicegui [--with pywebview] python app.py`, browser mode and native window; prep via pyarrow, shapely, numpy | Overture places and buildings, bbox 72.820,19.010,72.860,19.065 (central Mumbai); 300,000 random synthetic people | Session scratchpad `map-spike/`, deleted after verification at Sushrut's request | Works. Native macOS window (pywebview/WKWebView) with three NiceGUI map panels, each one MapLibre + deck.gl component: 1,717 stores (41 ms), 27,986 building footprints (0.4 s), 300k people (1.0 s). JS→Python click events and Python→JS `run_method` with a return value verified. Sushrut: "I love this". | Keep this map path; next test Windows/WebView2, drawing a snip, offline basemap, millions of points. |

## Notes

- No traffic experiments have run. Package setup checks are recorded in the daily note.
