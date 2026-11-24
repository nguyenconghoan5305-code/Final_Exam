# P5 locked final predictive test evaluation

## Authority and frozen protocol

The assignment PDF (page 1, Section 2.3; page 2, Part 5) requires all tuning, feature selection and threshold choice on development only, followed by one test use. The accepted split is development 1984Q1–2012Q4, excluded gap 2013Q1–2013Q4, and test 2014Q1–2025Q2. The last development target window ends 2013-12-31, before the first test prediction quarter. Global test outcome counts were inspected earlier solely to establish split feasibility; test predictors and predictive scores had not been used for model or threshold choice.

`final20261122a/frozen_plan.json` was committed before opening test predictors (SHA-256 `a2fdb045d85cb361084d8d059d24db0e51749eed23502ab0fad08edb32099ff2`). It records 171 accepted source hashes, exact split, seven predictor names/order, `numeric_median_scale_v1` preprocessing, two-thread limit, seed 42, the selected `HistGradientBoostingClassifier` parameters from the accepted `threshold20261122c` lock, and `SCORE >= 0.03210050445929286`. The lock SHA-256 is `06728e3e789e526777c57f2a581f85f64f8e499f6280c9b96333d360a57a45f5`. The fixed reference scenario is alpha 0.10, beta 0.01, and the separately estimated full-development FDIC COST scale is 10,821.559 nominal thousands of US dollars. The lock and scenario were not altered after test scores were observed.

The fresh complete pipeline fitted median imputation, standard scaling and bounded histogram gradient boosting on the 1,329,451 development rows only (9,677 positive). The 2013 gap and 2014–2025 test rows were excluded from fitting. The fitted pipeline was serialized and reloaded, with identical development probe scores, before test access. Its SHA-256 is `ccf80d8513f2a21705ec7903bfbe8b4f8794516dae4b5feabbb98f88ff325004` and is bound to the frozen plan in `model_binding.json`.

The fixed registry `runs/task_5_6_final/test_use_registry.json` is keyed to the lock and split hashes, not the run ID alone. A Windows process lock protects every transition. It recorded `planned`, `fitting`, `model_fitted`, **one** `test_access` transition at 2026-11-22 14:02:12 UTC, `scores_persisted`, and `complete`. Test access was recorded before opening predictors. The ZSTD score artifact was atomically promoted and bound to the model/plan before metrics were computed. A crash after score persistence can resume reporting from saved scores. If test access is recorded without a durable score artifact, automatic rescore and a new run ID are refused pending investigation. Verification reads saved scores and upstream hashes; it does not refit or rescore.

## Fixed result

The predeclared metrics are scikit-learn stepwise `average_precision_score`, ROC-AUC, Brier score, confusion counts, precision, recall, specificity/FPR, alarm rate, prevalence, and distinct represented/detected source events. A represented failure counts as detected if at least one of its eligible positive test bank-quarters alarms. No confidence interval was predeclared; repeated bank-quarters from the same bank or event are dependent.

| Measure | Locked test result |
|---|---:|
| Test bank-quarters / positive / negative | 246,083 / 178 / 245,905 |
| Distinct represented / detected / missed failure events | 53 / 38 / 15 |
| Prevalence | 0.072333% |
| Average precision | 0.172989033 |
| ROC-AUC | 0.876991628 |
| Brier score | 0.000659085 |
| TP / FP / TN / FN | 109 / 1,483 / 244,422 / 69 |
| Alarms / alarm rate | 1,592 / 0.646936% |
| Precision / recall | 0.068467 / 0.612360 |
| Specificity / false-positive rate | 0.993969 / 0.006031 |
| Locked reference surrogate per bank-quarter, nominal USD thousands | 0.955581967 |
| Fixed always-alarm / never-alarm surrogates per bank-quarter | 108.137314073 / 0.782759273 |

The reference surrogate comprises 160,483.71997 nominal USD thousands from 1,483 false-alarm bank-quarters and 74,668.75710 from 69 missed-positive bank-quarters, divided by 246,083 observations. These are **assumed row-level loss surrogates**, not literal total FDIC event losses or intervention benefits. The fixed never-alarm diagnostic has a lower test surrogate than the locked policy. This is an observed limitation of the development-selected policy under the declared assumptions; no test threshold scan or policy adjustment was made. The scores are uncalibrated. The 46-quarter test horizon can contain distribution shift, and only 53 distinct represented failures are available. Development OOF predictions served both model and threshold selection, so their earlier metrics are selection evidence rather than independent performance estimates.

## Execution and read-only verification

