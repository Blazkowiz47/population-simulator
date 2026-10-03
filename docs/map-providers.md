# Map providers, data rights, and import boundaries

Status: planning decision, with provider facts checked on **2026-10-03**. No Google account, billing address, API entitlement, key, or paid service has been inspected or configured. The interfaces below are proposed; they are not implemented capabilities.

The project will import persistent simulation geometry from OpenStreetMap or other independently licensed data. Google Maps is a planned optional presentation/reference provider. These roles allow support for both map interfaces without promising that Google exposes an interchangeable road-and-building-network importer.

## Capability contract

| Capability | OpenStreetMap / independent open data | Google Maps Platform |
| --- | --- | --- |
| Persistent road-network import | Planned from XML/PBF extracts or bounded Overpass queries | No regional topology importer planned |
| Buildings, land use, and mapped amenities | Import what exists; record missing coverage | No scraping, satellite tracing, or persistent inventory harvesting |
| Map display | Planned using a selected tile provider or self-hosted map | Optional official Maps JavaScript SDK view |
| Independent simulation overlays | Planned | Technically supported; activation depends on the applicable contract and chosen presentation |
| Replay without network access | Core requirement using locally imported data | Simulation replay remains available; Google basemap is an online dependency |
| Offline basemap package | Only with self-hosted tiles or a provider permitting offline distribution | No tile download/archive feature planned |
| Optional routing or place context | Separate independently licensed services can be assessed | Defer exact APIs, response fields, caching, and permitted use until the account and workflow are known |

Google's Roads API supplies point/path operations: snap-to-road, nearest-road, and speed-limit lookup. Snap and nearest requests accept up to 100 GPS points. This is not a bounding-box export of a complete road graph, buildings, lane geometry, or junction controls. Its documentation lists an Asset Tracking licence requirement for speed limits. These facts establish a technical capability limit independently of the contractual limits below. [Roads API overview](https://developers.google.com/maps/documentation/roads/overview).

Google's Maps JavaScript Data layer can display externally supplied GeoJSON points, lines, and polygons with application properties. This supports a possible renderer for independent simulation output. The documentation alone does not establish permission for every combination of third-party geometry and Google content. [Data layer documentation](https://developers.google.com/maps/documentation/javascript/datalayer).

## OpenStreetMap acquisition and offline operation

