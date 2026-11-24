# Part 3 Scaling Assessment and Assignment Audit

Status: all four Part 3 implementation milestones passed on 2026-11-15.
The report-ready material below has **not yet been placed in the final
submission report**. It answers Q1 only for the measured observed-panel
workload, not for a future feature or modelling pipeline.

## Evidence gate and comparability

The authoritative assignment PDF, page 2, requires efficient Polars core
transformations with an engine explanation, a fair pandas comparison,
matching PySpark results with physical-plan interpretation, and a measured
argument for when a cluster would be justified. Its page 3 processing rubric
requires all four elements; Q1 appears on page 1.

Read-only review `part3review20261115a` revalidated the accepted P3.1
contract and manifest, all 170 exact `treatment20261114a` source paths and
hashes, the three Polars optimized plans, both benchmark summaries and
environments, all process records, and all six Spark pre/post plans. It
independently compared **35 complete saved output sets** against the immutable
P3.1 reference (14 from the two-engine run and 21 from the three-engine run),
including every row and field in the observed, gap, and quality relations.
It hashed **300 accepted source/evidence/output artifacts** before and after
the review and found zero changes. The review result is in ignored
`runs/task_3_4_review/part3review20261115a/review.json`.

Both benchmark runs used frozen workload `observed_panel_quality_gaps_v1`:
170 Financials quarters, 1,682,616 source rows, 191,535,956 stored input
bytes, identical six projected fields, keys, null/absence semantics and
output schemas. Every preflight, warm-up, and measured output passed exact
parity without numeric tolerance. The earlier two-engine run is retained as
historical P3.2 evidence. **All three numbers in the comparison below come
from the same later run `panelbench20261115b`**, which repeated pandas and
Polars when Spark was added. Each engine performed one warm-up and five
measured fresh-process runs in a reproducibly rotated order. The same parent
monotonic timer spans process startup, complete input reading and
transformations, full Arrow materialization, three ZSTD Parquet writes, and
process exit. Spark session shutdown is part of that end-to-end boundary.
Input hash scans, parity comparisons, plan capture and reporting are outside
the measured interval. `environment.json` records OS, CPU, RAM, package
versions, input size, thread settings, order and cache policy.

## Report-ready comparison for Part 3

Peak memory is the **sampled sum of process-tree resident working sets**, in
MiB, at a 50-ms interval. It includes Spark's JVM and visible local workers.
Speedup relative to pandas = median pandas wall time / median engine wall time.
Memory ratio relative to pandas = median engine peak RSS / median pandas peak RSS.
Displayed figures are rounded from the exact nanosecond and byte records.
The saved `panelbench20261115b` unrounded medians verify the Polars speedup
as 35,599,391,700 / 2,947,318,000 = 12.0785716709 times; this documentation
check did not rerun the benchmark.

| Engine | Input scope | Measured runs | Median wall time (s) | Time range (s) | Median sampled peak RSS (MiB) | Peak RSS range (MiB) | Full-data parity |
| --- | --- | ---: | ---: | --- | ---: | --- | --- |
| pandas | 170 quarters; 1,682,616 rows; six projected fields | 5 | 35.599 | 18.068-38.498 | 1,097.199 | 1,095.086-1,097.523 | Exact, three relations |
| Polars | Same accepted inputs and output contract | 5 | 2.947 | 2.646-3.012 | 1,023.773 | 996.652-1,031.930 | Exact, three relations |
| local PySpark | Same accepted inputs and output contract | 5 | 62.499 | 39.956-65.496 | 2,360.633 | 2,284.035-2,483.090 | Exact, three relations |

For this six-column observed-panel workload, the ratio of pandas to Polars
median end-to-end wall time was 12.08× in the three-engine run (35.599 s /
2.947 s). The pandas-to-Spark median wall-time ratio was 0.570× (35.599 s /
62.499 s). Spark's median sampled process-tree RSS
was 2.152 times pandas's median. The five individual results for each engine
are retained in `panelbench20261115b/records.json`; pandas and Spark showed
material within-run timing variation, so the table does not imply stable
per-operator speed ratios.

The Polars optimized plans show Parquet scan projections of 6, 2 and 5 out
of 29 physical columns for observed, gaps and quality, followed by sorts,
the bank `shift` window and quarter aggregation as appropriate. Spark's
**executed** final plans show the same projected scan fields, global sorts,
an exchange and window for gaps, and hash aggregations plus exchanges for
quality. Adaptive Spark plans changed from `isFinalPlan=false` before action
to `true` after materialization. These operators identify work the engines
performed. The benchmark did not isolate their separate costs, so it does
not assign a causal share of the observed runtime to any one operator.

## Q1: laptop capacity and the cluster decision

