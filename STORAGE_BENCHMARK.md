# Task 1.6 Storage Experiment: Report-Ready P1.3 Evidence

## Accepted input and frozen comparison

The experiment uses the accepted `real20261111b` Financials snapshot: 170 quarterly Parquet files from 1984Q1 through 2026Q2, 1,682,616 rows, and Financials schema v2. All source paths came from the complete accepted manifest, not a directory glob. Before conversion and again after timing, all 170 files passed the original ParquetStore schema, partition, key, row-count, ZSTD codec, and SHA-256 checks. The physical raw schema has 28 columns: the 26 selected FDIC Financials fields plus `ID` and `SOURCE_MISSING_FIELDS`. Both provenance columns were validated in the raw schema and excluded explicitly from the common 26-column benchmark schema. The benchmark never rewrote raw files.

The checked-in specification is `config/storage_benchmark_v1.json`. Its source-derived Arrow type list and selective expected row count were frozen in ignored `runs/task_1_6_storage/frozen_spec.json` **before** the first benchmark conversion or timed read. Each representation contains exactly 170 quarterly files with identical logical rows, column order, and within-quarter row order. CSV is uncompressed; Parquet variants differ only by Snappy versus ZSTD level 3. All Parquet files passed physical footer codec checks. Raw v2 files are not one of the three comparison representations.

## P1.3 comparison table

Stored bytes sum all 170 files of each format, including for the 40-quarter selective workload. Each timing is a fresh worker process and includes file open, decoding, and full Arrow materialization. Values displayed here are rounded only for presentation; exact nanosecond durations, bytes, and calculations are in `runs/task_1_6_storage/timings.json`.

| Workload | Format | Files read | Rows × columns | Stored bytes (170 files) | Size / CSV | Five read times (s) | Median (s) | Min–max (s) | Speedup / CSV |
| --- | --- | ---: | ---: | ---: | ---: | --- | ---: | ---: | ---: |
| Full | CSV | 170 | 1,682,616 × 26 | 469211854 | 100.000% | 8.733188, 8.973475, 9.227246, 5.926109, 5.751443 | 8.733188 | 5.751443–9.227246 | 1.000× |
| Full | Snappy Parquet | 170 | 1,682,616 × 26 | 203331205 | 43.335% | 1.685006, 1.670056, 1.616103, 1.699006, 1.797475 | 1.685006 | 1.616103–1.797475 | 5.183× |
| Full | ZSTD-3 Parquet | 170 | 1,682,616 × 26 | 153574569 | 32.730% | 1.992929, 1.972710, 1.938333, 2.285467, 2.153304 | 1.992929 | 1.938333–2.285467 | 4.382× |
| Selective | CSV | 40 | 262,595 × 8 | 469211854 | 100.000% | 0.434444, 0.358413, 0.431895, 0.603207, 0.396176 | 0.431895 | 0.358413–0.603207 | 1.000× |
| Selective | Snappy Parquet | 40 | 262,595 × 8 | 203331205 | 43.335% | 0.081973, 0.084119, 0.097388, 0.088683, 0.081912 | 0.084119 | 0.081912–0.097388 | 5.134× |
| Selective | ZSTD-3 Parquet | 40 | 262,595 × 8 | 153574569 | 32.730% | 0.096854, 0.122024, 0.119073, 0.101553, 0.095882 | 0.101553 | 0.095882–0.122024 | 4.253× |

Size relative to CSV (%) is `100 × format bytes / 469211854`. Read-time speedup is `median CSV time / median format time` for the same workload. The denominators are unchanged across rows of the same calculation.

## Workloads and validation

The full workload uses all 170 quarters and all 26 selected columns. The selective workload uses 2010Q1–2019Q4, inclusive, with `CERT, REPDTE, ASSET, DEP, EQ, ROA, NIMY, RBCRWAJ`; its 262,595 expected rows were counted from verified source partitions before measurement. All six format/workload combinations passed exact per-quarter row-level Arrow equality to the accepted source, including decimals, dates, finite float64 values, null-versus-zero, key uniqueness, quarter boundaries, column order, and row order. Maximum observed float difference was zero; no tolerance was used. No source rows were dropped or imputed.