Start with one bounded pilot corridor and a reproducible OSM snapshot. Use a supplied `.osm`/`.osm.pbf` file or a small Overpass query. For repeated or city-scale preparation, download a regional PBF extract, clip locally, and retain the complete objects needed for topology. Geofabrik provides India and subregion extracts, polygon boundaries, and update files. Snapshot dates and checksums belong in the import manifest; a mutable `latest` download URL alone is insufficient provenance. [Geofabrik India downloads](https://download.geofabrik.de/asia/india.html).

The OSM editing API is intended for map editing. The OSMF directs large or frequent readers toward bulk downloads or alternatives such as Overpass. Each public Overpass instance has its own resource limits. Proposed acquisition behaviour: bounded queries, endpoint identity, explicit timeouts, retries with backoff, local snapshot reuse, and no automatic city-wide queries against a public endpoint. [OSMF API usage policy](https://operations.osmfoundation.org/policies/api/).

Keep raw vector acquisition separate from visual map tiles. The public `tile.openstreetmap.org` service prohibits bulk prefetching and offline archives. It is not the source for a downloadable simulation area. Interactive viewing must follow the selected provider's policy; offline maps require self-hosted tiles or a provider explicitly permitting that use. [OSMF tile usage policy](https://operations.osmfoundation.org/policies/tiles/).

After import, compilation and simulation must work offline. A missing basemap should fall back to the application's own network view, while replay, metrics, and scenario editing remain usable.

## OSM licensing and attribution

OSM data is licensed under the Open Database License (ODbL). Use visible OpenStreetMap contributor attribution and link to the copyright/licence information. Distributed raw or derived map data needs appropriate licence information; changes to or additions to an OSM-derived database can trigger share-alike requirements. [OpenStreetMap copyright and licence](https://www.openstreetmap.org/copyright).

The software's eventual licence and the imported database's licence are separate decisions. A rendered visual output and an extractable network database can fall into different ODbL categories. OSMF guidance distinguishes Produced Works from Derivative Databases and explains obligations for the underlying database. Evaluate the actual export format before distributing scenario/network bundles; do not claim that every simulation output must use the same software licence. [OSMF Produced Work guidance](https://osmfoundation.org/wiki/Licence/Community_Guidelines/Produced_Work_-_Guideline).

Proposed attribution locations are the map view, exported figures, and an accompanying licence/provenance manifest for reusable network data. Preserve separate attribution for independently acquired observations or overlays.

## Google terms: standard billing regime

The standard [Google Maps Platform Terms](https://cloud.google.com/maps-platform/terms) include these relevant provisions:

- **3.2.3(a), No Scraping:** prohibits extracting/exporting Google Maps Content outside the Services, including bulk roads information.
- **3.2.3(b), No Caching:** permits only express service-specific exceptions.
- **3.2.3(c), No Creating Content From Google Maps Content:** includes road/building tracing from satellite maps and Google-content use for AI/ML training, testing, validation, or fine-tuning.
- **3.2.3(e), No Use With Non-Google Maps:** constrains use with or near non-Google maps.
- **3.2.2(b), Attribution:** requires supplied attribution to remain visible and unchanged.

Under the standard [Service Specific Terms](https://cloud.google.com/maps-platform/terms/maps-service-terms), **Roads 17.1–17.3** and **Routes 19.1–19.3** allow use without a corresponding Google map, prohibit use in conjunction with a non-Google map, and permit latitude/longitude caching for up to 30 consecutive days. That exception does not authorize retaining every response field for 30 days.

Place IDs have a documented indefinite-storage exception. Most other Routes content remains subject to caching restrictions. A future context adapter needs field-specific retention policies rather than one expiry setting for complete responses. [Routes API policies](https://developers.google.com/maps/documentation/routes/policies).

The project therefore excludes scraping tiles, reconstructing topology by bulk routing requests, tracing satellite imagery, and automatically converting Google content into persistent simulation geometry. This is the chosen implementation boundary under the reviewed terms, not a conclusion that all traffic simulation uses of Google services are impossible.

## Google terms: EEA billing regime

**Billing-account address determines applicability.** The simulated city, user's current location, and Oslo timezone do not establish the billing regime.

For EEA integrations from **8 July 2025**, Google removed the blanket restrictions concerning non-Google maps, recreation of Google features, and embedded vehicle systems. Older unmodified integrations have transition provisions. The project must not apply the standard non-Google-map prohibition to every customer. [Google EEA changes FAQ](https://developers.google.com/maps/comms/eea/faq).

The [EEA Terms](https://cloud.google.com/terms/maps-platform/eea), **3.3.2(a)–(c)**, retain scraping, caching, and content-creation restrictions, including road/building tracing and AI/ML uses.

The current [EEA Service Specific Terms](https://cloud.google.com/terms/maps-platform/eea/maps-service-terms) differ from the standard terms:

- **18.1:** Roads API speed limits cannot be used “With any Map.”
- **18.2:** Roads latitude/longitude caching is limited to 30 days.
- **20.1:** Routes descriptions or steps cannot be used “With any Map.”
- **20.2:** Routes latitude/longitude caching is limited to 30 days.
- General **2:** users must be able to distinguish Google content from third-party content.

A Google renderer and a Google context adapter are separate decisions. EEA removal of a blanket restriction does not authorize scraping, unrestricted caching, or every response field in every display.

## Proposed provider interfaces

Provider selection must advertise capabilities, rather than imply identical import behaviour.

| Interface | Responsibility | Persistent data allowed by this design |
| --- | --- | --- |
| `NetworkSource` | Acquire OSM or other independently licensed vector snapshots | Raw snapshot plus provenance and licence |
| `NetworkCompiler` | Normalize topology, widths, turns, junctions, and simulation surfaces | Provider-neutral network with assumptions and source references |
| `SimulationEngine` | Execute scenarios against compiled geometry | Deterministic scenario, run manifest, metrics, and replay |
| `MapRenderer` | Draw a basemap and independent simulation features | Application-owned display settings and simulation output |
| `GoogleContextService` | Later approved reference lookups, if useful | Only approved response fields with documented expiry and display rules |

Use `import_network`, `render_basemap`, `render_independent_overlays`, `offline_replay`, and `context_lookup` as explicit capability flags. The OSM source supplies the simulation network regardless of which renderer is selected. Neither compiler nor engine should call Google APIs.

The importer should preserve source object IDs, acquisition timestamp, endpoint or file URI, checksum, source CRS, licence, attribution, and extraction bounds. Keep source facts separate from inferred width, lane count, speed, building occupancy, and demand parameters. Give manual corrections their own author/date/reason. Project geometry into metres for dynamics, with a declared transformation for display coordinates.

Any later Google responses must remain isolated from reusable OSM snapshots, inference tables, demand-model training data, persistent topology, and replay bundles. This conservative project boundary preserves reproducibility while the exact approved reference workflow remains unsettled.

## Google activation criteria

OSM implementation can proceed immediately. Enable a Google integration only after the following concrete checks:

1. Record the actual billing jurisdiction, applicable contract, and enabled APIs. Do not infer them from this research.
2. Describe the display workflow and overlay provenance; resolve the chosen third-party-overlay presentation under that contract.
3. Choose the official SDK and the independent-data fields it will display. Keep network import outside the SDK.
4. Specify attribution, terms/privacy links, any response storage, field expiry, and deletion behaviour.
5. Configure restricted keys, usage quotas, and an agreed spend limit when account setup is explicitly requested.
6. Verify that absent credentials or provider failure falls back to the open-map/network renderer and never blocks simulation.

No Google SDK, credentials, billing configuration, or API response cache is part of the present scaffold. No map download or simulation has been run. Recheck the cited provider policies when implementing the integration; this document records the current planning evidence and unresolved access questions.
