# P2.4 Report Availability and Point-in-Time Selection Rule

## Scope and source contracts

This fourth Part 2 milestone specifies and tests a reusable **selection rule**. It does not scan the 170-quarter panel, assign modelling labels, decide the final risk set, fit preprocessing, or train a model. The immutable P2.2 `panel20261115b` provides observed Financials bank-quarter rows with unique `(CERT, REPDTE)` keys and source-missingness provenance. Repaired P2.3 `repair20261115f` provides retrospective failure/assistance and last-report exit evidence. Neither its transaction dispositions nor its exit categories enter this module's predictors or population builder.

The [assignment PDF](final_report_bft65b.pdf) defines a prediction quarter `q` and the later failure target `(q, q + 4 quarters]`; it also limits modelling observations to 2025Q2. This module neither creates nor shifts that target. Its purpose is to prevent a Financials statement dated by `q` from being treated as if it had already been released by `q`.

## Four distinct dates

| Date | Meaning | Rule here |
| --- | --- | --- |
| `REPDTE` | Financial statement's report quarter-end. | It orders source reports; it is not a publication date. |
| Availability date | Earliest date on which a report is provisionally allowed as a feature. | `REPDTE + 120 calendar days`; if a separately verified revision publication date is later, use the later date. The 120-day part remains an assumption. |
| Prediction date `q` | Quarter-end at which a bank would be ranked. | A source version is eligible only if availability date `<= q`. |
| `FAILDATE` | FDIC outcome event date for later target construction. | Never read in this join or predictor allowlist. The official target remains `(q, q + 4 quarters]`. |

For example, at `q=2023-06-30`, a `REPDTE=2022-12-31` report reaches its provisional availability date on `2023-04-30` and can be selected. A `REPDTE=2023-03-31` report reaches that date on `2023-07-29` and cannot be selected at 2023Q2. The 120 days are **calendar days**, not four months or four quarters.

The accepted FDIC snapshot does not carry a verified per-bank historical publication timestamp or an archive of report vintages. An apparently old report may contain a revision made after `q`. Thus the rule enforces a conservative lag on observed statements but cannot prove that their exact values were public at `q`. A later task must seek release/vintage evidence if available, document this residual uncertainty, and run development-only lag sensitivity. A verified revision date only orders versions; it does not turn the provisional base lag into a verified release date.

## Population is separate from feature selection

`select_asof(population, reports)` requires explicit `(CERT, q)` population requests. Each request carries a population source, an evidence identifier, and the date that source became known; dates after `q` are rejected. Duplicate population keys are rejected. A `verified_registry` request may be supplied only by a future independently checked point-in-time registry. If that bank has no eligible Financials history, the result is `no_eligible_history` with `predictors=None`; no zero, imputation, or fabricated report is created.

`population_from_available_reports(q, reports)` provides a conservative **candidate** roster when no registry is supplied: include a CERT only if at least one Financials version has become eligible under the availability rule by `q`. The earliest eligible availability date becomes the membership evidence date. A quarter-`q` report that is not yet eligible cannot create a prediction at `q`, and a report added later cannot create a past roster member. This roster does not prove that every included bank was still open at `q`: a stale report can remain after an unobserved exit. Retrospective P2.3 exits, `ACTIVE`/`CHANGEC1` in the current Institutions snapshot, and future failure events must not be used to backfill membership. A future production risk-set stage must either verify dated, knowable-at-`q` exit evidence or explicitly retain and flag stale candidates; this milestone does not choose between those policies. The maximum modelling `q=2025-06-30` is a downstream observation-scope gate, not a reason to use later reports in this join.

## Deterministic source selection and revisions

For each population row, consider only reports with the same `CERT`. Require `REPDTE + 120 days <= q`. For a version with a documented and **verified** publication date, use `max(REPDTE + 120 days, verified publication date)`; require it to be `<= q`. Select the largest eligible `REPDTE`. If that bank-quarter has multiple verified versions, select the latest version whose publication date is no later than `q`.

First determine each version's effective availability: `max(REPDTE + 120 calendar days, verified publication date)` when a verified date exists, or the provisional 120-day date for an undated version. Discard versions with effective availability after `q` **before** checking version ambiguity. Among versions eligible at `q`, multiple versions of one `(CERT, REPDTE)` require distinct verified publication dates; an undated version paired with another eligible version, or tied eligible dates, triggers `AmbiguousRevisionError`. The code never assigns a publication date to an undated original or chooses a version by row order, source ID, or later observed values. A future-dated revision cannot change an earlier population or selection, even when the undated original is already provisionally eligible. The accepted P2.2 panel has one row per bank-quarter, so no version tie is observed there; the absence of stored versions does **not** prove that the retained values are the historical vintage.

