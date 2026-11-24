# BFT65B Project Plan

## Authority and objective

`final_report_bft65b.pdf` is the authoritative specification. `fdic_download_starter.py` is a starting point, not proof that its field list, download, or checks satisfy the rubric. Build a reproducible bank failure early-warning data pipeline on FDIC data. This is primarily a data-engineering project; conclusions must be supported by observed measurements. Do not invent row counts, split failure counts, quality findings, timings, costs, or model scores.

The required source scope is Financials for all insured banks from 1984Q1 through 2026Q2, plus complete Failures and Institutions tables. Financial amounts are in thousands of USD and ratios in percent. Select and justify fields using the FDIC dictionary. For modelling, use observation quarters no later than 2025Q2. The label for bank `i` at quarter `q` is 1 only when that bank fails in `(q, q + 4 quarters]`; a merger is not a failure.

## Proposed architecture and data flow

```text
FDIC API or bulk source -> acquisition -> immutable raw Parquet + source manifest
                    -> quality audit -> curated bank-quarter panel
                    -> point-in-time features + four-quarter labels
                    -> chronological development/validation/test split
                    -> training-only preprocessing and models -> locked test evaluation
                    -> tables, figures, execution plans, benchmark log, report
```

Use a Python package under `src/bft65b/` with `acquisition`, `quality`, `panel`, `processing`, `features`, `modelling`, and `reporting` modules. A central configuration records dates, selected fields, storage paths, random seeds, and benchmark settings. A CLI runs individual stages or the entire DAG; `tests/` holds fixture-based unit and integration tests. Record input identity, configuration, package versions, seed, run date, stage status, and validation outcomes in run manifests. Keep raw and derived datasets outside Git; commit code, tests, small synthetic fixtures, manifests without raw records, and documented outputs needed for review.

Use pandas for a baseline and interoperability, Polars lazy operations for the full panel, PySpark for a comparable local demonstration, and scikit-learn for preprocessing and modelling. Document the Parquet schema, quarter partitions, compression, and any type conversions. Verify that physical layouts and engine behavior meet claims with measurements rather than assumptions.

### Proposed storage contract (subject to FDIC dictionary and pilot validation)

Store each Financials quarter as `data_raw/fdic/financials/REPDTE=YYYYMMDD/part-000.parquet`; store `failures.parquet` and `institutions.parquet` separately. Treat `data_raw/` as an ignored, immutable landing zone. Retain source response metadata in the manifest (endpoint, query, retrieval time, reported total, field list, file checksum). Partition by quarter for pruning and resumability, not by `CERT`; sort records by `(CERT, REPDTE)` before writing for deterministic checks. Start with Zstandard level 3 and measure it against Snappy and CSV on identical records. Do not claim a compression winner before measurement.

Proposed logical types: Financials and Institutions `CERT` as non-null Int64 after validating identifiers; Failures `CERT` as nullable Int64 for bank linkage because the pilot observed 488 source nulls. Failures source `ID` is required, unique, and checked on every complete response; it is not a bank identifier. `REPDTE`, `FAILDATE`, and other source dates are Date after explicit format parsing; source names, state, class, change/reason codes are UTF-8 strings; ACTIVE is nullable Int8 (0/1) while source status semantics are retained; financial amounts are Decimal(20,4) in the FDIC's stated thousands-of-USD unit; Failures `COST` is float64 matching its official double source type; published ratios are Float64 in percent. Keep source nulls distinct from zero, preserve original field names, and record any field requiring a different type or scale. The versioned implementation and exact Arrow fields are in STORAGE_SCHEMA.md. Keep raw response values available for audit if a conversion fails; do not silently coerce or discard a record.

Verify schema and layout by inspecting Parquet metadata, reading each partition back, checking key uniqueness, null/type conversion counts, source-to-file row counts, and checksums. Measure total/on-disk bytes and repeated full-scan and selected-quarter/column read time for CSV, Snappy Parquet, and Zstandard Parquet on the same data; report writer settings, hardware, library versions, and any CPU/storage tradeoff.

### FDIC acquisition pilot and full-run gates

Before the full download, run a four-quarter pilot on 2023Q1-2023Q4, as suggested by the starter script, plus small-table endpoint probes. Validate field codes and response shapes against the official dictionary; compare every page's cumulative rows to the API `meta.total`, use a stable sort key, check duplicate `(CERT, REPDTE)` and duplicate page overlap, and detect changing totals or empty intermediate pages. Inject or simulate HTTP errors/timeouts to verify bounded retries and backoff. Write each response to a temporary file, validate Parquet read-back and checksum, then atomically replace its final target and mark the manifest complete last. Test interruption, truncated/corrupt existing files, and rerun behavior: only a file whose manifest and on-disk validation both pass may be skipped. Never promote a partial quarter. The full run repeats these gates for all required quarters and both complete small tables; reconcile the manifest with physical files before downstream use.

