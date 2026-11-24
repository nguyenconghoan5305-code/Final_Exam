# P2.1 Raw-Data Quality Audit

## Scope and status

This is the first Part 2 milestone: an automated, read-only audit of the accepted `real20261111b` snapshot. It covers 170 Financials quarters (1984Q1-2026Q2; 1,682,616 records), the complete Failures table (4,117 records), and the complete Institutions endpoint snapshot (27,835 records). All 172 Parquet files were independently validated against the accepted manifest for schema, partition, keys, row count, ZSTD codec, and SHA-256 before inspection and checked again for unchanged SHA-256 afterward. No source record was cleaned, imputed, dropped, relabelled, or written back.

The official assignment requires an automated quality report and, in later milestones, justified handling, a clean panel, exit treatment, and point-in-time correctness. This audit closes only the measurement milestone. `SOURCE_FIELD_INVENTORY.md` records the FDIC dictionary titles, definitions, and unresolved units used to limit the rules below. The assignment's general amount unit is thousands of USD and ratio unit is percent; field-specific historical comparability remains subject to review.

## Report-ready findings

Each prevalence uses the stated dataset's record count as its denominator. The machine-readable result also has per-quarter Financials counts, per-event-year Failures counts, and a single Institutions snapshot breakdown. An absent source field and a present JSON null are separate outcomes.

| Dataset / field | Period | Check | Observed count | Denominator | Prevalence | Interpretation |
| --- | --- | --- | ---: | ---: | ---: | --- |
| Financials / P3ASSET | 1984Q1-2026Q2 | Present but explicitly null | 971,925 | 1,682,616 | 57.763% | All records are null in 68 quarters, 1984Q1-2000Q4. The field is absent from zero source responses. |
| Institutions / CHANGEC1 | Endpoint snapshot | Requested field absent | 3,482 | 27,835 | 12.509% | Source-field absence, rather than an explicit null. The change-code mapping and historical applicability remain unresolved. |
| Failures / CERT | Event dates, all years | Explicitly null | 488 | 4,117 | 11.853% | These source records retain unique `ID` values; no bank CERT is inferred. |
| Failures / COST | Event dates, all years | Explicitly null | 637 | 4,117 | 15.472% | Estimated resolution cost is unavailable for these records; no zero is substituted. |
| Financials / CERT | 1984Q1-2026Q2 | No CERT in Institutions snapshot | 10,996 | 1,682,616 | 0.654% | Covers 235 of 24,064 distinct Financials CERT values (0.977%); a current endpoint snapshot is not a dated historical roster. |
| Failures / CERT | Event dates, all years | Non-null CERT with no Financials CERT | 346 | 4,117 | 8.404% | Coverage is restricted to Financials 1984Q1-2026Q2; an absent report does not prove an invalid failure source record. |
| Failures / CERT, FAILDATE | Event dates, all years | Only later Financials reports exist | 30 | 4,117 | 0.729% | A CERT match exists, but no `REPDTE <= FAILDATE` candidate exists. |
| Failures / CERT, FAILDATE | Event dates, all years | More than one prior Financials report | 3,220 | 4,117 | 78.212% | This is expected longitudinal history and a warning against a row-multiplying join, not a definite source defect. |

Financials-to-Institutions CERT equality matched 1,671,620 of 1,682,616 Financials **records** and 23,829 of 24,064 distinct Financials **CERT values**. The Institutions denominator is 27,835 distinct snapshot CERT values; 23,829 occur at least once in Financials. The snapshot has no event-effective date here, so this comparison does not establish historical institution validity.

The Failures linkage categories are mutually exclusive by **source record** and sum to 4,117: 488 missing CERT, 346 non-null CERT with no Financials CERT, 30 with only post-failure reports, and 3,253 with at least one `REPDTE <= FAILDATE` chronological candidate. All 4,117 failure source IDs were unique. Distinct CERT counts by category can overlap when one CERT appears in more than one failure source record; they must not be summed as a unique-bank denominator. A chronological candidate still lacks verified publication time and must not be treated as a valid point-in-time modelling join.

The 170 Financials quarters, 88 Failures event years, and one Institutions snapshot are reported separately. The audit observed zero duplicate endpoint primary keys, zero Financials partition/date mismatches, zero negative total assets, zero `ACTIVE` values outside 0/1, zero `DEPDOM > DEP` review observations, and zero recorded source conversion anomalies. These are observed results under the selected checks, not proof that all historical accounting relationships are valid.

## Rule boundaries and later decisions

- `ASSET < 0` is a definite violation of the total-assets concept. The synthetic fixture verifies its count and denominator; none occurred in the accepted snapshot.
- `DEPDOM > DEP` is a **review observation** because the dictionary identifies domestic deposits as a component but historical reporting bases and field-specific units need confirmation. It is not automatically repaired.
- A complete balance-sheet identity cannot be checked with the selected fields because total liabilities are not present. The titles alone do not justify asserting `LNRE + LNCI <= LNLSGR` for every historical period. Universal bounds for profitability, capital ratios, and P3ASSET/P9ASSET/NAASSET are not asserted while historical scale and formula questions remain.
- Source JSON types are summarized from accepted acquisition evidence. The typed Parquet files validate the Arrow schema but cannot reconstruct every original JSON numeric representation.
- `affected_records_unique` in the JSON counts a row once across overlapping **audit signals**, including explicit nulls and the expected multiple-report join warning. It is not a count of confirmed invalid records. The separate `missing_or_null_records_unique` count uses a union over fields.
- Treatment rules, before/after counts, merger/other-exit classification, historical institution validity, the conservative report-availability policy, feature design, and labelling remain open in subsequent milestones.

## Reproduction and evidence

Run `python -m bft65b.quality.audit audit` with the exact PowerShell parameters in README.md and a new run ID. The CLI refuses to overwrite an existing run directory. `show` reads saved findings without auditing again. The detailed result and its English report table are in ignored `runs/task_2_1_quality/<run-id>/findings.json` and `report.md`; `run_manifest.json` and `metadata.json` retain source hashes, configuration, environment, validation results, and command. Identical full-snapshot repeats are compared by the SHA-256 of `findings.json`, which contains no run timestamp or timing metadata. Tests use synthetic rows and never corrupt accepted source files.

Runs `audit20261114a` and `audit20261114b` each completed with 172 unchanged input hashes and the identical findings SHA-256 `eaf3f9cee2952eedef1c3a3a38eba63accdf21b813d6baa671adbeb62567de01`. The ignored `runs/task_2_1_quality/repeat_check.json` records the independent comparison. Focused synthetic tests passed 21/21 (`focused_junit.xml`); the complete regression suite passed 226/226 (`full_junit.xml`). Git tracked no raw files, virtual environment, or detailed audit output.
