# P4.2 Retrospective Active-Bank Study Cohort

## Methodological decision

The examination PDF, page 1 §2.3, defines the failure outcome over the next four quarters and excludes mergers as failures. Page 2 requires a point-in-time bank-quarter analysis and a leak-free feature table. It does not require a historically public institution-register vintage for every prediction quarter. The study therefore reconstructs the **historical activity cohort** using official effective structural dates in the captured 2026 Institutions and institution-role History snapshots. This is a retrospective research cohort, not a claim that an operational system knew the same complete register at historical `q`. Financial predictors remain limited by the accepted `REPDTE + 120 calendar days <= q` rule. Failures after `q` supply outcomes only.

The universe is the 24,064 distinct CERTs with an observed Financials record in `panel20261115b`. The policy evaluates all of those identities at every 1984Q1–2025Q2 quarter-end, including times with no available Financials history. It does not claim to cover an institution that never appears in the accepted Financials panel. A missing current Institutions record is not proof that a bank was inactive. A stale or missing report is not proof of closure.

## Versioned rule

Policy `retrospective_operational_v1` uses `ESTYMD` as operational entry and `ENDEFYMD` as effective inactivity when these dates and current `ACTIVE` are coherent. The operational interval is `[ESTYMD, ENDEFYMD)`: entry on `q` admits the bank; exit on `q` excludes it. A currently open record with `ACTIVE=1` and a `9999-12-31` end is treated as right censored, conditional on the current status field. This is a study convention for the sentinel, not an assertion that the official dictionary defines the literal 9999 value as a public historical release date. An inactive record with that sentinel has an unresolved end. `INSDATE` denotes insurance entry, **not** operational establishment; missing or 9999 `INSDATE` is never replaced by `ESTYMD`. Insurance-start sensitivity is reported separately.

For 235 observed CERTs missing from current Institutions, a History CERT overlap by itself is insufficient. A History-only interval is supported only with exactly one institution-level code 110 entry, one whole-institution ordinary merger (221–224) or voluntary closure (240) terminal, no certificate change or failure-resolution code, and every observed Financials report inside the interval. Other cases remain unresolved. Code 225 is a **partial** purchase and assumption; 810–812 describe merger participation; 410 describes insurance-status change; and 830 denotes open-bank assistance. None is a whole-institution exit by itself. Codes 211, 213, 215–217, 230, 235 and 260 describe failure-resolution or closure pathways, not ordinary unassisted merger; only the repaired FDIC Failures/Assistance event table can establish the binary failure outcome. Code 150 is a certificate change and leaves an unresolved identifier mapping after its effective date. The saved official `CHANGECODE_DESC` probe `codeprobe20261119a` validates each code used by this policy against the captured History index. Code 215 has three source examples; the validator accepts its three consistent descriptions rather than requiring five nonexistent examples.

Reporting gaps do not end activity. A later supported entry can reopen an interval; no real multiple-entry mapping is asserted without evidence. A conflicting report, certificate change, or discrepancy between Institution and History terminal dates creates explicit uncertainty from the relevant effective date. An event effective after `q` does not change the activity decision at `q`; a later correction to an earlier effective date can revise the **reconstructed cohort**. That is a historical-source revision, not permission to put future values into predictors. `PROCDATE` and `DATEUPDT` are processing/update dates, not verified first-publication timestamps. The older `complete_register_known_at_q` fixture remains a separate hypothetical deployment test and is not set to true for this study.

## Identity resolution from saved official snapshots

The 45 captured API responses in `activeaudit20261119e`, the code probe, 170 accepted panel quarters, and 166 accepted `pit20261115f` quarters are checksum checked. Source definitions and earlier snapshot reconciliation are in `ACTIVE_MEMBERSHIP_RESEARCH.md`.

| Group (unit: distinct observed CERT) | Total | Supported Institution interval | Supported strict History interval | Unresolved dates | Unresolved History overlap only | No Institution or History |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| All observed banks | 24,064 | 23,807 | 118 | 22 | 106 | 11 |
| No current Institution identity | 235 | 0 | 118 | 0 | 106 | 11 |
| Prior `INSDATE`/`ENDEFYMD` conflict group | 159 | 137 | 0 | 22 | 0 | 0 |