The repair was reproduced before code changes: for `REPDTE=2022-12-31`, the undated panel original became provisionally available on 2023-04-30. At `q=2023-06-30`, adding a same-quarter revision verified as published on 2023-07-01 incorrectly raised `AmbiguousRevisionError` in `population_from_available_reports()`. The expected earlier roster, source report, and predictor values were unchanged. The failing pre-fix JUnit is in ignored `tmp/p24_revision_repro_before.xml` (one failed test). The repaired rule passes that case through both `population_from_available_reports()` and `select_asof()`. A revision published exactly at `q` is eligible; an undated original plus an already-eligible revision, or two eligible revisions tied on publication date, still fails closed through both entry points. The inclusive `<= q` boundary and the official `(q, q + 4 quarters]` outcome target are unchanged.

The result records `source_repdte`, assumed/effective availability date, `report_age_days = q - REPDTE`, availability basis, and source/provenance diagnostics separately from the predictors. If no report is eligible for an externally evidenced population row, those source fields and the predictor dictionary are `None`. No imputation or missingness-based row removal occurs.

## Predictor boundary

The explicit raw Financials predictor allowlist contains 23 verified numeric fields: `ASSET`, `DEP`, `EQ`, `NETINC`, `ROA`, `NCLNLS`, `LNATRES`, `RBC1AAJ`, `CHBAL`, `SC`, `BRO`, `DEPDOM`, `EINTEXP`, `LNLSGR`, `LNRE`, `LNCI`, `P3ASSET`, `P9ASSET`, `NAASSET`, `NTLNLS`, `ELNATR`, `NIMY`, and `RBCRWAJ`. These are eligible **source columns**, not a fitted or final feature selection. Values and nulls are copied as-is; field-specific missingness and unit issues remain for later feature decisions.

`CERT`, `REPDTE`, `STNAME`, and source `ID` are keys or diagnostics, not predictors. `SOURCE_MISSING_FIELDS` is retained only as selected-report provenance. `DQ_FLAGS`, `FAILDATE`, `COST`, `RESTYPE`, `RESTYPE1`, bank-link dispositions, exit assessments, `ACTIVE`, `CHANGEC1`, snapshot-derived flags, and any future-derived join flags are excluded from predictor values. The adapter for a P2.2 row copies only the allowlisted predictor columns, even if those audit fields are present in the input mapping. The join reads no Failures, Institutions, or exit table.

## Synthetic acceptance matrix and evidence

The focused suite `tests/test_panel_asof.py` checks the calendar-day lag; eligibility one day before, on, and after a prediction quarter-end using dated revisions; latest of two eligible quarters; no eligible history; bank isolation; a later quarter's report not creating earlier population membership; population date/source validation; undated and tied versions failing closed; a later verified revision changing only a later prediction; and a perturbation that changes future reports, exit/assistance fields, Institutions snapshot fields, and `DQ_FLAGS` without changing the earlier roster, selected report, or predictor dictionary. It also checks that explicit nulls remain null and provenance stays diagnostic.

The fixture evidence is the ignored JUnit XML at `runs/task_2_4_asof/fixture_tests.xml`. These fixtures prove the implemented temporal boundaries, not the unpublished historical release/vintage times of the real FDIC statements. The accepted reference manifests and summaries were read as contracts, without scanning or altering their data files:

One accepted P2.2 Financials Parquet footer was also read without materializing records: it contains 29 physical columns (the 26 selected FDIC fields plus `ID`, `SOURCE_MISSING_FIELDS`, and `DQ_FLAGS`); `CERT` is `int64`, `REPDTE` is `date32[day]`, and the two provenance/quality fields are string lists. This matches the adapter's input contract, but is not a new 170-file data acceptance run.

