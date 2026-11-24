# P2.2/P2.3 Observed Panel and Exit Evidence

> **Historical baseline:** This document records immutable run `panel20261115b`. Its 2,775 `confirmed_failure` bank assessments predate FDIC transaction-type verification. The repaired P2.3 result is `repair20261115f` with 2,534 confirmed failures; see [PANEL_TRANSACTION_REPAIR.md](PANEL_TRANSACTION_REPAIR.md). The observed 170-quarter panel and previous evidence have not been rewritten.

## Scope and input contract

The selected accepted P2.1 source is `treatment20261114a`. Its complete manifest, metadata, summary SHA-256, 172 output paths, per-file SHA-256 values, row counts, quality marker, and ZSTD footers were checked before the panel build. The new manifest records the selected treatment run and all 172 treated-input checksums. The accepted Task 1.5 raw files and P2.1 artifacts remain unchanged.

Financials is the sole panel backbone. Each saved bank-quarter row preserves the treated Financials values, `SOURCE_MISSING_FIELDS`, and `DQ_FLAGS`. The panel contains observed `(CERT, REPDTE)` pairs only. Reporting gaps are saved separately and do not create null or zero bank-quarter rows. Event linkage, last-report exit assessments, and Institutions snapshot fields are separate **retrospective audit outputs**. They are not point-in-time features or modelling labels.

## Panel reconciliation

Unit: observed bank-quarter rows and distinct `(CERT, REPDTE)` keys. The full English 170-quarter table is in ignored `runs/task_2_2_panel/panel20261115b/report.md`; the machine-readable table is `summary.json` beside it.

| Period | Input rows | Input keys | Output rows | Output keys | Input duplicates | Output duplicates | Missing keys | Added keys |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1984Q1–2026Q2, 170 quarters | 1,682,616 | 1,682,616 | 1,682,616 | 1,682,616 | 0 | 0 | 0 | 0 |

All 170 individual quarters passed the same key and value-preservation checks, and every saved Parquet file passed atomic-write read-back and SHA-256 checks. The 10,996 historical Financials rows whose CERT is absent from the current Institutions snapshot remain in the panel with their quality flag. This snapshot mismatch is not a reason to delete a historical report.

## Failure source-event reconciliation

Unit: distinct Failures source `ID` records; denominator **4,117 source events**. One input source ID produces exactly one saved event. An event may have several earlier Financials reports without being duplicated. Non-null CERT appears in 3,629 events, spanning 3,585 distinct CERT values; these are different units and should not be added to the event categories.

| Bank-link disposition | Source events | Denominator | Interpretation |
| --- | ---: | ---: | --- |
| Chronological candidate | 2,814 | 4,117 | At least one `REPDTE <= FAILDATE` and no later report; report availability is unverified. |
| Post-event report conflict | 439 | 4,117 | The same CERT has a Financials report after `FAILDATE`; bank linkage needs review. |
| Future reports only | 30 | 4,117 | CERT exists only in reports after the event date. |
| Unmatched CERT | 346 | 4,117 | Non-null failure CERT has no Financials CERT in the accepted range. |
| Missing CERT | 488 | 4,117 | Source event remains keyed by ID; no CERT is invented. |

Across all dispositions, **3,220** events have multiple earlier Financials reports. This is an expected many-to-one join hazard, not 3,220 additional failure events. The saved event table retains source `ID`, `FAILDATE`, `CERT`, COST, provenance, quality flags, disposition, report counts, and evidence rule.

## Last-report exit assessments

Unit: distinct Financials `CERT` banks; denominator **24,064 observed banks**. This table evaluates the last observed report for each bank, not all 1,682,616 bank-quarters.

| Category | Banks | Denominator | Evidence rule |
| --- | ---: | ---: | --- |
| Confirmed failure | 2,775 | 24,064 | One linked FDIC Failures source event dated from the last report through the dataset boundary, with no later report or conflicting event. |
| Documented merger | 0 | 24,064 | No accepted dated merger-event source was ingested. |
| Documented other exit | 0 | 24,064 | No accepted dated other-exit source was ingested. |
| Ambiguous exit | 16,976 | 24,064 | Last report before the boundary without sufficient dated evidence, or conflicting/nonqualifying evidence; this is **not** a confirmed exit. |
| Observed at dataset boundary | 4,313 | 24,064 | Last Financials report is 2026Q2; dataset end alone does not establish an exit. |

The gap audit found **33** resumed reporting gaps in **32** banks, covering **92** missing intervening quarter positions. The banks resume reporting, so a temporary missing quarter does not establish an exit. No gap row is inserted into the panel. Detailed CERTs, source IDs, dates, and event references remain only in ignored Parquet and run evidence.

## Source authority and limits

The [official FDIC Institutions field dictionary](https://api.fdic.gov/banks/docs/institution_properties.yaml) identifies `CHANGEC1` as a structural-event code; the accepted Institutions selection also includes `ACTIVE`, but no dated structure-change history or verified code-to-exit mapping. The [FDIC Bank Structure Changes service](https://banks.data.fdic.gov/bankfind-suite/oscr) distinguishes effective from processed dates, and the [FDIC API documentation](https://api.fdic.gov/banks/docs/) exposes a separate History endpoint. **Inference:** the selected Institutions snapshot fields cannot establish a merger, another exit reason, or the applicable event date. `CHANGEC1` and `ACTIVE` are therefore retained only as snapshot audit fields, never used to assign a real exit category. Synthetic dated-event fixtures demonstrate the classifier's documented merger and other-exit branches without claiming real cases.

`FAILDATE` and source `ID` from the accepted Failures table are the event authority. CERT equality and report chronology are linkage evidence, but do not establish the report's public availability or resolve all historical identity conflicts. The exit table's `EVIDENCE_JSON` retains source, identifier, date, and rule for each candidate; Institutions snapshot flags are named as audit-only. The assignment's four-quarter target and the provisional `REPDTE + 120 days` feature-availability rule remain separate, unimplemented milestones.

## Reproduction and verification

README.md gives the exact PowerShell build and show commands. Final attempt `panel20261115b` has a complete manifest with 172 inputs, 173 outputs, and five passing validations. The first attempt `panel20261115a` is preserved. All 173 output Parquet SHA-256 values match across attempts; the first report omitted explicit zero rows for merger and other exits, which the final report includes. The 172 treated inputs and the 172 upstream accepted raw inputs were independently rehashed after the final run and matched their manifests. Focused synthetic tests passed 12/12; full pytest passed 262/262. The ignored `runs/task_2_2_panel/repeat_check.json` records the repeat comparison. Data and detailed IDs remain outside Git.