The 159 older conflicts mixed insurance start with operational end. The resolved 137 include special or missing insurance dates while operational establishment/end evidence remains coherent. The 22 unresolved are inactive records without a usable effective end. Overall, 23,925 of 24,064 observed identities have a supported interval and 139 remain unresolved; supported identities may still have time-local conflicts. The ignored bank-evidence Parquet keeps every CERT, source IDs, dates, rule and disposition. The subset must not be described as complete coverage of all historically active banks.

Across all 24,064 bank-evidence rows, 81 have a dated certificate-change code, six have a report or terminal conflict start, 13 have differing Institution/History terminal dates, and 391 have an unavailable or sentinel insurance date. These are overlapping **bank-level flags**, not additional mutually exclusive dispositions. The quarter rule turns relevant conflicts into uncertainty at or after the affected effective date; it does not apply a future change to an earlier q. Detailed CERTs remain in ignored evidence.

## Measured membership scope

Saved run `membership20261120a` has one decision for each of 24,064 observed CERTs at each of 166 prediction quarter-ends: 3,994,624 decisions. All 167 ZSTD Parquet outputs passed schema, key, row-count, read-back and SHA-256 validation. The candidate roster contributes 3,697,094 bank-quarters; 297,530 bank-quarters have no feature history under the accepted report-availability rule.

| Unit: bank-quarter decision | 1984Q1–2025Q2 | 2025Q2 |
| --- | ---: | ---: |
| Retrospective activity eligible | 1,660,450 | 4,427 |
| Ineligible | 2,298,952 | 19,419 |
| Uncertain | 35,222 | 218 |
| Eligible with feature history | 1,605,865 | 4,424 |
| Eligible without feature history | 54,585 | 3 |
| All observed CERT identities assessed | 3,994,624 | 24,064 |

The 2025Q2 candidate roster has 24,051 rows; 13 of the 24,064 assessed identities lack eligible report history. The prior P2.4 21,214 `eligible` candidate diagnostics and 16,600 older-report cases are reconciled separately in the review evidence. The cohort uses structural evidence, not a report-age cutoff.

### Worked 2025Q2 reconciliation

| Population at 2025Q2 (unit: distinct CERT bank-quarter) | Count | New disposition |
| --- | ---: | --- |
| All Financials-observed identities | 24,064 | 4,427 active, 19,419 inactive, 218 uncertain |
| As-of candidate roster | 24,051 | 4,424 active with history, 19,411 inactive, 216 uncertain |
| No as-of feature history | 13 | 3 active, 8 inactive, 2 uncertain |
| Earlier P2.4 `eligible` diagnostic | 21,214 | 4,424 active with history, 16,581 inactive, 209 uncertain |
| Earlier `eligible` with report age >365 days | 16,600 | 0 active, 16,464 inactive, 136 uncertain |

The production label table for the full 24,064 identities records 2 positive, 4,422 negative, 214 unresolved and 19,426 excluded decisions. Among the earlier 21,214 diagnostic `eligible` candidates, the new 4,424 active-with-history cases contain 2 positive, 4,419 negative and 3 unresolved decisions. The remaining 16,581 are excluded as inactive; the 209 uncertain remain unresolved. These denominators are not interchangeable. Detailed keys remain only in ignored Parquet files; `p42review20261120b/summary.json` contains the exact machine-readable cross-tab and all 166-quarter counts.

## Sensitivity and limitations

Requiring a verified insurance-start date by `q` would remove 78,060 otherwise activity-eligible bank-quarters, including one at 2025Q2. This is a sensitivity scenario, **not** the adopted operational-entry rule; it also excludes cases with a missing or special `INSDATE`. Treating an exit effective on `q` as still active could add at most 1,205 bank-quarter decisions across the study horizon, including one at 2025Q2, before time-local conflicts or feature-history filters. The adopted rule excludes same-day exits. Neither scenario establishes historical public availability.

The current-snapshot design can be affected by revised structural dates, incomplete old Institution identity coverage, and survivor/selection bias from the Financials-observed universe. Unknown identities and conflicting dates remain visible in the denominator and do not silently become negative labels. The modelling subset additionally requires available Financials history and a resolved label. These are distinct filters. The final report must state this scope and cannot present the cohort as a real-time 1984–2025 registry.
