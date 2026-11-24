# Part 4.2 Four-Quarter Label Rule and Population Decision Gate

## 2026-11-20 production decision: retrospective cohort accepted

The user-selected active-bank objective is implemented as the historical study cohort in `RETROSPECTIVE_COHORT_POLICY.md`. The authoritative PDF (page 1 §2.3; page 2 Part 2/4) requires a four-quarter failure outcome and point-in-time predictors, but does not require a complete public Institutions vintage at every historical prediction date. We therefore use verified effective activity dates **retrospectively for cohort reconstruction** and the accepted 120-day Financials rule for **predictors**. This distinction does not assert that a live system possessed the later registry at `q`. The older global `complete_register_known_at_q` fixture is not bypassed or set to true.

Run `membership20261120a` resolved 23,925/24,064 observed CERTs to supported operational intervals and retained 139 unresolved identities/dates. It produced 1,660,450 eligible, 2,298,952 ineligible and 35,222 uncertain bank-quarter decisions across 166 q. Of the eligible decisions, 54,585 lack as-of feature history. The ignored 167-file run passed saved-output schema, key, count, codec, SHA-256 and upstream-hash verification. The source snapshot and the old `pit20261115f` roster remain unchanged.

Run `labels20261120a` applies the unchanged `q < FAILDATE <= q + 4 calendar quarters` target through q=2025Q2, using all 4,117 repaired Failures/Assistance source records. A positive requires a verified FDIC failure transaction and the accepted P2.3 chronological bank link. A verified failure on or before q excludes the bank even if a structural snapshot appears active. Assistance alone is not failure or closure. An ordinary merger or other non-failure exit is **not positive**; with complete Failures follow-up and no conflicting failure evidence, the binary outcome is negative. The structural exit defines later cohort membership separately. Unknown transaction, conflicting same-bank events, uncertain identity and incomplete follow-up remain unresolved. A null-CERT event cannot overturn a separate verified positive. The accepted 488 null-CERT events end in 1976 and therefore do not intersect these target windows. The accepted outcome snapshot extends beyond the final 2026Q2 right endpoint. No model split or feature table is produced here.

| Unit: bank-quarter decision | 1984Q1–2025Q2 (3,994,624) | 2025Q2 (24,064) |
| --- | ---: | ---: |
| Positive | 10,117 | 2 |
| Negative | 1,647,629 | 4,422 |
| Unresolved | 37,023 | 214 |
| Excluded | 2,299,855 | 19,426 |
| With as-of feature history, any label disposition | 3,697,094 | 24,051 |

The 10,117 positive **bank-quarters** correspond to 2,563 distinct observed CERTs and 2,563 distinct qualifying FDIC source events; one failure can label multiple earlier q. The production modelling subset must still restrict to eligible members with as-of history and a positive/negative decision, then apply Part 4.3 features. It cannot use retrospective activity fields, event IDs or labels as predictors.

The saved `p42review20261120b` reconciliation accounts for the previous 2025Q2 baseline: 21,214 P2.4 `eligible` diagnostics comprise 4,424 retrospectively active with feature history, 16,581 inactive, and 209 uncertain. Within the 4,424, there are 2 positives, 4,419 negatives and 3 unresolved. All 16,600 baseline reports older than 365 days fall into 16,464 inactive and 136 uncertain under structural evidence; age itself is **not** the activity rule. The full 24,064 observed identities include 13 with no feature history at 2025Q2. The 235 missing Institution identities and 159 prior insurance/end-date conflicts have separate deterministic disposition tables in `RETROSPECTIVE_COHORT_POLICY.md` and machine-readable bank evidence outside Git.

The independent repeat `membership20261120b`/`labels20261120b` had zero SHA-256 differences in all 167 membership and 166 label Parquet outputs and identical per-quarter summary counts. `p42review20261120b` rechecked saved upstream inputs and both output sets. Focused tests passed 56/56 and full pytest passed 379 with three unrelated Spark-environment skips. The historical 2026-11-16 and 2026-11-19 sections below document the former blocked decision and are superseded by this dated, measured study-design decision. They are retained to preserve research provenance.

## 2026-11-19 rule repair and complete membership-source audit

