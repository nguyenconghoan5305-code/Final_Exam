# Part 6 Clean-Environment Reproducibility Audit

**Scope and status (2026-11-23):** This is the second Part 6 checklist gate. It separates live-source reacquisition, replay from the accepted source vintage, and verification of saved output. The original `final20261122a` remains the only predictive test evaluation. The original test-use registry, model, predictions, threshold, and accepted manifests are immutable. A fresh environment and a source snapshot were created for this audit; the full dependency-chain regeneration gate remains open until every required stage is demonstrated with independent outputs.

## Three distinct reproducibility modes

1. **A — live FDIC reacquisition:** `bft65b.acquisition.full --execute` requests the current official API, with its current index and totals, into a new raw namespace. It was **not run** here. A later API revision could change rows or values; matching the 2026-11-11 source vintage byte for byte is not guaranteed. The accepted live result is `real20261111b`, with 170 Financials partitions and complete Failures/Assistance and Institutions endpoints.
2. **B — captured-snapshot replay:** read the immutable `real20261111b` raw Parquet, original acquisition manifest/summary, saved supplementary FDIC transaction response and dictionary, captured Institutions/History responses and code descriptions, and regenerate downstream outputs under fresh ignored run IDs. This has no FDIC network request. The completed subset and dependency breaks are recorded below. Source manifests contain absolute `C:\NEU\BFT65B` paths, so the currently supported raw-snapshot replay resolves them at the original project root even while Python executes the independently copied source and clean venv. It is **not yet a relocatable, self-contained clone replay**.
3. **C — saved-artifact verification:** a stage's `verify` or `show` reads already accepted manifests/data and checks its gates. The `final_evaluation verify` command recomputes metrics from saved scores and hashes; it does **not** refit the model or regenerate test scores. Mode C must never be described as end-to-end regeneration.

## Clean setup, source snapshot, and protected inputs

An explicit allowlist copied **132 files, 1,911,605 bytes** into ignored `tmp/p6_repro_20261123a/repo/`: all `.py` under `src/` and `tests/`, all `config/*.json`, and top-level Markdown, `AGENTS.md`, `README.md`, `requirements.txt`, `pytest.ini`, `.gitignore`, `EVIDENCE_REGISTER.csv`, starter script and authoritative PDF. It includes current untracked implementation files; `.pyc`, credentials, datasets, existing runs, virtual environments and unrelated temporary files were excluded. `repo/source_inventory.json` records every relative path, byte count and SHA-256 **before the documentation edits of this milestone**. The 96 source/config/test/requirements/pytest files checked after execution still matched that inventory; README and status documents changed intentionally afterward. This source snapshot is not a Git checkout; a Git-only checkout currently omits untracked code. No Git index change was made.

The initial `tmp/p6_repro_20261123a/protected_before.json` independently hashed **172 accepted raw outputs** from the `real20261111b` manifest and **nine** protected final-policy/plan/binding/summary/manifest/registry/model/score artifacts. Every raw hash matched its accepted manifest. A separate `supplementary_inventory.json` checked 45 captured Institutions/History source pages against the accepted active-audit manifest and hashed four additional transaction/dictionary/code files. The postcheck found **0 changes among 181 protected files and 49 supplementary files**. The accepted raw source paths and supplementary evidence are **inputs**, not newly generated outputs; none was copied into the source-only snapshot. Existing supplementary source paths and hashes must be carried into a fully isolated Mode B design before claiming it is portable.

`py -0p` returned “No installed Pythons found!” in this shell, but the existing venv's `pyvenv.cfg` identified an intact base interpreter at `C:\Users\asus\AppData\Local\Programs\Python\Python311\python.exe`. That interpreter created `repo/.venv_clean` with `include-system-site-packages = false`. One `pip install --disable-pip-version-check -r requirements.txt` succeeded; `pip check` reported **No broken requirements found**. Python is 3.11.9 on `Windows-10-10.0.26200-SP0`; direct pinned versions are pandas 2.2.3, Polars 1.35.2, PyArrow 18.1.0, PySpark 3.5.7, scikit-learn 1.6.1, requests 2.32.3, NumPy 2.1.3 and pytest 8.3.4. Installed transitive versions are in ignored `tmp/p6_repro_20261123a/pip_freeze.txt`. `requirements.txt` pins direct packages, **not all transitive packages**; this install selected newer transitive versions than the historical environment. Thus `pip check` establishes consistency, not bit-for-bit dependency reproduction.

JDK `17.0.21` was available. The existing `winutils.exe` and `hadoop.dll` from the verified local setup were copied, with matching SHA-256 values `A0CA6E358357C41EF56EBDB02C38E4A4D55DA7CA7A13001678BB2EF7D644ADEA` and `D3DD64AFDC85F2A7EB5345ABF2ECAA744B0A157DE40859313337D47F81EE1C7B`, into the clone's ignored `tmp/hadoop-3.3.5/bin/`. They were **not downloaded** again. The Spark fixture requires `HADOOP_HOME` under the source checkout's own `tmp/hadoop-3.3.5`, `PYSPARK_PYTHON` and `PYSPARK_DRIVER_PYTHON` pointing at the clean venv, and that `bin` prepended to the current process PATH. This host-specific binary requirement is not supplied by `requirements.txt`.

| Ignored source input | Present provisioning and independent check | Role in replay |
| --- | --- | --- |
| `data_raw/task_1_5_full/fdic_extended_v2/` plus `runs/task_1_5_full_real20261111b{.json,_summary.json,_acceptance.json}` and the unlinked-ID audit | Original complete FDIC API acquisition; 172 source-file SHA-256 values in the accepted manifest and `protected_before.json` | Immutable raw Financials, Failures/Assistance and Institutions snapshot. A current live reacquisition is Mode A, not this original vintage. |
| `runs/task_2_3_transaction_repair/source_failures_20261115.json`, `source_capture.json`, `failure_properties_20261115.yaml` | Official supplementary Failures transaction query and dictionary captured for `repair20261115f`; hashes in `supplementary_inventory.json` | RESTYPE/RESTYPE1 source-ID reconciliation; not included in the 132-file code snapshot or a source-only Git checkout. |
| `runs/task_4_2_active_membership/activeaudit20261119a/{institutions,history}/` and `activeaudit20261119e/run_manifest.json` | 45 official captured response pages checked against the later accepted audit manifest; source paths/hashes in `supplementary_inventory.json` | Retrospective effective activity evidence; not a public historical registry vintage. |
| `runs/task_4_2_membership_code_probe/codeprobe20261119a.json` | Saved official code-description probe, hashed in `supplementary_inventory.json` | History code semantics. |
| `tmp/hadoop-3.3.5/bin/{winutils.exe,hadoop.dll}` | Already verified community native files; two fixed SHA-256 values above, copied only into the ignored clone | Local Windows Spark fixture/runtime. Not a data input or Python package. |

