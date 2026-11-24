# BFT65B: Bank Failure Early Warning

Phase 0 and the implementation gates for Parts 1–5 have passed. The accepted Part 4.2 runs `membership20261120a` and `labels20261120a` establish a retrospective activity cohort and four-quarter labels; they do not claim a historical public registry vintage. See RETROSPECTIVE_COHORT_POLICY.md and LABEL_POLICY.md. The Part 4.1 feature contract is in FEATURE_DICTIONARY.md. Part 4.3 produced and verified the seven-feature model table under `featuretable20261120a`, with a matching independent repeat `featuretable20261120b`; see FEATURE_TABLE_POLICY.md. P4.4 saved its temporal-leakage audit under `p44audit20261120a`; see P4_LEAKAGE_AUDIT.md. The first Part 5 milestone froze a chronological development/gap/test assignment under `split20261120b`, with `split20261120a` as independent repeat evidence; see P5_SPLIT_POLICY.md. The second Part 5 milestone fitted and compared development-fold logistic and bounded gradient-boosting models under `dev20261121a` and the matching independent repeat `dev20261121b`; see P5_DEVELOPMENT_VALIDATION.md. The third Part 5 milestone ran a bounded local Spark MLlib demo under `sparkdemo20261122a` and independent repeat `sparkdemo20261122b`; see P5_SPARK_MLLIB.md. The fourth Part 5 milestone audited development-only FDIC COST under `costaudit20261122d`, with `costaudit20261122e` as an independent repeat; see P5_COST_AUDIT.md. The fifth milestone locked a development-only alarm threshold under `threshold20261122c`, with `threshold20261122d` as its independent repeat; see P5_THRESHOLD_POLICY.md. The final Part 5 milestone refitted the selected model on full development and used the locked threshold once on test under `final20261122a`; see P5_FINAL_EVALUATION.md. Final report placement and Part 6 remain pending. The assignment PDF is authoritative. See PROJECT_PLAN.md, TASKS.md, PROJECT_STATUS.md, and EVIDENCE_REGISTER.csv for current scope and evidence.

## First Part 5 milestone: chronological split

The split counts distinct verified FDIC failure source IDs represented by eligible model-positive rows. It assigns prediction quarters, never report or failure dates. Its only test-label access is the global outcome-count feasibility audit; it does not train or evaluate a model. The frozen development period is 1984Q1–2012Q4, the excluded gap is 2013Q1–2013Q4, and test is 2014Q1–2025Q2. Reproduce or verify with:

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.split verify --project-root C:\NEU\BFT65B --run-id split20261120b
.\.venv_local\Scripts\python.exe -m bft65b.modelling.split show --project-root C:\NEU\BFT65B --run-id split20261120b
# Independent regeneration requires a fresh ignored run ID; it reads accepted files only.
.\.venv_local\Scripts\python.exe -m bft65b.modelling.split run --project-root C:\NEU\BFT65B --run-id split_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest tests\test_chronological_split.py -q --basetemp C:\NEU\BFT65B\tmp\pytest_p51_focus_new
```

The saved manifest and quarter/period counts are in `runs/task_5_1_split/split20261120b/`; the two ZSTD assignment/linkage files are in `data_processed/task_5_1_split/split20261120b/`. Both roots are ignored by Git. To inspect counts without regenerating the split:

```python
import json
from pathlib import Path
path = Path(r"C:\NEU\BFT65B\runs\task_5_1_split\split20261120b\summary.json")
summary = json.loads(path.read_text(encoding="utf-8"))
print(summary["bounds"])
print(summary["periods"])
print(summary["by_quarter"][-1])
```

## Second Part 5 milestone: development-only forward validation

`P5_DEVELOPMENT_VALIDATION.md` defines three expanding-window folds with four complete prediction-quarter gaps, two frozen scikit-learn candidates and average precision as the primary development-only ranking metric. The `plan` command validates accepted inputs and writes the candidate/fold specification before any fit. The separate `execute` command opens only development predictor partitions and saves candidate out-of-fold scores. The later 2013 gap and 2014–2025 test partitions are not used for fitting or scoring here.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.development plan --project-root C:\NEU\BFT65B --run-id dev_new_unique_id
# Review runs\task_5_2_development\dev_new_unique_id\frozen_plan.json before executing.
.\.venv_local\Scripts\python.exe -m bft65b.modelling.development execute --project-root C:\NEU\BFT65B --run-id dev_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.modelling.development verify --project-root C:\NEU\BFT65B --run-id dev_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.modelling.development show --project-root C:\NEU\BFT65B --run-id dev_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest tests\test_development_validation.py -q --basetemp C:\NEU\BFT65B\tmp\pytest_p52_focus_new
```

Use fresh run IDs for a repeat, then `compare --run-id <first-id> --compare-run-id <repeat-id>` to check key-level OOF scores within the documented numerical tolerance. All development evidence and OOF data remain in ignored `runs/task_5_2_development/` and `data_processed/task_5_2_development/`. A saved summary can be inspected without refitting:

```python
import json
from pathlib import Path
summary = json.loads(Path(r"C:\NEU\BFT65B\runs\task_5_2_development\dev20261121b\summary.json").read_text(encoding="utf-8"))
print(summary["selected_candidate"])
print(summary["fold_results"])
print(summary["pooled_results"])
```

## Third Part 5 milestone: local Spark MLlib demonstration

`P5_SPARK_MLLIB.md` defines a fixed logistic MLlib demonstration on the complete accepted development fold 3. It does not change the selected scikit-learn gradient-boosting candidate. The training quarters are 1984Q1–2007Q4, the excluded fold gap is 2008Q1–2008Q4, and validation is 2009Q1–2012Q4. Configure the verified Task 3.0 runtime in the current PowerShell process, hash-check its existing native files, and run the synthetic acceptance before fitting:

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:HADOOP_HOME = (Resolve-Path .\tmp\hadoop-3.3.5).Path
$env:PATH = (Join-Path $env:HADOOP_HOME 'bin') + ';' + $env:PATH
$env:PYSPARK_PYTHON = (Resolve-Path .\.venv_local\Scripts\python.exe).Path
$env:PYSPARK_DRIVER_PYTHON = $env:PYSPARK_PYTHON
Get-FileHash .\tmp\hadoop-3.3.5\bin\winutils.exe,.\tmp\hadoop-3.3.5\bin\hadoop.dll -Algorithm SHA256
.\.venv_local\Scripts\python.exe .\tests\spark_runtime_acceptance.py *> .\tmp\spark_p53_acceptance_new.log
if ($LASTEXITCODE -ne 0) { throw 'Spark runtime acceptance failed' }
.\.venv_local\Scripts\python.exe -B -m bft65b.modelling.spark_demo run --project-root C:\NEU\BFT65B --run-id sparkdemo_new_a
.\.venv_local\Scripts\python.exe -B -m bft65b.modelling.spark_demo verify --project-root C:\NEU\BFT65B --run-id sparkdemo_new_a
.\.venv_local\Scripts\python.exe -B -m bft65b.modelling.spark_demo run --project-root C:\NEU\BFT65B --run-id sparkdemo_new_b
.\.venv_local\Scripts\python.exe -B -m bft65b.modelling.spark_demo compare --project-root C:\NEU\BFT65B --run-id sparkdemo_new_a --compare-run-id sparkdemo_new_b
.\.venv_local\Scripts\python.exe -B -m pytest tests\test_spark_mllib_demo.py -q --basetemp C:\NEU\BFT65B\tmp\pytest_p53_focus_new
```

Use fresh IDs and ignored output directories. `verify` checks the saved scores and accepted input hashes without rerunning Spark. The final `compare` requires exact key/label coverage and a declared `1e-8` score tolerance. Inspect a saved result without fitting:

```python
import json
from pathlib import Path
s = json.loads(Path(r"C:\NEU\BFT65B\runs\task_5_3_spark_mllib\sparkdemo20261122a\summary.json").read_text(encoding="utf-8"))
print(s["training"], s["validation"], s["metrics"])
print(s["timings_seconds"], s["runtime"])
```

## Fourth Part 5 milestone: development-only COST audit

`P5_COST_AUDIT.md` distinguishes observed FDIC Estimated Loss (nominal thousands of US dollars) from an assumed preventable fraction and an assumed false-alarm investigation charge. The event denominator consists only of unique verified failure source IDs represented by development-positive model observations; events represented only in the gap or test do not enter COST estimation. No threshold or predictive test score is calculated.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.cost_audit verify --project-root C:\NEU\BFT65B --run-id costaudit20261122d
.\.venv_local\Scripts\python.exe -m bft65b.modelling.cost_audit show --project-root C:\NEU\BFT65B --run-id costaudit20261122d
# Run again only under a fresh ignored run ID.
.\.venv_local\Scripts\python.exe -m bft65b.modelling.cost_audit run --project-root C:\NEU\BFT65B --run-id cost_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest tests\test_cost_audit.py -q --basetemp C:\NEU\BFT65B\tmp\pytest_cost_new
```

The report-ready English table is in ignored `runs/task_5_4_cost/costaudit20261122d/report.md`; the frozen assumptions are in `scenario.json`. To view saved counts without rerunning the audit:

```python
import json
from pathlib import Path
s = json.loads(Path(r"C:\NEU\BFT65B\runs\task_5_4_cost\costaudit20261122d\summary.json").read_text(encoding="utf-8"))
print(s["measured"]["development"])
print(s["measured"]["oof_validation"])
print(s["measured"]["fold_training_costs"])
```

