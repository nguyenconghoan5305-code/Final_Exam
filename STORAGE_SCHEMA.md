# BFT65B Versioned Parquet Storage Contract

This is the typed landing format for the dictionary-checked fields. The assignment PDF defines the required scope; SOURCE_FIELD_INVENTORY.md records official field definitions and unresolved unit/history questions. The original 2023 pilot used Financials v1, Failures v2, and Institutions v1. The accepted full-history Financials snapshot later uses the separate extended Financials v2 schema described below.

## Layout and configuration

| Dataset | One-file target | Partition |
| --- | --- | --- |
| Financials | data_raw/fdic/financials/REPDTE=YYYYMMDD/part-000.parquet | One calendar quarter-end within 1984Q1-2026Q2 |
| Failures | data_raw/fdic/failures.parquet | Whole endpoint |
| Institutions | data_raw/fdic/institutions.parquet | Whole endpoint |

The caller supplies the configured raw root; BFT65B_DATA_ROOT can point outside the repository. All data paths and run manifests remain Git-ignored. The default writer codec is Zstandard at level 3. ProjectConfig.parquet_compression permits zstd or snappy; parquet_zstd_level defaults to 3. The writer verifies the Parquet footer's codec. The exact Zstandard level is recorded in configuration because the footer does not expose the writer setting. Task 1.6's separate, same-data storage comparison is documented in STORAGE_BENCHMARK.md.

Each file carries Arrow schema metadata: `bft65b.schema_version=1` for Financials and Institutions or `=2` for Failures, dataset, partition, provisional amount unit, and ratio unit. This per-dataset version change preserves the four previously verified Financials v1 partitions byte-for-byte. The physical partition path and schema metadata must agree. Dates are stored as Arrow date32 rather than strings. Nullable numeric values remain null; the writer never fills them with zero.

## Explicit Arrow schemas

All fields below retain their source names. Required keys are non-null. Financials monetary Decimal(20,4) values have up to 16 integer and 4 fractional digits; no rounding is permitted. These units are the assignment's general thousands-of-USD convention unless the official dictionary states otherwise. NCLNLS and LNATRES have unresolved field-specific definitions. Failures `COST` follows the dictionary's `double` source type and is stored as finite float64 in thousands of USD; 756 of 3,480 observed non-null values required more than four decimal places, so the original Decimal(20,4) conversion would have rounded or rejected them.

| Dataset | Field | Arrow type | Source conversion and unit |
| --- | --- | --- | --- |
| Financials | CERT | int64, required | Integer or digit string to canonical bank identifier |
| Financials | REPDTE | date32, required | Source YYYYMMDD to calendar quarter-end; must match partition |
| Financials | STNAME | utf8 | State name, no numeric conversion |
| Financials | ASSET, DEP, EQ, NETINC, NCLNLS, LNATRES | decimal128(20,4) | Provisional thousands of USD; exact conversion only |
| Financials | ROA, RBC1AAJ | float64 | Percent; finite values only. RBC1AAJ is the PCA leverage ratio |
| Failures v2 | CERT | nullable int64 | Bank linkage identifier when FDIC supplies it; 488 of 4,117 pilot rows have source JSON null. No missing value is invented |
| Failures | FAILDATE | date32, required | M/D/YYYY, ISO date or timestamp, or YYYYMMDD to event date |
| Failures v2 | COST | nullable float64 | Official estimated loss, source double, thousands of USD; non-finite values and N/A strings are rejected |
| Institutions | CERT | int64, required | Same canonical identifier rule |
| Institutions | STNAME | utf8 | State name |
| Institutions | ACTIVE | int8 | Official 0/1 status or null; not a failure label |
| Institutions | CHANGEC1 | utf8 | Structural-event code, including numeric API values converted to text; leading zeros in source text retained |
| Financials, Institutions | ID | nullable utf8 | Optional API response ID retained for provenance |
| Failures v2 | ID | required utf8 | Empirically observed source-record identity in the response data object; nonempty and unique across the complete result set. It is **not** a bank identifier or a selectable field in the official Failures YAML dictionary |
| All three | SOURCE_MISSING_FIELDS | list<utf8>, required | Names of optional requested fields absent from the source data object; distinguishes absence from explicit JSON null |