An authorized reader can provision the original vintage by copying or read-only mounting each captured input with its manifest and independently validating hashes; copying accepted **downstream** output files into a fresh run directory is not regeneration. The current code binds many absolute source/output paths, so such a copy alone does not make a different project root valid. No accepted manifest was edited to camouflage relocation.

The clean copy's `python -m bft65b --help` passed. The first full pytest attempt, with `HADOOP_HOME` pointed to the original root, recorded **455 passed, three failed, three errors** in `full_suite.log`/`.xml`: three acquisition fixture writes hit `WinError 5` during temporary-manifest replacement, and three Spark MLlib cases rejected a `HADOOP_HOME` outside the clone root at setup. After copying the hash-verified native files and setting the documented clone-local path, the three failed acquisition cases and entire Spark MLlib test module passed together (**11 passed**). A second fresh, sequential full suite passed **461 tests, zero skips**, with 36 PySpark deprecation warnings. These retries establish the observed result; they do not prove the cause of the transient `WinError 5`. The first failure evidence is retained.

## Stage inventory and execution contract

Commands below are module suffixes; invoke them as `python -m bft65b.<module> ...` with the clean interpreter, `PYTHONPATH` set to the source snapshot's `src`, and the indicated `--project-root`. “Accepted dependency” is an ignored input that a bare source checkout lacks. The README contains individual stage commands but currently lacks one portable end-to-end command and frequently uses `C:\NEU\BFT65B`, `.venv_local`, and accepted run IDs literally. The table records actual required inputs rather than treating these defaults as a portable execution plan.

| Stage | Documented command / code and configuration | Required source or upstream input | New output and gate | This audit |
| --- | --- | --- | --- | --- |
| P1 raw acquisition | `acquisition.full --config config/financials_full_v2.json --attempt NEW --execute`; API client, typed store, full runner | Live FDIC Financials, Failures/Assistance, Institutions; field dictionaries | 172 Parquet jobs, source totals/index, manifest, schema, keys, codec and SHA-256 | Mode A not run; accepted `real20261111b` used as immutable Mode B input |
| P1 storage experiment | `benchmarks.storage preflight/prepare/parity/run/verify`; frozen benchmark spec | Accepted raw manifest and 170 Financials files | Separate CSV/Snappy/ZSTD files, six parity gates and 30 timed reads | Not rerun; prior timings are accepted historical measurements, not clean-host measurements |
| P2 raw audit | `quality.audit audit --attempt real20261111b --run-id NEW`; `config/financials_full_v2.json` | Accepted 172 raw files, full manifest, summary and acceptance evidence | Findings, report, manifest; source hashes/schema/row checks | Regenerated from immutable raw source; see run log below |
| P2 treatment | `quality.treatment run --audit-run NEW --run-id NEW` | Fresh audit findings and accepted raw snapshot | 172 flagged derived Parquet outputs and before/after log; read-back/checksums | Regenerated; `repro20261123a_treatment` complete |
| P2 observed panel | `panel.build run --treatment-run NEW --run-id NEW` | Fresh treatment outputs | 170 bank-quarter partitions plus event, exit and gap evidence; key/reconciliation gate | Regenerated; `repro20261123a_panel` complete; exit-assessment semantics differ from historical accepted panel |
| P2 transaction repair | `panel.repair run --source-json SAVED --run-id NEW` | Captured supplementary Failures response, dictionary and panel | Repaired source-ID event and exit tables; one-to-one source-ID gate | Runner fixes `PREVIOUS_RUN=panel20261115b`, so it cannot consume a fresh panel without a scoped refactor |
| P2 as-of application | `panel.prediction run --run-id NEW` | Accepted raw, treatment, panel and repaired events | 166 prediction-quarter partitions; report availability and key checks | Hard-coded accepted upstream run IDs; not yet chained to fresh outputs |
| P3 Polars, pandas, Spark | `processing.polars_panel run`; `benchmarks.panel run --include-spark`; Part 3 review | Accepted treatment/panel reference and native Spark files | Observed/gap/quality relations, exact three-engine parity, plans and measured fresh-process times | Not rerun as a chained clean replay; prior benchmarks remain historical |
| P4 cohort and labels | `panel.retrospective run`; `features.production run --membership-run NEW`; feature contract | Captured Institutions/History/code descriptions, accepted observed panel, PIT and repaired FDIC events | Membership/label quarter tables, uncertainty, four-quarter target and hashes | Fixed source run IDs and ignored supplementary inputs block an isolated fresh chain |
| P4 feature table and audit | `features.table run --run-id NEW`; `features.leakage_audit run --run-id NEW` | Accepted panel, PIT, contract, membership, label and independent repeat | Seven-feature table, key/label reconciliation, future-perturbation gate | Fixed accepted upstream IDs; not chained to fresh outputs |
| P5 split and development | `modelling.split run`; `modelling.development plan/execute`; Spark demo | Accepted feature/label/leakage artifacts, frozen policy and native Spark files | Chronological assignments, folds, OOF predictions and MLlib demo | Not rerun; must preserve frozen selection and test boundary |
| P5 cost and threshold | `modelling.cost_audit run`; `modelling.threshold plan/execute` | Accepted development event linkage/COST, OOF scores and scenario | Unique-event COST audit, full empirical curve, locked reference threshold | Not rerun; accepted lock remains immutable |
| P5 final test | `modelling.final_evaluation verify --run-id final20261122a` | Accepted policy, model, score, bindings, registry, 171 upstream hashes | **Read-only** integrity and metric recomputation | No `plan`, `fit` or `evaluate` command is allowed in this milestone |

The assignment PDF, page 2, says the pipeline must rerun end to end from README. Hard-coded source IDs across `panel.prediction`, `processing.polars_panel`, `features.table`, `modelling.split`, `modelling.development`, `modelling.threshold` and other modules are a **measured implementation dependency**, not a PDF requirement. The bounded panel-repair dependency has been made explicit below. A complete independent chain still needs configurable upstream run references at the remaining edges. Changing only a README placeholder or relabelling old manifests would not satisfy the gate. No accepted manifest was rewritten to make a path appear valid.

## Replay results, comparison, and boundary

The clean source snapshot and clean interpreter executed against the original **immutable raw source paths** with fresh ignored IDs. Audit `repro20261123a_audit` produced 1,682,616 Financials, 4,117 Failures/Assistance, 27,835 Institutions records and 1,687 period findings. Its `findings.json` SHA-256 is `eaf3f9cee2952eedef1c3a3a38eba63accdf21b813d6baa671adbeb62567de01`, exactly matching accepted `audit20261114a/b`; source hashes remained unchanged. Treatment `repro20261123a_treatment` consumed **that fresh audit** and created 172 new Parquet outputs. Its `summary.json` and `treatment_log.json` SHA-256 values exactly match accepted `treatment20261114a`: `8c93be21cd5474529148dac89d722565dbc4fdd9a0eaa6a6ccf069817e262235` and `fbcb82c7c3c84eb7eea3b7ef652429606866c69e4d0465940ae093d2df0ce12a`. All three endpoint row totals and null/absence counts remained unchanged.

