# P5 COST audit: development-only estimated-loss proxy

## Scope and source meaning

The frozen split is `split20261120b`: development prediction quarters 1984Q1–2012Q4, excluded gap 2013Q1–2013Q4, and test 2014Q1–2025Q2. The accepted selected predictive candidate remains `bounded_hist_gradient_boosting` from `dev20261121a`/`dev20261121b`. This milestone does not refit it, select an alert threshold, or inspect predictive test scores.

The [FDIC Failures field dictionary](https://api.fdic.gov/banks/docs/failure_properties.yaml), captured under ignored `runs/task_2_3_transaction_repair/failure_properties_20261115.yaml` (SHA-256 `7e3137fb6fc93dd49371b4d032f48fad403efd3d9a7f5cc8ca5480c00807364c`), defines `COST` as **Estimated Loss**, a numeric double in **thousands of US dollars**. It is the estimated Deposit Insurance Fund disbursement to insured depositors less expected liquidation recoveries of the receivership estate. It excludes interest on certain DIF claims. The dictionary warns that comprehensive estimates are unavailable for FDIC-insured failures before 1986 and FSLIC-insured failures in 1934–1988; historical RTC accounting and later definition changes also limit comparability. `N/A` denotes unavailable information in the source documentation. The project retains the accepted raw nullable float64 values, including zero and negative values; missing is never rewritten to zero. These are current-vintage estimates, not verified final realized losses, total social losses, or amounts known at the historical prediction date.

The separate official response was captured on **2026-11-14 22:29:05 UTC** from the query and source index recorded in ignored `runs/task_2_3_transaction_repair/source_capture.json`. Its `RESTYPE`/`RESTYPE1` evidence was matched one-to-one by source ID to all accepted Failures records. The COST audit uses the repaired event identity and transaction status, and validates COST against the immutable accepted raw Failures file. It makes no new FDIC request. The monetary series is **nominal**, with no inflation adjustment; comparing amounts across decades has a purchasing-power limitation. Resolution costs may be revised after the extraction vintage.

## Population and observed coverage

The event-level denominator is the set of distinct, verified FDIC `SOURCE_ID` values linked to positive **development** model observations in `split20261120b/positive_linkage.parquet`. The same failure can label multiple prediction quarters, but contributes once to event-level COST statistics. The target interval remains `(Q, Q + 4 calendar quarters]`; 24 development-linked events occur after 2012Q4, within development's final outcome window ending 2013-12-31. They are retained. Events appearing only in the gap (18 distinct after excluding development overlaps) or test (53 distinct) do not enter cost estimation. The accepted source-ID map, CERT, FAILDATE, transaction disposition, bank link, target interval, and raw-versus-repaired COST are checked before aggregation. The selected model's 48 OOF validation quarters form a separately reported subset; descriptive coverage over all development is **not** a fold-local fitted statistic.

| Population (unit) | Positive bank-quarter observations | Distinct verified events | Missing COST events | Zero | Positive | Negative | Invalid/special | Usable finite nonnegative events |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| All development | 9,677 | 2,462 | 16 | 54 | 2,385 | 7 | 0 | 2,439 (99.07%) |
| Selected-model OOF validation | 2,011 | 512 | 0 | 14 | 498 | 0 | 0 | 512 (100%) |

`SOURCE_MISSING_FIELDS` identified zero source-absent COST cases among the 2,462 development events; the 16 missing COST values were stored nulls. No invalid/special values remained after the accepted typed storage conversion. This is an observed property of the accepted snapshot, not a guarantee for future vintages. The seven negative values remain in the detailed audit; they are excluded only from a nonnegative representative-cost statistic, not from failure events or labels. The development usable-value median is **10,821.559 nominal USD thousands**; the mean is **63,280.700498 nominal USD thousands**; the maximum is **11,979,120.801 nominal USD thousands**. The top 1%, 5%, and 10% of positive-cost events contribute **34.83%**, **61.38%**, and **73.25%** of the observed positive COST sum, respectively. This concentration and the 1984–1985 source caveat make any single representative amount sensitive to period composition and missingness. No clipping is performed.

| Failure-date period | Distinct events | Missing | Zero | Positive | Negative | Invalid/special | Usable coverage |
|---|---:|---:|---:|---:|---:|---:|---:|
| 1984–1985 | 145 | 0 | 5 | 139 | 1 | 0 | 144/145 (99.31%) |
| 1986–1993 | 1,756 | 15 | 34 | 1,701 | 6 | 0 | 1,735/1,756 (98.80%) |
| 1994–2007 | 73 | 1 | 4 | 68 | 0 | 0 | 72/73 (98.63%) |
| 2008–2013 | 488 | 0 | 11 | 477 | 0 | 0 | 488/488 (100%) |

Apparent non-null coverage in 1984–1985 does not establish comprehensive FDIC historical estimated-loss coverage; the official warning remains controlling. Missing COST could correlate with source era or resolution type, so the usable-value median is not asserted to be an unbiased mean loss for all failures.

## Declared assumptions and decision unit

The later threshold milestone will consider **one bank-quarter alarm** as one decision. Each alarm on a negative bank-quarter incurs one assumed investigation charge, including repeated alarms on the same bank in later quarters. A missed positive bank-quarter receives a simplified loss surrogate. Because several positive bank-quarters can share one FDIC failure, summing those row-level surrogates is **not literal total FDIC loss**. An event-level avoided-loss analysis would need a separate, predeclared policy attributing one event's loss among its advance alerts; this milestone does not compute one. Assistance, ordinary mergers, and unresolved labels remain outside confirmed failure COST estimates; they are not recoded as failures.

The machine-readable `scenario.json` freezes a reference **assumed preventable fraction 0.10** and an **assumed false-alarm investigation charge equal to 0.01 × the training-event median COST**. Sensitivity grids are preventable fractions **0.05, 0.10, 0.25** and investigation-cost fractions of the training median **0.001, 0.01, 0.05**. These dimensionless choices deliberately span a broad range because this project has no observed supervision investigation cost; they are illustrative assumptions, not FDIC measurements or externally validated plausible amounts. Once multiplied by a fold-training median in nominal USD thousands, both terms use the same currency unit and price basis. The preventable fraction does not establish that supervision causally avoids the stated share of DIF loss. No threshold has been calculated or optimized.

Raw missing COST remains missing. Later simplified scenario calculations may use a representative median fitted on **unique verified events in that fold's training prediction quarters only**, including events in their four-quarter outcomes. Such a statistic is a scenario parameter, not imputation of the raw event record. Negative COST remains reported but is excluded from the nonnegative median. If a training fold has no usable costs, the scenario gate fails rather than excluding its failures. Full-development descriptive estimates and final-policy estimates must be labelled separately from fold-local estimates.

| Fold training end | Positive training observations | Distinct training events | Usable events | Fold-training median COST (nominal USD thousands) |
|---|---:|---:|---:|---:|
| 1999-12-31 | 7,644 | 1,949 | 1,926 | 7,180.8385 |
| 2003-12-31 | 7,742 | 1,971 | 1,948 | 7,139.7405 |
| 2007-12-31 | 7,799 | 1,999 | 1,976 | 7,318.3360 |

Changing a validation event's COST cannot alter its fold's fitted median; the synthetic test asserts this and also confirms that a change to a training event can alter the statistic. Current-vintage COST is outcome evidence and must never enter the seven-feature predictor matrix.

## Reproduction and evidence

Use Python 3.11 and the pinned local environment. All detailed source IDs and event records stay under ignored directories.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.cost_audit verify --project-root C:\NEU\BFT65B --run-id costaudit20261122d
.\.venv_local\Scripts\python.exe -m bft65b.modelling.cost_audit show --project-root C:\NEU\BFT65B --run-id costaudit20261122d
# Reproduction requires a fresh run ID; existing evidence is immutable.
.\.venv_local\Scripts\python.exe -m bft65b.modelling.cost_audit run --project-root C:\NEU\BFT65B --run-id cost_new_unique_id
```

Accepted run: `costaudit20261122d`. Its `summary.json`, `scenario.json`, `report.md`, `metadata.json`, and `run_manifest.json` are in ignored `runs/task_5_4_cost/costaudit20261122d/`. Detailed one-event-per-ID Parquet is in ignored `data_processed/task_5_4_cost/costaudit20261122d/development_events.parquet`. Independent repeat `costaudit20261122e` had identical measured results, input and scenario hashes, and event-Parquet SHA-256 `d7ed73a307edad0d5e58873aa8e5e4c8698f83fd433746f080b621fb11749f8a`. Earlier runs `costaudit20261122a`/`b`/`c` remain intact as preliminary development evidence; the final pair also writes the report atomically. The validator rechecks saved output schema, ZSTD codec, keys, read-back, checksums, summary metrics, scenario hash, completion evidence, and input hashes.

Focused synthetic tests after the atomic-report fix: **8 passed**. An earlier full suite had **436 passed, 1 failed** because a legacy pilot fixture hit transient Windows `WinError 5` during `os.replace` of a temporary manifest; its isolated retry passed. A subsequent fresh suite passed **437 tests**. The final sequential full suite after the report-write fix, with the verified Spark environment, passed **438 tests, zero skips, 36 PySpark deprecation warnings** in 204.27 seconds. The transient failure did not involve the COST code or accepted data. Both final COST runs passed saved-artifact verification and upstream input hashes after execution.

`git diff --check` passed for working-tree changes; `git ls-files` returned no raw, processed, run, temporary, or virtual-environment artifacts, and representative paths matched `.gitignore`. A separately checked **pre-existing staged diff** still reports trailing blank-line notices in `PROJECT_STATUS.md` and `TASKS.md`; this milestone did not restage or rewrite that historical index state. Existing LF-to-CRLF conversion notices are informational.

Final report placement and the distinct threshold, full-development refit, and one-time test gates remain pending.