The user has selected the **active-bank-at-q** objective. The complete
supplementary source audit in `ACTIVE_MEMBERSHIP_RESEARCH.md` establishes
retrospective interval coverage but not a historically public register at
each q. The conditional rule in `src/bft65b/panel/membership_rule.py` requires
an independently verified complete register known at q; it does not promote
the current Institutions/History snapshot to production membership. The
second Part 4 checklist item and production labels remain **BLOCKED**.

The label function now resolves an independently verified, linked positive
before considering an unassigned null-CERT event in the same target window.
An unassigned event can still prevent a confirmed negative, and a meaningful
same-bank unknown transaction or conflicting link remains unresolved. A
known assistance transaction alone is neither a failure nor a closure:
with complete follow-up and no other censoring evidence it permits a
negative; it does not override a separately verified failure. An ordinary
documented merger is not a positive; if it ends observation before the
right endpoint, a negative is not established. The official target remains
`q < FAILDATE <= q + 4 calendar quarters`, with q no later than 2025Q2.

The accepted 4,117 repaired source events include 488 null-CERT events,
latest 1976-11-19. They have zero date overlap with prediction windows
beginning in 1984; the positive/null-CERT defect therefore has no measured
effect on the current candidate windows, while the rule remains correct for
future snapshots. No production label file was written.

**Decision (2026-11-16): BLOCKED for a production labelled dataset.** The
four-quarter outcome function and synthetic tests are implemented. The
aggregate-only preflight is complete in ignored
`runs/task_4_2_label_preflight/labelpreflight20261116b/`. No per-bank-quarter
production labels were written. The second Part 4 checklist box remains open.

## Assignment and three separate decisions

The authoritative examination PDF, page 1 §2.3, defines
`y[i,q] = 1{bank i fails in (q, q + 4 quarters]}`, says mergers are not
failures, and limits modelling observation quarters to 2025Q2. Page 2
requires a point-in-time correct bank-quarter panel and leak-free feature
table. It does **not** prescribe a report-age cutoff or equate an old report
with a bank still operating at `q`. The source Financials panel has
1,682,616 observed bank-quarters. The as-of candidate roster has 3,697,094
`(CERT,q)` rows across 166 prediction quarters. Neither is automatically an
approved active-bank risk set.

1. **Membership knowable at `q`:** `pit20261115f` offers a report-derived
   candidate roster. `population20261115b` offers an as-of four-calendar-
   quarter recent-reporter proxy. Neither proves active status. The accepted
   Institutions table is a later snapshot. The official History source was
   only probed; its `EFFDATE` and internal `PROCDATE` do not establish when
   each event became publicly knowable, and event coverage/identity remain
   unaudited for the full history. See `HISTORY_SOURCE_ASSESSMENT.md`.
2. **Identity and event link:** `repair20261115f` one-to-one reconciled all
   4,117 accepted source IDs with official FDIC transaction evidence. A
   verified failure transaction is distinct from assistance. Its P2.3
   `chronological_candidate` bank link requires a pre-event Financials report
   and no post-event report, but is not an independently verified historical
   identity register. Null `CERT`, unmatched `CERT`, post-event-report
   conflicts and multiple records remain explicit.
3. **Outcome after `q`:** For an approved member with resolved identity,
   `src/bft65b/features/label.py` sets `y=1` only for one corroborated,
   correctly linked `verified_failure` event satisfying
   `q < FAILDATE <= q + 4 calendar quarters`. It excludes a verified failure
   on or before `q`, keeps conflicting/merger/unknown evidence unresolved,
   and sets `y=0` only when the full right endpoint is covered without such
   evidence. A null-CERT failure in the outcome window prevents a confirmed
   negative because it cannot be assigned to a bank. The function requires
   externally supplied membership evidence under a verified-active policy
   or an explicitly approved recent-reporter objective; it does not derive
   membership from the proxy or retrospective exit flags. `label_batch()`
   rejects duplicate `(CERT,q)` keys. One unique source event can correctly
   label several earlier quarter observations.

The accepted Failures source index was created 2026-08-25 and the
supplementary response was retrieved 2026-11-14, both after the final
2025Q2 observation's right endpoint, 2026-06-30. Its latest event date is
2026-07-17. This supports endpoint observation in the accepted snapshot;
it does not certify that every historical event linkage or publication
vintage was complete. The 120-day Financials availability assumption is
unchanged and does not alter the official target window.

