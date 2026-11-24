# P5 development-only forward validation and bounded model comparison

## Scope and authority

The examination PDF, page 1 Section 2.3 and page 2 Part 5, requires a simple baseline, own model(s), validation suitable for time-ordered imbalanced outcomes, development-only tuning and a single final test use. The frozen split `split20261120b` provides development Q from 1984Q1–2012Q4, an excluded 2013 gap and test from 2014Q1–2025Q2. The prior split feasibility audit inspected global test outcome counts only. This milestone opens the seven predictor values **only for development partitions**. Gap and test predictors are never fitted or scored here.

The simple baseline is an unweighted L2 logistic regression using scikit-learn. The substantive nonlinear comparison is a bounded histogram gradient boosting classifier, also using scikit-learn. These are project-configured library estimators; neither is described as an algorithm coded from scratch. Both consume the same seven already approved predictors and the existing `make_preprocessing_pipeline()` in a fresh fold-local complete pipeline. No new feature formula, resampling, feature selection, class weight, early stopping, calibration or full-development refit is used.

## Frozen fold policy

Use the last 48 development prediction quarters as three consecutive, non-overlapping, 16-quarter validation blocks. Each block has an expanding training history from the first development quarter and four complete excluded prediction quarters immediately before it. Every training target right endpoint must be strictly before validation starts. Positive FDIC source IDs in train and validation must be disjoint. This schedule is derived from the accepted development scope and event coverage before any candidate fit. A validation block with a single class or undefined average precision is recorded and blocks candidate selection; difficult blocks are never removed after seeing scores.

| Fold | Training Q | Intervening Q | Validation Q |
|---|---|---|---|
| 1 | 1984Q1–1999Q4 | 2000Q1–2000Q4 | 2001Q1–2004Q4 |
| 2 | 1984Q1–2003Q4 | 2004Q1–2004Q4 | 2005Q1–2008Q4 |
| 3 | 1984Q1–2007Q4 | 2008Q1–2008Q4 | 2009Q1–2012Q4 |

All three validation blocks are inside development. Adjacent blocks are contiguous; a quarter can be an earlier fold's validation observation and a later fold's training observation after the required temporal gap. Initial development quarters through 2000Q4 have no out-of-fold prediction. The same CERT can appear at several Q values; this is future-period validation, not a bank-disjoint study. Distinct failure events are counted from accepted positive source IDs, separately from positive bank-quarter rows.

The accepted `dev20261121a/frozen_plan.json` was written before any candidate fitting (SHA-256 `298e8999660c1a07e65e314e322e3e9e888f9c84280f0a8efac0cacbcea19c0b`). Its measured fold coverage is:

| Fold | Training rows | Training positive rows | Distinct training banks/events | Validation rows | Validation positive rows | Distinct validation banks/events | Validation prevalence |
|---|---:|---:|---:|---:|---:|---:|---:|
| 1 | 880,486 | 7,644 | 1,949 / 1,949 | 149,382 | 77 | 21 / 21 | 0.0005154570 |
| 2 | 1,033,680 | 7,742 | 1,971 / 1,971 | 137,419 | 413 | 168 / 168 | 0.0030054068 |
| 3 | 1,173,760 | 7,799 | 1,999 / 1,999 | 122,293 | 1,521 | 442 / 442 | 0.0124373431 |

The first 68 development quarters through 2000Q4 contain 920,357 model rows with no OOF score. Each fold's training target endpoint equals the end of its four-quarter gap and strictly precedes validation Q. No training/validation FDIC source ID overlap was observed in the frozen plan.

The distinct training-bank counts above were independently counted from the accepted `split20261120b/positive_linkage.parquet` for each frozen training end date; this snapshot happens to have one represented positive source ID per such bank. The frozen plan records distinct training event counts and both distinct validation counts.

## Frozen candidates, metrics and selection

The machine-readable `frozen_plan.json` is written and SHA-256 recorded in the manifest before fitting. It records all fold boundaries, row/event counts, predictor names, candidate parameters, seed 42, two-thread limit, accepted input hashes and the selection rule. Candidate 1 is `LogisticRegression(C=1.0, solver='liblinear', max_iter=250, class_weight=None, random_state=42)`. Candidate 2 is `HistGradientBoostingClassifier(learning_rate=0.05, max_iter=40, max_leaf_nodes=15, max_depth=3, l2_regularization=1.0, early_stopping=False, random_state=42)`.

The primary ranking metric is `sklearn.metrics.average_precision_score`: non-interpolated, stepwise average precision over positive rows, **not** trapezoidal PR-AUC. Report it per fold, together with class prevalence, ROC-AUC and Brier score. ROC-AUC and AP are undefined for a single-class validation block; record null and stop selection. Brier is descriptive for uncalibrated scores, not a calibration claim. Pooled out-of-fold metrics are reported separately because the pooled rows come from different fitted fold models and periods. Positive rows can share one FDIC event; observation-level AP must not be presented as an event-level metric.

Select the highest unweighted mean of the three defined fold AP values. If means differ by at most `1e-12`, choose the higher minimum fold AP; a remaining tie goes to the lexicographically smaller candidate ID. This rule uses development predictions only. Save the selected candidate's out-of-fold scores for the later cost/threshold milestone. No operational alarm threshold is selected here, and no test prediction or predictive test score is computed.

## Fit isolation and output contract

