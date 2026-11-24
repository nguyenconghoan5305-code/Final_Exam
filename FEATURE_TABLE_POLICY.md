# P4.3 Production Feature–Label Table

## Inputs and temporal contract

The authoritative examination PDF (page 2, Part 4) requires a leak-free feature table joined to the four-quarter failure target, with learned preprocessing fitted on training data only. This implementation consumes the immutable accepted `panel20261115b` Financials history, `pit20261115f` as-of selections, `featurecontract20261116b`, `membership20261120a`, `labels20261120a`, and the checksum/repeat evidence in `p42review20261120b`. Their existing validators and saved hashes must pass before a new run writes any output. No FDIC network request is made.

The seven numeric feature formulas, types, units, rounding and null rules are frozen in `FEATURE_DICTIONARY.md` and `src/bft65b/features/contract.py`. `q` is a prediction quarter-end through 2025Q2. The selected source report `r` and **every** calendar-quarter predecessor required by a feature must satisfy the accepted availability rule at `q`: an undated original is assumed available at `REPDTE + 120 calendar days`; a verified later revision date delays that version. The accepted observed panel contains one report version per `(CERT, REPDTE)`, with no historical revision publication vintages. The production runner reads the complete accepted observed-report history in quarter order, retains the most recent four *observed* reports per CERT, and invokes the accepted feature contract when each new report becomes available. Exact calendar-quarter dates are checked inside that contract; a resumed reporting gap does not turn a nonconsecutive older row into `r-1` or `r-3`. Later report partitions are not used for an earlier `q`.

The runner independently compares every PIT selected report's key, `ASSET`, `DEP`, `EQ`, `ROA`, `RBC1AAJ`, source ID, source-missing-field provenance and assumed availability with the observed panel anchor. Any mismatch or unavailable report stops the run. The feature-candidate output retains selected report and availability dates, report age, basis, source provenance, and a per-feature lineage/reason JSON string **as audit fields**. These are excluded from the model matrix.

## Join and output contract

For every `q`, the runner requires unique `(CERT, Q)` on both the PIT feature candidate side and the accepted label side. A PIT candidate with no matching label is a failed completeness gate. A label without a PIT candidate is counted as `no_feature_history`. The candidate output contains all accepted PIT candidate keys and seven features, including genuine nulls. The model output includes exactly those rows whose accepted label has `MEMBERSHIP == eligible`, `FEATURE_HISTORY_ELIGIBLE == true`, and `LABEL_DISPOSITION` positive or negative. An unresolved, excluded, or no-history row is retained in the quarter and overall reconciliation, never silently converted to a negative or dropped from evidence.

The model Parquet schema is `CERT`, `Q`, `Y`, then these seven features in frozen order: `asset_size`, `equity_to_assets_pct`, `deposits_to_assets_pct`, `roa_reported_pct`, `pca_leverage_reported_pct`, `asset_growth_qoq_pct`, `roa_four_quarter_mean_pct`. `CERT`, `Q`, and `Y` are identifiers/outcome, **not predictors**. The only permitted predictor matrix columns are the seven names above. No failure date, FDIC source ID, membership evidence, structural end date, audit disposition, or future-derived quality flag is in that matrix.

Both quarterly output kinds use Arrow schemas and ZSTD Parquet under a fresh Git-ignored `data_processed/task_4_3_feature_table/<run-id>/` namespace. Same-filesystem temporary writes are read back before atomic promotion. The separate `runs/task_4_3_feature_table/<run-id>/` manifest records input hashes, output schema version, file hashes, row counts, codec, validation gates and completion status. A complete-looking summary with a missing or failed manifest is not accepted by `verify` or `show`.

## Unfitted preprocessing

`make_preprocessing_pipeline()` returns a new, unfitted scikit-learn `Pipeline`. A deterministic selector converts only the seven approved predictors to float64, mapping saved nulls to NaN; it rejects unsupported and nonfinite values. Conversion to float64 is required by scikit-learn and does **not** change the stored decimal128 source or derived values. `SimpleImputer(strategy="median", keep_empty_features=True)` learns medians from the caller's training fold only. An entirely missing training feature remains present and is filled with zero under scikit-learn's configured behavior. `StandardScaler` then learns mean and scale from that same training fold only. No clipping, encoding, feature selection or real-data fitting is performed here. Synthetic forward-time folds and held-out perturbations verify fit isolation and stable seven-column output.

## Measured production result

The independently verified `featuretable20261120a` run covers 166 prediction quarter-ends through 2025Q2. Its observed-report scan consumed 1,656,057 rows from 164 historical Financials partitions; the final eligible source quarter is 2024Q4 under the 120-day rule. The run wrote 166 candidate and 166 model Parquet files. Counts below use bank-quarter keys as the unit. A label-only key has no eligible report history and is retained in the reconciliation rather than silently dropped.

| Measure | Bank-quarter keys |
|---|---:|
| Accepted label decisions | 3,994,624 |
| PIT feature candidates | 3,697,094 |
| One-to-one matched keys | 3,697,094 |
| Feature-only keys | 0 |
| Label-only keys / no eligible feature history | 297,530 |
| Matched but excluded or unresolved | 2,093,911 |
| Model rows with positive target | 9,934 |
| Model rows with negative target | 1,593,249 |
| Total model rows | 1,603,183 |

Null counts in the 1,603,183-row model table are `asset_size` 0, `equity_to_assets_pct` 2,366, `deposits_to_assets_pct` 1, `roa_reported_pct` 0, `pca_leverage_reported_pct` 0, `asset_growth_qoq_pct` 23,433, and `roa_four_quarter_mean_pct` 69,395. No production imputation was applied. The corresponding candidate-table null counts and all period-level counts are in the ignored summary.

For 2025Q2, 24,064 label keys reconcile to 24,051 candidate keys and 13 without eligible report history. Of the 24,051 matched keys, 4,421 enter the model table (2 positive, 4,419 negative) and 19,630 remain excluded or unresolved. These numbers describe the retrospective cohort and the accepted label policy; they do not establish a live historical registry population.

The primary manifest, summary and 332 output hashes are under `runs/task_4_3_feature_table/featuretable20261120a/`; Parquet outputs are under `data_processed/task_4_3_feature_table/featuretable20261120a/`. Both directories are Git-ignored. `independent_readback_counts.json` independently read all 166 model files and reproduced 1,603,183 rows, 9,934 positive targets and all seven null counts. Use `pyarrow.parquet.ParquetFile(path).read()` for direct reads: PyArrow dataset discovery can infer `Q=...` as a partition column and conflict with the physical `Q` date column. The independent `featuretable20261120b` run passed full saved-output and upstream-checksum verification. Its `repeat_comparison.json` records zero differences among all 332 Parquet SHA-256 values, equal quarter summaries, equal overall counts, equal null counts and equal exclusion counts. The two summary JSON hashes differ because run IDs, paths and timestamps differ. Focused tests passed 16/16; full regression passed 387 with three unrelated Spark-runtime skips. Final report placement remains pending. The fixed 120-day lag is an assumption, not a reconstruction of historical release vintages. The activity cohort is retrospective as documented in `RETROSPECTIVE_COHORT_POLICY.md`; its structural evidence is never used as a predictor. Part 5 will choose real chronological development/test periods and fit this pipeline only inside development training folds.
