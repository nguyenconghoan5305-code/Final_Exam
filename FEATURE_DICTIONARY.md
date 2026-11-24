# Part 4.1 Bounded Point-in-Time Feature Contract

**Status:** implementation contract validated on 2026-11-16. This is a
dictionary and executable synthetic feature rule, not a production feature
table, label set, fitted preprocessing pipeline, or model.

## Authority, inputs, and time

The assignment PDF, page 2, calls for economically informative bank-risk
features, a leak-free feature table and training-only preprocessing. This
milestone implements the dictionary and input rule only. The observed report
history is the immutable P2.2 `panel20261115b` Financials v2 panel (170
quarters, 1,682,616 bank-quarters). The accepted P2.4 `pit20261115f` rule
selects the latest report for a separately established bank and prediction
quarter-end `q`. The executable contract validates both upstream runs and
their hashes. It does not infer active-bank membership: the available-report
candidate roster is not a verified active-bank population.

`r` below means the selected report's `REPDTE`, which can precede `q` by
several quarters. An undated source version is assumed available at
`REPDTE + 120 calendar days`. A verified later revision date delays that
version's availability. A report is usable only if its availability date is
at or before `q`; the boundary is inclusive. A future version is filtered
before revision ambiguity is assessed. Multiple eligible versions with
unorderable dates fail. For a feature requiring several reports, its earliest
input availability date is the maximum of those versions' dates; its first
possible prediction date is the first calendar quarter-end on or after that
maximum. Both are recorded as diagnostics. The 120-day lag is a conservative
assumption, **not** a reconstruction of historical publication vintages.
The accepted snapshot lacks timestamps for most original versions, so later
revisions without verified timing remain a limitation.