## Fifth Part 5 milestone: development-only alarm threshold

`P5_THRESHOLD_POLICY.md` records the frozen, data-scaled surrogate loss, complete empirical OOF curve, reference lock and nine sensitivity cells. It uses only the selected gradient-boosting candidate's accepted development OOF scores, the accepted fold-training COST medians, and the previously declared assumptions. `threshold20261122c` is the locked run; `threshold20261122d` is its independent repeat. No test predictions or full-development model refit were made.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold verify --project-root C:\NEU\BFT65B --run-id threshold20261122c
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold compare --project-root C:\NEU\BFT65B --run-id threshold20261122c --compare-run-id threshold20261122d
# Reproduction requires a fresh run ID and explicit review of frozen_plan.json between steps.
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold plan --project-root C:\NEU\BFT65B --run-id threshold_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.modelling.threshold execute --project-root C:\NEU\BFT65B --run-id threshold_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest tests\test_threshold.py -q --basetemp C:\NEU\BFT65B\tmp\pytest_threshold_new
```

The ignored `runs/task_5_5_threshold/threshold20261122c/` directory holds `frozen_plan.json`, `locked_policy.json`, `summary.json`, `sensitivity.json`, the English report, two SVG plots and the manifest. Four ZSTD Parquet files under ignored `data_processed/task_5_5_threshold/threshold20261122c/` contain every empirical policy's overall and per-fold measures. To view the saved lock without rescoring:

```python
import json
from pathlib import Path
root = Path(r"C:\NEU\BFT65B\runs\task_5_5_threshold\threshold20261122c")
locked = json.loads((root / "locked_policy.json").read_text(encoding="utf-8"))
print(locked["policy"], locked["selection_result"])
```

## Final Part 5 milestone: locked one-use test evaluation

`final20261122a` is the accepted one-use evaluation. Its registry records the sole test-access transition and its saved ZSTD scores, fitted model, frozen plan, manifest, report and figure. See `P5_FINAL_EVALUATION.md` for the fixed result and limitations. The following command is **read-only**: it verifies source/model/score hashes, test key and label coverage, alarm decisions and metrics from saved scores. It does not refit or rescore.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.final_evaluation verify --project-root C:\NEU\BFT65B --run-id final20261122a
```

The `plan`, `fit` and `evaluate` modes consume a final evaluation attempt. They were executed for `final20261122a` after the synthetic and full regression gates, and **must not be run under a new ID for this locked design**. The project-level registry refuses a second run ID. To inspect the saved result without invoking the runner:

```python
import json
from pathlib import Path
path = Path(r"C:\NEU\BFT65B\runs\task_5_6_final\final20261122a\summary.json")
print(json.loads(path.read_text(encoding="utf-8"))["result"])
```

## Windows PowerShell setup

On a fresh clone with no existing virtual environment, use CPython 3.11 x64 and a recent PowerShell. The commands below create a new environment. In the relocated C:\NEU\BFT65B copy, use the verified .venv_local commands under Task 1.1 and preserve the copied environments.

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --disable-pip-version-check -r requirements.txt
.\.venv\Scripts\python.exe -m pip check
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv\Scripts\python.exe -m bft65b --help
.\.venv\Scripts\python.exe -m bft65b show-config
.\.venv\Scripts\python.exe -m pytest -q
```

`PYTHONPATH` is needed until the package is installed as an editable distribution. The pinned dependencies include Polars, pandas, PyArrow, PySpark, and scikit-learn for later stages, although Phase 0 code only uses Python's standard library. Java may be required for the later local PySpark demonstration; it is not needed for Phase 0 tests. Dependency pins are project choices and must be validated in a clean Python 3.11 environment before claiming full reproducibility.

If `py -0p` does not list Python 3.11, verify a separately installed CPython 3.11 executable before creating the venv. On the audited laptop the launcher returned “No installed Pythons found!”, while `C:\Users\asus\AppData\Local\Programs\Python\Python311\python.exe` was verified as 3.11.9 and used with `-m venv` for the isolated audit. Do not assume that host-specific path exists elsewhere. The clean audit installed the direct pins once, ran `pip check`, and saved the resulting full transitive `pip freeze` list; the direct pins alone are not a complete transitive lockfile.

Keep raw data outside Git using BFT65B_DATA_ROOT or the ignored data_raw/ path. Processed data and run outputs default to ignored data_processed/ and runs/. Do not commit datasets. show-config prints the configured field codes but performs no API request. Task 1.1 verified those codes against official FDIC dictionaries, and the client rejects any code outside its checked allowlist.

For a quick standard-library check when the full environment is unavailable, run python -m bft65b --help after setting PYTHONPATH. Parts 1-5 have individual commands below, but a portable clean-environment end-to-end replay is still an open Part 6 gate; several stages pin accepted upstream run IDs and need ignored source snapshots.

### Reproducibility modes and the current clean-environment gate

See [P6_REPRODUCIBILITY.md](P6_REPRODUCIBILITY.md) for the stage inventory, source inventory, observed clean-environment results, and exact limitations. These three modes have different meanings:

- **Live-source reacquisition** uses `bft65b.acquisition.full --execute` and a new isolated raw namespace. It requests the FDIC API's *current* vintage. Current index/total changes can prevent byte-identical reproduction of `real20261111b`; the source must be reconciled, not silently mixed.
- **Captured-snapshot replay** reads the accepted immutable 172 raw Parquet files, `real20261111b` manifest/summary/acceptance evidence, the saved official Failures transaction response and dictionary, and the saved Institutions/History/code-description evidence. These ignored inputs are not in Git. A source-only checkout cannot run the later stages without separately provisioning them. The current raw manifest contains absolute `C:\NEU\BFT65B` paths; copying it and changing those paths is not acceptance. New derived outputs need fresh ignored run namespaces and explicit upstream lineage. The clean-environment audit and its downstream treatment example are recorded in `P6_REPRODUCIBILITY.md`; a complete portable replay is not yet claimed.
- **Read-only accepted-output verification** uses a stage's `verify` command. In particular, `final_evaluation verify` checks the existing `final20261122a` saved model/score bindings, registry, source hashes and metrics without fitting or generating test predictions. It is not regeneration of final test scores.

For this project's verified Windows Spark tests, JDK 17 and the hash-verified local `winutils.exe`/`hadoop.dll` must be placed under the **current checkout's** ignored `tmp/hadoop-3.3.5/bin/`; set `HADOOP_HOME` to that checkout-local directory and `PYSPARK_PYTHON`/`PYSPARK_DRIVER_PYTHON` to its venv interpreter in the current PowerShell process. These binaries are not in Git or a Python requirements file. The clean-replay audit reused the previously verified files and did not download them. `requirements.txt` pins direct packages; `requirements-lock-win-py311.txt` additionally freezes the 25-package transitive set actually installed and tested on this Windows/Python 3.11 host. The lock does not supply JDK/Hadoop native tools or guarantee identical wheels on other platforms.

### Bounded portable captured-vintage replay (raw through transaction repair)

The following commands provision a **new ignored root** from immutable accepted source evidence. They make no FDIC request and do not copy accepted derived audit, treatment, panel or repair outputs. Use fresh run IDs and an empty destination. The source-vintage bundle copies the original acquisition manifest, summary, acceptance and linkage audit byte-for-byte; `source_vintage_bundle.json` maps original recorded paths to relative bundle paths and verifies every SHA-256. Original manifests remain unchanged. `P6_REPRODUCIBILITY.md` records the executed run IDs and parity result.

```powershell
Set-Location C:\NEU\BFT65B
$source = (Get-Location).Path
$portable = 'C:\NEU\BFT65B\tmp\portable_replay_new'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
New-Item -ItemType Directory -Path $portable -Force | Out-Null
$src = (Resolve-Path .\src).Path
Get-ChildItem -LiteralPath $src -Recurse -File -Filter '*.py' | ForEach-Object {
    $rel = $_.FullName.Substring($src.Length + 1)
    $target = Join-Path (Join-Path $portable 'src') $rel
    New-Item -ItemType Directory -Path (Split-Path -Parent $target) -Force | Out-Null
    Copy-Item -LiteralPath $_.FullName -Destination $target
}
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.quality.source_vintage provision --source-root $source --destination-root $portable
if ($LASTEXITCODE -ne 0) { throw 'Source-vintage provisioning failed' }
& $python -m bft65b.quality.source_vintage verify --project-root $portable
if ($LASTEXITCODE -ne 0) { throw 'Source-vintage verification failed' }
& $python -m bft65b.quality.audit audit --project-root $portable --config config/financials_full_v2.json --attempt real20261111b --run-id portable_new_audit
if ($LASTEXITCODE -ne 0) { throw 'Portable audit failed' }
& $python -m bft65b.quality.treatment run --project-root $portable --config config/financials_full_v2.json --audit-run portable_new_audit --run-id portable_new_treatment
if ($LASTEXITCODE -ne 0) { throw 'Portable treatment failed' }
& $python -m bft65b.panel.build run --project-root $portable --config config/financials_full_v2.json --treatment-run portable_new_treatment --run-id portable_new_panel
if ($LASTEXITCODE -ne 0) { throw 'Portable panel build failed' }
& $python -m bft65b.panel.repair run --project-root $portable --source-json (Join-Path $portable 'runs/task_2_3_transaction_repair/source_failures_20261115.json') --panel-run portable_new_panel --run-id portable_new_repair
if ($LASTEXITCODE -ne 0) { throw 'Portable transaction repair failed' }
& $python -m bft65b.panel.compare_repair --accepted-root $source --accepted-run repair20261115f --replay-root $portable --replay-run portable_new_repair
if ($LASTEXITCODE -ne 0) { throw 'Repaired-result parity failed' }
```

Stop at the first nonzero exit code. The replay outputs appear only under `$portable\runs` and `$portable\data_processed`. On a different machine, create a new Python 3.11 venv and install `requirements-lock-win-py311.txt` before these steps; provision the captured source files with their accepted hashes. The comparison command reads accepted repair outputs **only for validation after regeneration**. The older raw-to-panel blocker description in `P6_REPRODUCIBILITY.md` records the initial clean replay and was resolved by the later portable transaction-repair follow-up. The complete Part 6 end-to-end gate remains open.

### Continue the captured-vintage replay through PIT candidates

Use the verified portable raw-to-repair chain above. For the established checkpoint, `$portable` is `C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable`. The PIT runner requires all four upstream IDs when a source-vintage bundle exists; it validates the selected audit/treatment/panel/repair lineage and maps acquisition paths through the bundle. It does not read the old absolute raw paths in the copied acquisition manifest. The four accepted upstream runs must not be rerun to obtain new IDs. The command below uses a **new** PIT ID; replace `portable_new_pit` before execution. It creates a candidate roster under the portable root, not an active-bank cohort or labels.

```powershell
Set-Location C:\NEU\BFT65B
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.quality.source_vintage verify --project-root $portable
& $python -m bft65b.panel.repair show --project-root $portable --run-id portable20261123b_repair
& $python -m bft65b.panel.prediction run --project-root $portable --run-id portable_new_pit --raw-run real20261111b --treatment-run portable20261123b_treatment --panel-run portable20261123b_panel --repair-run portable20261123b_repair
if ($LASTEXITCODE -ne 0) { throw 'Portable PIT replay failed' }
& $python -m bft65b.panel.prediction verify --project-root $portable --run-id portable_new_pit
if ($LASTEXITCODE -ne 0) { throw 'Portable PIT verification failed' }
$env:PYTHONPATH = 'C:\NEU\BFT65B\src'
& $python -m bft65b.panel.compare_pit --left-root C:\NEU\BFT65B --left-run pit20261115f --right-root $portable --right-run portable_new_pit --output C:\NEU\BFT65B\tmp\portable_new_pit_comparison.json
if ($LASTEXITCODE -ne 0) { throw 'Keyed PIT parity failed' }
```

`compare_pit` verifies both saved runs, compares every Parquet column by sorted `(CERT, Q)`, and checks quarter and overall summaries. Accepted PIT outputs are read only after regeneration for comparison. `P6_REPRODUCIBILITY.md` records the executed PIT ID and measured parity. The complete end-to-end gate remains open for cohort, labels, features and later stages.

```powershell
# Safe verification of the accepted final result; does not consume test again.
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.modelling.final_evaluation verify --project-root C:\NEU\BFT65B --run-id final20261122a
```

The original final `plan`, `fit` and `evaluate` commands printed earlier in this README are historical execution records for `final20261122a`, **not** instructions to run them again. A later frozen-design score reproduction, if required, needs a separately named protocol that preserves the original single test-use history.

## Task 1.1 API contract checks

The client, bounded probe, versioned storage writer, and four-quarter pilot are in src/bft65b/acquisition/. STORAGE_SCHEMA.md defines the Arrow schemas and safe rerun contract. The field definitions, candidate review, field-presence behavior, and pagination limits are documented in SOURCE_FIELD_INVENTORY.md. The 4,117-record Failures endpoint snapshot passed the single-page total and unique source-ID gate; future downloads must repeat that validation. The 488 null-CERT failures remain unresolved bank-linkage cases, never synthesized certificates.

From the verified relocated environment, run:

    $env:PYTHONPATH = (Resolve-Path .\src).Path
    .\.venv_local\Scripts\python.exe -m pytest -q tests\test_api_client.py tests\test_storage.py
    .\.venv_local\Scripts\python.exe -m bft65b.acquisition.probe --page-size 2 --manifest runs\task_1_1_api_probe.json

The live probe requests at most two pages of two records per endpoint. It writes only API totals, sample counts, field-presence counts, and validation outcomes to an ignored run manifest; it does not save raw rows.

## Task 1.3 pilot evidence

The accepted bounded 2023 pilot manifest is `runs/task_1_3_pilot_resume3.json`, with a summary in `runs/task_1_3_pilot_resume3_summary.json`. Both and the six Parquet outputs are ignored by Git. The summary includes actual totals, missing/null counts, file sizes, checksums, and the path/checksum of the 488-source-ID unresolved-linkage audit. Earlier failed attempts remain under `runs/` for diagnosis. Financials v1 partitions were revalidated and reconciled without rewriting; Failures v2 permits a null `CERT`, requires a unique source `ID`, and stores `COST` as the source-aligned float64 type. Institutions v1 uses the verified `CERT` key.

## Task 1.4 synthetic recovery checks

Task 1.4 passed on 2026-11-11 using only pytest temporary directories for destructive simulations. Run the focused recovery suite and save its individual outcomes under ignored `runs/`:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_api_client.py tests\test_storage.py tests\test_pilot.py --junitxml=runs\task_1_4_recovery_junit.xml
.\.venv_local\Scripts\python.exe -m pytest -q
```

