# P5 chronological split feasibility and freeze

## Scope and authority

The examination PDF, page 1, Section 2.3, requires prediction quarters no later than 2025Q2, a test period after development with at least four intervening quarters, and at least 50 failures in test. Its failure target is `(Q, Q + 4 calendar quarters]`. This milestone uses the accepted retrospective-cohort model table and verifies **distinct qualifying FDIC source events represented by eligible positive model rows**. A failure may label several earlier bank-quarter rows; counting those rows as separate failures would misstate the gate.

The cohort, label rule, seven predictors, and feature values remain unchanged. This stage reads only `(CERT, Q, Y)` from the accepted model table, eligible label metadata and FDIC event identity/date/disposition. It does not fit preprocessing, choose validation folds, train a model, inspect model scores, estimate costs, or select an alarm threshold.

## Frozen selection rule

1. Independently verify the accepted membership, labels, both feature-table runs, P4.4 leakage audit, and repaired P2.3 events. Check all consumed file hashes.
2. For each positive model row, require a unique label key with eligible membership and available feature history. Require exactly one `QUALIFYING_SOURCE_IDS` entry matching a unique repaired FDIC source ID, the same CERT, verified failure transaction, chronological bank link, corroborated supplementary match, and `Q < FAILDATE <= Q + 4 calendar quarters`. For every negative model row require a resolved negative label, complete target endpoint, and no qualifying source ID.
3. End test at the latest accepted modelling quarter, 2025Q2. Move its first quarter backward in calendar order until test contains at least 50 **distinct qualifying source IDs**. Choose the latest such start, so test is the shortest recent contiguous block meeting that count. Use the four immediately preceding complete prediction quarters as an excluded gap. Put all earlier accepted model quarters into development. Do not optimize these dates using feature values, scores, class weights, costs, or a preferred sample size.
4. Verify the development rightmost target endpoint is strictly before the first test prediction date. Require no qualifying source ID shared by development-positive and test-positive observations. The same CERT may occur at different prediction dates in both periods.
5. Assign every accepted model key to exactly one of development, gap or test. Gap rows are saved for reconciliation and excluded from fitting. Prediction Q controls the split; `REPDTE` and `FAILDATE` never substitute for Q.

Inspecting global outcome counts and event IDs for split feasibility is expressly permitted by `PROJECT_PLAN.md` and `AGENTS.md`. This means test **labels were inspected for feasibility**; the untouched-test protocol applies to predictive scores and all later modelling choices. Later work must use development folds only, freeze every choice, and evaluate test once.

## Reproduction and saved evidence

`python -m bft65b.modelling.split run --project-root C:\NEU\BFT65B --run-id <new-id>` writes an ignored run manifest, quarter-level counts, period boundaries, source-hash inventory and two ZSTD Parquet outputs: one `(CERT,Q,PERIOD,Y)` assignment per accepted model row and one positive-row FDIC source-ID linkage. `verify` independently checks the saved schema, codec, keys, row counts, checksums and upstream hashes. `show` checks saved evidence without rehashing upstream files. Use fresh IDs for independent repeats; compare logical summaries and output hashes. The summary records event counts, distinct positive banks and positive/negative bank-quarter rows separately.

The split estimates future-period generalization for the accepted retrospective study cohort. It does not imply that a bank-disjoint cohort or a historical public institution registry was used. The 120-day Financials availability lag and absent historical release vintages remain documented upstream. No final predictive performance is inferred from this split.

## Accepted measured result (2026-11-20)

Run `split20261120b` validated 343 consumed upstream data/evidence hashes and assigned all **1,603,183** accepted model rows (9,934 positive; 1,593,249 negative). Independent run `split20261120a` has identical quarter and period summaries, input hashes, and both output Parquet SHA-256 values. The union of positive source IDs across all quarters contains **2,520** distinct verified FDIC failure events. The selected test start, 2014Q1, yields **53** distinct represented events; moving the start one quarter later to 2014Q2 yields only **46**, below the specified gate. The outcome-count inspection used no predictor values or model scores.

| Period, inclusive Q | Quarters | Model bank-quarters | Positive rows | Negative rows | Distinct model banks | Distinct positive banks | Distinct qualifying events |
|---|---:|---:|---:|---:|---:|---:|---:|
| Development, 1984Q1–2012Q4 | 116 | 1,329,451 | 9,677 | 1,319,774 | 23,301 | 2,462 | 2,462 |
| Excluded gap, 2013Q1–2013Q4 | 4 | 27,649 | 79 | 27,570 | 7,017 | 38 | 38 |
| Test, 2014Q1–2025Q2 | 46 | 246,083 | 178 | 245,905 | 6,798 | 53 | 53 |