Panel `repro20261123a_panel` consumed the **fresh treatment**, passed its run gate, and wrote 1,682,616 observed bank-quarter rows, 4,117 source events, 24,064 last-report assessments and 33 resumed-gap events. All 173 output keys and row counts match accepted `panel20261115b`; **172 of 173 physical output SHA-256 values match**. The exception is `exit_assessments`. Its accepted baseline has 2,775 `confirmed_failure`, 16,976 `ambiguous_exit` and 4,313 `boundary_observed` banks; the current-code replay has 0, 19,758 and 4,306, respectively. Aggregate transitions are 2,775 former confirmations to ambiguous and seven former boundary observations to ambiguous. The accepted old panel predates the separate FDIC transaction repair and boundary-precedence changes; the current panel without supplementary transaction evidence conservatively cannot confirm those failures. `tmp/p6_repro_20261123a/panel_comparison.json` records the aggregate comparison without bank IDs. **This is a real semantic/version difference**, not something to hide with a row-count-only parity check. `repair20261115f` is the accepted later event classification. The repair runner fixes `PREVIOUS_RUN=panel20261115b`, so it cannot consume `repro20261123a_panel` through its CLI; the fresh chain stops there. All three fresh manifests and CLI `show` checks report complete for the work actually done.

```powershell
$snapshot = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo'
$python = Join-Path $snapshot '.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $snapshot 'src'
& $python -m bft65b.quality.audit audit --project-root C:\NEU\BFT65B --config C:\NEU\BFT65B\config\financials_full_v2.json --attempt real20261111b --run-id repro20261123a_audit
& $python -m bft65b.quality.treatment run --project-root C:\NEU\BFT65B --config C:\NEU\BFT65B\config\financials_full_v2.json --audit-run repro20261123a_audit --run-id repro20261123a_treatment
& $python -m bft65b.panel.build run --project-root C:\NEU\BFT65B --config C:\NEU\BFT65B\config\financials_full_v2.json --treatment-run repro20261123a_treatment --run-id repro20261123a_panel
```

The three commands above are **records of commands already executed**. Their run IDs now exist and must not be rerun as fresh attempts. Choose new IDs for an independent repeat. Their command/exit-code logs are `tmp/p6_repro_20261123a/{audit_replay,treatment_replay,panel_replay}_execution.json` with companion `.log` files. The measured outer command durations were **388.301 s** for audit, **998.996 s** for treatment and **95.052 s** for panel. These are descriptive runs on the same host with OS caches and other activity uncontrolled, not format/engine benchmark measurements. `repro_summary.json` records the timings, run paths, before/after hash checks and panel comparison.

The accepted final `final20261122a` passed the clean interpreter's **read-only** `final_evaluation verify --project-root C:\NEU\BFT65B --run-id final20261122a` (exit 0). A separate clean-environment calculation from saved `test_scores.parquet`, the locked threshold and assumptions matched AP **0.1729890328959421**, ROC-AUC **0.8769916284926879**, Brier **0.0006590848016157432**, TP/FP/TN/FN **109/1,483/244,422/69**, reference surrogate **0.9555819665316173** and fixed never-alarm surrogate **0.7827592730907865** within `1e-12`. Evidence is `tmp/p6_repro_20261123a/independent_saved_score_metrics.json` and `final_readonly_verify_execution.json`. This remains **verification**, not newly generated predictions. The registry is still `complete` with exactly **one** `test_access` transition. The authoritative PDF requires one test use; it does not require a second predictive use for reproducibility. If a separate frozen-design score reproduction is ever requested, it must be explicitly labelled a reproduction against a test already used and governed by a separate protocol. It cannot be called an untouched test.

## Reproducible commands and evidence locations

```powershell
Set-Location C:\NEU\BFT65B
$snapshot = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo'
$python = Join-Path $snapshot '.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $snapshot 'src'
$env:PYTHONDONTWRITEBYTECODE = '1'
$env:HADOOP_HOME = Join-Path $snapshot 'tmp\hadoop-3.3.5'
$env:PATH = (Join-Path $env:HADOOP_HOME 'bin') + ';' + $env:PATH
$env:PYSPARK_PYTHON = $python
$env:PYSPARK_DRIVER_PYTHON = $python
& $python -m pip check
& $python -m pytest -q -rs -p no:cacheprovider --basetemp C:\NEU\BFT65B\tmp\p6r_verify_new
# Safe, read-only verification of the original one-use test evidence:
& $python -m bft65b.modelling.final_evaluation verify --project-root C:\NEU\BFT65B --run-id final20261122a
```

The frozen source inventory, protected hash inventory, `pip_install*.log`, `pip_freeze.txt`, all first and second full-suite logs/JUnit and command records, targeted retry evidence, and stage execution logs are under ignored `tmp/p6_repro_20261123a/`. New derived replay outputs are under distinct ignored `runs/task_2_1_quality/`, `runs/task_2_1_treatment/` and `data_processed/task_2_1_treatment/` run IDs. The native binaries are present only in ignored local tools directories. No generated data, model, environment or evidence file is a tracked report attachment.

## Bounded portable source-to-transaction replay (2026-11-23)

This follow-up uses ignored `tmp/p6_repro_20261123b/portable/` as a **separate project/data root**. `quality.source_vintage provision` verified the original completed 172-job acquisition and copied **172 raw Parquet files plus eight source-evidence/configuration files**, 180 files total. The original manifest, summary, acceptance report and unresolved-linkage audit were copied **byte-for-byte**. The new `runs/source_vintage_bundle.json` (SHA-256 `5b3df440f1b153f44a0221acab5470f19abac00eb1ae7c6da9f59fd1725f3385`) records relative paths, original-path provenance, sizes, SHA-256 and the accepted acquisition attempt. The original absolute paths are mapped **only in memory** after the copied evidence and every raw file pass independent hashes; original manifests were not edited. The captured official Failures transaction response, capture metadata and YAML dictionary were copied with the bundle. A separate `source_code_inventory.json` records the Python source copy; the installed clean venv is outside the portable data root and is never a dataset input. No FDIC network call occurred.

The existing clean CPython 3.11.9 environment has 25 installed packages. `requirements-lock-win-py311.txt` pins all 25 and matched `pip freeze` exactly (**25/25, zero differences**); `pip check` passed. PySpark 3.5.7, Temurin JDK 17.0.21 and the previously verified local Hadoop native hashes remain the tested Windows runtime. No new binaries, system changes or package installation were needed for this bounded replay. The lock pins package versions but not platform-specific wheel hashes or JDK/native-tool binaries.