The focused suite passed 137 tests; the full suite passed 146. It injects HTTP 429/503/connection timeouts and exhausted retries, interruptions, missing/stale/failed/inconsistent manifests, missing/truncated/corrupt Parquet, and a manifest write failure after file promotion. It verifies Financials v1 and Failures v2 recovery, byte-preserving skips, source-total/index probes, no duplicate output keys, and the six-file completion gate. `STORAGE_SCHEMA.md` defines the exact skip and reconciliation policy. Use `--resume` only with a persisted **running** pilot manifest; an existing complete manifest is verified read-only, and a failed manifest requires a new run path. Never run destructive failure injections against the accepted 2023 files. At this Task 1.4 checkpoint, the full historical acquisition had not started.

## Task 1.5A extended Financials preflight

The verified 26-field selection is pinned in `config/financials_extended_v2.json`; its official titles, unit limitations, and bounded coverage measurements are in `SOURCE_FIELD_INVENTORY.md`. The extended runner's CLI permits only two four-quarter plans. It has no full-download CLI mode, and its callable runner rejects a 170-quarter or otherwise unapproved plan. The eventual full scope is generated deterministically as 170 Financials quarters plus complete Failures and Institutions, but remains disabled during Task 1.5A.

From `C:\NEU\BFT65B` in PowerShell, the bounded checks are reproducible with:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.acquisition.extended --help
.\.venv_local\Scripts\python.exe -m bft65b.acquisition.extended historical-profile --config config\financials_extended_v2.json --project-root C:\NEU\BFT65B
.\.venv_local\Scripts\python.exe -m bft65b.acquisition.extended extended-pilot --config config\financials_extended_v2.json --project-root C:\NEU\BFT65B
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_extended_acquisition.py
.\.venv_local\Scripts\python.exe -m pytest -q
```

The already accepted bounded manifests are `runs/task_1_5a_historical_profile.json` and `runs/task_1_5a_extended_pilot_2023.json`, with matching `_summary.json` files. Repeating a completed command without `--resume` refuses to overwrite its manifest; `--resume` independently validates local files and probes source total/index without rewriting healthy outputs. If a failed run needs recovery, use new ignored manifest and summary paths. The raw root in the JSON is `data_raw/task_1_5a`; the store appends `fdic_extended_v2`, so 1984Q1 is exactly `data_raw/task_1_5a/fdic_extended_v2/financials/REPDTE=19840331/part-000.parquet`. This is separate from the original accepted `data_raw/fdic/` pilot. At the Task 1.5A checkpoint, the eight sampled quarters did not establish all-quarter field coverage and full acquisition remained unchecked.

## Task 1.5B full-run entrypoint (historical preparation checkpoint)

The full-run configuration is `config/financials_full_v2.json`. The CLI accepts only the verified 26-field v2 set and exact assignment dates. Its dry-run lists all 170 Financials quarter jobs from 19840331 through 20260630, followed by complete Failures and Institutions jobs, with no network or file writes. Run this read-only command from `C:\NEU\BFT65B`:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.acquisition.full --config config\financials_full_v2.json --project-root C:\NEU\BFT65B --attempt first --dry-run
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_full_acquisition.py
```

The future **network execution command is documented for a separate authorized run; do not run it as part of Task 1.5B**:

```powershell
.\.venv_local\Scripts\python.exe -m bft65b.acquisition.full --config config\financials_full_v2.json --project-root C:\NEU\BFT65B --attempt first --execute
```

For recovery, preserve `runs/task_1_5_full_first.json` and its summary, choose a new attempt ID, inspect the prior failure, then invoke the same command with `--attempt recovery01 --recover-from first --execute`. Never edit or retry the old attempt in place. The runner independently checks each inherited file and source total/index, records hashes of prior attempt artifacts, and preserves healthy identical Parquet bytes. A promoted file without a summary or persisted output record is re-fetched and reconciled; a corrupt file is rebuilt through validated atomic promotion; a readable conflicting file or changed source stops the run. After all outputs pass physical validation, the runner re-probes all source totals/indexes before committing the complete manifest, then commits the complete summary. If that last commit fails, start a new recovery attempt; the complete-manifest/incomplete-summary mismatch remains as evidence. Current source metadata cannot prove unchanged row content if FDIC revises records while retaining both total and index fingerprint. No real full-run data, timing, or modelling result has been claimed.

