# Active-Bank Membership Source Review — 2026-11-16

## 2026-11-20 retrospective-cohort resolution

The complete captured sources from `activeaudit20261119e` were reused without a further FDIC download. The saved 36-code `CHANGECODE_DESC` probe `codeprobe20261119a` was checksum and source-index matched; every entry, exit, assistance, partial-transfer, identifier-change and failure-resolution code used by the new policy has a consistent official description in that probe. The complete, versioned decision and assumptions are in `RETROSPECTIVE_COHORT_POLICY.md`. It uses effective operational dates for a **retrospective historical study cohort** while keeping Financials predictors under the accepted 120-day as-of rule. It does not claim a historically public structural-register vintage.

The 235 current-Institutions-missing CERTs resolve to 118 strict History-supported intervals, 106 unresolved History-overlap cases, and 11 without a captured Institution/History identity. The earlier 159 `INSDATE`/`ENDEFYMD` conflicts resolve to 137 supported operational intervals and 22 unresolved inactive records with no usable end date. In total, 23,925/24,064 observed CERTs have supported intervals and 139 remain unresolved. These are measured in `membership20261120a`; the prior audit's retrospective diagnostic classifications and BLOCKED conclusion below remain dated historical findings, not the current production status.

`INSDATE=12/31/9999` is not used as insurance inception, and `ESTYMD` is never silently substituted for it. `ENDEFYMD=12/31/9999` is not treated as a literal exit date; the policy interprets it as right censored only with `ACTIVE=1` and marks `ACTIVE=0` plus that sentinel uncertain. This is an explicit study convention supported by the current-status field and source pattern; the official dictionary does not establish a historical first-publication date for that sentinel. A 78,060 bank-quarter insurance-start sensitivity and separate exit-day scenario quantify policy effects. See ignored `runs/task_4_2_review/p42review20261120b/summary.json`.

## Complete official-source coverage audit — 2026-11-19

The earlier bounded-probe assessment below is retained as historical
evidence. A separate ignored source run, `activeaudit20261119a`, subsequently
captured all 27,835 current Institutions records in three CERT-ordered pages
(10,000/10,000/7,835) and all 119,475 institution-role (`ORG_ROLE_CDE=FI`)
History events effective in 1984–2025 in 42 complete annual partitions.
The smallest annual result had 530 rows and the largest 6,742; no annual
result needed a second 10,000-row page. Every page retained its complete API
response, query, reported total, source index and SHA-256. The source indexes
were stable within each endpoint. This is a bounded full-history source pull,
not a historical public-vintage series. Branch and other non-institution
events were excluded by the official role field and checked row by row.

The first analysis of those immutable responses accidentally read a History
source ID from the response wrapper rather than `data.ID`. Its summary is
superseded. The corrected analysis verified 119,475 nonempty, unique History
IDs and reconciled the current Institutions snapshot to the accepted
`real20261111b` snapshot. The latter has the same 27,835 CERTs, no added or
removed CERTs, four changed `ACTIVE` values and nine changed `CHANGEC1`
values after normalizing the accepted string representation against the API
numeric representation. These are source revisions, not a byte-equality
failure. The current source index is `institutions_20261113100007`; the
accepted index was `institutions_20261106100008`.

| Coverage unit | Denominator | Observation | Count |
| --- | ---: | --- | ---: |
| Distinct observed Financials CERTs | 24,064 | Matched to current Institutions CERT | 23,829 |
| Distinct observed Financials CERTs | 24,064 | No current Institutions CERT | 235 |
| Observed CERTs missing in Institutions | 235 | Present in bounded FI History by CERT | 224 |
| Observed CERTs missing in Institutions | 235 | Absent from bounded FI History by CERT too | 11 |
| Matched observed CERTs | 23,829 | Invalid retrospective entry/end interval | 159 |
| Matched observed CERTs | 23,829 | Retrospective interval candidate | 23,670 |
| Repaired Failures source events | 4,117 | Null CERT; latest event 1976-11-19 | 488 |
| Distinct nonnull Failures CERTs | 3,585 | Also in 1984–2025 FI History CERT set | 2,410 |
| Distinct nonnull Failures CERTs | 3,585 | Absent from bounded FI History CERT set | 1,175 |

The 488 null-CERT events precede the first 1984 prediction quarter, so the
confirmed-positive/null-CERT code defect has no measured overlap with these
modelling windows. A History ID is not the Failures source ID: the sets have
zero exact matches. A shared CERT is only an identifier overlap, not a
one-to-one proof that a particular History event describes a particular
Failure. The full History pull includes 2,608 records without a usable CERT.
Of these, 2,594 omit the CERT field and 14 carry an explicit null; the raw
responses preserve that distinction. In the current Institutions response,
3,478 records omit CHANGEC1, none explicitly set it null, and 19 omit
DATEUPDT. The source type/date validation accepts these observed absences
without replacing missing values.
The current Institution fields establish retrospective entry and inactivity
dates, but their current values cannot establish what a modeller knew at
historical `q`.
The FI History response contains 36 distinct CHANGECODE values. A complete
official mapping from these codes to bank-level entry, ordinary merger,
failure resolution and other inactivity has not been independently verified
for this snapshot, so the audit retains the codes without promoting them
to exit classifications.

