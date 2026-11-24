# Part 2 Acceptance Review — 2026-11-15

**Later P4.2 study-design update (2026-11-20):** The five Part 2 implementation gates and their accepted outputs remain unchanged. `membership20261120a` now reconstructs a separate retrospective operational-activity cohort from audited effective structural dates; `labels20261120a` applies the official four-quarter outcome to that cohort. See `RETROSPECTIVE_COHORT_POLICY.md` and `LABEL_POLICY.md`. The 2026-11-15 decision below remains the historical assessment of the `pit20261115f` **candidate roster**; it is still not a verified real-time public active-bank register. Final report placement and the historical-vintage limitation remain open.

## Decision

The fifth checklist item's implementation gate **passed** for run `pit20261115e`: the accepted provisional 120-calendar-day rule was applied to the observed panel through prediction quarter 2025Q2, all 166 outputs passed independent verification, and the upstream files remained unchanged. The five written Part 2 checklist gates have evidence. **Part 2 is not yet ready to be described as a verified active-bank risk set or as fully resolved for the final report.** Historical activity and publication vintages remain unverified. The final report must present these limits rather than calling every retained candidate an active bank.

## Source and temporal contract

The authoritative assignment defines Financials through 2026Q2, modelling observations no later than 2025Q2, and the later failure target `(q, q + 4 quarters]`. This stage does not generate that target. It uses the 120-day provisional availability assumption from `PROJECT_PLAN.md` and `POINT_IN_TIME_RULE.md`: `REPDTE + 120 calendar days <= q`. The accepted source runs are `real20261111b` (raw), `treatment20261114a` (P2.1), `panel20261115b` (P2.2), and `repair20261115f` (P2.3). Their accepted manifests, evidence hashes, and physical files were checked before the run; 527 recorded upstream paths passed SHA-256 checks again afterward.

The population is an available-report **candidate** roster, not a confirmed roster of operating banks. A bank enters only after at least one of its Financials reports is provisionally available. The latest eligible report supplies 23 raw Financials predictor values. Repaired FDIC events are used only for dated exclusion disposition: verified failure with an as-of chronological bank link and `FAILDATE <= q` is excluded; conflicting or unresolved non-assistance event evidence is excluded as `unresolved_identity`; assistance alone is not an exit. The retrospective P2.3 `BANK_LINK_DISPOSITION` is not applied directly at earlier `q`, because later reports can change it. A null-CERT event cannot be assigned to a prediction bank. The chosen output retains `REPORT_AGE_DAYS` so stale candidates can be audited without falsely calling a reporting gap an exit.

## Measured prediction reconciliation

Unit: bank-quarter prediction candidates, except the two observed-q diagnostic columns (observed Financials rows). The complete 166-quarter English table is in ignored `runs/task_2_5_prediction/pit20261115e/report.md`; machine-readable counts are in `summary.json` beside it.

| Prediction scope | Candidates | Eligible for provisional feature selection | No eligible history in candidate roster | Excluded: verified prior failure | Excluded: unresolved identity | Observed-q reports, diagnostic | Observed-q reports without prior available history, diagnostic |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1984Q1–2025Q2, 166 quarters | 3,697,094 | 3,335,808 | 0 | 322,352 | 38,934 | 1,665,087 | 47,904 |
| 1984Q1 | 0 | 0 | 0 | 0 | 0 | 17,930 | 17,930 |
| 1984Q2 | 0 | 0 | 0 | 0 | 0 | 17,935 | 17,935 |
| 1984Q3 | 17,930 | 17,886 | 0 | 42 | 2 | 17,946 | 276 |
| 2023Q2 | 24,031 | 21,200 | 0 | 2,562 | 269 | 4,714 | 5 |
| 2025Q2 | 24,051 | 21,214 | 0 | 2,568 | 269 | 4,494 | 4 |

Candidate counts reconcile exactly to eligible plus the two exclusion reasons. `no_eligible_history=0` is structural under the available-report roster; the observed-q diagnostic quantifies the different population that would be introduced if the existence of a not-yet-published current-quarter report were used. It is never used to create candidates. No source value is imputed.

Across candidates, report age in days was min 181, p50 1,096, p90 9,586, max 15,066. Selected report dates ranged from 1984-03-31 through 2024-12-31. These long ages show why provisional feature availability must not be interpreted as proof of current bank activity. As a descriptive check at 2025Q2, 19,434 of 24,051 candidates had a selected report older than 365 days; 16,600 of those were in the `eligible` disposition. The 365-day threshold here is an audit illustration, **not** a verified exit rule or an exclusion criterion. The accepted data do not establish whether each of these banks was active at `q`.