Each candidate/fold creates a new `Pipeline(preprocessing=make_preprocessing_pipeline(), estimator=...)`; `fit()` receives only the fold's training rows and exactly the seven predictor columns. The first pipeline step selects approved numeric features, then median imputation and scaling learn only from that training slice. An entirely missing training feature retains a stable output column and receives the documented scikit-learn zero fill. Validation is transformed without refitting. `CERT`, `Q`, `Y`, FDIC event IDs, failure dates, cohort evidence and audit fields are never predictors.

The normal entry points reject nondevelopment keys, duplicate keys, role dates outside frozen fold boundaries, invalid labels and a training fold with one class. OOF output contains `CERT`, `Q`, `Y`, fold ID, candidate ID and a finite score in `[0,1]`; event linkage remains in the accepted split audit artifact. Each candidate must cover every frozen validation key exactly once. New ZSTD Parquet files, summary, manifest, hashes and environment belong only in ignored run/data namespaces. An independent new run ID repeats the experiment; key-level scores are compared within a declared absolute numerical tolerance instead of assuming byte-identical serialized model objects. Actual fit time is measured using a monotonic clock and recorded per fold/candidate.

The accepted retrospective cohort, publication-lag uncertainty and selected seven feature formulas remain as documented in `RETROSPECTIVE_COHORT_POLICY.md`, `FEATURE_TABLE_POLICY.md` and `P4_LEAKAGE_AUDIT.md`. The final English report will place measured fold metrics and methodological limitations after this implementation gate passes.

## Measured development results

The first execution, `dev20261121a`, opened the 116 development model partitions and produced 409,094 out-of-fold scores for each candidate across 48 validation quarters. Those rows include 2,011 positive bank-quarter observations and 512 distinct positive FDIC events. The score-based metrics below are **observation-level**. They do not estimate event-level detection or an operational alert threshold. AP is stepwise average precision; Brier uses the uncalibrated probability scores.

| Candidate | Fold | Validation rows | Positive rows | Prevalence | AP | ROC-AUC | Brier |
|---|---|---:|---:|---:|---:|---:|---:|
| Logistic baseline | 1 | 149,382 | 77 | 0.000515457 | 0.014679594 | 0.742936211 | 0.000864795 |
| Logistic baseline | 2 | 137,419 | 413 | 0.003005407 | 0.026402604 | 0.699025200 | 0.003072900 |
| Logistic baseline | 3 | 122,293 | 1,521 | 0.012437343 | 0.197070064 | 0.917274220 | 0.012022159 |
| Bounded histogram gradient boosting | 1 | 149,382 | 77 | 0.000515457 | 0.020989539 | 0.840328631 | 0.000530377 |
| Bounded histogram gradient boosting | 2 | 137,419 | 413 | 0.003005407 | 0.065534298 | 0.791548347 | 0.002929162 |
| Bounded histogram gradient boosting | 3 | 122,293 | 1,521 | 0.012437343 | 0.298658014 | 0.948438642 | 0.010670200 |

| Candidate | Mean fold AP (selection) | Minimum fold AP | Pooled OOF AP (descriptive) | Pooled OOF ROC-AUC | Pooled OOF Brier |
|---|---:|---:|---:|---:|---:|
| Logistic baseline | 0.079384087 | 0.014679594 | 0.116383095 | 0.860180293 | 0.004941861 |
| Bounded histogram gradient boosting | 0.128393950 | 0.020989539 | 0.227339193 | 0.914075828 | 0.004367314 |

The frozen ranking rule selects bounded histogram gradient boosting because its mean fold AP is higher. This selection is based solely on development-fold predictions. The first fold has only 21 distinct positive events, so its metric is sensitive to a small number of failures. The validation prevalence changes substantially across folds; pooled OOF scores combine models fitted on different historical training periods. These measurements do not establish future test performance or calibrated probabilities.

## Independent repeat and execution evidence

The fresh run `dev20261121b` used the same frozen fold and candidate policy, seed, two-thread limit and accepted input hashes. The saved `compare` check independently verified both manifests, source hashes, OOF schemas, keys, labels, codecs and checksums, then compared all **818,188** candidate/key scores. Maximum absolute score difference was **0.0**, below the declared `1e-10` tolerance. Both runs selected `bounded_hist_gradient_boosting`; both candidate OOF Parquet hashes matched exactly. The comparison is saved as `runs/task_5_2_development/dev20261121b/repeat_comparison.json`.

The first execution recorded **756.1814328999026 s** and the repeat **544.4138668000232 s** in their respective summaries. The monotonic timer covers development input loading, six fold fits, validation scoring, output writing and final checks. These are descriptive execution times, not a controlled speed comparison. The repeat run records Windows build 26200, Python 3.11.9, scikit-learn 1.6.1, pandas 2.2.3, PyArrow 18.1.0, 16 logical processors and a two-thread fit limit. Individual fit and prediction durations are in the saved fold results. The first run predates the explicit summary environment field; its manifest still records Python and dependency versions. Both saved summaries record zero test predictor files opened, zero test predictions, no full-development refit and no threshold.

The focused synthetic tests passed **8/8**. A final sequential full-suite run passed **422 tests with 3 unrelated Spark-environment skips** in 228.86 s. A prior concurrent full-suite attempt had one Windows `WinError 5` in an acquisition fixture; the isolated retry passed and the final sequential suite passed. This was not a development-model failure. Commands and saved-result inspection are documented in `README.md`. All final-report placement remains pending.