| Regenerated stage | Selected input | Actual new output and result |
| --- | --- | --- |
| P2.1 audit `portable20261123b_audit` | Copied accepted `real20261111b` raw snapshot; mapped original source manifest and bundle hash | 172 raw inputs all inside portable root; 1,682,616 / 4,117 / 27,835 rows by endpoint; 1,687 period findings; findings SHA-256 `eaf3f9cee2952eedef1c3a3a38eba63accdf21b813d6baa671adbeb62567de01`, equal to accepted audit |
| P2.1 treatment `portable20261123b_treatment` | **New audit** and same source bundle | 172 newly generated Parquet files; summary and treatment-log hashes `8c93be21cd5474529148dac89d722565dbc4fdd9a0eaa6a6ccf069817e262235` and `fbcb82c7c3c84eb7eea3b7ef652429606866c69e4d0465940ae093d2df0ce12a`, equal to accepted treatment |
| P2.2 panel `portable20261123b_panel` | **New treatment** | 173 outputs; 1,682,616 bank-quarters, 4,117 source events, 24,064 bank assessments, 33 resumed gaps. Before transaction repair: 0 confirmed, 19,758 ambiguous and 4,306 boundary-observed banks. |
| P2.3 repair `portable20261123b_repair` | **New panel** plus copied official RESTYPE/RESTYPE1 evidence | 4,117/4,117 source IDs matched uniquely, zero extra IDs; 3,522 verified-failure events, 580 assistance, 13 unverified-code assistance and two charter-uncertain failures. Repaired bank counts: 2,534 confirmed, 17,217 ambiguous, 4,313 boundary-observed. |

The old `panel20261115b` had 2,775 confirmed failures and seven additional boundary-observed banks relative to the current un-repaired panel. The new repair does **not** restore those obsolete pre-repair classes as an input assumption: it applies the verified transaction source to all 4,117 events. It moves 2,534 ambiguous banks to confirmed failure and seven ambiguous banks to boundary-observed; assistance and uncertain records remain explicit. Read-only `panel.compare_repair` sorted by source ID/CERT and compared **full schemas and every keyed field value** against accepted `repair20261115f`: both 4,117-row event and 24,064-row assessment outputs passed, with identical category and disposition counts. Their two physical Parquet SHA-256 values also match the accepted repaired outputs. `repair.show` passed for both accepted and portable runs.

The first portable panel attempt exited 1 **before creating either panel output directory**: a newly added configuration check compared JSON lists in the manifest directly to Python tuples. Its log and execution record are retained as `panel_failed_config.log` and `panel_failed_config_execution.json`. A bounded fix compares canonical JSON representations while still rejecting altered field lists; the synthetic regression passed, and the same unused run ID then passed. This failure was not treated as a successful attempt. The stage-by-stage hashes, input/output counts, selected upstream IDs and root confinement are in ignored `tmp/p6_repro_20261123b/lineage_check.json`: every newly generated manifest is complete, all validations pass, all input/output paths stay inside the portable root, and **zero** stored hashes differ. `repair_comparison.json` contains the keyed parity result without raw identifiers.

The read-only postcheck `protected_postcheck.json` found **0 changed among 181 accepted raw/final artifacts, 49 supplementary files and 180 original source-bundle inputs**; the project-level registry still has exactly one `test_access` transition. The previous one-use final test was neither refitted nor rescored. The original first-replay evidence and the failed panel attempt remain intact. Focused synthetic relocation/repair tests passed **27/27**; the first clean full suite passed **468/468** before the panel-config fix. The affected focused panel/bundle group then passed **20/20**, and the final sequential clean full suite passed **469/469, zero skipped**, with 36 PySpark deprecation warnings. Commands, JUnit XML and actual stdout are saved in ignored `tmp/p6_repro_20261123b/{focused,full,full_after_fix}.*`, `*_execution.json` and stage `.log` files; README contains fresh-ID provisioning and replay commands. The early synthetic comparator-fixture error and the first portable panel failure are retained as failed attempts rather than counted as acceptance.

## Bounded portable point-in-time replay (2026-11-24)

The earlier raw-to-panel blocker in the first clean-environment attempt is historical: it was resolved by the portable raw-to-transaction-repair follow-up above. This PIT continuation reused those four completed portable runs without modifying or rerunning them. `source_code_inventory_before_pit.json` preserves the prior 53-file source inventory; `portable/source_code_inventory.json` records the deliberately updated PIT runner source snapshot. The clean Python 3.11.9 venv and 25-package pinned lock were reused. The accepted source bundle hash remains `5b3df440f1b153f44a0221acab5470f19abac00eb1ae7c6da9f59fd1725f3385`.

`src/bft65b/panel/prediction.py` now requires explicit raw/treatment/panel/repair run IDs when a bundle is present, validates their audit-to-repair lineage and bundle hashes, and uses the verified in-memory raw-path mapping. Historical no-bundle CLI defaults and saved-run verification remain supported. The 120-calendar-day report rule, 2025Q2 prediction cutoff, candidate dispositions and 23 PIT predictor columns did not change. Saved verification reconstructs the selected upstream chain from recorded run IDs and reconciles manifest inputs with metadata hashes. A separate read-only `panel.compare_pit` compares every column by `(CERT, Q)` across roots.

The execution tool lost control of the first foreground process after it had written 33 quarter-progress records. A detached second process was observed with a `running` manifest and zero outputs immediately after its shell returned. **Those intermediate observations were not final outcomes:** later read-only inspection found `portable20261124a_pit` and `portable20261124b_pit` both had complete manifests, summaries and 166 outputs. They remain separate occupied run IDs; their full saved-run gates were not used for acceptance. The monitored `portable20261124c_pit` completed and was independently verified. It produced **166** ZSTD Parquet files and **3,697,094** unique candidate bank-quarters through 2025Q2: 3,335,808 eligible, 322,352 excluded for a prior verified failure, 38,934 unresolved identity and zero no-history candidates by the report-derived roster definition. These are PIT **candidates**, not a verified historical active-bank cohort or labels. All **528** recorded input hashes (including the portable source-bundle mapping) and 166 output read-backs passed; 528 input plus 166 output paths remained inside the portable root. The accepted `pit20261115f` verifier rechecked 527 input hashes. `tmp/p6_pit_20261124/parity.json` reports **166/166** quarter coverage, **3,697,094** keyed rows, zero full-schema/value/quarter-summary differences and **166/166** equal physical Parquet hashes against the immutable accepted PIT run. Different summary hashes reflect run-ID/provenance paths.

Focused synthetic portable-lineage, path-escape, full-column comparator, saved-run verifier, as-of and source-vintage tests passed **47/47** (`tmp/p6_pit_20261124/focused_final.xml`). The first expanded focused attempt stopped during collection on a comparator parenthesis typo; its failed JUnit `tmp/p6_pit_focus_04.xml` is retained, the typo was fixed, and the final focused rerun passed after a saved-run verifier case was added. The first clean full suite passed **481/481**; after that final test addition the sequential clean full suite passed **482/482, zero skipped**, with 36 PySpark deprecation warnings (`tmp/p6_pit_20261124/full_final.log` and `full_final.xml`). JDK 17.0.21 and the existing hash-verified `winutils.exe`/`hadoop.dll` were configured only for those test processes. `tmp/p6_pit_20261124/integrity_postcheck.json` found zero changes among 181 protected raw/final, 49 supplementary and 180 original bundle-source files; portable audit/treatment/panel/repair inputs and outputs retained zero path/hash failures. The final registry still has exactly one `test_access` transition. No FDIC request, cohort reconstruction, label generation, model fit or final-test inference occurred.

