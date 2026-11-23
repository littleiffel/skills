# Quality checklist for Earth-observation and geospatial changes

Use this while grilling when a change affects data. Not every area applies to every change: pick the ones where the change could make the output wrong, and turn each into a question with a recommended answer. The aim is to leave the grilling with metrics, thresholds, reference data and a test set that the implementer can run without asking anyone.

## Areas to cover

| Area | What can go wrong | Typical checks |
|------|-------------------|----------------|
| Correctness against a reference | Classes, masks or retrieved values are wrong | Accuracy, F1, IoU against labelled samples; RMSE, bias or R² against in-situ or a trusted reference product |
| Geometry | Output is shifted, resampled wrongly or clipped | CRS, resolution, extent and pixel-grid alignment match the spec; co-registration shift under a set number of pixels |
| Values | Wrong units, scaling or nodata handling | Value ranges, dtype, scale/offset, units, consistent nodata value, no NaN where nodata is expected |
| Completeness | Missing tiles, dates or areas | % coverage of the AOI, % nodata or cloud, no missing dates in the requested range |
| Temporal consistency | Jumps between dates caused by processing, not reality | Time series on stable targets stay flat; no step change at sensor or processing-version boundaries |
| Regression | Unintended change compared with the current version | Pixel-wise difference statistics against the current pipeline on the same inputs; expected changes only where intended |
| Format and metadata | Downstream tools can't read the output | Valid COG or other target format, STAC or metadata fields present and correct, file naming convention |
| Operations | The pipeline becomes too slow, expensive or flaky | Runtime, memory and cost per scene or tile; reruns are idempotent (no duplicates, same result) |

## Questions that usually need settling

- **Reference data.** What do we compare against: labelled samples, in-situ measurements, another product, or the current version? Does it exist and can the pipeline reach it? If not, getting it is a stage of its own.
- **Thresholds.** What number counts as good enough? If nobody knows an absolute number, "no worse than the current version" is a valid threshold, but it needs a measured baseline first.
- **Test set.** A small, fixed set of inputs that every stage runs on. Pick a few AOIs and dates that cover the hard cases (heavy cloud, coastline or water edge, orbit or tile boundary, seasonal extremes) and say where they are stored. Small enough to run in minutes, so it gets run.
- **Baseline.** If the current output has never been measured, make "measure the baseline" the first stage. Without it no later stage can show that it improved things or kept them the same.
- **Where the checks live.** Cheap invariants (CRS, dtype, ranges, nodata) go in unit or integration tests. Metrics on the test set go in a validation script or report that a stage can run. Say which, so "How to verify" in each stage can point at it.