The source fields are the verified 26-field Financials v2 allowlist. The
executable feature subset is `ASSET`, `DEP`, `EQ`, `ROA`, and `RBC1AAJ`.
The [official FDIC Financials dictionary](https://api.fdic.gov/banks/docs/risview_properties.yaml)
was retrieved on 2026-11-16; its SHA-256 was
`b257457e4e821d6a0d9f88ffc4e6e0da80e4ccf67fa394cadb21587b50b23d59`.
It defines ASSET as total assets excluding off-balance-sheet items; DEP as
total deposits; EQ as total equity capital; ROA as annualized net income
after taxes and extraordinary items divided by average assets, expressed
as a percentage; and RBC1AAJ as the PCA leverage percentage (`RBCT1 /
AVASSETJ`), distinct from the risk-based ratio RBCRWAJ. The accepted panel
stores the three amounts as Arrow `decimal128(20,4)` and the two reported
ratios as `float64`. The assignment uses thousands of USD for financial
amounts and percent for ratios. The dictionary does not attach a separate
unit annotation to every amount field, so the monetary unit remains the
assignment convention. The FDIC ratio percentage formulas confirm that
reported ROA and RBC1AAJ must **not** be multiplied by 100 again.

## Feature dictionary

All features are anchored at `r`, the latest report selected at `q`. A
lookback of 0 uses only `r`; lookback 1 also needs `r-1`; lookback 3 needs
`r` through `r-3`, all in **calendar quarters**, not prior observed rows.
Every required version must satisfy its individual availability date `<= q`.
For ratios derived from amounts, output is Arrow `decimal128(30,6)` percent,
rounded half-even only for the new derived value; source amounts are never
rounded or imputed. Negative EQ is retained because it can signal distress.
No fitted transformation or data-derived cutoff is used.

| Feature / output type | Source / unit and exact formula | Economic rationale | Lookback; minimum history | Null, gap and denominator policy | Earliest usable date |
| --- | --- | --- | --- | --- | --- |
| `asset_size` / decimal128(20,4) | `ASSET[r]`, thousands USD | Scale and potential exposure | 0; one report | Source absence and explicit null remain distinct in diagnostics; nonpositive ASSET yields null | availability(r), then next prediction quarter-end |
| `equity_to_assets_pct` / decimal128(30,6) | `100 × EQ[r] / ASSET[r]`, percent | Equity buffer | 0; one report | Null or absent input yields null; ASSET <= 0 yields null; negative EQ retained | availability(r), then next prediction quarter-end |
| `deposits_to_assets_pct` / decimal128(30,6) | `100 × DEP[r] / ASSET[r]`, percent | Deposit funding reliance | 0; one report | Null or absent input yields null; ASSET <= 0 yields null; a negative DEP value remains visible for quality review | availability(r), then next prediction quarter-end |
| `roa_reported_pct` / float64 | `ROA[r]`, already percent | Annualized profitability | 0; one report | Null/absent remains null; nonfinite values rejected; no rescaling | availability(r), then next prediction quarter-end |
| `pca_leverage_reported_pct` / float64 | `RBC1AAJ[r]`, already percent | PCA leverage capital | 0; one report | Null/absent remains null; nonfinite values rejected; no rescaling | availability(r), then next prediction quarter-end |
| `asset_growth_qoq_pct` / decimal128(30,6) | `100 × (ASSET[r] / ASSET[r-1] − 1)`, percent | Quarter-to-quarter balance-sheet change | 1; two consecutive reports | Missing calendar quarter, unavailable version, null/absent input or nonpositive prior ASSET yields null; no forward fill | max availability(r, r-1), then next prediction quarter-end |
| `roa_four_quarter_mean_pct` / float64 | arithmetic mean of reported `ROA[r]`, `ROA[r-1]`, `ROA[r-2]`, `ROA[r-3]`, percent | Persistence of reported profitability | 3; four consecutive reports | Any calendar gap, unavailable version, null/absent or nonfinite ROA yields null | max availability(r...r-3), then next prediction quarter-end |

The four-quarter ROA mean is an average of already annualized reported
percentages; it is **not** trailing net income or a newly annualized ratio.
The synthetic 2022Q4 to 2023Q2 resumed gap proves that missing 2023Q1
invalidates both quarter-over-quarter asset growth and the four-report mean,
even though a prior observed row exists. No bank-quarter is manufactured.

If the independently supplied population has no eligible report, all seven
feature values are null with `no_eligible_history`. `CERT` and `q` identify
the result but are not feature values. Source ID, source-absence provenance,
quality flags and availability diagnostics are kept outside the model-value
map. Failure outcomes, assistance and exit assessments, Institutions snapshot
status, and future-derived joins cannot enter the feature inputs.

## Deferred or omitted source fields

- `NETINC`: the dictionary defines net income but does not say whether the
  acquired BankFind API value is a year-to-date or quarter-only flow. The
  [FDIC BankFind methodology](https://banks.data.fdic.gov/bankfind-suite/help?helpTopic=disclaimers-and-methodologies)
  explains that Call Report income/expense values are generally year-to-date
  and quarterly values may be derived, with merger adjustments. That does
  not establish the exact period basis of this acquired `NETINC` field for
  every historical quarter. No NETINC difference, rolling sum, or growth
  feature is defined yet.
- `P3ASSET`: the official dictionary gives a title but no detailed formula
  or unit/percentage scale. The accepted acquisition measured 68 all-null
  quarters and 971,925 explicit nulls. It remains raw and visible in quality
  evidence, never converted to zero; it is omitted pending scale and
  historical-comparability verification.
- Liquidity (`CHBAL`, `SC`), other asset-quality (`P9ASSET`, `NAASSET`),
  funding (`BRO`), and risk-based capital (`RBCRWAJ`) candidates remain in
  the verified raw schema. Their detailed comparability, source-specific
  definitions, or consistent historical availability need a separate review
  before feature selection. No held-out or model-performance evidence was
  consulted for this dictionary.

## Acceptance and remaining limits

`src/bft65b/features/contract.py` contains the machine-readable specs,
schema gate and `compute_features()` fixture implementation. The accepted
run `runs/task_4_1_feature_contract/featurecontract20261116b/` contains an
ignored `contract.json` and manifest with upstream file hashes. It verifies
all 170 panel Financials schemas/checksums and the accepted PIT output and
upstream checksums without generating a production feature table. Synthetic
tests exercise exact availability boundaries, future revisions, source
absence versus explicit null, nonpositive denominators, and a cross-year
calendar gap. The Part 2 active-bank population decision remains blocked;
this dictionary does not resolve it. Final report placement is pending.
