# Source and field inventory

## Authority and acquisition scope

The [assignment PDF](final_report_bft65b.pdf) requires Financials for all banks from 1984Q1 through 2026Q2 and complete Failures and Institutions tables. Modelling observations stop at 2025Q2. Failures are labelled from FAILDATE in the four quarters after an observation. Task 1.1 verified the configured field **codes and endpoint membership** against the official FDIC YAML dictionaries on 2026-11-07. Task 1.3 subsequently acquired the four 2023 Financials quarters and complete Failures/Institutions endpoint responses. The full 1984Q1-2026Q2 Financials history has not been downloaded.

Official sources:

- [Financial API definitions](https://api.fdic.gov/banks/docs/risview_properties.yaml)
- [Failure API definitions](https://api.fdic.gov/banks/docs/failure_properties.yaml)
- [Institution API definitions](https://api.fdic.gov/banks/docs/institution_properties.yaml)
- [BankFind API documentation](https://api.fdic.gov/banks/docs/) and [OpenAPI specification](https://api.fdic.gov/banks/docs/swagger.yaml)

ProjectConfig.fields_verified=True authorizes only the exact dictionary-verified 11-field v1 or 26-field v2 Financials configuration, together with the unchanged Failures and Institutions sets. The client allowlist includes those verified codes; arbitrary subsets or additions cannot be certified by this Boolean alone.

## Selected Financials fields

The assignment states that financial amounts are in **thousands of USD** and ratios in **percent**. The Financial YAML does not provide a separate x-number-unit annotation for every selected amount. The unit column therefore identifies the assignment as the source where applicable; any field-specific exception remains subject to pilot review. Types below are YAML source types. The accepted Task 1.3 pilot froze the 11-field Financials v1 landing schema; Task 1.5A separately validates the extended v2 schema in STORAGE_SCHEMA.md.

| Code | Official title / definition | YAML type | Unit evidence and unresolved detail |
| --- | --- | --- | --- |
| CERT | FDIC Certificate #; unique institution identifier assigned by FDIC | number | Identifier, no monetary unit |
| REPDTE | Report Date; last day of the selected financial reporting period | string, keyword | Live source value YYYYMMDD; date parsing to be checked across periods |
| STNAME | STATE NAME | string, keyword | Geographic name; Financial YAML description is blank |
| ASSET | Total assets; sum of owned assets, excluding off-balance-sheet accounts | number | Thousands of USD per assignment |
| DEP | Total deposits; includes demand, savings, time, and foreign-office deposits | number | Thousands of USD per assignment |
| EQ | Equity capital; includes stock, surplus, and undivided profits | number | Thousands of USD per assignment |
| NETINC | Net income; income after listed noninterest items, provisions, and taxes | number | Thousands of USD per assignment; reporting-period interpretation needs review |
| ROA | Return on assets; annualized net income after taxes and extraordinary items as a percentage of average total assets | number, elastic double | Percent per official description |
| NCLNLS | TOTAL N/C-LOANS & LEASES | number | Detailed expansion of “N/C”, formula, and field-specific unit are not supplied by this dictionary entry; investigate before deriving ratios |
| LNATRES | ALLOW FOR LOANS LOSS ADJUSTED | number | Adjustment formula and field-specific unit are not supplied by this dictionary entry; investigate before deriving ratios |
| RBC1AAJ | LEVERAGE RATIO-PCA | number, elastic double | Percentage per YAML formula metadata; do not label it a risk-based capital ratio. Exact formula and historical comparability need review |

## Selected Failures fields

| Code | Official title / definition | YAML type | Unit evidence and unresolved detail |
| --- | --- | --- | --- |
| CERT | Cert; FDIC certificate identifier linking to institution data | string, keyword | Live JSON returned a number or null. It is a nullable bank linkage field, not a Failures source-record key; 488 nulls were observed |
| FAILDATE | Effective Date; date the failed/assisted institution ceased to exist as a privately held going concern, or entered government ownership | string, date-time | Live JSON sample used M/D/YYYY, despite YAML date-time; historical formats need audit |
| COST | Estimated Loss; estimated Deposit Insurance Fund disbursements less expected receivership recoveries | number, elastic double | Official x-number-unit: thousands of US dollars. The dictionary says comprehensive estimates are unavailable for some older failures and may appear as N/A; this is not an observed preventable loss |

## Selected Institutions fields

| Code | Official title / definition | YAML type | Unit evidence and unresolved detail |
| --- | --- | --- | --- |
| CERT | FDIC Certificate #; institution identifier | string | Live JSON returned a number; full-data identifier normalization is pending |
| STNAME | State Name; state or territory where the institution is physically located | string | Geographic name, no monetary unit |
| ACTIVE | Institution Status; 1 = currently open and FDIC-insured, 0 = closed or not FDIC-insured | number | Neither value alone proves failure; live JSON returned 0 in the sample |
| CHANGEC1 | Change Code; FDIC code for an institution structural event | string | Live JSON returned numeric code 223; the code mapping and historical availability remain unresolved |

## Task 1.1 live API observations

Minimal probes on 2026-11-07 used **two pages of two records per endpoint**, not full downloads. Each response had top-level meta, data, and totals. meta.total was an integer; meta.parameters echoed fields, filters, limit, and offset; meta.index supplied a name and create timestamp. Each item in data contained a nested data object and score; the requested fields and an ID appeared inside that nested object. The source totals below describe the API's reported result set, **not downloaded row counts**.

| Endpoint / query | API-reported total | Sampled rows | Observed index |
| --- | ---: | ---: | --- |
| Financials, REPDTE:20230331 | 4,740 | 4 | risview_20260819185831 |
| Failures, no filter | 4,117 | 4 | failures_1787667198788 |
| Institutions, no filter | 27,835 | 4 | institutions_20261106100008 |

The generic API documentation illustrates ISO date filters, but REPDTE:2023-03-31 returned **zero** Financials results while REPDTE:20230331 returned 4,740. The client therefore requires REPDTE:YYYYMMDD for Financials and rejects the ISO form. CERT appeared in ascending numeric order in live ten-record diagnostic probes, although the Failures and Institutions YAML types are strings. The client checks the observed numeric order and keeps raw response values. The official OpenAPI specification describes sort_by as a single field name. The client uses CERT for each endpoint. Later complete-table evidence found repeated and null Failures CERT values; the verified null-last ordering and source-ID key are described below. A repeated CERT across a page boundary remains ambiguous.

The ignored probe manifest is runs/task_1_1_api_probe.json. Its source_total entries are API metadata; its rows entries are sample counts only. The later Task 1.3 pilot findings below supersede the probe's open questions for the acquired endpoint snapshots; full historical Financials field coverage remains unverified.

## Task 1.1 technical refinement: pagination and field presence

The [FDIC Failures help page](https://banks.data.fdic.gov/bankfind-suite/failures/help) describes CERT as an institution identifier and the detail output as failure or assistance transactions. Neither that page nor the endpoint dictionaries reviewed here guarantee one record per CERT in every endpoint. The OpenAPI specification describes a single sort_by field and a maximum limit of 10,000; accepting a comma-separated sort string with HTTP 200 is not proof that a secondary tie-breaker was applied. The returned ID is not defined as a field in any of the three endpoint YAML dictionaries, so its uniqueness must be **validated on each complete acquisition** rather than assumed from the dictionary.

The safe strategy for complete Failures acquisition is a single API page with limit 10,000 **only if the reported total is at most 10,000 at download time**. The complete pilot reported 4,117; that is a snapshot, not a future guarantee. One page can retain distinct transactions sharing CERT without a page-boundary ambiguity. The client rejects duplicate source `ID` keys, including when CERT/date values differ; repeated CERT/date values with distinct source IDs are preserved for later quality review. If the total exceeds 10,000, or a repeated CERT crosses pages under a smaller page size, safe completion has **not** been established: the client raises AmbiguousPaginationError. Stop the dependent acquisition and use an FDIC [official bulk download](https://banks.data.fdic.gov/bankfind-suite/bulkData) or design and verify a partitioned API query that reconciles every partition and the overall source total. Do not merely disable the check. Financials quarter pages and Institutions pages still require unique keys and monotonic CERT; source violations stop their runs for review. A writer must not promote any page output until the full iterator and all count checks finish.

The API may return HTTP 200 while silently omitting an unknown requested field: a one-record request containing NOT_A_FDIC_FIELD did so on 2026-11-07. Thus the client checks requested codes against its verified allowlist before network access. For each returned row, it distinguishes a requested field **absent from the data object** from one **present with JSON null** in page.missing_fields and page.null_fields. Either state is allowed for optional financial measures and must be audited by field and period later; an absent or null Financials `CERT`/`REPDTE`, Failures source `ID`/`FAILDATE`, or Institutions `CERT` is rejected. Failures `CERT` may be null and is counted as an unresolved bank linkage. A mismatched Financials REPDTE is rejected. In two-row 1984Q1, 1990Q1, and 2023Q1 samples of the 11 configured fields, no requested field was absent or null. That small observation is not a historical coverage estimate. In a separate two-row 1984Q1 candidate-field probe, P3ASSET was present with null in both records, while it was present and non-null in both sampled 2023Q1 records.

## Task 1.3 complete-endpoint observations and key decision

On 2026-11-10/11 (Asia/Saigon), the [official Failures YAML](https://api.fdic.gov/banks/docs/failure_properties.yaml) again defined `CERT` as the FDIC institution certificate, `FAILDATE` as the effective event date, and `COST` as an estimated loss in thousands of USD with elastic `double` type. It does not list `ID` as a selectable property. Actual API items had outer `data` and `score` keys; the nested `data` object contained an `ID` string even when `fields=CERT,FAILDATE,COST` was requested. `ID` is treated as empirically observed **source-record identity**, never as a bank identifier or official certificate mapping.

Two complete single-page Failures requests each returned 4,117 rows against `meta.total=4,117`, with the same index fingerprint, 4,117 present nonempty string IDs, 4,117 unique IDs, the same ID set, and zero payload mismatches by ID. The pilot request passed the same total/ID-key gate. Every future complete request must repeat these checks; an absent, empty, or duplicated source ID blocks promotion. The API's numeric `CERT` values were nondecreasing, followed by all null `CERT` values at positions 3,629–4,116 (zero-based). There were 44 extra occurrences of repeated non-null `CERT`; source ID, not `(CERT, FAILDATE)`, therefore identifies a Failures row. The one-page strategy remains required while the API total is at most 10,000. A larger response needs a separately verified pagination or official bulk strategy.

Exactly 488 of the 4,117 Failures rows had `CERT` present with JSON null; none had `CERT` absent. Their `FAILDATE` years span 1934–1976: 1930s 312, 1940s 99, 1950s 28, 1960s 44, 1970s 5. No certificate is synthesized. All 488 source IDs are recorded in the ignored linkage audit named by `runs/task_1_3_pilot_resume3_summary.json`; the summary and manifest record its count and SHA-256. `COST` was present in every row, null in 637 and non-null in 3,480; 756 non-null values had more than four decimal places (maximum observed: 10). The provisional Decimal(20,4) type was rejected by the live gate, so Failures schema v2 uses nullable float64, matching the official double source type and avoiding an extra four-place rounding. No `N/A` string occurred in this snapshot; such a future value still fails conversion for review. The ignored aggregate-only audit is `runs/task_1_3_failures_identity_audit.json`.

Institutions returned 27,835 rows over three pages, reconciled to the API total with unique `CERT` keys and no overlapping pages. `CHANGEC1` was absent in 3,482 records and present in the others; it was not imputed. The four 2023 Financials partitions matched API totals and retained v1 schema/checksums unchanged. All six pilot files passed typed Parquet read-back and SHA-256. Full Financials historical coverage and eventual bank-linkage handling for the 488 null-CERT failures remain later-stage work.

CERT was observed as a JSON number in the initial small endpoint samples, even though the Failures and Institutions YAML definitions say string. The complete Failures pilot additionally observed null CERT. The client preserves raw values, normalizes non-null integer and digit-string CERT values for numeric-order checks, and requires unique source IDs for Failures. Non-integer non-null CERT identifiers stop iteration. It recognizes observed M/D/YYYY FAILDATE and common ISO/calendar forms for date validation while preserving the original source value. Any unrecognized date representation stops iteration pending an explicit parsing rule. Full historical Financials type frequencies and field-level missingness remain for later acquisition and quality audits.

Financials filters must be REPDTE:YYYYMMDD, a valid calendar quarter-end within the assignment's 1984Q1-2026Q2 range. This reflects the observed keyword representation of REPDTE; the general API documentation's ISO-date example did not match this field in the live probe. Returned REPDTE must equal the requested quarter. The official four-quarter failure target definition is unchanged.

## Part 4 candidate review at Task 1.1 (before the acquisition-list change)

The original 11-field Financials set supports size, deposits, equity, income, ROA, two loan-quality/reserve measures, and a leverage measure. It did not cover cash/securities liquidity, brokered funding, loan composition, past-due/nonaccrual exposure, net charge-offs, interest margin, or a risk-based capital ratio. The following **22 additional starter-script codes were found** in the official Financial YAML on 2026-11-07. At that checkpoint none had entered the acquisition list. Task 1.5A later selected 15 for a separate v2 acquisition; seven remain unselected. Economic interpretations below are proposed feature rationales, not measured predictive results.

| Code | Official YAML title | Proposed risk information | Unit / limitation |
| --- | --- | --- | --- |
| DEPDOM | Deposits held in domestic offices | Domestic deposit concentration | Amount, provisionally thousands of USD per assignment |
| BRO | BROKERED DEP | Reliance on potentially less stable funding | Amount; detailed definition and history need review |
| CHBAL | CASH & DUE FROM DEPOSITORY INST | Immediately available liquidity | Amount; verify coverage and comparability |
| SC | SECURITIES | Liquid or saleable asset buffer | Amount; liquidity depends on security type and market conditions |
| LNLSGR | LOANS AND LEASES-TOTAL | Overall lending intensity | Amount; candidate denominator for shares |
| LNLSNET | LOANS AND LEASES-NET | Net loan-book size | Amount; reconcile with gross loans and reserves |
| LNRE | RE LOANS | Real-estate concentration | Amount; category scope must be checked |
| LNCI | C&I LOANS | Commercial credit concentration | Amount; category scope must be checked |
| LNCON | CONSUMER LOANS | Consumer credit concentration | Amount; category scope must be checked |
| P3ASSET | 30-89 DAYS P/D-TOTAL ASSETS | Early-stage delinquency signal | Numeric; unit/denominator unclear from title; 1984 sample had nulls |
| P9ASSET | 90+ DAYS P/D-TOTAL ASSETS | Severe delinquency signal | Numeric; unit/denominator unclear from title |
| NAASSET | NONACCRUAL-TOTAL ASSETS | Nonaccrual exposure | Numeric; unit/denominator unclear from title |
| NTLNLS | TOTAL LN&LS NET CHARGE-OFFS | Realized credit losses | Amount; period basis and sign need review |
| ORE | OTHER REAL ESTATE OWNED | Distressed property exposure | Amount; valuation and history need review |
| INTINC | TOTAL INTEREST INCOME | Interest earning capacity | Amount; reporting-period basis needs review |
| EINTEXP | TOTAL INTEREST EXPENSE | Funding-cost pressure | Amount; reporting-period basis needs review |
| NONII | TOTAL NONINTEREST INCOME | Revenue diversification | Amount; reporting-period basis needs review |
| NONIX | TOTAL NONINTEREST EXPENSE | Cost efficiency | Amount; reporting-period basis needs review |
| ELNATR | PROVISIONS FOR CREDIT LOSSES | Credit-loss provisioning | Amount; policy and accounting changes need review |
| ROE | Return on Equity (ROE) | Equity profitability | Percent per official description; dictionary notes NA for negative retained earnings |
| NIMY | NET INTEREST MARGIN | Net interest spread | Percent per official formula metadata; denominator details need review |
| RBCRWAJ | TOTAL RBC RATIO-PCA | Risk-based capital adequacy | Percent per official formula metadata; not equivalent to configured RBC1AAJ leverage ratio |

The assignment gives financial amounts in thousands of USD and ratios in percent, but many individual YAML entries above have no explicit unit annotation or explanatory description. Their physical type, exact scale, numerator/denominator, null prevalence, revisions, and availability through 1984Q1-2026Q2 must be checked before selection. In a **two-record sample only** from 1984Q1 and 2023Q1, all 22 codes were returned; P3ASSET was null in both older rows. These observations do not establish historical coverage. Any selected feature must obey the project's point-in-time availability policy and be fitted/selected using development folds only.

## Task 1.5A verified Financials v2 selection and bounded coverage

On 2026-11-11 the 15 additions below were checked again against the [official Financials YAML](https://api.fdic.gov/banks/docs/risview_properties.yaml). Every code belongs to the Financials endpoint and has source `type: number`. The official YAML leaves the detailed `description` blank for 14 of the 15; the table records its **title**, not an invented formula. `DEPDOM` has an explicit description: the sum of domestic-office demand, money-market, other savings, and time deposits. `NIMY` has a percentage formula using `NIMA / ERNAST5`; `RBCRWAJ` has a percentage formula using `RBC / RWAJT`. The configured `RBC1AAJ` is a distinct PCA **leverage** percentage formula using `RBCT1 / AVASSETJ`. No field-specific YAML monetary unit was found for the new amount-like fields; the assignment's general thousands-of-USD convention is provisional until the full data audit. The scale/unit of `P3ASSET`, `P9ASSET`, and `NAASSET` is explicitly unresolved.

| Risk group | Selected code | Official YAML title | Definition / unit evidence | Economic motivation and caution |
| --- | --- | --- | --- | --- |
| Liquidity | CHBAL | CASH & DUE FROM DEPOSITORY INST | No detailed YAML description or unit; provisional amount | Cash buffer; balance alone does not measure stressed access. |
| Liquidity | SC | SECURITIES | No detailed YAML description or unit; provisional amount | Saleable-asset inventory; marketability varies by security. |
| Funding | BRO | BROKERED DEP | No detailed YAML description or unit; provisional amount | Funding mix; do not equate all brokered deposits with outflow. |
| Funding | DEPDOM | Deposits held in domestic offices | YAML defines the domestic-office deposit sum; amount under assignment convention | Domestic share of deposits; overlaps `DEP`. |
| Funding / earnings | EINTEXP | TOTAL INTEREST EXPENSE | No detailed YAML description or unit; provisional amount and period basis unresolved | Funding-cost pressure; period interpretation must be checked. |
| Lending | LNLSGR | LOANS AND LEASES-TOTAL | No detailed YAML description or unit; provisional amount | Loan intensity and denominator for possible shares. |
| Lending | LNRE | RE LOANS | No detailed YAML description or unit; provisional amount | Real-estate exposure; subset of gross loans. |
| Lending | LNCI | C&I LOANS | No detailed YAML description or unit; provisional amount | Commercial-loan exposure; subset of gross loans. |
| Asset quality | P3ASSET | 30-89 DAYS P/D-TOTAL ASSETS | No detailed YAML description, formula, or unit; numeric scale unresolved | Early delinquency candidate; all values null in two early complete quarters. |
| Asset quality | P9ASSET | 90+ DAYS P/D-TOTAL ASSETS | No detailed YAML description, formula, or unit; numeric scale unresolved | Later delinquency candidate; may overlap other credit-quality measures. |
| Asset quality | NAASSET | NONACCRUAL-TOTAL ASSETS | No detailed YAML description, formula, or unit; numeric scale unresolved | Nonaccrual candidate; may overlap `NCLNLS`. |
| Asset quality | NTLNLS | TOTAL LN&LS NET CHARGE-OFFS | No detailed YAML description or unit; provisional amount and period basis unresolved | Realized credit-loss signal. |
| Asset quality | ELNATR | PROVISIONS FOR CREDIT LOSSES | No detailed YAML description or unit; provisional amount and accounting comparability unresolved | Provisioning signal; overlaps existing reserve measures conceptually. |
| Profitability | NIMY | NET INTEREST MARGIN | YAML percentage formula `NIMA / ERNAST5`; denominator meaning requires review | Interest-margin signal; complements `NETINC` and `ROA`. |
| Capital | RBCRWAJ | TOTAL RBC RATIO-PCA | YAML percentage formula `RBC / RWAJT`; risk-weighted capital ratio | Complements, but is not interchangeable with, `RBC1AAJ` leverage. |

All 15 were selected for **raw acquisition**, yielding 26 Financials fields with the existing 11. Selection is based on coverage of distinct economic domains, not on observed target associations or model scores. Correlated or overlapping measures remain raw; feature selection, imputation, and scaling must be fitted within development training folds. The seven starter-script candidates not selected for v2 are `LNLSNET`, `LNCON`, `ORE`, `INTINC`, `NONII`, `NONIX`, and `ROE`. They remain possible later additions if a documented need warrants another version and acquisition.

The new [bounded historical profile](runs/task_1_5a_historical_profile_summary.json) contains **complete** Financials responses for 1984Q1, 1989Q4, 2009Q4, and 2026Q2. The separate [extended 2023 pilot](runs/task_1_5a_extended_pilot_2023_summary.json) contains all four 2023 quarters. Both summaries record every field's absent count, explicit null count/rate, source type counts, source index, retrieval timestamps, page offsets/counts, source totals, Parquet bytes, and SHA-256. The table highlights coverage differences; detailed counts for all 26 fields are in the ignored summaries.

| Quarter | API / downloaded rows | Pages (row counts) | Absent requested fields | `P3ASSET` null | `P9ASSET` / `NAASSET` null | `BRO` null | Conversion anomalies |
| --- | ---: | --- | ---: | ---: | ---: | ---: | ---: |
| 1984Q1 | 17,930 / 17,930 | 2 (10,000 + 7,930) | 0 | 17,930 (100%) | 3,452 each (19.25%) | 3,177 (17.72%) | 0 |
| 1989Q4 | 16,152 / 16,152 | 2 (10,000 + 6,152) | 0 | 16,152 (100%) | 2,897 each (17.94%) | 36 (0.22%) | 0 |
| 2009Q4 | 8,103 / 8,103 | 1 | 0 | 0 | 0 | 17 (0.21%) | 0 |
| 2023Q1 | 4,740 / 4,740 | 1 | 0 | 0 | 0 | 17 (0.36%) | 0 |
| 2023Q2 | 4,714 / 4,714 | 1 | 0 | 0 | 0 | 17 (0.36%) | 0 |
| 2023Q3 | 4,686 / 4,686 | 1 | 0 | 0 | 0 | 17 (0.36%) | 0 |
| 2023Q4 | 4,658 / 4,658 | 1 | 0 | 0 | 0 | 17 (0.36%) | 0 |
| 2026Q2 | 4,313 / 4,313 | 1 | 0 | 0 | 0 | 17 (0.39%) | 0 |

All eight queries reported the index fingerprint `risview_20260819185831` / `2026-08-19T18:58:33Z` and passed per-page stability and key checks. This is **eight of 170 quarters**, not a full historical availability audit. `P3ASSET` was retained despite complete nullness in the two sampled early quarters because its later observed coverage and economic meaning warrant a period-aware audit. Source `number` values for the three unresolved-scale fields were integers or null in these eight complete responses; Financials v2 stores them as float64 with an exact-integer guard, and their units remain unasserted. Values requiring a lossy string-to-float conversion fail storage validation. A future source representation outside this verified sample must stop for review. No feature or model was fitted.

## Task 1.5 observed full-history field coverage

The accepted full-run [aggregate acceptance audit](runs/task_1_5_full_real20261111b_acceptance.json) covers all 170 Financials quarters from 1984Q1 through 2026Q2, plus complete Failures and Institutions responses. It independently reconciled 1,682,616 Financials, 4,117 Failures, and 27,835 Institutions rows to their API totals and typed Parquet files. All 26 requested Financials fields were present in every source row; this does **not** mean their values were non-null. `P3ASSET` was explicitly null in 971,925 Financials rows (57.7627%); all rows were null in 68 consecutive quarters from 1984Q1 through 2000Q4, and the field was absent in zero quarters. Its historical nullness remains a feature-design issue for Part 4, not a reason to rewrite raw values. Other source absence was observed only for Institutions `CHANGEC1` (3,482 records). Failures retained 488 explicit null `CERT` values and 637 null `COST` values; the ignored linkage audit reconciles all 488 source IDs. Zero conversion anomalies were recorded. The acceptance JSON gives separate absent and explicit-null counts for every requested field; units, period comparability, and point-in-time availability remain analytical questions.

## Rubric inventory

PROJECT_PLAN.md identifies 23 traceability requirements: P1.1-P1.3, P2.1-P2.4, P3.1-P3.4, P4.1-P4.2, P5.1-P5.4, P6.1, R.1, and S.1-S.4. EVIDENCE_REGISTER.csv maps every ID to a future artifact, gate, and report output.
