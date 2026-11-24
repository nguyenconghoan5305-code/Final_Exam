# P2.3 Failure and Assistance Transaction Repair

## Authority, snapshot, and matching rule

The examination PDF requires failures to be distinguished from mergers and other exits. The accepted `real20261111b` Failures and Assistance snapshot has 4,117 source records. The earlier `panel20261115b` is an immutable P2.2 panel and retrospective P2.3 baseline; its 2,775 `confirmed_failure` bank assessments did not yet use transaction type.

The [FDIC Failure field dictionary](https://api.fdic.gov/banks/docs/failure_properties.yaml) defines `RESTYPE` (Resolution) with `Failure` and `Assistance` values and `RESTYPE1` (Transaction Type) with resolution methods. The [FDIC Failures help](https://banks.data.fdic.gov/bankfind-suite/failures/help) explains that assistance often leaves the charter open, whereas purchase-and-assumption and payoff close the failed institution's charter. A failed bank can be acquired through P&A; that is a failed-bank resolution, not evidence of an ordinary merger. The [FDIC Resolutions Handbook](https://www.fdic.gov/bank-failures/resolutions-handbook.pdf) describes a Deposit Insurance National Bank (DINB) as a vehicle for insured deposits of a failed institution. FDIC's dictionary text includes ABT under IDT, although ABT is absent from its current enumerated values.

On 2026-11-14 22:29:05 UTC, the official `/banks/failures` endpoint was queried once with `fields=CERT,FAILDATE,COST,RESTYPE,RESTYPE1&limit=10000&offset=0&sort_by=CERT&sort_order=ASC&format=json` and no filter. The exact encoded URL, response bytes, retrieval time, source index, and dictionary checksum are in ignored `runs/task_2_3_transaction_repair/source_capture.json`. The response SHA-256 is `1a07e288dec8254881f22c13d2646dc79fcf0ed67803592879e9f4abcae53977`; the dictionary SHA-256 is `7e3137fb6fc93dd49371b4d032f48fad403efd3d9a7f5cc8ca5480c00807364c`. `meta.total` and downloaded rows were both 4,117, under the verified single-page limit. The source index `failures_1787667198788` with timestamp `2026-08-25T14:13:18Z` matched the accepted acquisition manifest.

The supplementary result had 4,117 unique nonempty `ID` values. All 4,117 matched accepted source IDs one to one. `CERT`, normalized `FAILDATE`, and `COST` agreed for every matched ID; duplicate IDs, extra IDs, missing IDs, and corroboration conflicts were all zero. `ID` remains an empirically verified source-record key, not a bank identifier. No CERT/date fallback join is allowed. Null CERT source events remain intact.

All six retrieved fields (`ID`, `CERT`, `FAILDATE`, `COST`, `RESTYPE`, `RESTYPE1`) were present in every response row. `RESTYPE` and `RESTYPE1` had zero explicit nulls; `CERT` had 488 and `COST` had 637 explicit nulls, matching the accepted source profile.

## Three independent decisions

1. **Transaction:** `RESTYPE=FAILURE` plus PA, P&A, PI, IDT, ABT, PO, DINB, or MGR yields `verified_failure`. The FDIC dictionary places MGR in Category 2 and describes FSLIC management takeover; the effective date marks cessation as a privately held going concern, even if final sale occurred later. REP is Category 1 reprivatization, so its two FDIC failure-status events remain `failure_charter_uncertain` for last-report exit. `RESTYPE=ASSISTANCE` is retained separately. `OBAM` appears in the live source but is absent from the current dictionary enum; it is identified as `open_bank_assistance_code_unverified`, without asserting a verified code definition. Absent, unknown, or contradictory codes remain unresolved.
2. **Bank linkage:** the accepted `BANK_LINK_DISPOSITION` is preserved separately. A verified failure is eligible for a bank assessment only with a `chronological_candidate` link (matching CERT, at least one report on/before FAILDATE, and no later report). The residual uncertainty in certificate histories and report availability is documented; this audit does not create modelling labels.
3. **Last observed report:** one qualifying failure after the last observed REPDTE and by 2026-06-30 confirms a failure assessment, provided there is no conflicting event. One independently dated ordinary merger, with no competing failure/assistance event, confirms a merger. Evidence and conflicts are evaluated before applying the dataset-boundary fallback. Assistance alone never proves an exit. A last report on 2026-06-30 without qualifying exit evidence remains `boundary_observed`. Temporary reporting gaps remain separate and unchanged.

## Measured event reconciliation

Unit and denominator: 4,117 accepted source IDs. Transaction status and bank link are separate axes.

| Transaction disposition | Source events |
| --- | ---: |
| Verified FDIC failure transaction with documented method | 3,522 |
| Assistance transaction (A/A or REP) | 580 |
| Assistance with OBAM code, method definition unverified | 13 |
| FDIC Failure with REP; charter outcome uncertain | 2 |
| Unknown or unmapped transaction | 0 |
| Missing, duplicate, extra, or conflicting supplemental ID | 0 |

Of the 3,522 verified failure transactions, 2,572 have a chronological candidate bank link, 485 have null CERT, 195 have no matching Financials CERT, and 270 have reports after the event. The 950 cases outside the chronological candidate category are retained as unresolved bank-link cases; their sum is not a bank count. Some chronological candidates fall after the observed dataset boundary or conflict with other events, so 2,572 is not the number of confirmed bank exits.

## Bank-category transition

Unit and denominator: 24,064 distinct observed Financials CERTs. Counts are retrospective last-report assessments, not bank-quarter labels.

| Category | Previous banks | Repaired banks |
| --- | ---: | ---: |
| Confirmed failure | 2,775 | 2,534 |
| Documented merger | 0 | 0 |
| Documented other exit | 0 | 0 |
| Ambiguous exit | 16,976 | 17,217 |
| Boundary observed | 4,313 | 4,313 |

| Previous category | Repaired category | Banks |
| --- | --- | ---: |
| Confirmed failure | Confirmed failure | 2,534 |
| Confirmed failure | Ambiguous exit | 241 |
| Ambiguous exit | Ambiguous exit | 16,976 |
| Boundary observed | Boundary observed | 4,313 |

The 241 baseline confirmed banks that changed category are assistance transactions. All 2,775 baseline confirmed banks are accounted for (2,534 + 241). **Zero banks entered** `confirmed_failure`. Among the 4,313 banks last observed on the boundary date, zero changed category in this snapshot; synthetic boundary-day fixtures still demonstrate the corrected precedence for a failure, merger, assistance, and conflict on that exact date.

## Outputs, checks, and limitations

Final repair run: `repair20261115f`. Ignored evidence: `runs/task_2_3_transaction_repair/repair20261115f/{run_manifest.json,summary.json,metadata.json,report.md}`. Ignored data: `data_processed/task_2_3_transaction_repair/repair20261115f/{failure_events,exit_assessments}/part-000.parquet`. The event output retains every accepted source column and appends the two FDIC transaction fields, match disposition, transaction disposition, and rule. Both output files passed ZSTD Parquet read-back and SHA-256. The accepted 170-quarter panel, event/gap outputs, and four prior-run evidence files were rehashed before and after; all 177 hashes remained unchanged. The 1,682,616 accepted bank-quarter rows and keys remain byte-identical across 170 original files. Earlier repair attempts `repair20261115a` through `repair20261115e` remain in their ignored namespaces as development evidence; runs `a` through `d` have superseded MGR counts, while `e` has the same data result as `f` but an outdated report sentence.

The repaired event Parquet SHA-256 is `1c701397538bf62849a2664aefa688dd3c56d5fa060336949ca643a626c17f47`; the exit-assessment SHA-256 is `9356344af2bb7aff896a7bf4fa96c8fc3cbb3cf43190fddc506f8b8503910d85`. The final manifest records five passing validations and `status=complete`. An independent post-run check rehashed all 177 protected artifacts and both outputs; all checks agreed with the saved evidence.

`RESTYPE1=OBAM` is not in the current FDIC dictionary enum, so its detailed method meaning remains unverified even though FDIC explicitly labels all 13 records `ASSISTANCE`. The official A/A description includes both assistance to an acquirer and open-bank assistance, so the 579 A/A records cannot all be labelled open-bank cases from this code alone. The two `REP` failure-status events need dated charter-history corroboration to establish a last-report exit. No dated ordinary merger or other-exit source was acquired in this repair; their measured confirmed counts remain zero. Snapshot `ACTIVE` and `CHANGEC1` remain audit-only. Future point-in-time joins and four-quarter labels must use a separately justified information-availability rule.

The [FDIC's 1980s historical account](https://www.fdic.gov/history/1980-1989) notes that it sometimes counted open-bank assistance as a failure in historical statistics. This repair defines **confirmed last-report failure exit** conservatively using the source `RESTYPE` and charter-resolution evidence; it does not settle the later modelling-target policy for assistance transactions.

The focused synthetic suite passed 32/32 and the full regression suite passed 282/282 on the documented local Python environment. Synthetic tests cover failure versus assistance, unknown and conflicting supplementary evidence, failed-bank P&A versus ordinary merger, and boundary-day precedence. The run's machine-readable `summary.json` contains exact transition rows and zero counts.

## Reproduction commands

The original source capture was a single `requests.get` of the exact URL recorded in `source_capture.json`, with `(10, 90)` second connect/read timeouts. Its response bytes and the official dictionary bytes are retained for deterministic reruns. The repair and final tests were executed from `C:\NEU\BFT65B` in PowerShell:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.panel.repair run --project-root C:\NEU\BFT65B --source-json C:\NEU\BFT65B\runs\task_2_3_transaction_repair\source_failures_20261115.json --run-id repair20261115f
.\.venv_local\Scripts\python.exe -m bft65b.panel.repair show --project-root C:\NEU\BFT65B --run-id repair20261115f
$p = 'C:\Users\asus\.codex\visualizations\2026\11\06\01a1b056-7f64-77a1-84dc-0605edf8aa3a\pytest_p23_mgr_focus'
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp $p tests\test_panel_build.py tests\test_panel_repair.py
$p = 'C:\Users\asus\.codex\visualizations\2026\11\06\01a1b056-7f64-77a1-84dc-0605edf8aa3a\pytest_p23_mgr_full'
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp $p
```

The shown `run` command is the actual invocation and cannot overwrite its existing ID; use a fresh ID for another attempt. The short `--basetemp` path avoids a known Windows long-path issue in unrelated older synthetic tests.
