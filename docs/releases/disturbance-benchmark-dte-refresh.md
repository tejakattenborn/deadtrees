# Corrected DTE benchmark refresh (DT-1472)

The 28 October 2026 export refreshes the viewer predictions and scores after
regeneration of the DTE rasters. It supersedes the data in the
[v7 handoff](disturbance-benchmark-v7.md). The existing viewer and access controls
consume the same data contract; no frontend or authorization change is required.

## Release location

Use this run explicitly:

```text
/mnt/gsdata/projects/deadtrees/disturbances/11-Webviewer_visuals/runs/pylos-2026-10-28-dte-refresh
```

The same directory is available through the university NAS share under
`projects/deadtrees/disturbances/11-Webviewer_visuals/runs/`.
Only `export/` is the viewer payload. A prepared WebP version is under
`11-Webviewer_visuals/builds/pylos-2026-10-28-dte-refresh`.
Large assets and research data stay outside Git. The raw export has 114 sites,
16,641 files and 948,314,841 bytes (948.31 decimal MB). All 513 available product
evaluations passed original-raster/workbook checks; the audit found no errors
or missing references. All 93,706 workbook rows match the analysis snapshot.
The WebP build is 506,992,131 bytes (506.99 decimal MB), excluding its small
build manifest, with 663 additional previews. The completed conversion recovered
from a transient NAS stat failure by verifying every image and checking each JSON
against the publisher rewrite before finalizing; see `build-recovery-audit.json`.

The run's `analysis/` directory contains:

- `START_HERE_benchmark_20261028.html`: standalone report with the product-by-product
  matrices first, regional F1/recall/precision boxplots and the historical 39-case
  illustration. All chart data and images are embedded; no internet or sidecar
  files are required to view it.
- `benchmark_reproduce_20261028.zip`: portable Python scripts and frozen confusion
  counts, eligibility, configuration and results; no NAS access is needed to rerun.
- `README_REPRODUCE.txt`: executable reproduction steps and concise methods.
- `delivery_manifest.json`: report and reproduction-package SHA-256 hashes.
- `refresh_audit.json`: comparison with the earlier workbook snapshot and the
  independent sign correction.

## Definitions retained

DTE uses `dtbil10_fc_dw_comb_by`: deadwood increase OR forest-cover loss,
bilinear 10 percentage points. TCD uses `tcbil10_by`. Only before + current
periods contribute, never after. Source TIFFs and workbooks remain read-only.
The refreshed source snapshot has 342 workbooks and 93,706 metric rows.

F1 is recomputed directly as `2TP/(2TP+FP+FN)`. A covered product that detects
nothing while reference disturbance is present receives zero. Missing coverage
remains unavailable. Undefined precision remains null in the viewer; boxplots
show it as zero to reproduce the supplied plotting convention.

The viewer retains the `all` union reference and `da_by` DIST-ALERT comparator.
The scientific regional plots retain the dominant-area reference (`r_all` for
cover loss, `i_all` for standing dead trees) and `da_ann_by` DIST-ANN comparator.
These intentional differences mean viewer scores and regional scores need not
match. Both use original confusion counts, never display pixels, to calculate F1.

## Comparisons and method

There are 12 panels: Europe, Global, Tropical and South America, each split into
all cases, tree-cover-loss dominated and dieback dominated. For each pair, use
only its eligible shared site-years; average the per-case F1 differences with
equal site-year weight. A row-minus-column positive value favours the row.
Different pairs can use different samples, so this is not one universal ranking.

The 95% intervals use 10,000 cluster-bootstrap draws. Remove the final `_YYYY`
from each site ID to cluster repeated years, sample whole clusters and retain all
their years. Use NumPy seed 20261026 reset for each pair and percentile 2.5/97.5
bounds. One cluster has no interval. The embedded code states the exact eligibility,
header checks, reference consistency rules and sparse-data handling.

The report contains 189 unique pair/group comparisons and 2,634 paired records.
147 eligible no-detection cases remain F1=0. All-product intersections are only
20 globally, 4 in Europe, 8 in the tropics and 2 in South America; pairwise overlap
retains up to 92 cases for global DTE versus GFC.

## Verification

All 92 eligible DTE cases exactly match independently corrected TP/FP/FN/TN.
Three native-raster checks confirm later-minus-earlier deadwood change and
matching prepared inputs. The full refreshed viewer export independently checks
each available product's original prediction/reference raster counts against
its workbook, including alignment, CRS and valid binary pixels.

The report's checks cover 12 panels, all 378 directional matrix cells, paired
records, downloads and absence of external runtime dependencies. Subgroup counts
and effects reconcile with all-case comparisons. Static figures were visually
inspected. The reproduction bundle was rerun from a separate directory.
Browser layout and production authentication were not revalidated in this
data-only refresh; frontend code is unchanged.

Export completion, image/reference checks and exact asset sizes are recorded in
`handoff-audit.json`. `overlay-sampling-audit.json` records zero mismatches in 38,912 sampled pixels
across 19 evaluations;
`web-build-audit.json` confirms all 513 prediction masks are lossless and all
counts/scores are unchanged after WebP conversion. Review those files before
deployment. `viewer-webp.tar` and `viewer-webp.tar.sha256` provide the packaged
build and its checksum.

## Release after review

Follow the [internal deployment guide](disturbance-benchmark-internal.md). This
branch only records the reviewed data refresh; it does not publish it or change
the live `current` release. Merging documentation alone does not switch data.

```bash
RUN=/mnt/gsdata/projects/deadtrees/disturbances/11-Webviewer_visuals/runs/pylos-2026-10-28-dte-refresh
PY=~/.local/share/deadtrees/benchmark-venv/bin/python
export BENCHMARK_BUILDS_ROOT=/mnt/gsdata/projects/deadtrees/disturbances/11-Webviewer_visuals/builds
$PY scripts/disturbance_benchmark_publish.py build "$RUN"
$PY scripts/disturbance_benchmark_publish.py publish "$RUN" --dry-run
# Only after approval for the production data update:
$PY scripts/disturbance_benchmark_publish.py publish "$RUN"
```

The publisher preserves the previous release for rollback. After switching, reload
the viewer and check a corrected DTE site, its reference/prediction layers and scores.
Existing date warnings and TMF missing-zero-scene limitations still apply.
The 10% threshold remains the selected benchmark setting, not a claim of an
independently validated optimum.
