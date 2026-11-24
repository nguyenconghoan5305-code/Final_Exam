# P3.3 Local PySpark Reproduction and Three-Engine Comparison

Status: P3.3 implementation and measurement passed on 2026-11-15. This
document reports the accepted three-engine run `panelbench20261115b`.
The later P3.4 cluster assessment is in `P3_SCALING_ASSESSMENT.md`.

## Runtime and frozen workload

The Task 3.0 synthetic runtime acceptance was rerun successfully before this
workload. The current host has Windows build 10.0.26200, Python 3.11.9,
Temurin Java 17.0.21, PySpark/Spark 3.5.7 with Hadoop 3.3.4 JARs, and the
previously hash-verified Hadoop 3.3.5 native files. The JVM reported its
native library loaded. Full acceptance stdout/stderr is in ignored
`tmp/spark_3_3_runtime_recheck.log`; no new binary was downloaded.

The input is exactly the 170 checksummed `treatment20261114a` Financials
partitions frozen by P3.1: 1,682,616 rows and 191,535,956 stored bytes.
Spark reads those explicit paths and projects the same six fields. Native
DataFrame expressions implement quarter arithmetic, `lag` by `CERT`, list
membership, null/absence counts, distinct keys, and quarter aggregation.
The final global sorts match the P3.1 ordering. No records are deduplicated,
imputed, synthesized, or filtered except the documented positive-gap output.

Actual settings from `plans/settings.json`: `local[4]`, effective default
parallelism 4, 16 shuffle partitions, adaptive query execution enabled,
Arrow-backed `toPandas` enabled, and Spark UI disabled. `spark.local.dir`
points to an ignored per-run local directory. Spark logged the standard
warning that a cluster manager can override that setting; this run used
local mode. `PYSPARK_PYTHON` and its driver counterpart used `.venv_local`.

## Full-data parity

Each engine generated all three relations in a preflight process and passed
the exact, full-row P3.1 comparator before timing. Every warm-up and all 15
measured outputs also passed after its timer stopped. The comparator checks
ordered Arrow schemas, every key, list/flag value, date, null, and finite
Float64 value. It uses no numeric tolerance. The Spark sink fully collects
every selected column via Arrow-backed `toPandas`, safely converts to the
frozen Arrow schema, and writes all three Parquet outputs. A bare `count()`
is never used as the terminal benchmark action.

| Relation or invariant | Polars | pandas | PySpark | Comparison |
| --- | ---: | ---: | ---: | --- |
| Observed bank-quarter rows and unique keys | 1,682,616 | 1,682,616 | 1,682,616 | Exact schema, order, keys, six fields and values |
| Resumed gap records | 33 | 33 | 33 | Exact `(CERT, prior, next, missing quarters)` records |
| Quarter-quality rows | 170 | 170 | 170 | Every field and quarter count exact |
| `P3ASSET` null / source absent / explicit null rows | 971,925 / 0 / 971,925 | Same | Same | Exact provenance counts |
| Snapshot-unmatched Financials rows | 10,996 | 10,996 | 10,996 | Exact flag membership |
| Preflight and measured parity failures | 0 | 0 | 0 | All outputs compared to immutable P3.1 reference |

The accepted input and reference checksums were verified again at the end
of the run. A failed comparator would have stopped later timing and saved
failure evidence; no real-data mismatch occurred.

## Executed physical plans

The three Spark query plans are saved separately before and after their
preflight materialization under ignored `runs/task_3_2_panel_benchmark/
panelbench20261115b/plans/`. Each before-plan had
`AdaptiveSparkPlan isFinalPlan=false`; each after-plan had `isFinalPlan=true`
and an AQE final plan with coalesced shuffle reads. These are the DataFrame
queries used by the benchmark worker, captured during an untimed preflight.

| Relation | Scan projection | Visible native operations in the final plan |
| --- | --- | --- |
| Observed | 6 fields over 170 Parquet paths | Project, range-partition exchange, global sort `(REPDTE, CERT)` |
| Gaps | `CERT`, `REPDTE` | Partition/sort by `CERT`, window `lag(REPDTE)`, calendar-quarter expression, positive-gap filter, range-partition exchange and final sort |
| Quality | `CERT`, `REPDTE`, `P3ASSET`, `SOURCE_MISSING_FIELDS`, `DQ_FLAGS` | Partial/final hash aggregation including distinct `CERT`, shuffle exchanges, final quarter sort |