## Measured aggregate preflight, not a modelling sample

Units below are **candidate bank-quarters**, not distinct banks, failure
events, or positive labels. `P2.4 eligible` and the recent-reporter
intersection are retrospective diagnostics only; their intersection is
**not** adopted as the modelling population. All counts were reconciled by
quarter from the accepted `pit20261115f` and `population20261115b` saved
summaries, after their independent physical-file and source-hash verifiers
passed.

| Prediction period | Candidate bank-quarters | P2.4 eligible diagnostic | Recent-reporter proxy | Activity-uncertain stale | Recent and P2.4 eligible diagnostic | P2.4 eligible outside recent proxy |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 1984Q1–2025Q2 (166 quarters) | 3,697,094 | 3,335,808 | 1,695,045 | 2,002,049 | 1,682,864 | 1,652,944 |
| 2025Q2 | 24,051 | 21,214 | 4,617 | 19,434 | 4,614 | 16,600 |

At 2025Q2, the 21,214 P2.4 eligible diagnostics partition into 4,614
recent reporters and 16,600 outside the four-quarter proxy. The broader
24,051 candidate roster partitions into 4,617 recent and 19,434 stale.
The 365-day audit illustration in prior evidence is not an exclusion rule.
Across the full horizon, P2.4 has 322,352 prior-failure diagnostics and
38,934 unresolved-identity diagnostics. These may overlap the recency
categories; they cannot simply be intersected to produce an approved
population. The preflight includes all 166 quarter rows and source-event
counts by event quarter in `preflight.json`.

The accepted event table contains 4,117 unique source IDs: 3,522 verified
failure transactions, 580 assistance transactions, 13 open-bank-assistance
codes outside the verified dictionary enum, and two failure-charter-uncertain
records. Among the 3,522 verified failure event records, 2,572 have the
P2.3 chronological-candidate link; the other 950 have missing, unmatched,
or conflicting bank linkage. These are **events**, not 2,572 distinct banks
or positive bank-quarters. The 488 null-CERT events have latest FAILDATE
1976-11-19, before the modelling quarters; their source IDs remain preserved.
No positive/negative observation count is reported because no production
population has been approved.

## Population options and selected objective

| Choice | What the evidence permits now | Consequence |
| --- | --- | --- |
| Preserve an active-bank objective | Continue the historical activity/exit source investigation. Verify event keys, full coverage, effective dates and when information was knowable at each `q`. | Production labelled dataset remains blocked; exact eligible count is unknown. |
| Approve a narrower recent-reporter early-warning study | Explicitly state that the model predicts among banks with a report within a chosen as-of calendar-quarter window; retain stale/uncertain cases as out of scope, and independently resolve identity and censoring. | Four-quarter proxy covers 1,695,045 candidate bank-quarters, but this is an audit baseline, **not** a final eligible count. At 2025Q2 it covers 4,617 candidates; the 4,614 intersection is diagnostic only. A policy and sensitivity analysis would still be required. |

A saved proxy run is not user approval. On 2026-11-16 the user explicitly
selected the active-bank objective. The recent-reporter objective remains
unapproved. `ACTIVE_MEMBERSHIP_RESEARCH.md` documents the subsequent official
source review and why historical membership remains unverified; production
labels remain blocked pending adequate source-vintage and identity evidence.

## Reproduction and limitations

Run the aggregate-only preflight with a new ignored run ID; it revalidates
accepted event, candidate and population manifests, physical outputs and
hashes. The final `preflight.json` and manifest are internally labelled
`production_label_status=BLOCKED`; the manifest's complete status refers
only to the finished **preflight**. The source count, right endpoint,
transaction/link dispositions, all 166 quarter reconciliations and input
fingerprints are machine-readable. No label Parquet file exists in this run.

The label tests cover failure at `q`, one day after `q`, at the right
endpoint and one day after; prior failure, assistance, merger, unresolved
identity/linkage, null CERT, incomplete follow-up and multiple source IDs.
The accepted FDIC snapshot has no verified historical publication vintages
for all records. The function's `chronological_candidate` rule inherits
that P2.3 limitation; it is not a claim that bank identity is perfectly
resolved for every event. Split dates, feature fitting and models remain
untouched.