The writer rejects unmodelled fields rather than silently dropping them. Financials and Institutions require non-null integer `CERT`; Financials require unique `(CERT, REPDTE)`. Failures require nonempty source `ID` and `FAILDATE`, with `ID` as the unique source-record key; repeated `CERT` and null `CERT` are preserved. Missing requested `CERT` is distinguished from explicit JSON null through `SOURCE_MISSING_FIELDS`. The writer rejects duplicate keys, non-quarter or out-of-scope Financials dates, Financials rows outside their partition, non-finite values, decimals needing rounding, invalid ACTIVE values, and source row counts that do not match the supplied API total. A source N/A string or unfamiliar historical date fails with an explicit error pending a documented cleaning rule. Parsed raw values must be retained by the acquisition caller until promotion succeeds; conversion failure never promotes a file.

## Atomic write and rerun contract

1. Convert and validate all source rows; require a nonempty source query and an API-reported total equal to the actual row count.
2. Write Parquet to a temporary file **in the target directory and filesystem**.
3. Reopen the temporary file and verify Arrow schema/metadata, codec, row count, keys, partition dates, every value against the typed in-memory table, and SHA-256.
4. Promote only with os.replace after the temporary file passes. Reopen and check the final file, then append a complete output record and validation to the atomic JSON run manifest. The caller marks the overall run complete only after all intended files have passed.
5. `can_skip` reads the **persisted** manifest. The caller must supply the expected source query and API total. The manifest must be `running` or `complete`, have no failed validations, and contain a complete storage stage, passing storage validation, and complete output entry. The output's dataset/path/partition, source total and row count, query, schema version, and codec must match. The final on-disk file must independently pass schema, row-count, key, codec, and SHA-256 checks. An in-memory-only manifest update cannot authorize a skip.

A healthy file with different rows, schema, codec, or completed-manifest checksum raises StorageConflictError and is not overwritten. A healthy identical file with no complete manifest entry is revalidated and reconciled without rewriting the raw bytes. A truncated/unreadable file cannot be skipped; after a new temporary file passes validation, the old bytes are copied to a .corrupt.<hash> backup and the verified file is atomically promoted. If the manifest write fails after promotion, a later run must revalidate and reconcile; it cannot skip from an unpersisted record. A future API source update is not detected solely by local validation; Task 1.3 must compare source metadata and totals before deciding whether a prior download is still complete.

## Task 1.4 recovery contract

`run_pilot(..., resume=True)` resumes only a persisted `running` pilot manifest. For each previously completed dataset it verifies the strict `can_skip` gate, then requests one small API page and compares the current API total and index fingerprint with the stored source metadata. A changed total/index stops rather than accepting old bytes. A missing, corrupt, or stale output is fetched again. Identical healthy raw bytes are reconciled without rewriting; a corrupt file is replaced only after temporary-file validation. The Failures v2 unresolved-linkage audit is checked against its checksum and the exact set of null-CERT source IDs before its file can be skipped. All four Financials quarters, Failures, and Institutions undergo a final physical read-back before the manifest is marked `complete`.

A `complete` pilot manifest is immutable: `--resume` performs read-only physical/source-metadata verification and returns its summary when checks pass. A failed manifest cannot authorize a skip or be resumed in place; start a new manifest path and re-fetch/reconcile against healthy existing raw files. A wrong persisted checksum causes a conflict and stops for investigation. No Task 1.4 test corrupts the accepted 2023 pilot files: all destructive tests use pytest temporary directories. The complete source content cannot be proven unchanged from a one-page metadata probe if an upstream provider were to revise records while keeping both total and index fingerprint unchanged; later full acquisition must retain source-version and conflict checks.

PyArrow reads individual files through ParquetFile, avoiding accidental Hive partition-column injection when a path contains REPDTE=YYYYMMDD. Task 3.0 has separately passed on the verified local Spark runtime.

## Verification status

Task 1.2 originally used synthetic fixtures only. On 2026-11-08, tests/test_storage.py passed 26 synthetic PyArrow tests and the complete pytest suite passed 89 tests. Task 1.3 later validated the versioned schemas against the four complete 2023 Financials quarters, the complete Failures endpoint, and the complete Institutions endpoint. The exact row counts, checksums, and read-back results are in `runs/task_1_3_pilot_resume3.json` and its summary. No storage speed claim was made at that checkpoint; the later Task 1.6 result is separate below.

Task 1.4's synthetic recovery suite passed 137/137 focused tests and the full regression suite passed 146/146 on 2026-11-11. The machine-readable focused results are in the ignored `runs/task_1_4_recovery_junit.xml`. The accepted Task 1.3 manifest retained `status=complete`; all six real output checksums and counts were independently revalidated after recovery testing.