The plans show projection at the Parquet scan and no partition filter: all
170 requested paths are read. The post-action plan, rather than only the
pre-action estimate, supports the description of adaptive exchanges.
These plans do not establish which operator caused the runtime difference.

## Shared benchmark protocol and results

This run repeated **all three engines** on the same P3.2 harness, rather than
mixing the prior two-engine measurements with a new Spark result. Each engine
had one untimed warm-up and five measured fresh-process runs. The 15-run
order was rotated deterministically and is stored in `environment.json`.
The parent monotonic timer includes interpreter startup, file reading,
all transformations and full materialization, writing three ZSTD-3 Parquet
outputs with the same PyArrow writer options, and process completion. Spark
also includes local session startup and shutdown. Input checksum scans,
parity checks, plan capture, and JSON reporting are outside measured runs.

Peak memory is the maximum 50-ms sampled sum of resident working-set bytes
of the full process tree, including the Spark JVM and visible Python workers.
Spark runs showed up to seven processes. Two Spark samples had a process
disappear before its RSS could be read; all five measured Spark runs had
hundreds of successful samples. Summed resident sets can count shared pages
more than once and sampling can miss short peaks. OS caches were not flushed
or controlled; these are **not cold-cache** measurements.

Speedup relative to pandas = median pandas wall time / median engine wall time.
Memory ratio relative to pandas = median engine peak RSS / median pandas peak RSS.
All individual exact nanosecond and byte values are in `records.json` and
`summary.json`; the displayed values below are rounded only for the table.

| Engine | Five measured times (s) | Median (s) | Range (s) | Median sampled peak RSS (MiB) | Peak range (MiB) | Speedup vs pandas | Memory ratio vs pandas |
| --- | --- | ---: | --- | ---: | --- | ---: | ---: |
| pandas | 18.068, 38.498, 35.999, 35.599, 34.615 | 35.599 | 18.068–38.498 | 1,097.199 | 1,095.086–1,097.523 | 1.000 | 1.000 |
| Polars | 3.012, 2.947, 2.780, 2.646, 2.990 | 2.947 | 2.646–3.012 | 1,023.773 | 996.652–1,031.930 | 12.079 | 0.933 |
| PySpark | 39.956, 63.794, 62.499, 61.789, 65.496 | 62.499 | 39.956–65.496 | 2,360.633 | 2,284.035–2,483.090 | 0.570 | 2.152 |

Polars had the lowest observed median on this host and workload; Spark had
the highest sampled process-tree memory. Pandas and Spark times varied
materially by run. Because the timed region includes startup, three scans,
shuffles or sorts, collection, writing, and Spark teardown, this experiment
cannot isolate a causal mechanism. It supports no claim that a cluster is
needed or that these ratios generalize to other hardware, cache states, or
larger workloads. That inference belongs to the remaining P3.4 milestone.

## Reproduction and evidence

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:HADOOP_HOME = (Resolve-Path .\tmp\hadoop-3.3.5).Path
$env:PATH = (Join-Path $env:HADOOP_HOME 'bin') + ';' + $env:PATH
$env:PYSPARK_PYTHON = (Resolve-Path .\.venv_local\Scripts\python.exe).Path
$env:PYSPARK_DRIVER_PYTHON = $env:PYSPARK_PYTHON
.\.venv_local\Scripts\python.exe .\tests\spark_runtime_acceptance.py *> .\tmp\spark_3_3_runtime_recheck.log
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel show --project-root C:\NEU\BFT65B --run-id panelbench20261115b
# New measurements require a fresh ignored run ID.
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel run --project-root C:\NEU\BFT65B --run-id panelbench_new_unique_id --include-spark
```

Evidence: ignored `runs/task_3_2_panel_benchmark/panelbench20261115b/`
contains `summary.json`, `parity.json`, `records.json`, `environment.json`,
three pre/post plan pairs, `plans/settings.json`, and per-process logs. The
complete derived outputs are in ignored
`data_processed/task_3_2_panel_benchmark/panelbench20261115b/`. The synthetic Spark fixture tests passed
3/3, combined focused tests 19/19, and full pytest 341/341. The 36 pytest
warnings are PySpark `distutils` deprecations; Spark stderr has only its
local-directory configuration warning and no Arrow fallback or exception.
