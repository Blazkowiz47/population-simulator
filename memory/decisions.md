# Decisions

Record meaningful project decisions and why they were made.

| Date | Decision | Reason | Consequence | Revisit? |
|---|---|---|---|---|
| 2026-10-03 | Build an independent engine and take inspiration only from open-source work. | Explicit user direction. | SUMO is a reference, not the simulation runtime; no upstream code copied. | On explicit scope change. |
| 2026-10-03 | Focus the product on Indian mixed traffic, including Bengaluru, Mumbai, and Hyderabad. | Explicit user interest. | Local demand/behaviour validation is needed; pilot region remains open. | When choosing the pilot. |
| 2026-10-03 | Bootstrap a packaged Python 3.12+ uv application. | User requested uv; a headless Python package supports inspectable experiments. | uv.lock and an editable local package exist; acceleration remains a profiling decision. | After performance measurements. |
| 2026-10-03 | Plan OSM network import and a separate optional Google Maps display adapter. | Current API capabilities and provider terms differ. | Persistent Google road extraction is not promised; account/workflow review precedes activation. | If suitable licensed access becomes available. |
| 2026-10-03 | Publish the project as a public GitHub repo named `population-simulator`, default branch `master`. | Explicit request from Sushrut. | First commit and remote exist (M0's 'no commits or remote' no longer holds). Local dir, package and CLI are still `traffic-simulator`. No licence file, so default copyright applies. | When the rename and licence are settled. |

## Notes

- The detailed milestones and model settings in `docs/PLAN.md` and `docs/architecture.md` are proposals. The project's own software licence has not been selected.