## Task 1.5A Financials v2 extension (separate landing area)

The accepted Financials v1 remains the 11-field schema under `data_raw/fdic/financials/`. The extended 26-field configuration in `config/financials_extended_v2.json` selects the 11 existing fields plus 15 dictionary-verified codes listed in SOURCE_FIELD_INVENTORY.md. `ProjectConfig.fields_verified=True` is accepted only for the exact v1 or v2 Financials tuple and the unchanged Failures/Institutions tuples. `FDICClient.VERIFIED_FIELDS` permits the 15 new codes; the bounded runner always requests its exact configured tuple. `ParquetStore` chooses schema and `schema_version` from that tuple, and `can_skip()` checks the matching version in the persisted output entry and on-disk Arrow metadata. A v1 file cannot satisfy a v2 skip.

The configured raw root for Task 1.5A is `C:\NEU\BFT65B\data_raw\task_1_5a` when run from the documented project root. `ParquetStore.target_path()` **appends** `fdic_extended_v2`, then the dataset path. Thus 2023Q1 v2 resolves exactly to `C:\NEU\BFT65B\data_raw\task_1_5a\fdic_extended_v2\financials\REPDTE=20230331\part-000.parquet`; 1984Q1 uses the same layout with `REPDTE=19840331`. This namespace remains separate even if a caller accidentally supplies the original raw root, and `data_raw/` is Git-ignored. Future v2 Failures/Institutions targets under this root would be `fdic_extended_v2/failures.parquet` and `fdic_extended_v2/institutions.parquet`; they were **not** fetched by the Task 1.5A live bounded runs. The original v1/v2/v1 pilot targets, manifests, and linkage audit stay in place byte-for-byte.

| New Financials v2 fields | Arrow type | Unit and conversion contract |
| --- | --- | --- |
| CHBAL, SC, BRO, DEPDOM, EINTEXP, LNLSGR, LNRE, LNCI, NTLNLS, ELNATR | `decimal128(20,4)` nullable | Amount-like source `number`; provisional thousands of USD from the assignment, with field-specific YAML unit absent. Exact decimal conversion; no rounding or replacement of null. |
| P3ASSET, P9ASSET, NAASSET | `float64` nullable | Source `number`, scale/unit **unverified**; the eight complete bounded quarters returned integers or null. Integers beyond float64's exact range and lossy numeric strings are rejected. Do not label these percentages without further source evidence. |
| NIMY, RBCRWAJ | `float64` nullable | Official YAML percentage formula; finite numeric conversion. `RBCRWAJ` is PCA risk-based capital, distinct from the existing `RBC1AAJ` leverage ratio. |

The extended Arrow schema carries `bft65b.schema_version=2`, `bft65b.dataset`, `bft65b.partition`, provisional amount and ratio units, and `bft65b.unverified_numeric_units=P3ASSET,P9ASSET,NAASSET`. Existing Financials v1, Failures v2, and Institutions v1 metadata and physical files were not changed. The existing `SOURCE_MISSING_FIELDS` list continues to distinguish an absent requested field from a present JSON null. Zstandard level 3 remains the provisional writer setting and was verified from the Parquet footer; no storage benchmark has been run.

The bounded Task 1.5A summaries and manifests under ignored `runs/task_1_5a_*.json` record the actual page counts, source totals/indexes, field absence/null/type counts, timestamps, output sizes, checksums, and validation outcomes. The 1984Q1 v2 file passed a 17,930-row two-page read-back and SHA-256 gate; all four extended 2023 quarters and three other profile quarters passed the same storage gate. These eight observed quarters do not establish 170-quarter type or availability coverage. A new historical conversion or conflicting healthy file must stop before promotion.

## Task 1.5B full-run namespace and recovery contract

The dedicated full-run raw root is `C:\NEU\BFT65B\data_raw\task_1_5_full`; `ParquetStore.target_path()` appends `fdic_extended_v2`. Thus the first Financials target is `C:\NEU\BFT65B\data_raw\task_1_5_full\fdic_extended_v2\financials\REPDTE=19840331\part-000.parquet`, the last uses `REPDTE=20260630`, and the two complete small-table targets are `fdic_extended_v2\failures.parquet` and `fdic_extended_v2\institutions.parquet`. All are outside the accepted Task 1.3 and Task 1.5A namespaces and ignored by Git. Each attempt has its own `runs/task_1_5_full_<attempt>.json`, `_summary.json`, and `_summary_unlinked_ids.json` paths. A path or configuration that could alias accepted artifacts is rejected before requests.