**Measured:** This specific 170-quarter observed-panel workload completed
with exact outputs on a laptop with 16,780,767,232 bytes of physical RAM.
The largest sampled process-tree RSS across the 15 measured runs was Spark's
2,603,708,416 bytes, or **15.5% of physical RAM**. The 191,535,956-byte input
was small relative to the 249,920,110,592 bytes of free disk recorded before
the three-engine run. Among the capacity quantities measured, memory reached
the larger fraction of its known capacity; even that observed fraction did
not approach exhaustion. CPU utilization, spill volume and throughput under
concurrent workloads were not measured. No run failed for memory or storage.

**Conclusion for the tested workload:** A Spark cluster is **not justified
by the available performance or capacity evidence**. Local Polars completed
the task with the lowest observed median time and no measured resource limit.
This says nothing definitive about larger data, concurrent users or a
different transformation pipeline. The local Spark measurement does not
demonstrate distributed-cluster speedup, cost savings or a numerical
break-even point; no crossover or capacity limit was measured.

**Conditional next evidence:** Consider a cluster only after a specified
larger or concurrent workload breaches a documented laptop memory/storage
limit, incurs measured spill or unacceptable completion time, or misses a
defined service-level target despite local optimization. Run the same parity
gate and end-to-end benchmark at increasing scales, measure CPU, spill,
shuffle/network traffic, memory and total cost, then compare a real cluster
with the local baseline. Spark's visible exchanges and window/aggregation
stages identify places where distributed execution could matter at greater
scale; they are not evidence of a crossover at the current scale.

## Part 3 requirement-by-requirement audit

| ID | Implementation artifact | Executable test or review gate | Report-ready evidence | Implementation status |
| --- | --- | --- | --- | --- |
| P3.1 | `src/bft65b/processing/polars_panel.py`; frozen contract and optimized plans; `P3_WORKLOAD.md` | Polars fixture tests, accepted P2.2/P2.1 row and quarter parity, review of plan checksums | Workload/plan table and 1,682,616/33/170 output reconciliation | PASS |
| P3.2 | `src/bft65b/processing/pandas_panel.py`, shared `benchmarks/panel.py`, process-tree sampler; `P3_BENCHMARK.md` | Pandas/harness tests; exact full-output comparator; five fresh processes per engine; saved-run review | Same-run pandas/Polars wall-time and memory rows above | PASS |
| P3.3 | `src/bft65b/processing/spark_panel.py`; captured before/final plans; `P3_SPARK.md` | Task 3.0 runtime gate, Spark fixtures, 35-set review with full Spark parity, final-plan checks | Spark parity, runtime/memory row and plan-operator interpretation | PASS |
| P3.4 | This assessment and `src/bft65b/benchmarks/part3_review.py` | Read-only `verify` gate: input/output hashes, all saved values, plans, run counts, formulas and environment | Capacity context, measured-limit statement and conditional cluster criteria | PASS for implementation; final report placement pending |

The final report must still include the selected tables and Q1 discussion.
This Part 3 acceptance does not resolve the separate Part 2 active-bank
population limitation and does not authorize Part 4 model work.

## Limitations and reproducibility

- The benchmark projects only the six frozen fields for this observed-panel
  workload; it does not measure a complete feature/model pipeline.
- OS caches were not controlled, so no cold-cache claim is made. The time
  includes startup, output writes and Spark shutdown, and does not isolate
  compute-only transformations.
- Process-tree RSS is sampled, can miss short peaks and can double-count
  shared pages. Two transient Spark process reads were unavailable, though
  all five measured Spark runs had hundreds of valid samples and up to seven
  visible processes.
- Local Spark uses a JVM and Arrow collection to the Python driver; this
  local demonstration cannot predict a distributed cluster's network,
  scheduling, performance or costs.
- The accepted Windows native Hadoop setup is host-specific. The earlier
  `NativeIO$Windows.access0` failure was not reproduced in the later passing
  Task 3.0 and P3.3 runs, but its historical cause remains unconfirmed.

From `C:\NEU\BFT65B` in PowerShell:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.part3_review show --project-root C:\NEU\BFT65B --run-id part3review20261115a
# Revalidate all accepted data, plans and results under a new ignored ID.
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.part3_review verify --project-root C:\NEU\BFT65B --run-id part3review_new_unique_id
```

`show` reads only the saved review. `verify` rehashes accepted inputs and
evidence and compares all 35 complete output sets, without rerunning a
benchmark or contacting FDIC. The review result and hash inventory remain
outside Git under `runs/task_3_4_review/`.

Final focused Part 3 tests passed **23/23**. The complete pytest regression
suite passed **345/345**, with 36 PySpark `distutils` deprecation warnings.
No accepted input or benchmark output was rewritten by this review.