For the established checkpoint, the executed command was:

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.panel.prediction run --project-root $portable --run-id portable20261124c_pit --raw-run real20261111b --treatment-run portable20261123b_treatment --panel-run portable20261123b_panel --repair-run portable20261123b_repair
& $python -m bft65b.panel.prediction verify --project-root $portable --run-id portable20261124c_pit
$env:PYTHONPATH = 'C:\NEU\BFT65B\src'
& $python -m bft65b.panel.compare_pit --left-root C:\NEU\BFT65B --left-run pit20261115f --right-root $portable --right-run portable20261124c_pit --output C:\NEU\BFT65B\tmp\p6_pit_20261124\parity.json
```

The first command above is an execution record and must **not** be rerun with the occupied ID. README provides a fresh-ID continuation. The bounded next replay gate is explicit source selection and parity for the retrospective cohort, followed separately by labels and features; this task did not start those stages.

The final focused and full test commands (run sequentially with fresh ignored temp paths) were:

```powershell
Set-Location C:\NEU\BFT65B
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = 'C:\NEU\BFT65B\src'
& $python -m pytest tests/test_portable_pit.py tests/test_panel_prediction.py tests/test_panel_asof.py tests/test_source_vintage.py -q -p no:cacheprovider --basetemp C:\NEU\BFT65B\tmp\p6_pit_focus_06 --junitxml C:\NEU\BFT65B\tmp\p6_pit_20261124\focused_final.xml
$env:PYTHONDONTWRITEBYTECODE = '1'
$env:HADOOP_HOME = 'C:\NEU\BFT65B\tmp\hadoop-3.3.5'
$env:PATH = (Join-Path $env:HADOOP_HOME 'bin') + ';' + $env:PATH
$env:PYSPARK_PYTHON = $python
$env:PYSPARK_DRIVER_PYTHON = $python
& $python -m pytest -q -rs -p no:cacheprovider --basetemp C:\NEU\BFT65B\tmp\p6_pit_full_02 --junitxml C:\NEU\BFT65B\tmp\p6_pit_20261124\full_final.xml *> C:\NEU\BFT65B\tmp\p6_pit_20261124\full_final.log
```

Both exited 0. The final suite ran 482 tests in 280.64 s. `git diff --check` exited 0, and `git ls-files -- data_raw data_processed runs tmp .venv .venv_local` returned no tracked files. The existing staged diff still has trailing blank-line warnings in `PROJECT_STATUS.md` and `TASKS.md`; this task did not alter the Git index. `git check-ignore -v` confirmed the portable PIT Parquet and new JUnit evidence are excluded by `tmp/`.

## Portable membership-source and retrospective-cohort checkpoint (2026-11-24)

The prior raw source-vintage bundle remains unchanged. A separate ignored `runs/membership_source_bundle_v1.json` (SHA-256 `d457558e0f697aa72f83422ae4e7d7bdd6d0e5ecc320013c9ae38fa638654536`) maps 49 captured files: 45 Institutions/History response pages, their original source summary and manifest, the code-description probe, and the assignment PDF. The original capture evidence was copied byte-for-byte, with absolute paths relocated only in memory after hash validation. There was no live FDIC request. The prior portable source-code inventory is preserved as `source_code_inventory_before_membership.json`; the updated inventory contains 54 files.

`portable20261124aaudit` reanalysed those captured responses against the explicit portable panel, PIT and repaired-event runs. Its substantive summary agrees with `activeaudit20261119e`; the run ID, repaired-event source ID and timestamp differ as expected. The portable audit manifest records the selected run IDs and both source-bundle hashes. An initial cohort invocation failed before creating outputs because the historical audit validator expected 45 manifest inputs, whereas the portable audit correctly records the extra membership bundle as a 46th input. The corrected cohort runner checks 46 in portable mode and retains the historical 45-input rule.

`portable20261124acohort` regenerated 166 quarterly membership files plus one bank-evidence file from the portable audit, panel and PIT selection. Saved read-back validated 167/167 ZSTD Parquet files, schemas, keys, row counts, checksums and source hashes. All 167 physical hashes agree with accepted `membership20261120a`, and the substantive summary is equal after excluding run-specific lineage, paths and timestamps. The 3,994,624 bank-quarter decisions are 1,660,450 eligible, 2,298,952 ineligible and 35,222 uncertain. Among eligible decisions, 54,585 lack available feature history; this is not recoded as inactivity. The study design remains a retrospective activity cohort, not a claim that the historical registry was publicly available at each `q`.

Focused cohort tests passed **12/12** after a retained sandbox-denied temp-directory attempt; the sequential clean full suite passed **484/484, zero skipped**, with 36 PySpark deprecation warnings in 402.81 seconds. The successful suite's log and JUnit are `tmp/p6_cohort_full_01.log` and `tmp/p6_cohort_full_01.xml`. The clean suite used the documented hash-verified local Hadoop natives and a fresh short ignored `--basetemp`. These tests did not use the real final test predictors.

The executed portable commands used the clean Python environment and fresh ignored run IDs:

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.panel.membership_source_vintage verify --project-root $portable
& $python -m bft65b.panel.active_audit --project-root $portable --run-id portable20261124aaudit --verify
& $python -m bft65b.panel.retrospective verify --project-root $portable --run-id portable20261124acohort
```

The first two run commands above are historical execution records and must not be repeated with occupied IDs. For a new replay, use fresh IDs and pass `--from-run activeaudit20261119a --panel-run <portable-panel> --pit-run <portable-pit> --repair-run <portable-repair>` to `active_audit`, followed by `retrospective run --audit-run <new-audit> --panel-run <portable-panel> --candidate-run <portable-pit> --code-probe codeprobe20261119a`.

## Portable production-label checkpoint (2026-11-24)

`production.py` now takes an explicit `--repair-run`. In portable mode, it requires the chosen repair run to match the audit lineage recorded by the chosen membership run. Historical defaults remain available for verification of accepted runs. An incorrect or omitted portable repair ID fails before output creation. The prior portable source-code inventory was preserved as `source_code_inventory_before_labels.json` before the updated runner was copied into the separate root.

`portable20261124alabels` consumed only `portable20261124acohort` and `portable20261123b_repair` as derived execution inputs. It generated 166 quarterly label files containing 3,994,624 decisions: 10,117 positive, 1,647,629 negative, 37,023 unresolved and 2,299,855 excluded. The feature-history subset includes 9,934 positive and 1,593,249 negative bank-quarters. Saved read-back passed schema, quarter/key uniqueness, ZSTD codec, input hashes and all 166 output hashes. All 166 physical hashes and the substantive summary agree with accepted `labels20261120a`; only lineage, path, timestamp and run ID differ. Focused production-label/label-rule tests passed **15/15**. This is regeneration from the captured vintage; the accepted label files were read only afterward for comparison.