| Accepted evidence | Status | SHA-256 |
| --- | --- | --- |
| `runs/task_2_2_panel/panel20261115b/run_manifest.json` | complete | `9B2CACA86A834C0A5D71474CBF88D465C8E1D8BC88B916AB6CB1BDB9C13D4CD0` |
| `runs/task_2_2_panel/panel20261115b/summary.json` | complete | `E19BBEFF861F911261CAEB8A6535F75E5937ED391CD13A2C22539DEA6B9AC8DF` |
| `runs/task_2_3_transaction_repair/repair20261115f/run_manifest.json` | complete | `35C72BD625F28F457A70CCAAE982ACCB034A827576C851029FA869853FAE74A9` |
| `runs/task_2_3_transaction_repair/repair20261115f/summary.json` | complete | `2CD4A449C2CAFFE05F12D845A799B0FB3C951B36426D3F4976ABA1541E89BCC3` |

On Python 3.11.9 with pytest 8.3.4, focused synthetic tests passed **14/14** (`fixture_tests.xml`: 14 tests, zero failures/errors/skips; SHA-256 `A913910D6CC9FFD333F24CDA8B642BFF82A334EEAB778AFC68203053EF01F6EF`). The relevant P2.2/P2.3/P2.4 suite passed **46/46**. The complete project regression passed **296/296** (`regression_tests.xml`: 296 tests, zero failures/errors/skips; SHA-256 `ED81E6CB0ACCCAE7583EB3DAD4093853017520F4B7F8A7F26ECCA408445A1E1B`). These are synthetic rule and regression results, not a full-panel PIT join or historical vintage audit.

## Reproduction

From `C:\NEU\BFT65B` in PowerShell:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
$testTmp = 'C:\Users\asus\.codex\visualizations\2026\11\06\01a1b056-7f64-77a1-84dc-0605edf8aa3a\pytest_p24_asof_evidence'
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp $testTmp --junitxml runs\task_2_4_asof\fixture_tests.xml tests\test_panel_asof.py
$testTmp = 'C:\Users\asus\.codex\visualizations\2026\11\06\01a1b056-7f64-77a1-84dc-0605edf8aa3a\pytest_p24_panel_regression'
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp $testTmp tests\test_panel_build.py tests\test_panel_repair.py tests\test_panel_asof.py
$testTmp = 'C:\Users\asus\.codex\visualizations\2026\11\06\01a1b056-7f64-77a1-84dc-0605edf8aa3a\pytest_p24_full'
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp $testTmp --junitxml runs\task_2_4_asof\regression_tests.xml
```

The `--basetemp` location is short to accommodate existing Windows synthetic tests. Use a new temp directory name for a repeat. The original rule-only milestone had no full-data join command or output; the later application and repair evidence are recorded below.

## 2026-11-15 revision-selection repair evidence

The original pre-fix regression failed **1/1** as expected with `AmbiguousRevisionError` at `q=2023-06-30`; ignored JUnit: `tmp/p24_revision_repro_before.xml`. After filtering future-unavailable versions before ambiguity checks, focused as-of/prediction tests passed **26/26**, the final focused suite including the keyed comparator passed **29/29**, and the final full regression passed **311/311**. JUnit files are `tmp/p24_revision_focused.xml`, `tmp/p24_revision_final_focused.xml`, and `tmp/p24_revision_final_regression.xml`.

The corrected production command used a fresh run ID, `pit20261115f`. Its `verify` command passed 166 Parquet read-backs and **527** upstream SHA-256 checks. The independent keyed comparison with immutable `pit20261115e` covered **3,697,094** shared `(CERT, q)` rows over 166 quarters. Previous-only and current-only keys were **0** each; disposition, selected `REPDTE`, availability date, report age, and every one of the 23 predictor fields had **0** differences. Machine-readable per-quarter and per-field counts: ignored `runs/task_2_4_revision_repair/pit20261115e_vs_pit20261115f.json`. The original run's 166 Parquet files plus four evidence files retained all **170** pre-run SHA-256 values (`tmp/p24_revision_protected_hashes_before.json`). The accepted snapshot contains one version per bank-quarter, so identical real-data results are expected but were measured; synthetic revision behavior is the substantive change. No population-policy, failure-label, or target-window change was made.

Reproduce the repair checks in PowerShell from `C:\NEU\BFT65B` with a fresh short `--basetemp` directory for test reruns:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_panel_asof.py tests\test_panel_prediction.py tests\test_panel_revision_compare.py
.\.venv_local\Scripts\python.exe -m bft65b.panel.prediction show --project-root C:\NEU\BFT65B --run-id pit20261115f
.\.venv_local\Scripts\python.exe -m bft65b.panel.prediction verify --project-root C:\NEU\BFT65B --run-id pit20261115f
.\.venv_local\Scripts\python.exe -m bft65b.panel.compare_revision --project-root C:\NEU\BFT65B --previous-run pit20261115e --current-run pit20261115f --output C:\NEU\BFT65B\runs\task_2_4_revision_repair\another_fresh_comparison.json
```

