# P3.1 Frozen Observed-Panel Workload

Status: P3.1 implementation evidence from `polars20261115a`. This document
defines the workload to be implemented unchanged by later pandas and PySpark
milestones. It is not a performance comparison, prediction population, label,
or model feature table.

## Input and output contract

The authoritative assignment PDF, page 2, requires efficient Polars core
transformations, a later pandas comparison, a later PySpark reproduction, and
engine-plan interpretation. This P3.1 scope uses the 170 Financials partitions
of accepted P2.1 treatment run `treatment20261114a`, 1984Q1–2026Q2. The
exact ordered paths, SHA-256 values, and row counts are frozen in ignored
`runs/task_3_1_polars/polars20261115a/contract.json`; no glob can add a
partition. The P2.2 `panel20261115b` is an independent row-level reference.
The P2.1 `audit20261114b` provides independent quarter-level quality counts.
The scan sets `hive_partitioning=False`: the physical `REPDTE` is Arrow Date,
and inferring the `REPDTE=YYYYMMDD` directory as a second column would create
a conflicting type. The partition inventory is validated from manifests
instead.

| Relation | Ordered columns and logical types | Key and order |
| --- | --- | --- |
| Observed | `CERT Int64`, `REPDTE Date`, `ID String`, `P3ASSET Float64`, `SOURCE_MISSING_FIELDS List(String)`, `DQ_FLAGS List(String)` | Unique `(CERT, REPDTE)`; order `(REPDTE, CERT)` |
| Gaps | `CERT Int64`, `PREVIOUS_REPDTE Date`, `NEXT_REPDTE Date`, `MISSING_INTERVENING_QUARTERS Int32` | One event for each pair of consecutive observed reports with at least one intervening quarter; order `(CERT, NEXT_REPDTE)` |
| Quality | `REPDTE Date`, `ROW_COUNT UInt32`, `DISTINCT_KEY_COUNT UInt32`, `P3ASSET_NULL_ROWS UInt32`, `P3ASSET_SOURCE_ABSENT_ROWS UInt32`, `P3ASSET_EXPLICIT_NULL_ROWS UInt32`, `UNMATCHED_INSTITUTIONS_SNAPSHOT_ROWS UInt32` | One row per quarter; order `REPDTE` |

`ID` is retained to compare source-record identity. `SOURCE_MISSING_FIELDS`
distinguishes fields absent from an FDIC response from explicit null values;
`DQ_FLAGS` is retained verbatim. `P3ASSET_NULL_ROWS` counts all null values;
source-absent and explicit-null counts partition it exactly. An absent field
with a non-null value fails validation. The unmatched count tests exact flag
`financial_cert_absent_from_institutions:CERT`. This is a comparison with a
later Institutions snapshot and **not** a point-in-time predictor. No raw
value is imputed or replaced, and no bank-quarter is synthesized or
deduplicated. Duplicate `(CERT, REPDTE)` keys fail the workload.

For gaps, map a quarter end to `year*4 + month//3 - 1`. Sort each bank's
observed dates and use consecutive pairs. The missing-quarter count is
`quarter_index(next) - quarter_index(previous) - 1`; emit only positive
values. This works across year boundaries. A temporary gap is not an exit.

The three relations are materialized when their Polars `LazyFrame.collect()`
calls execute, after their optimized plans are captured. They are stored as
three separate ZSTD-3 Parquet files in ignored
`data_processed/task_3_1_polars/polars20261115a/`. The accepted
`contract.json` specifies the schemas, ordered input inventory, null/key/
ordering semantics, and expected output rows and checksums. The CLI
`compare` reads all three candidate relations and requires exact types,
column order, row order, nulls, list values, keys, and field values against
these frozen outputs. It does not use an aggregate-only or tolerant match.

## Observed acceptance

| Check | Independent result |
| --- | ---: |
| Financials quarters scanned | 170 |
| Observed bank-quarters and unique keys | 1,682,616 / 1,682,616 |
| Projected row/value differences versus P2.2 | 0 |
| Resumed gap events / distinct banks | 33 / 32 |
| Missing intervening quarters | 92 |
| Exact gap-record differences versus P2.2 | 0 |
| `P3ASSET` null / source-absent / explicit-null rows | 971,925 / 0 / 971,925 |
| Financials rows flagged unmatched to Institutions snapshot | 10,996 |
| Quarterly quality-count differences versus P2.1 | 0 across 170 quarters |

The run validated the complete accepted treatment manifest, all 172 treated
output checksums and schemas, the 171 relevant P2.2 reference Parquet hashes,
and the P2.1 findings checksum before processing. It rehashed the 170
Financials inputs, reference evidence/files, and findings afterward. Three
new Parquet outputs passed atomic write/read-back and SHA-256. The saved
manifest has passing validations; the contract was marked frozen only after
exact parity. The environment recorded in `metadata.json` was Windows build
10.0.26200, Python 3.11.9, Polars 1.35.2, with all dependency versions and
paths recorded there. No read-time or memory benchmark was performed.

## Actual optimized plans

The three complete plans are saved beside the contract as
`observed_optimized_plan.txt`, `gaps_optimized_plan.txt`, and
`quality_optimized_plan.txt`. The observed scan shows `PROJECT 6/29 COLUMNS`
followed by sort `(REPDTE, CERT)`. The gap scan shows `PROJECT 2/29 COLUMNS`,
then sort `(CERT, REPDTE)`, a `shift(1).over(CERT)` window, calendar-quarter
arithmetic, a positive-gap filter, and final sort. The quality scan shows
`PROJECT 5/29 COLUMNS`, then list-membership evaluation, aggregation by
`REPDTE`, and final sort. These plans support a claim of scan-level column
projection. They do **not** show quarter pruning (all 170 paths are scanned)
or a gap-filter pushdown into Parquet. The plan's estimated row count is an
optimizer estimate, not the measured 1,682,616 rows.

## Reproduction

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:PYTHONDONTWRITEBYTECODE = '1'
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel show --project-root C:\NEU\BFT65B --run-id polars20261115a
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel plans --project-root C:\NEU\BFT65B --run-id polars20261115a --relation gaps
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel compare --project-root C:\NEU\BFT65B --run-id polars20261115a --candidate-dir C:\NEU\BFT65B\data_processed\task_3_1_polars\polars20261115a
# Use a fresh run ID to reproduce without replacing prior evidence.
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel run --project-root C:\NEU\BFT65B --run-id polars_new_unique_id
```

Later pandas and PySpark code must use the exact input inventory and logical
contract. Comparative warm-up, repetition, runtime, peak-memory, and Spark
plan interpretation belong to the remaining Part 3 milestones.

Acceptance commands executed with `.venv_local` and `PYTHONPATH=src`:

```powershell
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_polars_panel.py
# 6 passed in 1.54s
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_polars_panel.py tests\test_panel_build.py tests\test_quality_audit.py tests\test_quality_treatment.py
# 63 passed in 21.64s after the final saved-manifest check
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp C:\NEU\BFT65B\tmp\pytest_p31_final
# 328 passed in 55.75s on final code
git diff --check
git ls-files data_raw data_processed runs .venv_local tmp
git check-ignore -v data_processed/task_3_1_polars/polars20261115a/observed.parquet runs/task_3_1_polars/polars20261115a/contract.json .venv_local/Scripts/python.exe
```

`git diff --check` exited 0 with existing LF-to-CRLF notices; `git ls-files`
returned no raw, processed, run, temporary, or environment file. All three
representative generated paths matched `.gitignore`. The pytest elapsed
times above are test-run observations, **not** workload benchmark results.
