# P5 development OOF alarm-threshold policy

## Authority, scope and input lock

The assignment PDF (page 1, Section 2.3; page 2, Part 5) requires development-only threshold choice from false-alarm and missed-failure costs, followed by one test evaluation. This milestone uses the previously selected `bounded_hist_gradient_boosting` candidate and the immutable split `split20261120b`. It does not refit the model, open test predictors, generate test scores, or optimize on test outcomes. The globally inspected test failure counts from the earlier split feasibility audit are separate from this threshold exercise.

The accepted inputs are `dev20261121a` and its independent `dev20261121b` OOF repeat, plus `costaudit20261122d` and `costaudit20261122e`. The OOF file SHA-256 is `76bc87c4a2efcb5489cb1f95644eb644eeda936943f1acf83b655b15744e2320`. The accepted scenario, fold boundaries, selected configuration, OOF and source hashes, decision unit, candidate construction, objective, and tie rule were saved to `threshold20261122c/frozen_plan.json` **before** threshold evaluation (SHA-256 `ba386f9784daeb12fb7e85ae5ba11776c69c8a178eca7e763ded0bab4f4b0f10`). The independent `threshold20261122d` plan has the same hash. No predictor partitions or COST values belonging only to the gap or test were used.

The selected OOF contains **409,094 unique validation bank-quarters**, **2,011 positive rows**, **407,083 negative rows**, and **512 distinct represented verified FDIC failure events**. The first **920,357** development model rows have no OOF score and remain outside threshold selection. Their in-sample predictions were not manufactured. The accepted positive source-ID linkage, `(CERT,Q,Y)` split assignments, fold IDs, row counts, and selected candidate scores all reconcile. The three training-only COST medians are **7,180.8385**, **7,139.7405**, and **7,318.3360 nominal USD thousands**. A separate full-development representative median, **10,821.559 nominal USD thousands**, is recorded for later fixed-policy reporting; it does not reoptimize this threshold.

## Cost scaling and frozen objective

For a fold with positive median (M_f), assumed preventable fraction (α), and assumed false-alarm charge fraction (β), its bank-quarter surrogate loss is

\[
L_f(t)=M_f\{\alpha FN_f(t)+\beta FP_f(t)\},\qquad M_f>0.
\]

Multiplying every candidate threshold's loss in one fold by the same positive (M_f) cannot change that fold's argmin. The synthetic test checks this. For one global threshold across folds, however, the frozen reference objective is

\[
L(t)=\frac{\sum_f M_f\{\alpha FN_f(t)+\beta FP_f(t)\}}{\sum_f N_f}.
\]

Different medians therefore weight folds differently. A synthetic two-fold fixture demonstrates that the optimum can change when the medians differ. The separately reported diagnostic \((\alpha\sum FN+\beta\sum FP)/\sum N\) removes this scale and **does not choose the lock**. On the accepted OOF data, its minimum happens to select the same threshold `0.03210050445929286`; this measured agreement is not a general identity. The positive-median gate rejects null, nonfinite, zero, or negative values.

This transparent, data-scaled surrogate uses measured FDIC Estimated Loss as an event-level scale and explicitly assumed fractions. It satisfies the PDF's request to derive an alarm tradeoff from data-informed costs while disclosing what the data cannot establish: a supervisory intervention's causal preventable share and the actual cost of investigating false alarms. COST values are current-vintage estimates in nominal thousands of US dollars, not inflation-adjusted welfare losses. Repeated negative bank-quarter alarms each incur the assumed investigation charge. Repeated positive quarters for one source event incur repeated row-level surrogate misses; their sum is **not** literal total FDIC loss.

## Complete empirical curve and selected reference policy

The frozen candidate set contains one `SCORE >= threshold` policy at **every one of the 32,386 distinct OOF scores**, with tied scores entering together, plus explicit `always_alarm` and `never_alarm` policies. Thus **32,388 policies** are evaluated. No coarse grid or probability-calibration assumption substitutes for this empirical scan. The reference scenario is α=`0.10`, β=`0.01`. Selection orders policies by exact float64 objective, then fewer false positives, then higher threshold; `never_alarm` ranks above finite values and `always_alarm` below them for the last tie. Precision is null for a policy with no alarms; recall is null if there are no positives, and false-positive rate is null if there are no negatives.