## Point-in-time and physical checks

- Every output has unique `(CERT, q)` keys; all selected availability dates are `<= q`. The 23 predictor columns are explicitly allowlisted. Failure, assistance, exit, snapshot, and future-derived quality fields are outside predictors. The accepted raw Financials history through 2026Q2 was not shortened or modified.
- Each of 166 ZSTD Parquet outputs passed write-time row-level read-back, and the independent `verify` command checked schema marker, codec, row count, SHA-256, key uniqueness, 120-day boundary, report age and disposition counts. It checked all 527 upstream SHA-256 values again: `status=verified`, 166 quarters, 3,697,094 candidate rows.
- A bounded real-data perturbation at `q=2023-06-30` compared 100 banks from 2022Q4; it appended 100 reports dated 2023Q2, added a 2023-07-01 failure event, and changed `DQ_FLAGS`, `EXIT_CATEGORY`, `TRANSACTION_DISPOSITION`, and `INSTITUTION_ACTIVE_SNAPSHOT`. Population membership, selected `REPDTE`, availability date, report age, all 23 predictors, and risk-set disposition at `q` remained identical. This cannot establish actual historical release or revision vintages.
- Focused panel tests passed **56/56** and the full regression suite passed **306/306**. The earlier rule-only JUnit evidence passed **14/14**. Test XML and all detailed output paths are Git-ignored under `runs/task_2_5_prediction/pit20261115e/`.

## Individual gate review

| Checklist item / rubric | Verified evidence | Gate result | Remaining report or research work |
| --- | --- | --- | --- |
| 1 — P2.1 raw audit | `audit20261114b`: complete manifest, 172 inputs, four passing validations, matching `findings.json` SHA-256 | PASS | Place measured problems and denominators in the final English report. |
| 2 — P2.1 treatment | `treatment20261114a`: complete manifest, 172 outputs, four passing validations, matching summary SHA-256; source rows/keys retained | PASS | Place before/after counts and flag-only rationale in the report. |
| 3 — P2.2/P2.3 observed panel and exit repair | `panel20261115b`: 173 outputs, five passing validations; `repair20261115f`: two outputs, five passing validations and 4,117 reconciled events | PASS for documented evidence classification | Dated merger/other-exit and historically knowable active-status evidence are unavailable in the accepted snapshot. Do not promote ambiguous cases to verified exits. |
| 4 — P2.4 availability/as-of rule | `POINT_IN_TIME_RULE.md`, `panel/asof.py`, 14 passing synthetic fixtures including future perturbation | PASS | Historical release and report-vintage evidence remain unavailable. |
| 5 — P2.4 full application | `pit20261115e`: complete manifest, 166 outputs, four passing validations; independent `verify`; 56 focused and 306 full tests | PASS for the stated application gate | Report the stale candidate population and define an independently supportable active-risk-set policy before modelling labels. |

The initial `pit20261115a` run failed before output promotion because PyArrow inferred a partition directory's `REPDTE` type while reading a physical file. `pit20261115b` failed after 92 outputs when Windows denied a concurrent manifest replacement during monitoring. `pit20261115c` was deliberately stopped after 21 progress files on discovering a retrospective bank-link leakage risk; its original running manifest was preserved as `interrupted_manifest.json` and its final manifest records the failure. `pit20261115d` wrote 166 files but failed an exact list-child schema comparison (`item` versus Parquet `element`). All four failed attempts remain in separate ignored namespaces and are not accepted. Physical-file reading, per-quarter progress evidence, as-of link computation, and the explicit `element` schema were corrected before `pit20261115e` passed. No accepted upstream artifact was rewritten.

## Unresolved limits and overall status

**Overall Part 2 status: BLOCKED for a final claim that the prediction population contains only banks active at `q`.** The accepted Institutions endpoint is a later snapshot, and the present sources lack verified dated exits for many banks. A fixed 120-day lag also cannot recover the values of an earlier report vintage or its actual publication date. The output is a reproducible, point-in-time guarded **candidate** panel with explicit ages and exclusions; it is not yet a validated active-bank modelling risk set. The fifth checklist implementation gate is complete, while final report placement and this population decision remain open. No labels, split dates, feature fitting, or model training were produced.
# Population-policy follow-up (2026-11-15)

