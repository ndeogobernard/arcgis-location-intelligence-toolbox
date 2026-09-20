# ArcGIS Location Intelligence Toolbox

A config-driven ArcGIS Python toolbox (`.pyt`) for multi-criteria site selection — screen a
parcel universe against hard filters, analyse it on a road network, score candidates against
weighted criteria, and test how stable the ranking is.

Built so the analysis is **reproducible and testable**, not a sequence of remembered clicks.

## Design

Three rules shape the whole toolbox:

**1 · The toolbox is a thin wrapper.** Tools define parameters, validate them, and log. Every
piece of real logic lives in the importable `li` package under `src/`, so it can be unit-tested,
called from a CLI, and reviewed as ordinary Python.

**2 · Nothing analytical is hard-coded.** No paths, thresholds, or weights in code — all of it
comes from `config/`. A magic number in a `.py` file is treated as a bug. Changing a screening
threshold or a weighting scenario means editing YAML, not hunting through source.

**3 · The tests do not need arcpy.** arcpy cannot be installed on a hosted CI runner, so
everything except geodatabase construction is written to work without it. The suite runs in
GitHub Actions in under a second, and CI asserts arcpy is genuinely *absent* so the guarantee
cannot lapse silently.

## Tools

| # | Tool | Status |
|---|---|---|
| 1 | **BuildGeodatabaseSchema** — build the full schema from YAML | ✅ |
| 2 | IngestAndStandardize — download, reproject, field-map, load | ◻ |
| 3 | RunQAQC — geometry, duplicates, CRS, domains, topology | ◻ |
| 4 | BuildNetworkDataset — travel modes, restrictions | ◻ |
| 5 | ScreenCandidateSites — hard filters, auditable fail reasons | ◻ |
| 6 | BuildServiceAreas — batched, resumable | ◻ |
| 7 | BuildODMatrices — OD cost matrices | ◻ |
| 8 | ComputeWorkforceMetrics — ACS / LODES aggregation | ◻ |
| 9 | ComputeCriteriaScores — winsorize + min–max | ◻ |
| 10 | WeightedSuitability — composite, rank, percentile | ◻ |
| 11 | SensitivityRunner — OAT + Monte Carlo | ◻ |
| 12 | ExportSiteProfiles — profiles, map-series index | ◻ |
| 13 | PublishToAGOL — hosted feature layers | ◻ |

Tool 1 is complete and verified. The rest are specified and land as the parent study reaches
each phase — this repository tracks that work rather than promising it.

## Using it

Open `toolbox/LocationIntelligence.pyt` in the ArcGIS Pro Catalog pane, or call it from Python:

```python
import arcpy
arcpy.ImportToolbox(r"toolbox/LocationIntelligence.pyt", "li")
arcpy.li.BuildGeodatabaseSchema("config/schema.yaml", r"C:\GIS\out.gdb", None, True, True)
```

Every run gets a `run_id` of `YYYYMMDD_HHMM_<scenario>` and logs the **git commit hash**, so any
output traces back to the exact code that produced it.

## Tests

```bash
python -m pytest tests/ -q
```

26 tests, no arcpy, sub-second. They validate the schema and the configuration together —
domains declared before use, domain types matching field types, relationship keys present on
both sides, subtype fields integer, and criteria staying in step with weights.

## Requirements

- **ArcGIS Pro 3.x** with **Network Analyst** and **Spatial Analyst** for the full pipeline
- Tool 1 needs neither extension
- A cloned conda environment — `arcgispro-py3` is read-only:

```bash
conda create --clone arcgispro-py3 --name li-toolbox
```

## Notes from building it

Things that cost time and are not obvious from the documentation:

- Attribute rules require **Global IDs** first, or you get `ERROR 002710`.
- Subtype fields must be **SHORT or LONG** — a text field cannot be one.
- `arcpy.da.Describe()` does **not** expose topology rules. To verify them, export an XML
  workspace document and parse it.
- ArcGIS caches `.pyt` modules between runs, so the toolbox reloads its own imports — otherwise
  edits appear not to take effect until Pro restarts.

## License

MIT — see [LICENSE](LICENSE).

---

A component of [dsg-dfw-site-selection](https://github.com/ndeogobernard/dsg-dfw-site-selection),
a DFW regional-DC site-selection system.