An attempt ID is write-once. A failed, interrupted, or complete-but-summary-incomplete attempt is recovered only through a **new** ID with `--recover-from <previous-id>`. Previous manifest and summary hashes are recorded as lineage inputs; previous files are not changed. A prior completed job may be inherited only when its persisted source query, total, index, page/field audit, output schema version, row count, codec, key uniqueness, SHA-256, current source probe, and (for Failures) unresolved-linkage audit reconcile. Otherwise the source is re-fetched and the existing file is validated against the new typed table. A corrupt file is backed up and replaced through validated same-filesystem atomic promotion; a healthy differing file is a conflict. Null versus absent fields, source types, page offsets/counts, retrieval timestamps, and conversion anomalies remain in the per-job summary. A missing/empty quarter and an incomplete Failures/Institutions endpoint cannot complete the plan. The final gate validates all 172 outputs and re-probes all 172 source totals/indexes before marking the manifest complete. The summary commits last; a commit failure retains the mismatch as recoverable evidence. All Task 1.5B execution tests used synthetic temporary data; no real full acquisition or storage benchmark was run.

The shared full-run root now requires an OS-owned, nonblocking lock at `data_raw/.task_1_5_full.lock`. Ownership begins before any attempt manifest/summary/audit/raw write and lasts through final validation and commit. The lock file is persistent and ignored; never unlink it because that could create two independently locked file identities. Windows releases the byte-range lock on process termination; a Linux runtime uses `flock`. A competing process fails before changing attempt evidence, and `ParquetStore.write()` refuses full-root writes outside the ownership context. A no-network dry-run never creates or takes the lock. A failed final read-back is reported under `validation:final_parquet:<endpoint>:<quarter>`; a failed final source probe uses `validation:final_source:<endpoint>:<quarter>`. Plan-wide errors use `plan`. Completed acquisition/storage stages are retained. Synthetic two-process and fault-injection results are recorded in ignored `runs/task_1_5b_followup_focused.xml`; the full regression result is in `runs/task_1_5b_followup_full.xml`.

## Task 1.5 accepted real snapshot

The accepted recovery attempt `real20261111b` reused all 172 SHA-256-identical files first acquired by the failed `real20261111a` attempt, then passed final physical read-back and source-total/index probes. Its `runs/task_1_5_full_real20261111b.json` and matching summary are complete; `tests/real_full_acceptance.py` independently checked every source/query/count, Arrow schema, key, REPDTE partition, ZSTD codec, read-back, SHA-256, and Failures linkage audit. Aggregate evidence is in ignored `runs/task_1_5_full_real20261111b_acceptance.json`. All 22 Task 1.3/1.4/1.5A protected SHA-256 values were unchanged. The 172 full-run Parquet files occupy 191,632,376 bytes in this observed snapshot; this single raw-snapshot size is **not** the separate CSV/Snappy/ZSTD comparison below.

## Task 1.6 isolated benchmark representations

Task 1.6 is now complete on the accepted Financials snapshot. Its fixed logical schema is the 26 extended Financials source fields in their configured order, with Arrow types from `EXTENDED_FINANCIAL_BASE_FIELDS`. The physical source's `ID` and `SOURCE_MISSING_FIELDS` fields remain intact and are checked by the original `ParquetStore`; they are deliberately excluded from all three 26-field comparison outputs. The checked-in contract `config/storage_benchmark_v1.json` and source-derived ignored `runs/task_1_6_storage/frozen_spec.json` were frozen before conversion. The latter records exact logical types and the observed 262,595 selective-workload rows.

Benchmark copies live only in ignored `data_processed/storage_benchmark_v1/{csv,snappy,zstd}/REPDTE=YYYYMMDD/part-000.{csv,parquet}`. There are 170 files per format. CSV is uncompressed UTF-8/LF with explicit null marker, ISO dates, exact decimal strings, and round-trip float text; Snappy and ZSTD-3 Parquet use identical writer options apart from compression. Every copy passed row-exact read-back against the immutable source, including the 40-quarter projected workload. The 170-file stored-byte totals were 469,211,854 (CSV), 203,331,205 (Snappy), and 153,574,569 (ZSTD-3). Six warm-ups and 30 measured reads passed; the complete method, timings, environment, cache limitations, and report-ready P1.3 table are in STORAGE_BENCHMARK.md and ignored `runs/task_1_6_storage/`. The accepted raw Parquet files and their hashes were unchanged after the experiment.
