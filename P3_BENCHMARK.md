# P3.2 Pandas and Polars Observed-Panel Benchmark

Status: implementation and measurement passed on 2026-11-15. This is the
historical second Part 3 milestone. The later three-engine comparison is in
`P3_SPARK.md`, and the final Part 3 scaling assessment is in
`P3_SCALING_ASSESSMENT.md`. All measurements below come from ignored run
`panelbench20261115a`.

## Frozen workload and exact parity

Both engines read the exact 170 checksummed Financials partitions from
`treatment20261114a` listed in the accepted P3.1 `contract.json` (191,535,956
stored bytes, 1,682,616 rows). The six projected fields, three output schemas,
ordering, calendar-quarter gap rule, null/absent split, and snapshot flag rule
are specified in `P3_WORKLOAD.md`. No decimal field appears in this frozen
workload; its `P3ASSET` value is binary Float64 and received no conversion or
tolerance. The pandas baseline reads projected fields via PyArrow, sorts by
the same keys, computes consecutive-bank gaps and per-quarter quality counts,
and converts to the exact frozen Arrow schemas with safe conversion. No row is
deduplicated or imputed.

Before timing, each engine independently produced all three complete outputs
and passed the exact P3.1 comparator: 1,682,616 observed rows, 33 gap events,
and 170 quarter-quality rows, with identical schemas, order, keys, lists,
nulls, and every value. All warm-ups and measured runs were also compared
after their timers stopped. No floating-point tolerance was used. A mismatch
reports the first relation, row, field, and values, and stops later timing.

## Measurement contract

- One fresh-process, untimed warm-up per engine preceded measurement. Five
  fresh-process measured runs per engine followed in alternating rounds:
  pandas/Polars, Polars/pandas, pandas/Polars, Polars/pandas, pandas/Polars.
  The order uses no randomness.
- Parent `time.perf_counter_ns()` spans subprocess startup, package imports,
  reading the 170 files, all transformations, full materialization, and
  writing all three ZSTD-3 Parquet outputs with identical PyArrow options.
  Input checksum scans, parity comparisons, JSON reporting, and source
  inventory checks are outside this interval. These are startup-inclusive
  end-to-end times, not isolated transformation times.
- Peak memory is the largest sampled sum of resident working-set bytes for
  the worker process tree, sampled every 50 ms by the parent. It includes
  visible descendants but can miss short-lived peaks. Individual sample
  counts and unreadable-process counts are in `records.json`.
- The worker sets PyArrow CPU count to 4 and sets `POLARS_MAX_THREADS`,
  `OMP_NUM_THREADS`, `MKL_NUM_THREADS`, `OPENBLAS_NUM_THREADS`, and
  `NUMEXPR_NUM_THREADS` to 4. These are configuration settings, not measured
  CPU utilization. Identical output writer settings are ZSTD level 3,
  65,536-row groups, dictionary encoding on, statistics on, Parquet 2.6.
- The OS file cache was not flushed or controlled. Results represent repeated
  reads under an uncontrolled warm-cache policy; they are not cold-cache
  measurements. The machine was Windows build 10.0.26200, Intel Core
  i5-1340P, 16 logical processors, 16,780,767,232 bytes RAM, Python 3.11.9,
  pandas 2.2.3, Polars 1.35.2, and PyArrow 18.1.0. The evidence records the
  full environment, input bytes, remaining disk, and executable path.

## Report-ready comparison

Both engines read 170 files and write the same three relations. Speedup
relative to pandas = median pandas wall time / median engine wall time.
Memory ratio relative to pandas = median engine peak RSS / median pandas peak RSS.
Five durations and five memory peaks per engine are retained individually in
`records.json`.

| Engine | Measured times (s; all 5) | Median time (s) | Time range (s) | Sampled peak RSS (MiB; all 5) | Median peak RSS (MiB) | Peak range (MiB) | Speedup vs pandas | Memory ratio vs pandas |
| --- | --- | ---: | --- | --- | ---: | --- | ---: | ---: |
| pandas | 18.978080, 19.146766, 18.042065, 18.810940, 19.058042 | 18.978080 | 18.042065–19.146766 | 1095.043, 1089.688, 1078.102, 1095.766, 1096.859 | 1095.043 | 1078.102–1096.859 | 1.000 | 1.000 |
| Polars | 2.786324, 2.833062, 2.821770, 2.847512, 2.793517 | 2.821770 | 2.786324–2.847512 | 995.570, 1002.000, 1013.902, 1007.668, 996.355 | 1002.000 | 995.570–1013.902 | 6.726 | 0.915 |

On this machine and workload, the Polars end-to-end median was lower than
the pandas median, and its sampled peak RSS was also lower. The workload
includes scan projection, sorting, window/gap work, aggregation, and output
writing, so the measurement cannot isolate a single cause. Cache state and
hardware limit generalization. PySpark remains unmeasured at this gate.

## Evidence and reproduction

Ignored evidence is under
`runs/task_3_2_panel_benchmark/panelbench20261115a/`: `summary.json`,
`parity.json`, `records.json`, `environment.json`, and per-process stdout/
stderr logs. Generated outputs are under
`data_processed/task_3_2_panel_benchmark/panelbench20261115a/`. The accepted
input and P3.1 reference hashes were checked before and after the run. The
benchmark namespace is ignored by Git. Repeating requires a fresh run ID.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel show --project-root C:\NEU\BFT65B --run-id panelbench20261115a
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel run --project-root C:\NEU\BFT65B --run-id panelbench_new_unique_id
```

Final focused verification passed 15/15 tests and the full regression suite
passed 337/337 after adding first-mismatch diagnostics. Git tracked no raw,
processed, run, temporary, or environment files; `git check-ignore` matched
all three generated namespaces. `git diff --check` exited 0 with existing
LF-to-CRLF notices for several tracked project files.