The subsequent sequential clean full suite passed **485/485, zero skipped**, with 36 PySpark deprecation warnings in 347.06 seconds. Its ignored log and JUnit are `tmp/p6_labels_full_01.log` and `tmp/p6_labels_full_01.xml`. The source-code inventory before the label runner update is preserved as `source_code_inventory_before_labels.json`.

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.features.production verify --project-root $portable --run-id portable20261124alabels
# For a new replay, use fresh IDs and the independently verified upstream runs:
& $python -m bft65b.features.production run --project-root $portable --run-id <fresh-label-run> --membership-run portable20261124acohort --repair-run portable20261123b_repair
```

The second command is a template and was not executed with a placeholder. The executed run used `--run-id portable20261124alabels` and the same explicit upstream flags. Final test predictors, scores and registry were not touched.

## Portable seven-feature and model-table replay checkpoint (2026-11-24)

The apparently interrupted `portable20261124afeatures` run was **active**, not orphaned: a current process check found its Python writer and the candidate-quarter count increased from 151 to 153 while its manifest remained `running`. The writer records its manifest every 20 quarters, so the 160 recorded outputs observed in a prior review were not a completion test. No competing writer or replacement run was launched. The original process exited 0 and committed a complete manifest with 332 output files.

The feature contract was regenerated as `portable20261124acontract` from the portable panel and PIT. Full saved acceptance checked its complete manifest, seven-feature specification, selected lineage, 180 input hashes confined to the portable root, and output checksum; its substantive definitions equal accepted `featurecontract20261116b`. A second portable cohort/label pair (`portable20261124bcohort`, `portable20261124blabels`) was independently generated for the existing P4.2 review gate. The new `portable20261124areview` compares both pairs: zero differences among 167 cohort and 166 label physical hashes, matching summaries, and the 2025Q2 PIT baseline of 21,214. Historical accepted outputs were not execution inputs.

`portable20261124afeatures` consumed explicit portable panel, PIT, contract, membership, labels and review IDs. Saved verification checked all 332 ZSTD Parquet outputs, schemas, quarter/key uniqueness, input hashes and root confinement. Its 166 quarters contain 3,697,094 feature-candidate keys and 1,603,183 model-ready keys: 9,934 positive and 1,593,249 negative. Candidate/model missingness counts and the full substantive summary match accepted `featuretable20261120a`; **332/332 physical SHA-256 hashes match**. The seven predictor definitions and order, 120-calendar-day availability assumption, calendar-quarter rolling rules and pre-existing exclusions were preserved.

Source changes after the previous 485-pass suite were limited to explicit portable upstream run selection, audit/repair lineage and input-root checks in `features/contract.py`, `features/table.py`, `panel/retrospective_review.py`, `panel/retrospective.py` and `features/production.py`, plus synthetic tests. The prior `source_code_inventory_before_features.json` remains in the portable root. An initial focused run had 33 passes and one test-fixture failure because the new review test patched a class not exported by that module; the patch target was corrected, the single test passed, and the final focused suite passed **45/45** (`tmp/p6_feature_focus_final.log`, `.xml`). These fixtures did not touch accepted data.

The first post-feature full suite (`tmp/p6_feature_full_final.*`) had **486 passed, two failed**; both failures were Windows `PermissionError [WinError 5]` at `RunManifest.write()`'s same-directory `os.replace` in synthetic pilot tests. A fresh isolated retry of their parametrized case passed **3/3**. A separate pilot-module run (`tmp/p6f_pilotmodule.*`) had **21 passed, one failed** with the same error in another test; that case passed **1/1** in isolation. A second full suite (`tmp/p6f_full_retry.*`) had **487 passed, one failed**, again the same Windows replacement error in a different pilot case. The test's destination and temporary file were inside a fresh short ignored directory, and `RunManifest.write()` closes its file handle before promotion. These observations demonstrate an intermittent Windows replacement denial under suite load, but do not establish which external process, if any, held the file.

A bounded repair in `src/bft65b/manifest.py` retries **only** Windows `PermissionError` with `winerror == 5` for the manifest's atomic replacement, at most five total attempts over at most 0.5 seconds of backoff. Any other error, or persistent denial, still fails and removes the temporary file without changing the previous manifest. New synthetic cases exercise one transient denial and persistent denial; the manifest plus complete pilot module passed **26/26** (`tmp/p6f_manifest_focus.*`). `source_code_inventory_before_manifest_retry.json` preserves the prior portable code snapshot. No data, accepted run or final-test registry was changed to make the tests pass.

The final sequential clean full suite passed **490/490, zero skipped**, with 36 PySpark deprecation warnings in 343.48 seconds (`tmp/p6f_full_afterfix.log`, `.xml`). The protected-hash check saved `tmp/p6_repro_20261123b/feature_protected_postcheck.json`: zero changes among 181 protected raw/final, 49 supplementary and 180 original raw-bundle source files; the registry still records exactly one `test_access` transition. Both portable bundles verified (180 raw-bundle files and 49 membership-bundle files), and a separate check found zero changes among the 49 original membership-source files (`tmp/p6_feature_membership_originals_check.json`). Working-tree `git diff --check` exited 0; the pre-existing staged diff retains trailing-blank-line warnings in `PROJECT_STATUS.md` and `TASKS.md`. No data/run/temp/environment files are tracked by Git.

Read-only saved verification commands:

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.features.contract show --project-root $portable --run-id portable20261124acontract
& $python -m bft65b.features.table verify --project-root $portable --run-id portable20261124afeatures
```

The historical generation used `features.contract validate --run-id portable20261124acontract --panel-run portable20261123b_panel --pit-run portable20261124c_pit`, then `features.table run --run-id portable20261124afeatures --panel-run portable20261123b_panel --pit-run portable20261124c_pit --contract-run portable20261124acontract --membership-run portable20261124acohort --label-run portable20261124alabels --review-run portable20261124areview`, always with `--project-root $portable`. Those occupied IDs must not be reused for execution. New replays require fresh run IDs and the same explicit upstream selection.

## Remaining acceptance and submission work

## Portable leakage-audit replay checkpoint (2026-11-24)

The independent feature-table run `portable20261124bfeatures` finished with a complete manifest. Its verifier passed schema, keys, codec, read-back and input hashes for 332 Parquet files. All 332 output hashes, quarter counts, overall counts and null counts agree with `portable20261124afeatures`. Both runs were generated from the explicitly selected portable panel/PIT/contract/cohort/label/review chain; neither copied accepted feature output.