The comparison output path must be new; the CLI refuses to overwrite existing evidence. The production run command for a further fresh run is documented in `README.md`. The prior `pit20261115e` evidence is a historical accepted checkpoint, not overwritten by this repair.
# Prediction population refinement (2026-11-15)

The assignment PDF, page 1, specifies Financials as observed bank-quarter
reports, Failures as the event authority, Institutions as a bank/snapshot
source, the 1984Q1–2026Q2 Financials range, and the unchanged later failure
target `(q, q + 4 quarters]`. Page 2 requires a clean bank-quarter panel and
point-in-time correct analysis. It does not prescribe a numerical activity
cutoff. These are three different units:

1. **Observed Financials panel:** a bank-quarter exists only when a Financials
   report exists; denominator is distinct `(CERT, REPDTE)` keys.
2. **P2.4 candidate roster:** a `(CERT, q)` exists after at least one report is
   provisionally available by `q`; denominator is candidate bank-quarters.
   A current-quarter report published later cannot create membership at `q`.
3. **Prediction population:** a bank must be plausibly operating and assessable
   at `q`. Neither prior reporting alone nor the later Institutions snapshot
   verifies that status. `panel/population.py` therefore produces a scoped
   *recent-reporter proxy* and retains stale candidates as activity-uncertain.
   It does not certify an active-bank population.

The primary proxy starts at the first provisionally available report and
includes a candidate when its latest selected report is no more than four
quarter-end steps before `q`. This is a declared coverage rule based on
quarterly reporting and the provisional 120-calendar-day publication delay,
not an FDIC closure rule. At `q`, the newest normally selectable quarter may
be two quarter-ends old. A four-quarter window permits a temporary reporting
gap; two- and eight-quarter alternatives quantify coverage sensitivity. A
stale bank remains in the output as `activity_uncertain_stale` rather than
being called closed. A later resumed report can return it to `recent_reporter`.
The policy does not inspect labels, model scores, or an intended sample size.

The official FDIC [History API dictionary](https://api.fdic.gov/banks/docs/history_properties.yaml)
defines `EFFDATE` as effective date and `PROCDATE` as the date a change/event
is processed. `CERT` identifies the institution; `ID` identifies a source
row; `CHANGECODE` is an activity event code. The [Bank Structure Changes
interface](https://banks.data.fdic.gov/bankfind-suite/oscr) offers effective-
and processed-date searches. The [FDIC Bank Data Guide](https://www.fdic.gov/system/files/2024-06/guide.pdf)
lists the History endpoint as weekly, with the application shown from 1999
and a footnote that bank structure changes reach back to 1970. A one-record
official API probe reported 585,705 History rows in the current index;
bounded `ORG_ROLE_CDE:FI`/effective-year probes reported 4,126 rows for 1984
and 704 for 2025, both at index `history_20261113100007`. These totals are
**not** a complete event/identifier/coverage audit. The first unfiltered
sample was a branch event (`ORG_ROLE_CDE=BR`), demonstrating why all History
rows cannot be treated as bank entry/exit events. No historical public
release timestamp or archived as-of vintage was verified. `PROCDATE` is not
silently promoted to public knowledge time. No History event is used to decide
real-snapshot membership in this policy version.

For future verified event evidence, `DatedEvent` separates effective date
from `known_at`. Only a documented, verified `known_at <= q` and effective
date `<= q` may exclude a bank. A same-day known failure/merger/other exit
excludes; a next-day event does not. Assistance does not establish an exit.
Conflicting known events leave identity unresolved. The accepted P2.4
`ELIGIBILITY_DISPOSITION` is retained as a diagnostic, not reused as
as-of membership evidence, because historical failure publication timing is
unverified. Multiple events cannot duplicate `(CERT, q)`.

The 120-day report lag and current source values do not reconstruct statement
vintages. The four-quarter outcome definition stays `(q, q + 4 quarters]`;
this population stage creates no labels. Until official, complete and
historically knowable entry/exit evidence is reconciled, any claim that this
proxy equals all banks active at `q` remains blocked. The user must decide
whether the narrower, explicitly labelled recent-reporter research population
is acceptable for the modelling objective or whether to obtain/validate a
historical active-bank register first.