The Task 1.1 contract review found no official guarantee that CERT is unique for every Failures transaction. The documented API accepts one sort_by field and at most 10,000 records per page. The 2023 pilot confirmed repeated and null Failures CERT values. If the complete Failures total remains at most 10,000, acquire it in one page, require a present and unique source `ID` for every row, retain all distinct source records, and reconcile the returned count. The observed API sort puts null CERT after numeric CERT; changing sort behavior fails validation. If the total exceeds 10,000 or any sort tie crosses pages, stop rather than claim completeness. An FDIC bulk export or a partitioned API method is an alternative only after its own totals, key policy, and source identity are verified. For every endpoint, preserve absent-versus-null requested-field information for the quality audit and reject missing source keys or Financials rows outside the requested quarter. Null Failures CERT values remain unresolved bank-linkage cases, not inferred failures for a specific bank.

### Temporal contract and leakage boundary

`REPDTE` is the financial statement's quarter-end; `FAILDATE` is the outcome event date. Neither is a publication timestamp. Define the prediction date for observation quarter `q` as that quarter-end, and retain the official target `FAILDATE in (q, q + 4 quarters]`. For a conservative first implementation, a report dated `r` becomes eligible as a feature only when `r + 120 calendar days <= prediction date`; use the latest eligible report and record its age. This 120-day lag is a modelling assumption, not a verified FDIC release schedule. Verify any available historical publication metadata and run sensitivity checks with alternative lags using development data only. Do not shift the target window when feature availability changes. Rows with no eligible financial history require an explicit missing-history policy; banks already failed by the prediction date are outside the risk set. Report residual uncertainty from revisions, delayed filings, and unavailable publication timestamps.

Global historical inspection may count raw rows, enumerate identifiers/field coverage, audit quality, and count outcomes by time to establish a feasible chronological split. It must not choose feature transformations, imputation values, scaling, winsorization cutoffs, encodings, feature selection, model parameters, class weights, cost ratios, or alarm thresholds using validation/test observations. Within each development fold, fit all statistical transformations on that fold's training slice and apply them unchanged to its later validation slice. After selection, refit on the permitted full development period and freeze before the single test evaluation. Leakage tests should shift a future record or extreme held-out value and prove earlier features and fitted statistics stay unchanged; test the exact target endpoints, publication-lag boundary, rolling windows, exit classification, fold isolation, and test-access log.

### Benchmark and cost protocols

Benchmark pandas, Polars, and local PySpark on identical immutable input and semantically equivalent transformations. Use one untimed warm-up per engine, then at least five measured runs in separate fresh processes with execution fully materialized and output written to the same sink or excluded consistently. Measure wall time with a monotonic clock and peak process-tree resident memory with periodic sampling (including Spark driver and local workers); record sampling interval, hardware/OS, thread/worker counts, library versions, cache policy, input size, and run order. Report all runs plus median and range; distinguish startup-inclusive from transformation-only timing if both are shown. Before comparing speed, canonicalize output ordering, null representation, date/decimal types, and check keys, row counts, and numeric values with a stated tolerance.

For cost-sensitive validation, `COST` from Failures is an observed deposit-insurance-fund resolution cost, not the causal loss preventable by an alarm. Summarize its coverage, units, missingness, distribution, and time variation using development outcomes only. Use it as a documented proxy for missed-failure severity, with explicit assumed preventable fraction. FDIC tables do not directly provide supervisory investigation cost for false alarms: define that parameter transparently as an assumption, show a plausible range, and perform a two-dimensional sensitivity analysis over investigation cost and preventable fraction. Choose the threshold using development-fold predictions and expected cost under a declared reference scenario; report threshold stability and avoid presenting proxy costs as observed benefits. Freeze the reference scenario and threshold before test evaluation.

## Rubric traceability

Each row requires a deliverable, an executable check or explicit review gate, and report evidence. `TASKS.md` defines the work order.