**Locked policy:** `score_ge`, **threshold `0.03210050445929286`**, equality alarms. The locked policy artifact is `threshold20261122c/locked_policy.json` (SHA-256 `06728e3e789e526777c57f2a581f85f64f8e499f6280c9b96333d360a57a45f5`).

| Scope | N | TP | FP | TN | FN | Alarms | Alarm rate | Precision | Recall | Reference surrogate cost/row (nominal USD thousands) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Fold 1 | 149,382 | 16 | 482 | 148,823 | 61 | 498 | 0.3334% | 0.0321 | 0.2078 | 0.524927745 |
| Fold 2 | 137,419 | 59 | 278 | 136,728 | 354 | 337 | 0.2452% | 0.1751 | 0.1429 | 1.983679784 |
| Fold 3 | 122,293 | 960 | 2,695 | 118,077 | 561 | 3,655 | 2.9887% | 0.2627 | 0.6312 | 4.969931270 |
| Overall | 409,094 | 1,035 | 3,455 | 403,628 | 976 | 4,490 | 1.0975% | 0.2305 | 0.5147 | 2.343710378 |

The overall normalized diagnostic objective at this policy is **0.0003230308926554777 per bank-quarter**. Its overall false-alarm and missed-positive surrogate components are **251,689.27536** and **707,108.57815 nominal USD thousands**, respectively, before division by 409,094 rows. They are algebraic surrogate components, not observed total FDIC losses.

## Two-dimensional sensitivity, frozen reference unchanged

The grid was inherited unchanged from `costaudit20261122d/scenario.json`: α in `{0.05, 0.10, 0.25}` and β in `{0.001, 0.01, 0.05}`. Each row reports its own optimum **and** the already locked reference policy under that scenario. Always/never costs and all confusion metrics are in the machine-readable table. Values below are nominal USD thousands per bank-quarter surrogate.

| α | β | Scenario-optimal policy | Optimal threshold | Optimal alarms | Optimal cost/row | Locked-reference cost/row |
|---:|---:|---|---:|---:|---:|---:|
| 0.05 | 0.001 | score_ge | 0.011926682170693396 | 12,684 | 0.750753107 | 0.925760868 |
| 0.05 | 0.01 | score_ge | 0.056704059968158 | 2,562 | 1.388876151 | 1.479473090 |
| 0.05 | 0.05 | never_alarm | boundary | 0 | 1.788443053 | 3.940416300 |
| 0.10 | 0.001 | score_ge | 0.008226758984362955 | 21,045 | 1.185214409 | 1.789998156 |
| **0.10** | **0.01** | **score_ge** | **0.03210050445929286** | **4,490** | **2.343710378** | **2.343710378** |
| 0.10 | 0.05 | score_ge | 0.2436650232349658 | 596 | 3.414507333 | 4.804653588 |
| 0.25 | 0.001 | score_ge | 0.004044059744634767 | 52,492 | 2.204565547 | 4.382710020 |
| 0.25 | 0.01 | score_ge | 0.01590816551079814 | 8,911 | 4.619452134 | 4.936422242 |
| 0.25 | 0.05 | score_ge | 0.056704059968158 | 2,562 | 6.944380757 | 7.397365452 |

The two grid cells with α/β=`5` select the same decision set. Their absolute objective levels differ because multiplying both fractions scales every surrogate loss. At α=`0.05`, β=`0.05`, `never_alarm` is the empirical optimum; this is reported without forcing a nontrivial threshold. No sensitivity cell changes the reference lock.

The ignored run includes report-ready visual evidence: `reference_curve.svg` and `sensitivity.svg`. The curve plot samples points only **for display**; selection and Parquet evidence contain every candidate policy. The four ZSTD Parquet curves record overall and each fold's confusion, alarm rate, precision/recall/FPR, cost components, reference objective, and normalized diagnostic at every policy. `verify` recomputes the optimum and sensitivity from saved per-fold counts and checks arithmetic, schema, keys, codec and hashes.

