# P4.4 Temporal-Leakage Audit

## Scope and authority

The examination PDF (page 1, Section 2.3; page 2, Part 4) defines the failure target as `(q, q + 4 calendar quarters]`, limits modelling prediction quarters to 2025Q2, and requires a leak-free feature table with preprocessing fitted on training data only. This audit does not choose a real development/test split or fit on the production table. The accepted retrospective activity cohort defines a historical study population; it is not a claim that the later structural registry was publicly available at each historical `q`. Financials predictors still follow the accepted provisional `REPDTE + 120 calendar days <= q` rule.

The read-only audit uses `panel20261115b`, `pit20261115f`, `featurecontract20261116b`, `membership20261120a`, `labels20261120a`, `featuretable20261120a`, and the independent repeat `featuretable20261120b`. It runs the accepted P4.3 saved-output and upstream validators before making its own evidence directory. It hashes all recorded accepted inputs and outputs before and after the audit. New evidence belongs only under the ignored `runs/task_4_4_leakage_audit/<run-id>/` directory.

## Executable checks

1. `ReportHistoryState`, the production report-cache component, advances source quarters only when their report date plus 120 days is at or before `q`. It rejects unordered/duplicate source quarters, wrong-quarter rows, duplicate bank-quarter keys, and backward prediction dates. Tests cover periods without a new report, cross-year resumed gaps, row-order invariance across banks, and bank isolation.
2. The accepted feature contract filters unavailable future reports and verified revisions before resolving revision ambiguity. Exact-at-`q` availability is inclusive. Tied or undated eligible versions retain the ambiguity error. Missing calendar quarters produce null rolling features rather than forward fill or use of a previous observed row.
3. The production join rejects duplicate/wrong-quarter keys, a candidate with no label, a candidate whose label claims no feature history, a label-only key claiming feature history, a resolved positive/negative outside eligible membership, and inconsistent `Y`/disposition. Legitimately label-only, no-history rows remain reconciled.
4. The audit independently reads all 166 accepted label, candidate and model partitions. For every quarter it compares the exact candidate keys with label-side feature-history eligibility; compares the exact model keys with the eligible, history-available, positive/negative subset; compares every model predictor value and `Y` with its candidate and label rows; and checks availability dates, source age, counts and field nulls.
5. A fixed-seed SHA-256 ranking selects eight model rows each from ordinary, calendar-gap and other-missing categories. It then recomputes all seven features and lineage from the accepted panel history for those selected `(CERT, q)` keys. Future reports, a revision verified only after `q`, and future/outcome/audit metadata are perturbed in memory; earlier values and lineage must remain unchanged. Detailed sampled identifiers remain in ignored evidence. This sample is not presented as an exhaustive feature-value recomputation.
6. Synthetic forward-time folds fit fresh scikit-learn pipelines only on their training rows. Extreme held-out values and labels must leave the training matrix, imputer statistics, scaler mean/variance/scale, feature names and dimensions unchanged. An intentionally contaminated combined-period fit must change the appropriate statistics; a deliberately early report must be rejected. These are negative controls, never edits to accepted data.
7. Synthetic outcome fixtures cover failure on `q`, immediately after `q`, at and after the right endpoint, incomplete follow-up, assistance, ordinary merger and uncertain identity. Future outcomes may change labels while leaving predictors for the same eligible keys unchanged.