| ID | Requirement | Code or deliverable | Test / acceptance gate | Report evidence |
| --- | --- | --- | --- | --- |
| P1.1 | Reproducible, preferably resumable acquisition of all required data | Downloader, configuration, download manifest | Full quarter range present; Failures and Institutions totals checked against source; rerun skips only verified complete files | Method, date, field choice, counts, completeness table |
| P1.2 | Efficient format, schema, partitioning, compression | Documented Parquet layout and schema validator | Read-back preserves keys/types; partitions and compression inspected | Layout description and rationale |
| P1.3 | Demonstrate advantage over raw CSV | Same-data CSV/Parquet benchmark | Identical rows/columns; repeatable size and read-time measurement | Size and timing table with environment |
| P2.1 | Automated audit of actual raw-data problems | DQ checks and machine-readable report | Synthetic defect fixtures trigger each check; full-data audit recorded | Problems, prevalence, handling, before/after counts |
| P2.2 | Clean bank-quarter panel | Panel builder and quality log | Unique `(CERT, REPDTE)`; valid chronology and documented missing-value policy | Panel counts and cleaning decisions |
| P2.3 | Correct failure versus other exit handling | Failure/exit classification logic | Fixtures for failure, merger, other exit, and ambiguous records | Exit categories and worked examples |
| P2.4 | Point-in-time correctness | Availability-time policy and join guards | No post-observation fields in features; temporal boundary tests | Timeline and leakage controls |
| P3.1 | Efficient Polars core transformations and engine explanation | Lazy Polars implementation and captured plan | Correct output; plan inspected for scans, projections, filters, joins | Design choices and engine behavior |
| P3.2 | Fair pandas baseline | Equivalent pandas implementation and benchmark harness | Same input/output; fixed environment, warmup, repetitions, memory and runtime protocol | Comparable benchmark table |
| P3.3 | Reproduce key steps in PySpark and explain plan | Spark implementation and plan capture | Key, null, aggregate, and numeric parity within stated tolerance | Spark plan and parity results |
| P3.4 | Argue when a cluster is justified | Scaling analysis | Conclusion tied to measured laptop workload and stated bottlenecks | Q1 answer with measured thresholds/limits |
| P4.1 | Motivated, point-in-time risk features and label | Feature dictionary, feature/label builder | Four-quarter label boundary and no-future-data fixtures | Feature rationale, lineage, missingness |
| P4.2 | Train-only preprocessing | scikit-learn `Pipeline`/`ColumnTransformer` | Fit-state and fold-isolation tests; held-out values do not affect fit | Preprocessing and split description |
| P5.1 | Baseline, own model(s), short Spark MLlib demo | Model training and MLlib demonstration | Fixed seed; comparable input and split; demo runs | Model comparison table |
| P5.2 | Chronological, imbalanced-data validation | Split manifest and development-only tuning | Test follows development; gap >= 4 quarters; test >= 50 failures; no random CV | Periods and failure counts; justified metrics |
| P5.3 | Data-based false-alarm/missed-failure threshold | Development-only cost estimation and threshold selection | Threshold frozen before test; assumptions and units recorded | Cost source, tradeoff curve, chosen threshold |
| P5.4 | Evaluate on test once | Locked evaluation stage and run record | One final test evaluation after choices frozen | Held-out results and uncertainty/limits |
| P6.1 | Answer Q1-Q3 and discuss Vietnam | Conclusions section | Each answer cites pipeline evidence; limitations and transfer issues reviewed | Evidence-backed conclusions |
| R.1 | End-to-end rerun from README | README, pinned `requirements.txt`, CLI | Clean-environment smoke run follows README | Reproduction instructions and run metadata |
| S.1 | English report, <=20 pages plus <=10 appendix pages | PDF or DOCX report | Page count and language review | Final report |
| S.2 | Required first-page information | Report cover/first page | Name, student ID, GitHub URL, method/date, row counts, periods and failure counts present | First page |
| S.3 | Public GitHub repo, code/notebooks with outputs and fixed seeds; no data committed | Repository and release checklist | Public link resolves during grading window; tracked-file scan excludes data; seeds and outputs checked | GitHub URL and README |
| S.4 | Cite sources including code, papers and AI tools | Citation register in README and report | Every external source has attribution | References |

## Method decisions and risk controls

- **Publication lag:** apply the temporal contract above before feature construction. The assumed 120-day lag does not prove actual availability; document historical release evidence if found and retain the uncertainty. No feature may depend on a later availability date.
- **Bank exits:** identify failures from Failures by `CERT` and `FAILDATE`; use Institutions to investigate other reporting exits. `ACTIVE`, `ENDEFYMD`, and `CHANGEC1` must not be treated as standalone failure labels or historical features without availability evidence. Audit unmatched and ambiguous identifiers.
- **API completeness:** validate FDIC field codes and endpoint behavior against official documentation; check stable pagination, per-query totals, duplicates, empty/changed pages, and read-back. An incomplete quarter must not be marked complete or silently used downstream.
- **Data quality:** audit missingness by field/period, type and unit changes, impossible values, duplicate keys, inconsistent dates, and join coverage. Preserve raw values and log every exclusion/imputation rule with counts.
- **Validation:** inspect actual failure counts before fixing split dates. Development includes all tuning, feature selection, preprocessing, cost estimation, and threshold selection. Use forward-time folds within development; retain at least four excluded quarters before the later test period. Evaluate test once and record the locked specification.
- **Censoring:** observations without four quarters of outcome follow-up cannot receive a confirmed negative label. Keep financial acquisition through 2026Q2, but stop modelling observations at 2025Q2 per the PDF and verify actual outcome coverage.

## Submission gates

The final English report answers Q1-Q3 and covers all six parts with tables, figures, or measurements. It is at most 20 pages plus a 10-page appendix. Its first page states name, student ID, GitHub URL, download method/date, row counts, and development/test periods with failure counts. The repository must be public from the deadline through grade release and contain a rerunnable README, pinned requirements, code or notebooks with outputs and fixed seeds, and no data files. Cite every external source, including AI assistance, in both README and report. These are future gates; no result or submission status is claimed by this plan.