The full runner acquires a nonblocking, cross-process lock at `data_raw/.task_1_5_full.lock` **before** creating an attempt manifest, summary, audit, or raw output. It holds ownership through final validation and both commits. A competing attempt raises `FullAcquisitionBusyError` promptly and leaves the active attempt's evidence and Parquet bytes untouched. Leave the lock file in place: Windows releases its byte-range lock automatically when the owner exits, including abrupt termination. The full-run `ParquetStore.write()` also rejects a direct write without ownership. `--dry-run` takes no lock and writes no files. Synthetic tests used two signalled real processes to verify exclusion, byte preservation, and recovery after termination; see the ignored `runs/task_1_5b_followup_focused.xml` and `runs/task_1_5b_followup_full.xml`.

Failed final validation records `failure.phase`, `failure.endpoint`, and `failure.quarter` in the summary and a matching failed validation/stage in the manifest. `final_parquet` identifies read-back; `final_source` identifies source-total/index reconciliation. A plan-wide mismatch has null endpoint/quarter and a plan-level stage. Earlier completed acquisition/storage stages remain complete in this evidence. The original exception still propagates to the caller; recover with a new attempt ID after investigating the failure.

## Task 1.5 real full acquisition: accepted snapshot

The authorized real run used the same 26-field configuration and exactly 170 Financials quarter jobs plus complete Failures and Institutions. Attempt `real20261111a` fetched 172 files but **failed**, correctly, during its final source sweep when Financials `20210331` received HTTP 429 after four retries. The failed `runs/task_1_5_full_real20261111a.json`, matching summary, linkage audit, and console log remain unchanged. A single later probe returned HTTP 200 with `X-Ratelimit-Limit: 20` and `X-Ratelimit-Remaining: 19`; the reviewed [FDIC API documentation](https://api.fdic.gov/banks/docs/) did not specify the limit's time window. The production runner now spaces one-row source probes by at least 3.2 seconds and uses five attempts with 5/10/20/40-second bounded exponential backoff (60-second cap). These are conservative operational choices, not an official FDIC limit claim.

Attempt `real20261111b --recover-from real20261111a` independently rechecked source total/index and physical file integrity, inherited all 172 identical Parquet SHA-256 values, and completed the final source sweep. Its ignored `runs/task_1_5_full_real20261111b.json` and `_summary.json` are `complete`, with 172 sources, 172 outputs, and 518 passing validations. The unresolved-Failures linkage audit is `runs/task_1_5_full_real20261111b_summary_unlinked_ids.json`. The independent, read-only `tests/real_full_acceptance.py` audit passed every job and saved aggregate findings to `runs/task_1_5_full_real20261111b_acceptance.json`: Financials 1,682,616 rows, Failures 4,117, Institutions 27,835, zero conversion anomalies, and 488 null-CERT Failures source IDs. Requested Financials fields were present throughout; Institutions `CHANGEC1` was absent from 3,482 source records. `P3ASSET` was explicitly null in 971,925 Financials records, with 68 complete all-null quarters from 1984Q1 through 2000Q4. Source absence, explicit null counts, and all-null quarter IDs remain separate in the acceptance JSON. All 22 protected earlier artifact hashes and the four failed-attempt evidence hashes were unchanged. No raw/run/environment file is tracked in Git. The full raw snapshot is under `data_raw/task_1_5_full/fdic_extended_v2`; it is **not** a storage performance benchmark.

The actual PowerShell execution and acceptance commands were:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.acquisition.full --config config\financials_full_v2.json --project-root C:\NEU\BFT65B --attempt real20261111a --execute
.\.venv_local\Scripts\python.exe -m bft65b.acquisition.full --config config\financials_full_v2.json --project-root C:\NEU\BFT65B --attempt real20261111b --recover-from real20261111a --execute
.\.venv_local\Scripts\python.exe tests\real_full_acceptance.py --project-root C:\NEU\BFT65B --config config\financials_full_v2.json --attempt real20261111b
.\.venv_local\Scripts\python.exe -m pytest -q
```

These immutable attempt IDs cannot be executed again. To acquire a later source snapshot, first run the dry-run with a **new** ID, inspect the current source and disk capacity, then use that new ID with `--execute`; retain all prior attempt evidence. The accepted acquisition does not change the modelling observation cutoff of 2025Q2. Task 1.6 has since passed; Part 2 remains open.

## Task 1.6 storage experiment (accepted Task 1.5 snapshot)

Run from `C:\NEU\BFT65B` with the working Python 3.11 environment. Each command has a separate acceptance gate. `preflight` reads and independently validates all 170 accepted Financials v2 files, the complete manifest/summary, physical provenance fields, keys, ZSTD codec, and SHA-256; it writes the source inventory and frozen contract before any conversion. `prepare` creates isolated CSV/Snappy/ZSTD copies; `parity` checks exact row-level equality for both workloads; `run` launches one fresh process per warm-up/read and records every timing. `show` displays the stored result without rereading or repeating the benchmark. No command makes an FDIC API request.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.storage preflight
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.storage prepare
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.storage parity
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.storage run
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.storage verify
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.storage show
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_storage_benchmark.py
.\.venv_local\Scripts\python.exe -m pytest -q
```

The checked-in contract is `config/storage_benchmark_v1.json`; its source-derived selective row count and exact Arrow types are frozen in ignored `runs/task_1_6_storage/frozen_spec.json`. Representations are in ignored `data_processed/storage_benchmark_v1`; the original Task 1.5 data remain under `data_raw/task_1_5_full`. The completed attempt is write-once: `preflight` and `run` refuse to overwrite its evidence. To repeat the experiment, copy the checked-in specification to a new name, change only `storage_root` and `evidence_root` to new ignored namespaces, and pass `--spec <new-spec-path>` to each command. The CSV uses UTF-8/LF, a header, comma delimiter, RFC-style minimal quoting, a collision-checked `__BFT65B_NULL__` sentinel, ISO dates, exact decimal strings, and round-trip binary64 text. The CSV reader applies an explicit Arrow schema. Both Parquet variants use identical v2.6 writer settings (65,536-row groups, dictionary off, statistics on, 1 MiB data pages, page version 1.0), differing only in codec and ZSTD level 3. The benchmark projects 26 source fields; `ID` and `SOURCE_MISSING_FIELDS` are validated in the immutable source but excluded from the fixed logical comparison.

The full workload reads 170 files and all 26 fields. The selective workload reads 40 quarters (2010Q1–2019Q4) and `CERT, REPDTE, ASSET, DEP, EQ, ROA, NIMY, RBCRWAJ`. Quarter selection occurs by explicit file paths for all formats. PyArrow reads only requested columns from Parquet; CSV must parse row-oriented text before its `include_columns` conversion/projection. Every read is a fresh process with PyArrow reader threads disabled; one untimed warm-up precedes five measured reads per format/workload. Format order rotates cyclically. Time is measured with `time.perf_counter_ns` around file open, decode, and full Arrow materialization; peak worker process working set is recorded. OS caches are not controlled: these are OS-managed cache measurements, not verified cold-cache timings. Conversion and parity are outside read timings. Machine-readable evidence is under ignored `runs/task_1_6_storage/`.

## Part 2.1 read-only raw-data quality audit

The first Part 2 milestone audits the accepted `real20261111b` raw snapshot without changing data. It validates all 172 source files against their accepted manifest, reads the three endpoints in place, distinguishes source-field absence from explicit nulls, and reports keys, dates, justified numeric rules, and join coverage with explicit denominators. Financials are broken down by quarter, Failures by event year, and Institutions as a snapshot. The resulting findings and report table are under ignored `runs/task_2_1_quality/<run-id>/`; `QUALITY_AUDIT.md` summarizes the accepted run. No command in this section calls the FDIC API or builds a cleaned panel.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.quality.audit --help
.\.venv_local\Scripts\python.exe -m bft65b.quality.audit audit --project-root C:\NEU\BFT65B --config config\financials_full_v2.json --attempt real20261111b --run-id audit20261114c
.\.venv_local\Scripts\python.exe -m bft65b.quality.audit show --project-root C:\NEU\BFT65B --run-id audit20261114a
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_quality_audit.py
.\.venv_local\Scripts\python.exe -m pytest -q
```

Choose a fresh run ID for `audit`; an existing output directory is never overwritten. The deterministic `findings.json` omits timestamps and timing metadata, so its SHA-256 can be compared across repeats. `metadata.json` and `run_manifest.json` record actual execution time, input hashes, configuration, dependencies, and validations. `show` reads saved findings without rerunning the audit. The subsequent flag-only treatment records before/after counts without changing source values.

## Part 2.1 lossless treatment and before/after reconciliation

The second Part 2 milestone reads the complete accepted audit and independently rechecks the immutable raw snapshot. It applies an explicit, fail-closed action to every observed rule, preserving all rows, source IDs, null values, and source-absence provenance. The only derived column is `DQ_FLAGS` (sorted strings). Each source table is read back after atomic ZSTD Parquet promotion. The 1,687 period issue entries and their denominators, actions, and before/after counts are in ignored `runs/task_2_1_treatment/<run-id>/treatment_log.json`; `summary.json` and `report.md` contain dataset reconciliation and an English table. No FDIC request, panel join, imputation, or label is created.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.quality.treatment --help
.\.venv_local\Scripts\python.exe -m bft65b.quality.treatment run --project-root C:\NEU\BFT65B --config config\financials_full_v2.json --audit-run audit20261114a --run-id treatment20261114c
.\.venv_local\Scripts\python.exe -m bft65b.quality.treatment show --project-root C:\NEU\BFT65B --run-id treatment20261114a
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_quality_treatment.py tests\test_quality_audit.py
.\.venv_local\Scripts\python.exe -m pytest -q
```

Use a new run ID for a repeat; the CLI refuses to overwrite a previous data or evidence directory. Derived Parquet files are under ignored `data_processed/task_2_1_treatment/<run-id>/`, separate from accepted raw inputs. `show` reads only saved manifest, metadata, and summary evidence without rerunning treatment. It reports completion only when the selected run ID, manifest status and validations, metadata status, summary status and counts, and recorded summary SHA-256 agree; incomplete or inconsistent evidence exits nonzero with a diagnostic. The verified host may require a shorter pytest `--basetemp` path for older tests that create deep synthetic paths inside the Codex sandbox; this affects test temporary files, not project data.

## Part 2.2/2.3 observed panel and exit evidence

The final accepted build uses the completed P2.1 treatment run `treatment20261114a`. It rechecks the 172 treated input hashes and schemas, writes exactly one observed Financials row per `(CERT, REPDTE)` into 170 quarter partitions, and saves separate Failures event, last-report exit-assessment, and resumed-gap tables. The final run is `panel20261115b`; all outputs and detailed source identifiers are Git-ignored. `PANEL_EXIT_EVIDENCE.md` explains the evidence rules, measured categories, and why Institutions `ACTIVE`/`CHANGEC1` are audit-only. The full 170-quarter reconciliation and separate event/exit tables are in the ignored report and JSON summary.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.panel.build --help
.\.venv_local\Scripts\python.exe -m bft65b.panel.build run --project-root C:\NEU\BFT65B --config config\financials_full_v2.json --treatment-run treatment20261114a --run-id panel20261115c
.\.venv_local\Scripts\python.exe -m bft65b.panel.build show --project-root C:\NEU\BFT65B --run-id panel20261115b
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_panel_build.py
.\.venv_local\Scripts\python.exe -m pytest -q
```

Use a fresh run ID for another build; the runner never overwrites an existing attempt. Inspect `runs/task_2_2_panel/panel20261115b/summary.json` for machine-readable quarter, event, exit, and gap counts, or `report.md` in the same directory for English tables. The CLI `show` inspects saved manifest, metadata, and summary evidence without scanning the Parquet data. This stage is retrospective audit evidence only; it does not implement feature availability, a prediction-date join, or the four-quarter target.

### P2.3 FDIC transaction repair

The accepted `panel20261115b` bank-quarter files remain immutable. Its original 2,775 failure assessments are a historical baseline. The separate `repair20261115f` run matches an official one-page Failures response containing `RESTYPE` and `RESTYPE1` to all 4,117 accepted source IDs, verifies CERT/FAILDATE/COST and source index, and saves revised event and bank-assessment tables only. The current verified count is 2,534 confirmed-failure banks; see [PANEL_TRANSACTION_REPAIR.md](PANEL_TRANSACTION_REPAIR.md) for all categories and limitations. The official source JSON, field dictionary, retrieval metadata, detailed IDs, and derived data are under ignored `runs/task_2_3_transaction_repair/` and `data_processed/task_2_3_transaction_repair/`. New `panel.build` runs without supplementary transaction evidence conservatively leave failure status unverified; use the repaired run for P2.3 conclusions.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.panel.repair show --project-root C:\NEU\BFT65B --run-id repair20261115f
# For an independent run, reuse the saved and checksummed official response with a fresh ID.
.\.venv_local\Scripts\python.exe -m bft65b.panel.repair run --project-root C:\NEU\BFT65B --source-json C:\NEU\BFT65B\runs\task_2_3_transaction_repair\source_failures_20261115.json --run-id repair_local_01
.\.venv_local\Scripts\python.exe -m pytest -q tests\test_panel_build.py tests\test_panel_repair.py
```

The saved `summary.json` and `report.md` under `runs/task_2_3_transaction_repair/repair20261115f/` show the exact source-event dispositions, full bank-category transition, and boundary-day results. Reproduction requires the ignored official response and its `source_capture.json` and `failure_properties_20261115.yaml` companions. No further FDIC download or point-in-time modelling stage is implicit in `show` or `run`.

## P2.4 report availability and as-of rule (synthetic acceptance)

`POINT_IN_TIME_RULE.md` is the English contract for the fourth Part 2 item. `REPDTE` is a report quarter-end; prediction date `q` is a quarter-end; a report becomes provisionally eligible at `REPDTE + 120 calendar days`. `FAILDATE` is reserved for later target construction. `src/bft65b/panel/asof.py` implements an explicit, knowable-at-`q` population request and latest-eligible-report selection, with 23 allowlisted Financials source predictors. The available-report roster is only a candidate population. Undated or tied revisions fail closed; historical publication and revision vintages are not verified in the accepted FDIC snapshot. This rule-only module does not create labels.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$testTmp = 'C:\Users\asus\.codex\visualizations\2026\11\06\01a1b056-7f64-77a1-84dc-0605edf8aa3a\pytest_p24_asof_evidence'
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp $testTmp --junitxml runs\task_2_4_asof\fixture_tests.xml tests\test_panel_asof.py
```

Use a fresh `--basetemp` name and JUnit path when repeating a run. The accepted synthetic fixture result was 14/14; the related panel tests passed 46/46 and the full regression passed 296/296. Ignored JUnit files are under `runs/task_2_4_asof/`. See the specification for exact test commands and evidence hashes.

## P2.4 prediction-quarter application and Part 2 review

The fifth Part 2 item applies the accepted rule to `panel20261115b` and the repaired `repair20261115f` event evidence. Run `pit20261115e` is the historical 166-quarter result; revision-corrected `pit20261115f` is the accepted as-of reference through 2025Q2. It retains the full raw Financials scope through 2026Q2. The output is a report-derived **candidate** roster, not by itself an active-bank risk set. `eligible` means a report was provisionally available and no dated, resolved failure exclusion was found; it does not prove that the bank was open at `q`. Report ages and unresolved identity dispositions are explicit. `PART2_ACCEPTANCE.md` records the earlier population blocker; the later accepted retrospective study cohort is documented in `RETROSPECTIVE_COHORT_POLICY.md`. This stage itself creates no label or model.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
# Choose a fresh run ID for another full application; existing runs are immutable.
$runId = 'pit_new_unique_id'
.\.venv_local\Scripts\python.exe -m bft65b.panel.prediction run --project-root C:\NEU\BFT65B --run-id $runId
.\.venv_local\Scripts\python.exe -m bft65b.panel.prediction show --project-root C:\NEU\BFT65B --run-id pit20261115f
.\.venv_local\Scripts\python.exe -m bft65b.panel.prediction verify --project-root C:\NEU\BFT65B --run-id pit20261115f
$testTmp = 'C:\Users\asus\.codex\visualizations\2026\11\06\01a1b056-7f64-77a1-84dc-0605edf8aa3a\pytest_p25_final_focus'
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider --basetemp $testTmp tests\test_panel_asof.py tests\test_panel_prediction.py tests\test_panel_build.py tests\test_panel_repair.py
```

Use new temporary and JUnit paths when repeating tests. The accepted focused result was 56/56 and full regression 306/306. `show` reads saved evidence; `verify` independently scans all 166 outputs and 527 upstream hashes. Read the saved summary without rerunning the pipeline:

```powershell
$code = @'
import json
from pathlib import Path
path = Path(r"C:\NEU\BFT65B\runs\task_2_5_prediction\pit20261115f\summary.json")
summary = json.loads(path.read_text(encoding="utf-8"))
print(summary["overall"])
print(summary["by_quarter"][-1])
'@
.\.venv_local\Scripts\python.exe -c $code
```

The complete English quarter and overall table is in ignored `runs/task_2_5_prediction/pit20261115f/report.md`. To reproduce the full regression, run `python -m pytest -q -p no:cacheprovider --basetemp <new-short-test-directory>` with `.venv_local\Scripts\python.exe` and `PYTHONPATH=src` as above.

## Part 2 prediction-population audit (recent-reporter proxy)

`pit20261115f` is the accepted, revision-corrected P2.4 candidate roster.
`bft65b.panel.population` reads it and writes a separate, ignored membership
audit; it does not alter candidate predictors, create labels, or prove that
every recent reporter was active at `q`. The primary window is four quarter
ends, with two- and eight-quarter sensitivity counts. Stale reports remain
`activity_uncertain_stale`. See [POINT_IN_TIME_RULE.md](POINT_IN_TIME_RULE.md),
[HISTORY_SOURCE_ASSESSMENT.md](HISTORY_SOURCE_ASSESSMENT.md), and
[PART2_ACCEPTANCE.md](PART2_ACCEPTANCE.md).

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:PYTHONDONTWRITEBYTECODE = '1'
.\.venv_local\Scripts\python.exe -m bft65b.panel.population show --project-root C:\NEU\BFT65B --run-id population20261115a
.\.venv_local\Scripts\python.exe -m bft65b.panel.population verify --project-root C:\NEU\BFT65B --run-id population20261115a
# Use a fresh ID to reproduce without overwriting the accepted run.
.\.venv_local\Scripts\python.exe -m bft65b.panel.population run --project-root C:\NEU\BFT65B --run-id population_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_panel_population.py
```

The saved `summary.json` has all 166 quarter reconciliations and the
two-/four-/eight-quarter sensitivity counts. `report.md` is an English
report-ready table; the 166 Parquet membership files contain no predictor
values. Read a saved summary without rerunning the pipeline:

```powershell
$code = @'
import json
from pathlib import Path
p = Path(r"C:\NEU\BFT65B\runs\task_2_6_population\population20261115a\summary.json")
s = json.loads(p.read_text(encoding="utf-8"))
print(s["overall"])
print(s["by_quarter"][-1])  # 2025Q2
'@
.\.venv_local\Scripts\python.exe -c $code
```

The current official FDIC History source was probed one row at a time; it
was not fully acquired or reconciled. Its `PROCDATE` is a process date, not
a verified historical public-release date. The modelling objective still
requires a decision on whether a labelled recent-reporter study is an
acceptable scope or a complete active-bank history must be established.

## Task 3.0 Spark runtime acceptance on the verified Windows host

The local synthetic acceptance passed twice on 2026-11-10. It requires Python 3.11.9 in `.venv_local`, JDK 17.0.21, the pinned PySpark 3.5.7/Hadoop 3.3.4 JARs, and the **already present** `tmp/hadoop-3.3.5/bin` native files. No new binary was downloaded for this task. The native files were previously sourced from `cdarlint/winutils` commit `7386986d5d8a079b5cd4464f4599766dd27e7d13`; verify SHA-256 before reuse: `winutils.exe` = `A0CA6E358357C41EF56EBDB02C38E4A4D55DA7CA7A13001678BB2EF7D644ADEA`, `hadoop.dll` = `D3DD64AFDC85F2A7EB5345ABF2ECAA744B0A157DE40859313337D47F81EE1C7B`. These ignored native files are not distributed with the repository. On another host without them, the Windows command below is not yet a validated setup; use a separately verified Linux runtime rather than assuming Spark Parquet works.

From `C:\NEU\BFT65B` in PowerShell:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:HADOOP_HOME = (Resolve-Path .\tmp\hadoop-3.3.5).Path
$env:PATH = (Join-Path $env:HADOOP_HOME 'bin') + ';' + $env:PATH
$env:PYSPARK_PYTHON = (Resolve-Path .\.venv_local\Scripts\python.exe).Path
$env:PYSPARK_DRIVER_PYTHON = $env:PYSPARK_PYTHON
Get-FileHash .\tmp\hadoop-3.3.5\bin\winutils.exe -Algorithm SHA256
Get-FileHash .\tmp\hadoop-3.3.5\bin\hadoop.dll -Algorithm SHA256
.\.venv_local\Scripts\python.exe .\tests\spark_runtime_acceptance.py *> .\tmp\spark_3_0_acceptance_manual.log
$LASTEXITCODE
Select-String .\tmp\spark_3_0_acceptance_manual.log -Pattern 'PASS|Traceback|Exception'
.\.venv_local\Scripts\python.exe -m pytest -q
```

The standalone check uses only synthetic records and writes its Parquet files and manifest under ignored `tmp/`. It verifies SparkSession startup, a DataFrame transformation, a Spark Parquet round trip, Spark reading the existing PyArrow `ParquetStore` output with dates, Decimal(20,4), nulls and normalized identifiers, and Spark MLlib fit/transform. A nonzero exit code is a failed gate. Full logs of the two verified runs are `tmp/spark_3_0_acceptance_run2.log` and `tmp/spark_3_0_acceptance_run3.log`; neither is tracked. The earlier `NativeIO$Windows.access0` failure was not reproduced in the corrected runs, so its exact historical cause is still unknown. An initial diagnostic with no `PYSPARK_PYTHON` failed at Python worker startup through the Windows App Execution Alias, which is a separate confirmed configuration error. WSL2 is not currently installed and was not used for this acceptance.

## P3.1 Polars lazy observed-panel workload

The accepted workload `observed_panel_quality_gaps_v1` reads the 170
Financials partitions of `treatment20261114a` and compares all projected
bank-quarter values with `panel20261115b`. It produces the observed relation,
resumed reporting gaps, and per-quarter quality counts. It does not use the
prediction population or create labels. The exact file list, schema, keys,
ordering, null/absence semantics, optimized plans, and output hashes are in
ignored `runs/task_3_1_polars/polars20261115a/contract.json` and
[P3_WORKLOAD.md](P3_WORKLOAD.md). This milestone does not measure comparative
runtime or memory.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:PYTHONDONTWRITEBYTECODE = '1'
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel show --project-root C:\NEU\BFT65B --run-id polars20261115a
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel plans --project-root C:\NEU\BFT65B --run-id polars20261115a --relation gaps
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel compare --project-root C:\NEU\BFT65B --run-id polars20261115a --candidate-dir C:\NEU\BFT65B\data_processed\task_3_1_polars\polars20261115a
# A new run ID writes a separate ignored output; accepted outputs stay immutable.
.\.venv_local\Scripts\python.exe -m bft65b.processing.polars_panel run --project-root C:\NEU\BFT65B --run-id polars_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_polars_panel.py
```

The pandas baseline and shared benchmark use this same contract. PySpark
reproduction remains the next Part 3 milestone.

## P3.2 pandas baseline and shared benchmark

The accepted `panelbench20261115a` run compared the pandas baseline and
Polars on all 170 accepted treatment partitions. Both engines passed exact
full-output parity with the frozen P3.1 reference before timing and after
every run. Each had one warm-up and five measured fresh processes. The timer
includes startup, complete transformation, materialization, and writing the
three outputs; the process-tree RSS sampler runs every 50 ms. The OS cache
was not flushed, so these are not cold-cache measurements. See
[P3_BENCHMARK.md](P3_BENCHMARK.md) for the full protocol, measured table,
environment, and limitations. A new benchmark requires a fresh run ID.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:PYTHONDONTWRITEBYTECODE = '1'
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel show --project-root C:\NEU\BFT65B --run-id panelbench20261115a
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel run --project-root C:\NEU\BFT65B --run-id panelbench_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_pandas_panel.py tests\test_panel_benchmark.py tests\test_polars_panel.py
```

The ignored `runs/task_3_2_panel_benchmark/panelbench20261115a/summary.json`
contains median/range and explicit pandas-denominator ratios. `records.json`
contains each process's duration, sampled peak RSS, sample count, exit code,
parity status, and log paths; `environment.json` records machine and cache
policy. The corresponding output Parquet files are under ignored
`data_processed/task_3_2_panel_benchmark/panelbench20261115a/`.

## P3.3 local Spark parity and three-engine benchmark

The accepted `panelbench20261115b` run uses the same frozen P3.1 workload
with native Spark DataFrame expressions. It passed exact full-row parity for
the observed relation, resumed gaps, and quarterly quality counts, then
reran pandas, Polars, and Spark together under the shared startup-inclusive
benchmark protocol. Spark's before/after adaptive physical plans and actual
settings are saved in the ignored run directory. The report-ready parity,
measurement, plan interpretation, and limitations are in
[P3_SPARK.md](P3_SPARK.md). P3.4 scaling assessment remains open.

From `C:\NEU\BFT65B` in PowerShell, with the documented Task 3.0 native
runtime present and hash-verified:

```powershell
$env:PYTHONPATH = (Resolve-Path .\src).Path
$env:HADOOP_HOME = (Resolve-Path .\tmp\hadoop-3.3.5).Path
$env:PATH = (Join-Path $env:HADOOP_HOME 'bin') + ';' + $env:PATH
$env:PYSPARK_PYTHON = (Resolve-Path .\.venv_local\Scripts\python.exe).Path
$env:PYSPARK_DRIVER_PYTHON = $env:PYSPARK_PYTHON
.\.venv_local\Scripts\python.exe .\tests\spark_runtime_acceptance.py *> .\tmp\spark_3_3_runtime_recheck.log
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel show --project-root C:\NEU\BFT65B --run-id panelbench20261115b
# A repeat writes a fresh ignored namespace and all 15 measurements.
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.panel run --project-root C:\NEU\BFT65B --run-id panelbench_new_unique_id --include-spark
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_spark_panel.py tests\test_panel_benchmark.py tests\test_pandas_panel.py tests\test_polars_panel.py
```

## P3.4 scaling assessment and saved-evidence review

[P3_SCALING_ASSESSMENT.md](P3_SCALING_ASSESSMENT.md) contains the English
report-ready three-engine table, the measured laptop-capacity argument for
Q1, and the P3.1-P3.4 rubric audit. The review gate independently checked
the same 170 source files, 35 complete saved output sets, 300 protected
artifact hashes, both benchmark records and the executed Polars/Spark plans.
It did not rerun performance measurements or access the FDIC API. The
historical two-engine results remain separate from the same-run three-engine
comparison. Final report placement is still pending.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
# Fast: view the saved review without rescanning data or rerunning benchmarks.
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.part3_review show --project-root C:\NEU\BFT65B --run-id part3review20261115a
# Slower read-only gate: use a fresh ID for a new ignored evidence record.
.\.venv_local\Scripts\python.exe -m bft65b.benchmarks.part3_review verify --project-root C:\NEU\BFT65B --run-id part3review_new_unique_id
```

## P4.1 bounded feature dictionary and executable contract

[FEATURE_DICTIONARY.md](FEATURE_DICTIONARY.md) defines seven source-lineage
features, exact calendar-quarter lookbacks, null and denominator rules, and
the accepted provisional availability timing. This milestone validates the
accepted panel and PIT evidence; it does not create labels, a production
feature table, or fitted preprocessing. Its ignored run is
`runs/task_4_1_feature_contract/featurecontract20261116b/`.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
# Fast saved result; no FDIC network request or source-data rescan.
.\.venv_local\Scripts\python.exe -m bft65b.features.contract show --project-root C:\NEU\BFT65B --run-id featurecontract20261116b
# Recheck accepted panel and PIT inputs using a fresh ignored run ID.
.\.venv_local\Scripts\python.exe -m bft65b.features.contract validate --project-root C:\NEU\BFT65B --run-id featurecontract_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_feature_contract.py tests\test_panel_asof.py tests\test_panel_prediction.py
```

The saved machine-readable dictionary can be inspected without rerunning
the validation:

```python
import json
from pathlib import Path

path = Path(r"C:\NEU\BFT65B\runs\task_4_1_feature_contract\featurecontract20261116b\contract.json")
contract = json.loads(path.read_text(encoding="utf-8"))
for feature in contract["features"]:
    print(feature["name"], feature["formula"], feature["availability_rule"])
```

## P4.2 retrospective cohort and production labels

The accepted historical study cohort is `membership20261120a`; the accepted
four-quarter labels are `labels20261120a`. The policy in
[RETROSPECTIVE_COHORT_POLICY.md](RETROSPECTIVE_COHORT_POLICY.md) uses official
effective activity dates for retrospective membership. It does not claim that
the current structural snapshot was publicly known at every historical q.
Financials predictors still obey the provisional 120-day as-of rule. The
accepted label table preserves positive, negative, unresolved and excluded
decisions, including rows with no eligible feature history. No feature-label
join, split or model is part of this milestone.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.panel.retrospective verify --project-root C:\NEU\BFT65B --run-id membership20261120a
.\.venv_local\Scripts\python.exe -m bft65b.features.production verify --project-root C:\NEU\BFT65B --run-id labels20261120a
# Recreate the full outputs only with fresh, unique ignored run IDs:
.\.venv_local\Scripts\python.exe -m bft65b.panel.retrospective run --project-root C:\NEU\BFT65B --run-id membership_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.features.production run --project-root C:\NEU\BFT65B --membership-run membership_new_unique_id --run-id labels_new_unique_id
# Reconcile saved runs; a new review ID writes only ignored aggregate evidence:
.\.venv_local\Scripts\python.exe -m bft65b.panel.retrospective_review --project-root C:\NEU\BFT65B --membership-run membership20261120a --label-run labels20261120a --review-id review_new_unique_id
# Recheck both accepted pairs and compare all 333 physical output hashes:
.\.venv_local\Scripts\python.exe -m bft65b.panel.retrospective_review --project-root C:\NEU\BFT65B --membership-run membership20261120a --label-run labels20261120a --repeat-membership-run membership20261120b --repeat-label-run labels20261120b --review-id repeat_review_new_unique_id
# Read the saved 2025Q2 reconciliation without downloading or recomputing:
.\.venv_local\Scripts\python.exe -c "import json; from pathlib import Path; p=Path(r'C:\NEU\BFT65B\runs\task_4_2_review\p42review20261120b\summary.json'); s=json.loads(p.read_text(encoding='utf-8')); print(s['q2025q2'])"
```

The complete quarter-level counts and SHA-256 source lineage are in the
ignored `runs/task_4_2_membership/membership20261120a/`,
`runs/task_4_2_production_labels/labels20261120a/`, and
`runs/task_4_2_review/p42review20261120b/` directories. For a concise read:

```python
import json
from pathlib import Path

root = Path(r"C:\NEU\BFT65B\runs")
review = json.loads((root / "task_4_2_review" / "p42review20261120b" /
                     "summary.json").read_text(encoding="utf-8"))
print(review["all_bank_resolution"])
print(review["q2025q2"])
```

## P4.3 production feature–label table and preprocessing

The accepted seven-feature formulas and lineage rules are in
[FEATURE_DICTIONARY.md](FEATURE_DICTIONARY.md). The production table policy,
join contract, and preprocessing rules are in
[FEATURE_TABLE_POLICY.md](FEATURE_TABLE_POLICY.md). This workflow reuses the
accepted observed report history, PIT selection, retrospective membership,
and production labels without making FDIC requests. Run IDs must be new;
the accepted runs are immutable.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m bft65b.features.table run --project-root C:\NEU\BFT65B --run-id featuretable_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.features.table verify --project-root C:\NEU\BFT65B --run-id featuretable_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.features.table show --project-root C:\NEU\BFT65B --run-id featuretable_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_feature_table.py tests\test_feature_preprocessing.py tests\test_feature_contract.py
# Inspect the accepted run and its independent repeat evidence without rebuilding:
.\.venv_local\Scripts\python.exe -m bft65b.features.table show --project-root C:\NEU\BFT65B --run-id featuretable20261120a
Get-Content .\runs\task_4_3_feature_table\featuretable20261120b\repeat_comparison.json
```

`verify` rechecks saved output schemas, keys, codecs, checksums, and recorded
upstream hashes; `show` reads only the saved completion evidence. The model
Parquet files contain `CERT`, `Q`, `Y`, and the seven ordered predictors. Use
only `PREDICTOR_COLUMNS` from `bft65b.features.preprocessing` as model input.
The factory `make_preprocessing_pipeline()` returns an **unfitted** pipeline;
Part 5 must fit a fresh instance inside each development training fold.

To inspect saved counts and nulls without rebuilding the table:

```python
import json
from pathlib import Path

path = Path(r"C:\NEU\BFT65B\runs\task_4_3_feature_table\featuretable20261120a\summary.json")
summary = json.loads(path.read_text(encoding="utf-8"))
print(summary["overall"])
print(summary["model_nulls"])
print(summary["by_quarter"][-1])
```

## P4.4 temporal-leakage audit

[P4_LEAKAGE_AUDIT.md](P4_LEAKAGE_AUDIT.md) defines the production-path
fixtures, the all-quarter alignment, the fixed-seed source-history sample,
negative controls, and measured limitations. The audit reads accepted runs
without rewriting them. Use a new run ID when reproducing it.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_leakage_audit.py tests\test_feature_table.py tests\test_feature_contract.py tests\test_feature_label.py tests\test_feature_preprocessing.py
.\.venv_local\Scripts\python.exe -m bft65b.features.leakage_audit run --project-root C:\NEU\BFT65B --run-id p44audit_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.features.leakage_audit verify --project-root C:\NEU\BFT65B --run-id p44audit_new_unique_id
# Inspect the accepted result without recomputing the full audit:
.\.venv_local\Scripts\python.exe -m bft65b.features.leakage_audit show --project-root C:\NEU\BFT65B --run-id p44audit20261120a
Get-Content .\runs\task_4_4_leakage_audit\p44audit20261120a\summary.json
# Use a short ignored pytest temp path on Windows:
.\.venv_local\Scripts\python.exe -m pytest -q -rs -p no:cacheprovider --basetemp C:\NEU\BFT65B\tmp\pytest_new_unique_id
```

The accepted run's `hash_inventory.json` contains 1,192 before/after input
hashes; `sample_checks.json` contains detailed identifiers outside Git. A
saved summary may be read directly without repeating the scan:

```python
import json
from pathlib import Path

path = Path(r"C:\NEU\BFT65B\runs\task_4_4_leakage_audit\p44audit20261120a\summary.json")
summary = json.loads(path.read_text(encoding="utf-8"))
print(summary["full_alignment"]["totals"])
print(summary["sample"])
print(summary["checks"])
```

## Historical P4.2 aggregate preflight and former population blocker

[LABEL_POLICY.md](LABEL_POLICY.md) separates membership knowable at `q`,
event identity/linkage and the later FDIC failure outcome. The pure label
function and synthetic tests implement the official four-quarter window.
The accepted recent-reporter run is a proxy, not approval to create a
modelling sample. `labelpreflight20261116b` is an ignored aggregate-only
audit with `production_label_status=BLOCKED`; it contains no bank-quarter
label dataset. The user selected the active-bank objective. The bounded
complete official-source coverage audit and its unresolved historical-vintage gate are in
[ACTIVE_MEMBERSHIP_RESEARCH.md](ACTIVE_MEMBERSHIP_RESEARCH.md).

The following paragraph records the 2026-11-19 checkpoint, which was
superseded by the accepted retrospective-cohort runs above.

The supplemental 2026-11-19 audit saved all 27,835 current Institutions rows
and all 119,475 institution-role History events effective in 1984–2025 in an
ignored namespace. Its first analysis misread the History ID location and
is superseded; use `activeaudit20261119e` for the corrected coverage report.
The conditional rule in `src/bft65b/panel/membership_rule.py` is tested on
synthetic q-known events only. No production membership or label dataset has
been published.

```powershell
Set-Location C:\NEU\BFT65B
$env:PYTHONPATH = (Resolve-Path .\src).Path
# A repeat requires a fresh ignored run ID; no FDIC request or labels are written.
.\.venv_local\Scripts\python.exe -m bft65b.features.label --project-root C:\NEU\BFT65B --run-id labelpreflight_new_unique_id
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_feature_label.py tests\test_feature_contract.py tests\test_panel_repair.py tests\test_panel_population.py tests\test_panel_prediction.py
# Reanalyse the 45 checksummed FDIC source responses without network access.
.\.venv_local\Scripts\python.exe -m bft65b.panel.active_audit --project-root C:\NEU\BFT65B --from-run activeaudit20261119a --run-id activeaudit_new_unique_id
.\.venv_local\Scripts\python.exe -m bft65b.panel.active_audit --project-root C:\NEU\BFT65B --run-id activeaudit20261119e --verify
.\.venv_local\Scripts\python.exe -m pytest -q -p no:cacheprovider tests\test_membership_audit.py tests\test_feature_label.py
```

To inspect the saved reconciliation without rerunning the audit:

```python
import json
from pathlib import Path

path = Path(r"C:\NEU\BFT65B\runs\task_4_2_label_preflight\labelpreflight20261116b\preflight.json")
result = json.loads(path.read_text(encoding="utf-8"))
print(result["production_label_status"])
print(result["population_scope"]["overall"])
print(next(row for row in result["population_scope"]["by_prediction_quarter"]
           if row["q"] == "20250630"))

audit = json.loads(Path(r"C:\NEU\BFT65B\runs\task_4_2_active_membership\activeaudit20261119e\summary.json").read_text(encoding="utf-8"))
print(audit["snapshot_reconciliation"])
print(audit["panel_cert_coverage"])
print(audit["candidate_2025q2"])
```

## Reproducibility and attribution

### Captured-vintage portable membership continuation

The portable root is an ignored, separate data namespace. Its original raw source bundle and completed audit → treatment → panel → transaction repair → PIT outputs must pass their saved validators before cohort replay. A separate 49-file membership-source bundle provisions the captured Institutions/History pages, official code probe and assignment; no live FDIC request is made. See [P6_REPRODUCIBILITY.md](P6_REPRODUCIBILITY.md) for the exact accepted run IDs and integrity evidence. Use fresh IDs for new runs; the IDs below are saved-run **verification** commands:

```powershell
$portable = 'C:\NEU\BFT65B\tmp\p6_repro_20261123b\portable'
$python = 'C:\NEU\BFT65B\tmp\p6_repro_20261123a\repo\.venv_clean\Scripts\python.exe'
$env:PYTHONPATH = Join-Path $portable 'src'
& $python -m bft65b.quality.source_vintage verify --project-root $portable
& $python -m bft65b.panel.prediction verify --project-root $portable --run-id portable20261124c_pit
& $python -m bft65b.panel.membership_source_vintage verify --project-root $portable
& $python -m bft65b.panel.active_audit --project-root $portable --run-id portable20261124aaudit --verify
& $python -m bft65b.panel.retrospective verify --project-root $portable --run-id portable20261124acohort
& $python -m bft65b.features.production verify --project-root $portable --run-id portable20261124alabels
& $python -m bft65b.features.contract show --project-root $portable --run-id portable20261124acontract
& $python -m bft65b.features.table verify --project-root $portable --run-id portable20261124afeatures
```

To generate a **new** cohort from the captured vintage, provision the membership bundle into a new portable root with `bft65b.panel.membership_source_vintage provision --source-root C:\NEU\BFT65B --destination-root <new-portable-root>`, then use `bft65b.panel.active_audit --from-run activeaudit20261119a --panel-run <portable-panel> --pit-run <portable-pit> --repair-run <portable-repair> --run-id <fresh-audit>` and `bft65b.panel.retrospective run --audit-run <fresh-audit> --panel-run <portable-panel> --candidate-run <portable-pit> --code-probe codeprobe20261119a --run-id <fresh-cohort>`. Each command requires `--project-root <new-portable-root>` and a clean Python `PYTHONPATH` pointing at that root's `src`. Historical accepted manifests are never rewritten for relocation.

The next label step is `bft65b.features.production run --project-root <new-portable-root> --run-id <fresh-label> --membership-run <fresh-cohort> --repair-run <portable-repair>`. Its verifier reads saved evidence with `bft65b.features.production verify --project-root <new-portable-root> --run-id <fresh-label>`. In portable mode the repair ID must match the cohort audit's recorded repair lineage; the CLI will not use the historical repair default.

Feature replay next validates the same seven-feature contract with `bft65b.features.contract validate --project-root <new-portable-root> --run-id <fresh-contract> --panel-run <portable-panel> --pit-run <portable-pit>`. The P4.2 review requires two independently regenerated cohort/label pairs; run `bft65b.panel.retrospective_review --project-root <new-portable-root> --membership-run <primary-cohort> --label-run <primary-labels> --repeat-membership-run <repeat-cohort> --repeat-label-run <repeat-labels> --candidate-run <portable-pit> --review-id <fresh-review>`. Then run `bft65b.features.table run --project-root <new-portable-root> --run-id <fresh-feature-table> --panel-run <portable-panel> --pit-run <portable-pit> --contract-run <fresh-contract> --membership-run <primary-cohort> --label-run <primary-labels> --review-run <fresh-review>`. The CLI verifies lineage, source hashes and keys before writing. Use fresh IDs; the saved IDs above are for read-only verification. This replay does not fit preprocessing or use final-test predictors.

The portable temporal-leakage audit requires both independently generated feature tables and the same explicit upstream chain. The established repeat `portable20261124bfeatures` passed all 332 output checksums against `portable20261124afeatures`; the audit `portable20261124aleakage` passed saved verification. These are read-only checks of occupied IDs:

```powershell
& $python -m bft65b.features.table verify --project-root $portable --run-id portable20261124bfeatures
& $python -m bft65b.features.leakage_audit verify --project-root $portable --run-id portable20261124aleakage
```

For a new isolated audit, use a fresh run ID and pass `--panel`, `--pit`, `--feature-contract`, `--membership`, `--labels`, `--feature-table`, `--independent-repeat` and `--p42-review` with the validated run IDs from that portable root. The command rejects omitted portable selections and mismatched feature-table lineage. It does not fit or score the final test.

The chronological split can now consume the verified portable lineage without falling back to historical derived runs. This command verifies the occupied replay; it does not refit a model or score test predictors:

```powershell
& $python -m bft65b.modelling.split verify --project-root $portable --run-id portable20261124asplit
```

To regenerate in a separate portable root, use a fresh run ID with `bft65b.modelling.split run --project-root <new-portable-root> --run-id <fresh-split> --membership <portable-cohort> --labels <portable-labels> --model-table <portable-primary-features> --model-repeat <portable-repeat-features> --leakage-audit <portable-leakage-audit> --repaired-events <portable-transaction-repair>`. The runner verifies upstream hashes and lineage and requires the already frozen 1984Q1–2012Q4 development, 2013Q1–Q4 gap and 2014Q1–2025Q2 test boundaries. The split reads model keys and labels for the previously authorized feasibility audit; it does not access test predictor values for fitting or scoring.

The frozen development replay is verified without fitting again by:

```powershell
& $python -m bft65b.modelling.development verify --project-root $portable --run-id portable20261124adev
& $python -m bft65b.modelling.development verify --project-root $portable --run-id portable20261124bdev
```

The accepted A/B repeat evidence is in the second run's `repeat_comparison.json`. For a **new** clean replay, first run `bft65b.modelling.development plan --project-root <new-portable-root> --run-id <fresh-development> --split-run <verified-portable-split>` and inspect the frozen plan. Then run `bft65b.modelling.development execute --project-root <new-portable-root> --run-id <fresh-development>`. The runner opens only development predictors, retains the accepted fold/candidate/seed policy and does not score the final test. Do not reuse the occupied IDs above.

The completed portable Spark MLlib demonstration is checked without fitting again by:

```powershell
& $python -m bft65b.modelling.spark_demo verify --project-root $portable --run-id portable20261124aspark
& $python -m bft65b.modelling.spark_demo verify --project-root $portable --run-id portable20261124bspark
```

These runs used the verified portable native Hadoop files and explicit `--split`, `--table`, `--development-primary`, `--development-repeat` and `--leakage` selections. See `P6_REPRODUCIBILITY.md` for parity, settings and the current-process `HADOOP_HOME` requirement. A new execution requires a fresh run ID; neither command evaluates the final test.

The completed portable development COST audit is checked without reprocessing FDIC events by:

```powershell
& $python -m bft65b.modelling.cost_audit verify --project-root $portable --run-id portable20261124acost
& $python -m bft65b.modelling.cost_audit verify --project-root $portable --run-id portable20261124bcost
```

New replay runs require a fresh ID and explicit `--split`, `--development-primary`, `--development-repeat` and `--repair` IDs from the same captured source vintage. These COST values are retrospective outcomes in nominal USD thousands and never enter predictors. The accepted scenario and test boundary remain unchanged.

Future runs will store configuration, Python/dependency versions, input identities, source totals, checksums, validation outcomes, timestamps, and status in JSON manifests. The report must cite actual measurements and all external code, papers, and AI assistance. Current sources include the course assignment PDF and starter script, the [official FDIC API documentation](https://api.fdic.gov/banks/docs/), the [OpenAPI specification](https://api.fdic.gov/banks/docs/swagger.yaml), and the endpoint YAML dictionaries linked in SOURCE_FIELD_INVENTORY.md. Project planning and implementation used OpenAI Codex assistance; the final report must disclose that use.