The focused synthetic gate passed **7 tests in 10.04 s**. The full suite, with the documented Spark runtime configured and a short Windows temporary path, passed **459 tests, zero skips, 36 PySpark deprecation warnings in 345.35 s** before any real predictive test use. The evaluation command exited 0. Fitting including accepted development reads took **28.712465 s** inside the worker; test read, inference and score persistence took **15.990086 s**. The outer fit and evaluation commands took approximately 34.23 s and 94.03 s, respectively, including process startup, saved-result verification and source hashing. These boundaries are not a model-speed benchmark. The recorded runtime was `Windows-10-10.0.26200-SP0` as reported by Python, Python 3.11.9, scikit-learn 1.6.1, pandas 2.2.3, NumPy 2.1.3 and PyArrow 18.1.0; the run manifest records the full dependency inventory.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
# The following commands were executed once for final20261122a; do not rerun them for an accepted test evaluation.
.\.venv_local\Scripts\python.exe -m bft65b.modelling.final_evaluation plan --project-root C:\NEU\BFT65B --run-id final20261122a
.\.venv_local\Scripts\python.exe -m bft65b.modelling.final_evaluation fit --project-root C:\NEU\BFT65B --run-id final20261122a
.\.venv_local\Scripts\python.exe -m bft65b.modelling.final_evaluation evaluate --project-root C:\NEU\BFT65B --run-id final20261122a
# Safe read-only verification after completion:
.\.venv_local\Scripts\python.exe -m bft65b.modelling.final_evaluation verify --project-root C:\NEU\BFT65B --run-id final20261122a
```

The accepted run manifest, frozen plan, model binding, score binding, summary, metadata, English `report.md`, and `confusion.svg` are under ignored `runs/task_5_6_final/final20261122a/`. The fitted joblib pipeline and full key-level ZSTD test scores are under ignored `data_processed/task_5_6_final/final20261122a/`. The score SHA-256 is `ab3b125140057916a2e6e36d3e89776af530feefccb8049f2e3fbacccff4bb4b`. The read-only `verify` command recomputes all metrics from that saved score artifact, reconciles every test key/label and alarm, checks codec and all 171 source hashes, and exited 0. A separate read-only calculation, `python tmp/p56_independent_verify.py`, matched saved AP, ROC-AUC, Brier, confusion and reference cost exactly within `1e-12`; its JSON output is `tmp/p56_independent_verify.json`. Ignored `tmp/p56_focus.xml`, `tmp/p56_full_pretest.xml`, `tmp/p56_full_pretest.log`, `tmp/p56_fit.log`, `tmp/p56_evaluate.log`, and their execution records retain the actual commands and results.

This completes the Part 5 implementation gate. Placement in the final English report, Part 6 conclusions, and a clean-environment end-to-end reproduction remain separate submission work.

## Post-evaluation integrity-verifier repair (2026-11-23)

This section records a validator repair **after** the original one-use test evaluation. No production plan, model fit, test score, policy, metric, manifest or registry was regenerated or rewritten. Python evaluates `a != b != c` as `(a != b) and (b != c)`. The former `verify_saved()` condition therefore missed a changed model file when the summary and binding still agreed. An isolated synthetic model-byte mutation first made the new regression fail as expected: `tmp/p56_integrity_red.xml` records **1 failed** because the old verifier did not raise. After replacing the chain with explicit OR mismatch checks, the identical regression passed (**1 passed**, `tmp/p56_integrity_green.xml`).

The bounded verification change also checks the frozen plan against model and score bindings, the project-level registry and manifest; checks recorded model and score hashes separately against file bytes; and reconciles manifest output paths, hashes, row counts and source-hash inventory. Synthetic tests mutate each reference independently while preserving unrelated checksums where needed. A consistent synthetic run remains a passing control. This change affects acceptance verification only, not fitting, inference, threshold choice or metric calculations.

The first focused repair suite passed **9 tests** (`tmp/p56_integrity_focus.xml`). The first full regression attempt remains recorded as **457 passed, 4 failed, zero skipped**, with 36 PySpark deprecation warnings (`tmp/p56_integrity_full.log` and `.xml`). Three failures were `PermissionError: [WinError 5] Access is denied` at `RunManifest.write()` while replacing a synthetic temporary `pilot.json`; the fourth was a Spark Python-worker connect-back timeout (`java.net.SocketTimeoutException`, then worker `WinError 10061`). Directory ACL inspection showed write permission and no relevant Python/Java process remained after the failed suite. The existing Spark fixtures use module-scoped sessions and call `session.stop()` at teardown. The installed Python 3.11.9, JDK 17.0.21, PySpark 3.5.7 and previously verified native-file hashes were unchanged. These observations do **not** establish a definitive root cause.

The three pilot cases passed **3/3** on a fresh short temporary path (`tmp/p56_pilot_retry1.*`); the Spark case passed **1/1** in 102.45 s (`tmp/p56_spark_retry1.*`); all four passed together **4/4** in a new pytest process (`tmp/p56_combined_probe.*`). These retries show non-reproduction in isolation and together, not a proven cause. No broad retry, security change, Spark change or unrelated acquisition-code change was made. The final focused suite passed **9/9 in 8.45 s** (`tmp/p56_closeout_focus.*`). The final sequential full suite with Spark configured passed **461 tests, zero skipped, 36 PySpark deprecation warnings in 240.07 s** (`tmp/p56_closeout_full.*`). Logs, JUnit XML and UTC command/exit-code records are retained under ignored `tmp/`; the failed full-suite evidence remains intact.

After the successful suite, the **read-only** `python -m bft65b.modelling.final_evaluation verify --project-root C:\NEU\BFT65B --run-id final20261122a` exited 0 (`tmp/p56_closeout_verify.log` and `_execution.json`). It reproduced the original 246,083 test rows, 178 positives, 53 distinct represented failures, AP 0.172989033, ROC-AUC 0.876991628, Brier 0.000659085, and TP/FP/TN/FN of 109/1,483/244,422/69. All **nine** protected policy/plan/binding/summary/manifest/registry/model/score SHA-256 values in `tmp/p56_integrity_before.json` matched `tmp/p56_closeout_after.json`; the registry still has exactly **one** historical `test_access` transition and remains `complete`. The locked policy's reference test surrogate, 0.955581967 nominal USD thousands per bank-quarter, remains above the fixed never-alarm diagnostic, 0.782759273. No threshold or model change was made in response.