The PDF (page 1 dataset/target scope; page 2 Part 2) requires an observed
bank-quarter panel and point-in-time correct failure-risk work. It does not
name an activity-age cutoff. The accepted P2.4 candidate roster remains
unchanged; `pit20261115f` is the revision-corrected reference. A new,
separate `recent_reporter_4_quarters_v1` membership audit is saved under
ignored `runs/task_2_6_population/population20261115a/` and
`data_processed/task_2_6_population/population20261115a/`. It classifies a
recent-reporter **proxy** and retains stale activity-uncertain candidates.
It does not certify all banks active at `q` or select modelling features.

| Period and unit | Candidate bank-quarters | P2.4 baseline eligible | P2.4 prior-failure diagnostic | P2.4 identity diagnostic | Recent reporters, 4-quarter proxy | Stale, activity uncertain | Recent entrants vs prior q | Baseline eligible with report age >365 days |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1984Q3, bank-quarters | 17,930 | 17,886 | 42 | 2 | 17,930 | 0 | 17,930 | 0 |
| 2025Q2, bank-quarters | 24,051 | 21,214 | 2,568 | 269 | 4,617 | 19,434 | 5 | 16,600 |
| 1984Q1–2025Q2, bank-quarters | 3,697,094 | 3,335,808 | 322,352 | 38,934 | 1,695,045 | 2,002,049 | 24,056 entry/re-entry transitions | 1,657,082 |

At 2025Q2, the **21,214** baseline eligible candidates reconcile to
**4,614** recent reporters and **16,600** stale/uncertain candidates. Three
additional recent reporters carried a retrospective P2.4 exclusion
disposition; that diagnostic does not change the as-of recency decision.
The full 2025Q2 candidate roster of 24,051 reconciles to 4,617 recent and
19,434 stale. Sensitivity on all candidates: two-quarter window **4,560**,
four-quarter **4,617**, eight-quarter **4,729** recent reporters. These are
coverage counts, not verified open-bank counts. Overall distinct candidate
and ever-recent bank counts are each 24,051; summing quarter counts measures
bank-quarters, not distinct banks. The 24,056 entry/re-entry transitions
include returns after stale intervals.

The official [History source assessment](HISTORY_SOURCE_ASSESSMENT.md)
documents bounded live probes, identifiers, `EFFDATE` versus `PROCDATE`,
role filtering, coverage limits, and the lack of verified historical public
availability. No History event is applied to real membership. A synthetic
event can change a decision only if its effective date and separately
verified knowledge date are both no later than `q`; assistance is not an
exit. The accepted P2.4 failure/identity disposition remains audit metadata
because its public knowledge date was not verified. The 120-day report lag
also remains provisional. This repair does not modify the target window or
create labels.

Implementation evidence: the 166 population Parquet files in each of
`population20261115a` and `population20261115b` passed logical write/read
checks, one row per `(CERT, q)`, and independent verification of their schema,
ZSTD codec, SHA-256, and 170 protected P2.4 file/evidence hashes. The P2.4
source verifier also checked 527 earlier upstream hashes before each run.
All 166 output SHA-256 values, 166 quarter statistics, and overall counts
were identical between runs. Focused as-of/prediction/population tests passed
40/40; the complete pytest suite passed 322/322. Git tracked no raw, derived,
run, or environment files. The accepted P2.4 bounded real perturbation
already compared 100 banks, all 23 predictors, and future report/exit/snapshot
changes; the new population tests exercise the same `classify()` path used
by the full runner for future and conflicting evidence. Exact observed
commands and results are saved in ignored
`runs/task_2_6_population/verification_20261115.json`.
The five existing Part 2 checklist gates retain their PASS decisions: P2.1
audit/treatment, P2.2 observed panel, P2.3 documented event classification,
and P2.4 as-of rule/application. **The active-bank modelling population is
still BLOCKED.** The user must decide whether an explicitly scoped
recent-reporter study is acceptable or whether complete, historically
knowable active-status evidence must be obtained and reconciled. Final
English report placement of P2.1–P2.4 is still pending.

## Scope decision update — 2026-11-16

The user selected the active-bank objective. The recent-reporter proxy is
not approved as the modelling population. The subsequent bounded official
source review in `ACTIVE_MEMBERSHIP_RESEARCH.md` found dated retrospective
Institution fields but no complete historical public-knowledge vintages;
the current Institutions index also differs from the accepted snapshot.
The active-bank membership gate and production labels remain BLOCKED. This
update does not alter the accepted Part 2 implementation evidence above.