## Reproduction

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_leakage_audit.py tests\test_feature_table.py tests\test_feature_contract.py tests\test_feature_label.py tests\test_feature_preprocessing.py
.\.venv_local\Scripts\python.exe -m bft65b.features.leakage_audit run --project-root C:\NEU\BFT65B --run-id p44audit_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.features.leakage_audit verify --project-root C:\NEU\BFT65B --run-id p44audit_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.features.leakage_audit show --project-root C:\NEU\BFT65B --run-id p44audit_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest -q -rs -p no:cacheprovider
```

The run ID must be fresh. `verify` rehashes all recorded accepted inputs; `show` checks saved manifest, summary, metadata and evidence consistency without rehashing the 1,192 accepted files. No command downloads FDIC data, rewrites prior runs, fits the production table, or uses a real test period.

## Measured findings and acceptance

Run `p44audit20261120a` completed nine named checks. Its independent, full-data alignment read every saved candidate, label and model quarter and reproduced the following counts. The unit is a distinct `(CERT, Q)` bank-quarter decision; there are 166 prediction quarters through 2025Q2.

| Check / population | Measured count | Result |
|---|---:|---|
| Accepted label keys | 3,994,624 | Reconciled to the label artifact |
| Candidate keys with available report history | 3,697,094 | Exactly the label-side history-eligible keys |
| Label-only keys without feature history | 297,530 | Explicitly reconciled |
| Final model keys | 1,603,183 | Exactly eligible membership, history available and resolved binary labels |
| Positive model keys | 9,934 | Exact `Y=1` and label disposition match |
| Negative model keys | 1,593,249 | Exact `Y=0` and label disposition match |
| Accepted upstream files hashed before and after | 1,192 | Zero changed SHA-256 values |
| Deterministic source-history rows recomputed | 24 across 23 banks | All seven values and lineage matched; all future perturbations invariant |

The 24 recomputed rows comprise eight ordinary, eight calendar-gap and eight other-missing cases, selected by a frozen SHA-256 ranking. Their prediction quarters span 1984-09-30 to 2022-09-30. The full-data pass checked every model predictor against its candidate row, every model key and `Y` against the accepted label subset, candidate availability and report age, and all seven model null counts. This is an exhaustive join/value comparison **between accepted derived artifacts**; recomputation from the original Financials source history is the stated 24-row sample. It must not be described as exhaustive source-history recomputation.

| Model predictor | Null bank-quarter values / 1,603,183 |
|---|---:|
| `asset_size` | 0 |
| `equity_to_assets_pct` | 2,366 |
| `deposits_to_assets_pct` | 1 |
| `roa_reported_pct` | 0 |
| `pca_leverage_reported_pct` | 0 |
| `asset_growth_qoq_pct` | 23,433 |
| `roa_four_quarter_mean_pct` | 69,395 |

At 2025Q2, 24,064 label keys reconcile to 24,051 candidate keys and 4,421 model keys (two positive and 4,419 negative). The summary contains all 166 quarter rows, nine named pass/fail checks, sample coverage, and limitations. `hash_inventory.json` records the 1,192 before/after SHA-256 pairs; `sample_checks.json` retains the detailed sampled keys and perturbation results outside Git. Saved `verify` rehashed all accepted inputs and passed.

### Bounded repair and test evidence

A synthetic regression exposed that the P4.3 join previously treated a positive label on an ineligible membership row as an ordinary exclusion. The test failed before the repair (`DID NOT RAISE ValueError`). The join now rejects that contradiction, inconsistent `Y`, and a label-only row that claims feature history. An independent read-only scan found **zero** such contradictions and **zero** missing PIT keys with a true history flag among the 3,994,624 accepted labels. No accepted P4.3 Parquet, manifest or summary was rewritten. The report-cache loop was extracted into `ReportHistoryState` without changing the accepted one-version-per-bank-quarter behavior, enabling direct tests of advancing `q`, unchanged cached anchors, ordering and gaps.

Focused pytest passed **35/35**. The first full pytest attempt under the application's long sandbox temporary path reported 27 failures, 369 passes and three skips; the failures were `FileNotFoundError` from legacy acquisition fixtures calling `tempfile.mkstemp` beneath that long Windows path. A fresh short ignored temp path, `C:\NEU\BFT65B\tmp\pytest_p44_20261120a`, resolved the environment issue: full pytest passed **396/396**, with three unrelated Spark-runtime skips. The skip reason is “Task 3.0 verified Spark runtime environment is not configured” in the current test shell. Git confirmed `runs/` and `tmp/` are ignored and no raw, processed, run or virtual-environment files are tracked. `git diff --check` found no whitespace errors; Git emitted existing LF/CRLF conversion warnings.

Environment recorded in the run manifest: Windows `10.0.26200`, Python 3.11.9, PyArrow 18.1.0, pandas 2.2.3, Polars 1.35.2, scikit-learn 1.6.1 and pytest 8.3.4. The exact manifest, summary, hash inventory, sample checks and metadata are under `runs/task_4_4_leakage_audit/p44audit20261120a/`.

The fixed 120-day availability lag is an assumption, not a reconstructed historical publication timestamp. The accepted panel has one version per bank-quarter and lacks historical revision vintages; synthetic revision tests establish rule behavior but cannot prove unobserved historical vintages. The retrospective cohort and its source limitations remain documented in `RETROSPECTIVE_COHORT_POLICY.md`. These limitations and final report placement remain open; the P4.4 implementation and evidence gate passes without selecting a real split or fitting a predictive model.
