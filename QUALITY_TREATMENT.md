# P2.1 Raw-Quality Treatment

## Scope and decision

The accepted `real20261111b` raw snapshot and the complete `audit20261114a` findings are the inputs. The audit has 1,687 nonzero issue-by-period entries covering 26 dataset-field-rule combinations. Every entry, including its period, denominator, prevalence, rationale, action, and unchanged before/after count, is in the ignored `runs/task_2_1_treatment/<run-id>/treatment_log.json`. The report-ready aggregate table is in `report.md` beside it.

Treatment is deliberately lossless: preserve all 172 raw files and source rows, retain every original value and `SOURCE_MISSING_FIELDS` list, and add a sorted `DQ_FLAGS` list to isolated derived Parquet. The accepted audit identified no confirmed duplicate key, invalid date, negative total asset, or other verified defect requiring a source-value correction. The observed nulls, source absences, and join limitations require flags and later analytical decisions. Nulls are not replaced with zero, and no historical Financials row is removed because it is absent from the current Institutions snapshot. Each Failures source `ID` remains one event even when several older Financials reports share its CERT.

## Measured reconciliation

| Dataset | Source and derived rows | Unique source keys before and after | Distinct keys carrying a flag | Raw values changed | Rows excluded |
| --- | ---: | ---: | ---: | ---: | ---: |
| Financials | 1,682,616 / 1,682,616 | 1,682,616 / 1,682,616 | 980,662 | 0 | 0 |
| Failures | 4,117 / 4,117 | 4,117 / 4,117 | 4,085 | 0 | 0 |
| Institutions | 27,835 / 27,835 | 27,835 / 27,835 | 3,482 | 0 | 0 |

Flagged-key counts are unions over overlapping issues. They cannot be reconstructed by adding field-level counts. Full field-level before/after absence and explicit-null counts are identical and are retained in `summary.json`; all 1,687 period issue counts reconcile to row flags. The derived Parquet preserves the source schema and its provenance columns and adds `DQ_FLAGS` with treatment marker `flag_only_v1`. It is stored under ignored `data_processed/task_2_1_treatment/<run-id>/`, separately from raw data and earlier benchmark outputs.

## Disposition of every observed issue type

| Dataset / field | Rule | Affected / denominator | Treatment and reason |
| --- | --- | ---: | --- |
| Failures / CERT | No Financials CERT match | 346 / 4,117 | Keep source event and flag; the accepted Financials scope cannot establish a bank link. |
| Failures / CERT | Null bank linkage | 488 / 4,117 | Keep unique source `ID` and null CERT; do not infer an identifier. |
| Failures / CERT | Explicit source null | 488 / 4,117 | Preserve null and its distinction from zero. |
| Failures / CERT, FAILDATE | Multiple prior reports | 3,220 / 4,117 | Keep one source event and flag the many-to-one join hazard. |
| Failures / CERT, FAILDATE | Only later reports | 30 / 4,117 | Keep event; a later `REPDTE` cannot justify a pre-failure observation. |
| Failures / COST | Explicit source null | 637 / 4,117 | Preserve unavailable cost; do not treat it as zero. |
| Financials / BRO | Explicit source null | 16,761 / 1,682,616 | Keep null and flag; defer any model-specific handling. |
| Financials / CERT | Absent from Institutions snapshot | 10,996 / 1,682,616 | Keep historical row; the endpoint is not a dated historical roster. |
| Financials / CHBAL | Explicit source null | 1 / 1,682,616 | Keep null and flag. |
| Financials / DEP | Explicit source null | 1 / 1,682,616 | Keep null and flag. |
| Financials / DEPDOM | Explicit source null | 1 / 1,682,616 | Keep null and flag. |
| Financials / EINTEXP | Explicit source null | 6,437 / 1,682,616 | Keep null and flag. |
| Financials / ELNATR | Explicit source null | 43,166 / 1,682,616 | Keep null and flag. |
| Financials / EQ | Explicit source null | 4,148 / 1,682,616 | Keep null and flag. |
| Financials / LNATRES | Explicit source null | 4,149 / 1,682,616 | Keep null and flag. |
| Financials / LNCI | Explicit source null | 1 / 1,682,616 | Keep null and flag. |
| Financials / LNLSGR | Explicit source null | 1 / 1,682,616 | Keep null and flag. |
| Financials / LNRE | Explicit source null | 1 / 1,682,616 | Keep null and flag. |
| Financials / NAASSET | Explicit source null | 77,730 / 1,682,616 | Keep null and flag. |
| Financials / NCLNLS | Explicit source null | 2,499 / 1,682,616 | Keep null and flag. |
| Financials / NETINC | Explicit source null | 4,692 / 1,682,616 | Keep null and flag. |
| Financials / NTLNLS | Explicit source null | 79,918 / 1,682,616 | Keep null and flag. |
| Financials / P3ASSET | Explicit source null | 971,925 / 1,682,616 | Keep null across 68 all-null quarters; do not imply zero delinquency. |
| Financials / P9ASSET | Explicit source null | 77,731 / 1,682,616 | Keep null and flag. |
| Financials / SC | Explicit source null | 1 / 1,682,616 | Keep null and flag. |
| Institutions / CHANGEC1 | Field absent from response | 3,482 / 27,835 | Keep null, preserve source-absence provenance, and flag it separately from explicit null. |

These are source-quality dispositions, not trained imputation rules. Each period-specific finding retains its own denominator in the machine-readable log. The number of flagged Failures source IDs includes other null or linkage signals and is not a count of invalid failures.

## Remaining analytical decisions

- Verify reporting availability before any historical bank linkage is used as a prediction feature. A `REPDTE <= FAILDATE` candidate alone is insufficient.
- Investigate official linkage for null-CERT and unmatched Failures without fabricating identifiers; classify exits only in the later exit milestone.
- Decide feature eligibility and training-fold-only imputation for historically unavailable fields such as P3ASSET in the feature and modelling milestones.
- Establish time-aware institution validity before using the current Institutions snapshot as historical evidence.
- Preserve COST missingness in later development-only cost analysis; do not equate observed resolution cost with preventable loss.

No panel, exit category, point-in-time join, modelling imputation, feature, or label is created here.

## Reproduction and acceptance evidence

The exact PowerShell commands are in `README.md`. Runs `treatment20261114a` and `treatment20261114b` each completed with 172 validated inputs, 172 derived outputs, four passing manifest validations, and all 172 accepted raw SHA-256 values unchanged. Their `summary.json` files are identical (SHA-256 `8c93be21cd5474529148dac89d722565dbc4fdd9a0eaa6a6ccf069817e262235`); their `treatment_log.json` files are identical (SHA-256 `fbcb82c7c3c84eb7eea3b7ef652429606866c69e4d0465940ae093d2df0ce12a`). All 172 derived files have identical logical and physical hashes across the two runs. The independent comparison is in ignored `runs/task_2_1_treatment/repeat_check.json`.

Focused audit/treatment tests passed 29/29. The complete regression suite passed 234/234 using a short synthetic-test temporary path. `git ls-files` returned no raw, processed, run, or virtual-environment files; `git check-ignore -v` confirmed representative paths. The derived data and detailed source identifiers remain outside Git. P2.1 implementation and report-table evidence are ready; incorporation into the final submission report remains pending.