`portable20261124aleakage` then consumed the two portable feature runs and their explicitly selected ancestors. Saved verification passed 9/9 named checks, 1,191 unchanged upstream-file hashes, full alignment over 3,697,094 candidate keys and 1,603,183 model keys (9,934 positive; 1,593,249 negative), plus 24 deterministic source-history recomputations with future-data perturbations. Its substantive overall, null, per-quarter and sample-category counts agree with accepted `p44audit20261120a`. The audit is sampled for independent feature recomputation; it does not claim to recompute every feature value from raw reports. The first invocation failed before evidence creation with Windows `PermissionError [WinError 5]` at the portable audit directory; the authorized retry used the same ID and completed. It did not alter upstream outputs.

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.features.table verify --project-root $portable --run-id portable20261124bfeatures
& $python -m bft65b.features.leakage_audit verify --project-root $portable --run-id portable20261124aleakage
```

The occupied audit run used `features.leakage_audit run --project-root $portable --run-id portable20261124aleakage --panel portable20261123b_panel --pit portable20261124c_pit --feature-contract portable20261124acontract --membership portable20261124acohort --labels portable20261124alabels --feature-table portable20261124afeatures --independent-repeat portable20261124bfeatures --p42-review portable20261124areview`. A new run requires a fresh ID. Focused leakage/feature tests passed **16/16** (`tmp/p6_leak_focus_02.log`, `.xml`). The sequential clean full suite after this code change passed **491/491, zero skipped**, with 36 warnings in 340.15 seconds (`tmp/p6_leak_full.log`, `.xml`). An independent postcheck found zero changed files among 181 protected raw/final, 49 supplementary and 180 original bundle files; the registry still records one `test_access` transition (`tmp/p6_repro_20261123b/leak_protected_postcheck.json`).

## Portable chronological-split replay checkpoint (2026-11-24)

`modelling.split` now accepts explicit membership, labels, primary/repeat feature tables, leakage audit and repaired-event runs. Portable mode rejects the historical defaults, validates cross-stage lineage and root confinement, and records selected IDs, source vintage and input hashes. Historical verification remains compatible with the accepted run. The runner also rejects any result outside the frozen development 1984Q1–2012Q4, gap 2013Q1–2013Q4 and test 2014Q1–2025Q2 boundaries.

`portable20261124asplit` completed from the portable chain. Saved verification passed. A complete Arrow-table comparison with accepted `split20261120b` found identical schema and all row values for 1,603,183 assignment keys and 9,934 positive linkage rows; both physical Parquet SHA-256 values also match. Bounds, quarter counts, period counts, exclusions and selection rule agree. Development has 1,329,451 rows (9,677 positive; 2,462 distinct represented failures), gap 27,649 (79 positive), and test 246,083 (178 positive; 53 distinct represented failures). The gap has four complete quarters, and the development target right endpoint is 2013-12-31, before the test begins. This split construction reads only model keys/labels and accepted outcome linkage for feasibility; it does not read test predictors for fitting or scoring.

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.modelling.split verify --project-root $portable --run-id portable20261124asplit
```

The occupied execution used `modelling.split run --project-root $portable --run-id portable20261124asplit --membership portable20261124acohort --labels portable20261124alabels --model-table portable20261124afeatures --model-repeat portable20261124bfeatures --leakage-audit portable20261124aleakage --repaired-events portable20261123b_repair`. New executions require a fresh run ID. The ignored complete comparison is `tmp/p6_repro_20261123b/split_parity.json`; focused synthetic tests passed **21/21** (`tmp/p6_split_focus.log`, `.xml`). The sequential post-change clean full suite passed **494/494, zero skipped**, with 36 warnings in 379.15 seconds (`tmp/p6_split_full.log`, `.xml`). Protected postcheck: zero changes among 181 raw/final, 49 supplementary and 180 original bundle files; one historical test access (`tmp/p6_repro_20261123b/split_protected_postcheck.json`).

## Portable frozen-development replay checkpoint (2026-11-24)

`modelling.development` now accepts an explicit `--split-run` at plan time and records its selected split/model/label/leakage lineage in the frozen plan and manifest. The execution and saved verifier follow the recorded split rather than the historical constant; portable mode rejects the historical default and paths outside its root. Accepted historical verification remains compatible. The accepted folds, seven predictors, two candidate configurations, seed 42, two-thread limit and selection rule were unchanged. Only 116 development model partitions were opened; no gap/test predictor was loaded or scored.

`portable20261124adev` and independently generated `portable20261124bdev` both completed from `portable20261124asplit`. Each saved verifier passed. Both selected `bounded_hist_gradient_boosting`. The accepted `dev20261121a` comparison found identical fold/candidate/selection definitions, complete OOF schema, keys, labels and fold IDs for **409,094 validation rows per candidate**, with maximum score and predictive-metric difference **0.0** against a predeclared `1e-10` absolute score tolerance. The A/B comparator checked **818,188 scores** across both candidates and likewise found maximum difference **0.0**. Evidence: ignored `tmp/p6_repro_20261123b/development_parity.json` and `portable/runs/task_5_2_development/portable20261124bdev/repeat_comparison.json`. The 68 initial development quarters (920,357 rows) remain without OOF scores by design.

Timing is a separate observation: accepted aggregate fold fit/prediction times were 479.203/48.475 seconds, versus 153.232/13.405 seconds in portable A. Different execution conditions and cache states were not controlled, so these values are not a causal speedup comparison. The first focused test attempt had eight passes and one fixture failure because the synthetic project root lacked the assignment PDF required by the existing root validator. Adding the fixture file, without changing the gate, yielded **9/9** focused tests (`tmp/p6_dev_focus_retry.log`, `.xml`; first attempt retained as `tmp/p6_dev_focus.*`). The sequential clean full suite after code changes passed **495/495, zero skipped**, with 36 warnings in 409.00 seconds (`tmp/p6_dev_full.log`, `.xml`). The portable development source hash matches the tested repository source. A new protected postcheck found zero changed files among 181 raw/final, 49 supplementary and 180 original bundle files; the final registry still has one `test_access` transition (`tmp/p6_repro_20261123b/development_protected_postcheck.json`).

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.modelling.development verify --project-root $portable --run-id portable20261124adev
& $python -m bft65b.modelling.development verify --project-root $portable --run-id portable20261124bdev
```

For a new isolated replay use a fresh ID with `modelling.development plan --project-root $portable --run-id <fresh-development> --split-run portable20261124asplit`, inspect the frozen plan, then `modelling.development execute --project-root $portable --run-id <fresh-development>`. These occupied IDs must not be rerun. The development replay does not refit the final model or open final-test predictors.

## Portable Spark MLlib demonstration checkpoint (2026-11-24)

`spark_demo.py` now selects the portable split, feature table, two completed development runs and leakage audit explicitly, validates their cross-stage lineage, and records the selected IDs in new evidence. It retains historical accepted-run verification. The two existing trusted native files were copied into the portable root's ignored `tmp/hadoop-3.3.5/bin` with the recorded SHA-256 values (`winutils.exe` `a0ca6e35...d644adea`, `hadoop.dll` `d3dd64af...7f81ee1c7b`); no binary was downloaded. The current-process `HADOOP_HOME` pointed there. The native library loaded in JVM; the actual runtime was Windows, Python 3.11.9, Java 17.0.21, Spark 3.5.7, Hadoop 3.3.4, `local[2]` and two default parallel tasks.

Independent `portable20261124aspark` and `portable20261124bspark` regenerated the fixed fold-3 MLlib demonstration from portable inputs. Each saved verifier passed. Training covered 1,173,760 keys with 7,799 positive rows; validation covered 122,293 keys with 1,521 positives. Portable A measured AP 0.164908239636995, ROC-AUC 0.927045458568939 and Brier 0.0122797354509375. Full-key/label/score comparison with accepted `sparkdemo20261122a` found maximum score difference `2.220446049250313e-16` under a declared `1e-8` tolerance, identical training medians and metrics within `1e-12`; physical hashes differ because the `RUN_ID` provenance column differs. A/B repeat comparison found the same maximum score difference and identical fitted training statistics across all 122,293 scores. Evidence: ignored `tmp/p6_repro_20261123b/spark_accepted_parity.json` and B `repeat_comparison.json`.

Portable A newly measured 8.635 seconds for Spark median fitting, 26.490 seconds for pipeline fit and 3.952 seconds for transform plus full collect, with 99.258 seconds before session shutdown. These are local descriptive timings, not a three-engine benchmark or a cluster speedup estimate. Focused Spark tests passed **6/6**, zero skipped (`tmp/p6_spark_focus.log`, `.xml`); the sequential clean full suite after the code change passed **496/496**, zero skipped, with 36 warnings in 271.82 seconds (`tmp/p6_spark_full.log`, `.xml`). Protected postcheck: zero changed files among 181 raw/final, 49 supplementary and 180 original bundle files; one historical final-test access (`tmp/p6_repro_20261123b/spark_protected_postcheck.json`).

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.modelling.spark_demo verify --project-root $portable --run-id portable20261124aspark
& $python -m bft65b.modelling.spark_demo verify --project-root $portable --run-id portable20261124bspark
```

