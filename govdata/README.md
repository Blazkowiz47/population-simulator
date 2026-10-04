# Government records

Official statistics used to **build** synthetic populations and to **check** simulated years against published numbers. **We do not republish government data.** Downloaded files stay on the local machine; the repo holds only the catalogue, per-dataset manifests, and our own comparison results (simulated figure vs official figure, with a citation). The catalogue of what exists, where it lives and under what terms is in [`catalog.yaml`](catalog.yaml).

## Layout

```text
govdata/
  README.md
  catalog.yaml               # every known dataset: publisher, URL, period, geography, licence, access, role
  <dataset-id>/              # created only when a dataset is actually fetched
    manifest.json            # tracked: source URL, retrieved_at, sha256 per file, licence, access, notes
    raw/                     # NOT tracked (git-ignored): files exactly as downloaded
    comparisons/             # tracked: our simulated-vs-official results, each citing the official table and version
```

Dataset IDs are lowercase slugs that include the vintage, for example `census-2011-hh14-karnataka` or `plfs-2025-unit`.

## Rules

1. **Never commit downloaded government files**, open or restricted. This includes Census tables, MoSPI unit data (PLFS, HCES, TUS, NSS), NFHS/DHS and IHDS. Everything under `raw/` is git-ignored.
2. **Publish only our statements.** A comparison records our simulated value, the official value with its table, version and URL, the official margin of error if published, and whether that dataset was used to build the model.
3. **Registration and credentials stay with the user.** Datasets behind a login or API key are downloaded by the user, or by a script reading a key from an environment variable. Keys never go in the repo (`.env` is git-ignored).
4. **Every file is traceable.** Record the source URL, retrieval date, checksum, reference period and finest geography. A file without a manifest entry must not feed a model.
5. **Mark each dataset's role** as `build` (used to construct the population) or `check` (held out for validation). A dataset used to build cannot back a "simulation matches" claim for the same quantity, because that match is guaranteed by construction.
6. **India boundary maps:** use Survey of India-conformant boundaries for any map of India. Note this in the catalogue entry for boundary datasets.