The corrected, independently verified analysis is `activeaudit20261119e`.
Its ignored `summary.json` contains all 166 prediction-quarter breakdowns,
the 45 source file hashes, both source indexes, source queries and retrieval
time. The `run_manifest.json` is complete for the **audit only**; its
`production_membership_status` is `BLOCKED`. The 3,697,094 candidate
bank-quarters divide into 1,544,548 inside a *retrospective* Institution
interval, 2,102,694 outside it, 26,104 without an Institution identity and
23,748 with an invalid or incomplete interval. These are candidate
bank-quarters, not a verified historical active-bank population.

| 2025Q2 unit: candidate bank-quarters | All candidates (denominator 24,051) | P2.4 eligible diagnostic (denominator 21,214) | P2.4 eligible with report age >365 days (denominator 16,600) |
| --- | ---: | ---: | ---: |
| Inside retrospective interval | 4,431 | 4,431 | 0 |
| Outside retrospective interval | 19,228 | 16,391 | 16,274 |
| No Institution identity | 233 | 233 | 177 |
| Unresolved interval | 159 | 159 | 149 |
| **Total** | **24,051** | **21,214** | **16,600** |

This reconciles the prior 21,214 and 16,600 baselines without treating old
reports as evidence of closure. The P2.4 `eligible` status is a candidate
diagnostic, not active membership. The 4,431 inside-interval cases cannot
be promoted to a production active roster because the interval is compiled
from the 2026 current snapshot. No availability-lag sensitivity estimate is
presented as evidence: the official sources do not establish a justified
historical publication bound for structural events. Any hypothetical lag
would be a scenario, not a validated as-of population rule.

### Conditional active-bank rule and unresolved gate

`src/bft65b/panel/membership_rule.py` specifies the rule **conditional on a
complete register proven public at q**: admit a bank after a known entry
effective on or before q; a known exit effective on or before q takes
precedence even on the same day; a later independently verified entry can
reopen membership. Assistance and Financials reporting gaps do not end
membership. Unresolved identity or incomplete register coverage yields
uncertain, and an event learned after q cannot alter the prior decision.
Synthetic tests exercise these conditions; no production run supplies a
historical complete-register certificate.

