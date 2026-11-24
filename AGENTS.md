# BFT65B Working Instructions

## Source of truth and scope

- Treat `final_report_bft65b.pdf` as the authoritative assignment and grading rubric. Use `fdic_download_starter.py` only as an optional starting point. Resolve any conflict in favor of the PDF and record the decision.
- Keep code, comments, docstrings, project documentation, README, and the final report in English. A conversational explanation to the user may be in Vietnamese.
- Implement work in the dependency order in `TASKS.md`. Phase 0 builds only the foundation; do not treat an unchecked or unverified later-stage task as complete.
- Never invent measurements, dataset counts, data-quality findings, split failure counts, model scores, cost estimates, or citations. Mark unknowns as pending measurement.

## Codex task execution rules

- Take one bounded `TASKS.md` milestone at a time and limit edits to its assigned scope. Preserve the architecture and rubric mapping unless the authoritative PDF requires a change; explain any cross-scope edit.
- Before marking a milestone complete, run its acceptance gate and relevant unit/integration tests. Record the actual command, result, and artifact or manifest path. Never check a task box solely because code was written.
- At the end of each implementation task, report files changed, commands executed, actual results, and unresolved issues. Separate measured facts from assumptions.
- If a validation fails or source behavior contradicts the plan, stop the dependent work, report the blocker and evidence, and repair or seek a decision. Do not bypass, weaken, or silently skip a failed gate.

## Data and reproducibility

- Acquire Financials for 1984Q1-2026Q2 and complete Failures and Institutions data using a documented method. Verify field definitions against the FDIC dictionary before use.
- Keep raw and derived datasets outside Git. Use an ignored local data directory or a configured external path. Do not commit credentials, raw records, large outputs, or downloaded files. Small synthetic test fixtures are allowed.
- Pin dependencies, fix random seeds, document commands and environment, and write manifests for source scope, versions, configuration, status, and checks. A stage is complete only when its acceptance checks pass.
- Preserve immutable raw input. Record all cleaning rules, exclusions, conversions, and resulting counts. Tests should use synthetic fixtures with known expected outcomes; full-data checks should report observed values.
- Complete and validate the four-quarter pilot before the full download. A valid skip requires both a complete manifest entry and a healthy file with matching checksum, row count, schema, and key checks. Use temporary writes and atomic promotion; never promote a partial file.

## Temporal and modelling safeguards

- Distinguish `REPDTE` (report quarter-end), assumed/verified availability date, prediction date (quarter-end `q`), and `FAILDATE` (outcome date). Provisionally allow a report as a feature only after `REPDTE + 120 calendar days`; verify release evidence where possible, document uncertainty and development-only lag sensitivity, and never shift the official `(q, q + 4 quarters]` target.
- Label failure only from the Failures table using `FAILDATE` within `(q, q + 4 quarters]`. A merger or other reporting exit is not a failure. Investigate unmatched or ambiguous `CERT` records and keep their disposition explicit.
- Use modelling observation quarters no later than 2025Q2. Do not assign confirmed negative labels to observations lacking complete four-quarter outcome follow-up.
- Choose split dates after measuring actual failures: development precedes test, with at least four intervening quarters, and test has at least 50 failures. Use forward-time validation within development; no random cross-validation.
- Fit all preprocessing, select features/models, estimate costs, and select the alarm threshold with development data only. Freeze these decisions before one final test evaluation. Record test use so that subsequent tuning cannot be presented as untouched validation.
- Global inspection may audit raw records, field coverage, identifiers, quality, and outcome counts for split feasibility. Fit imputation, scaling, clipping, encoding, feature selection, class weights, model parameters, cost ratios, and threshold only within each development training fold; refit on full development only after selection. Use perturbation tests to prove future and held-out values cannot alter earlier features or fitted statistics.
- Use metrics suited to rare failures and explain false alarms, missed failures, and data-derived threshold assumptions. Do not claim model accuracy is the primary project goal.

## Evidence and final review

- For every requirement ID in `PROJECT_PLAN.md`, provide an implementation artifact, an executable test or explicit review gate, and a report table, figure, or measurement. Keep the traceability register current.
- Compare Polars and pandas fairly on identical data; use PySpark to reproduce key transformations, verify parity, capture execution plans, and make scaling claims from measured evidence.
- The report must answer Q1-Q3, explain limitations and transfer to Vietnam, be in English, and stay within 20 pages plus a 10-page appendix. Its first page must contain every item required by the PDF.
- README must rerun the pipeline end to end; the GitHub repository must be public for the required grading window and contain pinned requirements, code/notebooks with outputs and fixed seeds, no data files, and citations for code, papers, and AI tools in both README and report.