A new execution needs a fresh ID and the five flags `--split`, `--table`, `--development-primary`, `--development-repeat`, `--leakage`, along with the documented current-process Spark variables and hash-verified portable native files. The accepted scikit-learn model choice remains unchanged; the Spark demo did not score the final test.

## Portable development COST checkpoint (2026-11-24)

The final sequential clean suite after the COST and threshold lineage code changes passed **499/499, zero skipped**, with 36 PySpark deprecation warnings in 283.70 seconds (`tmp/p6_cost_threshold_full.log`, `.xml`). This supersedes the pending-result sentence below; it does not assert threshold execution has passed.

The COST runner now requires explicit split, primary/repeat development and repaired-event run IDs in a portable root. It validates their lineage, source-vintage bundle, dictionary units, source IDs, period boundaries, full input checksums and output read-back. Historical accepted-run verification remains available. `portable20261124acost` and independent `portable20261124bcost` each completed from `portable20261124asplit`, `portable20261124adev`, `portable20261124bdev` and `portable20261123b_repair`. Their 2,462 unique development events, 9,677 positive bank-quarter links, 512 OOF events and 2,439 usable COST values match accepted `costaudit20261122d`. Both event Parquet SHA-256 values equal the accepted `d7ed73a307edad0d5e58873aa8e5e4c8698f83fd433746f080b621fb11749f8a`; measured tables and frozen scenario are also equal. Evidence: ignored `tmp/p6_repro_20261123b/cost_parity.json`, two COST manifests and `cost_threshold_source_inventory.json`. The initial focused fixture failure (synthetic root lacked the assignment stub) is preserved in `tmp/p6_cost_focus.xml`; corrected focused COST tests passed 10/10 in `tmp/p6_cost_focus_retry.xml`. COST/threshold focused tests passed 25/25 in `tmp/p6_cost_threshold_focus.xml`. The sequential clean full suite passed 499/499 with zero skips (`tmp/p6_cost_threshold_full.xml`); this checkpoint does not select an alarm or open test predictors.

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.modelling.cost_audit verify --project-root $portable --run-id portable20261124acost
& $python -m bft65b.modelling.cost_audit verify --project-root $portable --run-id portable20261124bcost
```

A new isolated COST replay needs a fresh ID and all four flags `--split`, `--development-primary`, `--development-repeat`, `--repair`. The source is the captured raw Failures table, not current live FDIC data. The scenario remains the accepted nominal row-level surrogate, not observed investigation or preventable welfare loss.

## Portable development threshold checkpoint (2026-11-24)

`threshold.py` now accepts an explicit portable split, two development runs and two COST runs at plan time. The saved plan records those run IDs, OOF/source hashes, fold-only COST medians, accepted alpha/beta scenario, score comparison and deterministic tie rule. Execute follows the frozen plan; saved verification follows its recorded COST lineage. Historical accepted-run verification is retained. A portable root rejects historical default upstream IDs and paths outside that root.

`portable20261124athreshold` and independent `portable20261124bthreshold` each froze plan SHA-256 `fe1882cf5e5942c7d413bc8d22a1db8f969a81b665a4505cc387c3927d3255d1`, then completed the full 32,388-policy empirical curve over 409,094 selected-model OOF rows (2,011 positive; 512 distinct events). Both selected the accepted `SCORE >= 0.03210050445929286` policy, with TP 1,035, FP 3,455, TN 403,628 and FN 976. The reference objective is 2.3437103783238085 nominal USD thousands per OOF bank-quarter under the declared row-level surrogate. Full reference and sensitivity results, all four curve SHA-256 values, and policy contents match accepted `threshold20261122c` and each other. Evidence: ignored `tmp/p6_repro_20261123b/threshold_parity.json`, both run manifests, plans, locks and curves; the normal A/B `compare` command returned `matching` under `1e-10` metric tolerance. Focused COST/threshold tests passed 25/25; the final sequential clean suite after these code changes passed 499/499 with zero skips (`tmp/p6_cost_threshold_full.xml`). Protected postcheck found zero changes in 181 raw/final, 49 supplementary and 180 original bundle files, and the registry still records one historical test access (`threshold_protected_postcheck.json`).

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.modelling.threshold verify --project-root $portable --run-id portable20261124athreshold
& $python -m bft65b.modelling.threshold compare --project-root $portable --run-id portable20261124athreshold --compare-run-id portable20261124bthreshold
```

New replay needs a fresh plan ID and all five upstream flags `--split`, `--development-primary`, `--development-repeat`, `--cost-primary`, `--cost-repeat`. Execute then uses the frozen plan and no independent replacements. Development OOF served both model and threshold selection; these are not final test performance. The FDIC median cancels within a fold's threshold minimization but weights folds differently in the global monetary-scale objective. The scenario is nominal and surrogate, not literal avoided FDIC loss. No model refit, test scoring, test threshold search or second predictive test use occurred.

**End-to-end clean-environment regeneration is not yet established.** The captured-vintage chain now reaches the locked development threshold. Required storage and processing benchmark replay, complete README/traceability review and final report remain open. The accepted final model/scores were only verified read-only and were never regenerated; the original final-test registry must stay intact.

The final PDF/DOCX, page-count check, student's name/ID, verified public GitHub URL, citation completeness and publication remain separate Part 6 submission tasks. No final report assembly or repository publication occurred here.