`EFFDATE` is an event's effective date; `PROCDATE` is an internal processing
date; `DATEUPDT` is the Institution record's last update. None is defined as
the first public availability date. In this complete pull, 114,595 of
119,475 FI History events have `PROCDATE` later than `EFFDATE`; a sampled
1984-effective event was processed in 2000. The [FDIC bulk-download page](https://banks.data.fdic.gov/bankfind-suite/bulkData/bulkDataDownload)
states that structure downloads expose the current weekly update, not
historical vintages. Assigning a fixed publication lag to these events
would be an unsupported assumption. The known 120-day Financials-report
availability rule does not license reusing that lag for institution events.

Therefore the completed source audit supports retrospective interval
diagnostics only. **Production membership and labels remain BLOCKED** pending
historical public-vintage evidence or an independently justified upper bound
for entry and exit information, a verified event-code interpretation, plus
resolution of the 235 Institution-missing
identities (224 with History CERT overlap but no current Institution interval)
and 159 conflicting intervals for any claimed all-bank population. The
active-bank objective selected by the user remains in force. The recent-
reporter proxy has not been adopted.

**Selected objective:** retain a population of banks demonstrably active at
prediction quarter-end `q`. This follows the user's explicit decision after
the P4.2 label preflight. **Status: BLOCKED for production membership and
labels.** This review made bounded official-source probes only; it did not
download the complete Institutions or History endpoint, change accepted
snapshots, or publish bank-quarter labels. Sanitized machine-readable probe
results are in ignored `runs/task_4_2_active_membership_research/
activeprobe20261116a/observations.json`.

## What the official fields establish

The [FDIC Institutions dictionary](https://api.fdic.gov/banks/docs/institution_properties.yaml)
was retrieved successfully (94,832 bytes; SHA-256
`6ed1763c66a9e240904a45712b23d825c0f31beaadb6cd0213fc1bf00b2af1b9`).
It defines `CERT` as the FDIC certificate number; `INSDATE` as the date an
institution obtained federal deposit insurance; `ESTYMD` as the date it
began operations; `ENDEFYMD` as the end/closing date of its last structural
event, representing the inactivity date for closed institutions; `ACTIVE=1`
as **currently** open and FDIC-insured and `ACTIVE=0` as closed or no longer
insured; and `DATEUPDT` as the last data update. These definitions make
entry/inactivity dates potentially useful retrospective evidence. None of
these fields identifies when a historical value or event first became
publicly knowable at a prior `q`. The current `ACTIVE` flag is not historical
status at each `q`.

The [official History dictionary](https://api.fdic.gov/banks/docs/history_properties.yaml)
and the prior bounded assessment in `HISTORY_SOURCE_ASSESSMENT.md` distinguish
`EFFDATE` (event effective date) from `PROCDATE` (internal process date).
`PROCDATE` is not documented as a historical public-release timestamp.
The [FDIC bulk-download page](https://banks.data.fdic.gov/bankfind-suite/bulkData/bulkDataDownload)
states that structure-related bulk downloads are limited to the current
weekly update. The [FDIC Bank Data Guide](https://www.fdic.gov/bank-data-guide/quick-reference-table)
describes the institution/location download as current. These sources do not
provide complete bank-level public vintages for all 1984Q1–2025Q2 quarters.

## Bounded live observations

| Probe | API-reported total | Source index | What it establishes |
| --- | ---: | --- | --- |
| Accepted Task 1.5 Institutions snapshot | 27,835 | `institutions_20261106100008` | Accepted snapshot and existing hashes; selected schema lacks the dated fields. |
| Current `/institutions`, six verified fields, `limit=2` | 27,835 | `institutions_20261113100007` | Same total but a different source index. Two rows do not establish full-key or value parity. |
| Current `ENDEFYMD:[2025-07-01 TO 2025-12-31]`, `limit=1` | 84 | `institutions_20261113100007` | At least the endpoint reports 84 records in this later-end-date filter; the single sampled record has `ACTIVE=0`, `ENDEFYMD=08/02/2025`, `DATEUPDT=09/16/2025`. This does **not** establish how many were members of the 2025Q2 candidate roster. |

Two earlier inactive records in the two-row probe have `ENDEFYMD` in 1982
and 1978, but `DATEUPDT` in 1990 and 1995 respectively. This observed lag
and the field's official *last update* definition rule out treating
`DATEUPDT` as the first public-knowledge date of the inactivity event. A
current `ACTIVE=0` for a bank with an end date after 2025Q2 also illustrates
why filtering on the present flag would use future information to decide
membership at 2025Q2. The 84 filtered records are an endpoint total, not a
count of affected candidate bank-quarters.

The current Institutions index differs from the accepted index despite
equal reported totals. A new dated-field pull cannot be silently joined to
the accepted Institutions snapshot; all identifiers, source versions,
revisions and conflicts would have to be reconciled first. No new field has
been added to the accepted acquisition allowlist or raw schema.

## Decision and required evidence

The selected active-bank objective **does not authorize** use of the
four-quarter recent-reporter proxy as the production risk set. The 2025Q2
preflight reconciles 21,214 P2.4 eligible diagnostics to 4,614 recent
reporters and 16,600 outside that proxy; these are coverage diagnostics,
not verified active/inactive labels. The full-horizon 3,697,094 candidate
bank-quarters likewise cannot be promoted to an active-bank population.

Before production labels, establish a reproducible bank-level membership
source and rule covering 1984Q1–2025Q2:

1. Obtain and audit complete institution entry, insurance and exit records,
   their identifiers and changes across source indexes. Reconcile them to
   the accepted 24,064 panel banks, 166 candidate quarters, and repaired
   failure events without silently dropping unresolved identities.
2. Establish when entry/exit information was actually public or a justified
   conservative knowability bound. `EFFDATE`, `ENDEFYMD`, `PROCDATE`,
   `DATEUPDT` and current `ACTIVE` must not be interchanged.
3. Define same-day entry/exit precedence, conflict handling, observation
   at the dataset boundary and how missing historical evidence is treated.
   Run future-perturbation tests before calculating an approved risk set.
4. Only then apply the already-tested four-quarter outcome rule and save
   unique `(CERT,q)` labels with complete right-endpoint follow-up.

The official sources checked here have **not** yet satisfied those gates.
The historical public-knowledge/vintage question is the decisive blocker;
full endpoint retrieval alone would not resolve it. The correct next work
is source-vintage research and a complete identity/coverage audit, not
label publication or model training.

## Reproducible bounded probes

From `C:\NEU\BFT65B` with the existing pinned environment:

```powershell
$base = 'https://api.fdic.gov/banks/institutions'
$fields = 'CERT,INSDATE,ESTYMD,ENDEFYMD,ACTIVE,DATEUPDT'
$all = Invoke-RestMethod -Uri "${base}?fields=$fields&limit=2&offset=0&sort_by=CERT&sort_order=ASC&format=json" -TimeoutSec 35
$all.meta | Select-Object total,index,parameters
$filter = [uri]::EscapeDataString('ENDEFYMD:[2025-07-01 TO 2025-12-31]')
$late = Invoke-RestMethod -Uri "${base}?fields=CERT,ACTIVE,ENDEFYMD,DATEUPDT&filters=$filter&limit=1&format=json" -TimeoutSec 35
$late.meta | Select-Object total,index,parameters
```

The API is live and may change; a future repetition must record its own
index, retrieval time, total and field values. The table above reports only
the observed 2026-11-16 probes. It is not a full endpoint completeness test.