CSV uses UTF-8, LF, a header, comma delimiter, minimal RFC-style quoting, a collision-checked `__BFT65B_NULL__` marker, exact decimal strings, ISO `YYYY-MM-DD` dates, and Python binary64 `repr` strings. PyArrow CSV reads use an explicit type schema, UTF-8 checking, one reader thread, and requested-column conversion. Parquet reads use `ParquetFile.read(..., use_threads=False)`. The CSV reader still parses row-oriented input; it cannot obtain Parquet's native column projection. Each Parquet writer uses format 2.6, 65,536-row groups, dictionary encoding off, statistics on, 1 MiB data pages, page version 1.0, page index off, and byte-stream split off. Conversion and parity validation are outside all read timings.

One untimed warm-up preceded five measured reads per format/workload: six warm-ups and 30 successful measurements. Format order rotated cyclically by repetition. Each read launched a new Python process; `time.perf_counter_ns` measured read and materialization, and Windows `GetProcessMemoryInfo` recorded peak process working set (maximum observed measured worker: 735662080 bytes). Garbage collection ran immediately before the timer. PyArrow reader threads were disabled; 16 logical processors were available. The OS cache was not flushed or verified, so these are OS-managed cache measurements, not cold-cache results. The CSV full-read times declined in later repetitions; this is a visible ordering/cache limitation, not a discarded observation.

Observed on this host, ZSTD-3 occupied fewer bytes than Snappy, while Snappy had a lower median read time in both workloads. Both Parquet formats read faster than CSV in these measurements. Columnar projection is a relevant architectural explanation for the selective workload, but this experiment does not isolate projection from encoding/compression or generalize to untested machines, caches, writers, or workloads.

## Environment, commands, and evidence

Windows build 10.0.26200; 13th Gen Intel Core i5-1340P, 16 logical processors, 16,780,767,232 bytes RAM; Samsung NVMe `MZVL4512HBLU-00BTW` with NTFS C: volume. Python 3.11.9, PyArrow 18.1.0, pandas 2.2.3, Polars 1.35.2, PySpark 3.5.7, NumPy 2.1.3, scikit-learn 1.6.1. C: free space at the beginning of measured reads was 245725630464 bytes. Exact machine, library, storage, worker, rotation, memory, and timing records are in the ignored evidence listed below. The tested host and OS-managed cache limit portability of the numeric findings.

Run the five commands in README's Task 1.6 section, then run `verify` or `show` without repeating the benchmark. `run` refuses to overwrite an existing timing attempt. Use a new benchmark namespace/specification to repeat the complete experiment. This recorded run had zero failed timed reads. Early development preflight/test failures and their resolutions are listed separately in ignored `runs/task_1_6_storage/unsuccessful_attempts.json`.

- `runs/task_1_6_storage/source_inventory.json`: accepted paths, rows, bytes, original SHA-256 values.
- `runs/task_1_6_storage/frozen_spec.json`: source-derived row expectation and logical types before conversion.
- `runs/task_1_6_storage/representations.json`: 510 benchmark paths, bytes, hashes.
- `runs/task_1_6_storage/parity.json`: six exact parity gates.
- `runs/task_1_6_storage/timings.json`: six warm-ups, all 30 individual measured reads, per-read peak memory, methods, environment, and summary.
- `runs/task_1_6_storage/source_postcheck.json`: unchanged accepted source hashes after timing.
- `runs/task_1_6_storage/hardware.json`: CPU model, disk model, filesystem and free space.
- `runs/task_1_6_storage/verification_console.json`: independent stored-evidence verification result.
- `runs/task_1_6_storage/focused_junit_final.xml` and `full_junit_final2.xml`: 16 focused and 205 complete-regression tests passed.

All paths above are relative to `C:\NEU\BFT65B` and are Git-ignored; they must be retained with the local evidence package. The source code, frozen configuration template, tests, and this report-ready table are the versionable artifacts.