The last development prediction date is **2012-12-31**. Its four-quarter target window ends **2013-12-31**; the first test prediction date is **2014-03-31**, after four complete excluded prediction quarters. No qualifying source event appears in both development-positive and test-positive rows. Some source events label several earlier quarters and therefore overlap development with the gap (20 events) or the gap with test (13 events). This is expected because the gap is excluded from fitting; the development/test intersection is zero.

The test block spans 46 quarters because the requirement is evaluated on distinct, eligible, verified events and recent years contain few such events. This is a temporal generalization study over a long later era; it is not evidence of uniform failure incidence or a bank-disjoint evaluation. **6,725 CERTs** occur at different Q values in both development and test, as permitted by the time-ordered study design; no `(CERT,Q)` key is shared. Final report placement, development validation folds, model fitting, costs and one-time predictive test evaluation remain pending.

All accepted model rows have exactly one assignment; none are unassigned. The 27,649 gap rows are preserved in the assignment file but excluded from fitting. The upstream P4.3 reconciliation explains why **2,391,441** accepted label decisions are outside the model table: 297,530 have no eligible feature history, 2,063,045 are membership-ineligible, 28,184 have uncertain membership, 2,677 have unresolved labels, and 5 have excluded labels despite feature history. These counts are copied from the verified P4.3 summary as split-lineage evidence, not reclassified by this milestone; quarter-level upstream reasons remain in that summary. Test rows are withheld from all future fitting and threshold selection.

### Acceptance commands and files

The following commands ran from `C:\NEU\BFT65B` with `$env:PYTHONPATH='C:\NEU\BFT65B\src'` and `.venv_local\Scripts\python.exe`:

```powershell
.\.venv_local\Scripts\python.exe -m bft65b.modelling.split run --project-root C:\NEU\BFT65B --run-id split20261120a
.\.venv_local\Scripts\python.exe -m bft65b.modelling.split run --project-root C:\NEU\BFT65B --run-id split20261120b
.\.venv_local\Scripts\python.exe -m bft65b.modelling.split verify --project-root C:\NEU\BFT65B --run-id split20261120b
.\.venv_local\Scripts\python.exe -m bft65b.features.leakage_audit verify --project-root C:\NEU\BFT65B --run-id p44audit20261120a
.\.venv_local\Scripts\python.exe -m pytest tests/test_chronological_split.py -q --basetemp C:\NEU\BFT65B\tmp\pytest_p51_focus_d
.\.venv_local\Scripts\python.exe -m pytest -q --basetemp C:\NEU\BFT65B\tmp\pytest_p51_full_c
git ls-files data_raw data_processed runs tmp .venv_local
git check-ignore -v data_processed/task_5_1_split/split20261120b/assignments.parquet runs/task_5_1_split/split20261120b/summary.json tmp/pytest_p51_full_c
git diff --check
```

Both split runs and independent saved-output verification completed; the focused suite passed **18/18**, and the full suite passed **414** with **3 skips** from `tests/test_spark_panel.py` because `HADOOP_HOME` and `PYSPARK_PYTHON` were unset in this test shell. Post-run P4.4 verification passed 9/9 checks and all 1,192 upstream hashes. `git ls-files` returned no data, runs, temporary or environment paths; ignore probes matched `.gitignore`. `git diff --check` returned zero, with existing line-ending conversion warnings only.

The accepted output SHA-256 values are `c4709551fdef8eee4a04640a118ddbc5d83841a958bbb1a3a5cecb5d7a39096c` for `assignments.parquet` and `354c961a68473d4f2f99bd838ee995930bdfae77f6fbae6d438ad70bd118daed` for `positive_linkage.parquet`; both independent runs matched. The ignored `runs/task_5_1_split/split20261120b/repeat_comparison.json` records equal bounds, 166 quarter rows, period counts, 343 input hashes and two output hashes. Its SHA-256 is `d42b48bd51d73b097b71609020db5e714c58c654c80d2eb8b8337df6defd41ec`. The accepted run manifest and summary contain the environment, validation states, row counts and file paths; generated data remain outside Git.