## Execution, recovery and limitations

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold verify --project-root C:\NEU\BFT65B --run-id threshold20261122c
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold show --project-root C:\NEU\BFT65B --run-id threshold20261122c
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold compare --project-root C:\NEU\BFT65B --run-id threshold20261122c --compare-run-id threshold20261122d
# A new reproduction requires a fresh ignored run ID and two explicit steps.
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold plan --project-root C:\NEU\BFT65B --run-id threshold_new_unique_id
# Review frozen_plan.json before this command:
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold execute --project-root C:\NEU\BFT65B --run-id threshold_new_unique_id
```

Accepted `threshold20261122c` and independent repeat `threshold20261122d` have matching frozen-plan hashes, locked threshold, confusion counts, objectives, all nine sensitivity results, and SHA-256 for all four Parquet curves (comparison tolerance `1e-10` for floating metrics). Both runs and detailed records are ignored by Git. An earlier plan `threshold20261122a` remains **failed evidence**: its initial NumPy `int8` accumulation overflowed a positive count to `-37`, so execution was stopped; a new checked count and synthetic 300-positive regression repaired this before the accepted plan. `threshold20261122b` is an earlier complete run without the additional normalized-optimum report field; it remains unchanged. No accepted upstream run was rewritten.

Development OOF scores served both candidate selection and threshold selection, so the displayed metrics are selection evidence rather than an unbiased final performance estimate. Any later test workflow must consume this immutable lock, freeze its remaining full-development fit and application procedure before opening test predictors, and execute one predictive test evaluation. Final report placement remains pending.

## Acceptance closeout (2026-11-22)

The current-code `verify` command passed for both `threshold20261122c` and `threshold20261122d`; `compare` reported matching frozen plans, locked policy, results, and all four Parquet curve hashes with a `1e-10` metric tolerance. The accepted lock remains `SCORE >= 0.03210050445929286`, and `locked_policy.json` has SHA-256 `06728e3e789e526777c57f2a581f85f64f8e499f6280c9b96333d360a57a45f5`. These commands recheck the saved outputs and upstream hashes without rerunning threshold selection.

With the documented local Spark environment configured, `tests/test_threshold.py` passed **14 tests in 10.78 s** (exit 0; eight ordinary tests and six parameterized invalid-median cases). The sequential full suite passed **452 tests, zero skips, 36 PySpark deprecation warnings in 567.51 s** (exit 0). The previous full-suite attempt ended with one Spark Python-worker connection timeout after 451 passes; its isolated retry passed **1 test in 276.54 s** before the fresh full suite passed. The timeout and retry logs are retained as diagnostic evidence, not counted as the final suite result.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:HADOOP_HOME = (Resolve-Path .\tmp\hadoop-3.3.5).Path
$env:PATH = (Join-Path $env:HADOOP_HOME 'bin') + ';' + $env:PATH
$env:PYSPARK_PYTHON = (Resolve-Path .\.venv_local\Scripts\python.exe).Path
$env:PYSPARK_DRIVER_PYTHON = $env:PYSPARK_PYTHON
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold verify --project-root C:\NEU\BFT65B --run-id threshold20261122c
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold verify --project-root C:\NEU\BFT65B --run-id threshold20261122d
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold compare --project-root C:\NEU\BFT65B --run-id threshold20261122c --compare-run-id threshold20261122d
.\.venv_local\Scripts\python.exe -m pytest tests/test_threshold.py -q -rs -p no:cacheprovider --basetemp C:\NEU\BFT65B\tmp\pytest_p55_focus_closeout --junitxml tmp\task_5_5_closeout\focused.xml
.\.venv_local\Scripts\python.exe -m pytest -q -rs -p no:cacheprovider --basetemp C:\NEU\BFT65B\tmp\pytest_p55_full_closeout --junitxml tmp\task_5_5_closeout\full.xml
```

The ignored `tmp/task_5_5_closeout/` directory contains the exact command, UTC start/end timestamps, exit codes, stdout/stderr logs and JUnit XML for the focused and full checks, plus both validator and comparison logs. After the documentation and traceability update, `tests/test_evidence.py` passed **1 test in 0.23 s**. Both saved-run validators were run again after the full suite and exited 0; their `post_full_verify_*.log` files confirm accepted outputs and upstream hashes remained valid. No full-development model refit or predictive test evaluation occurred during this closeout.
