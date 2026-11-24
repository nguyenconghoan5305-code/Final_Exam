# FDIC History Source Assessment for Prediction Membership

Date of bounded live probe: 2026-11-15 (final metadata retrieval
`2026-11-15T09:10:46.2754212Z`). The official History YAML response was
63,898 UTF-8 bytes, SHA-256
`71c01823ef906c768aceb9da7c9cdd0d0b78edd5d68e0d55e1585ea976922af3`.
This assessment does not download or
claim completeness of the FDIC History dataset. It does not alter accepted
raw, panel, repair, or P2.4 candidate files.

## Official source and dictionary

- [FDIC BankFind API documentation](https://api.fdic.gov/banks/docs/) lists
  `/history` for structure-change events and links its
  [official History field dictionary](https://api.fdic.gov/banks/docs/history_properties.yaml).
- The dictionary defines `EFFDATE` as **Effective Date**, `PROCDATE` as
  **Process Date** (the date an institution's change/event is processed),
  `CERT` as the institution's FDIC certificate number, `CHANGECODE` as an
  activity event code, and `ORG_ROLE_CDE` as an organization role including
  `FI` for financial institution and `BR` for branch. The observed API
  response also contains `ID` and `TRANSNUM`; neither was established as a
  complete, stable, unique historical event key by this bounded probe.
- The [Bank Structure Changes interface](https://banks.data.fdic.gov/bankfind-suite/oscr)
  supports search by processed or effective date. The [FDIC Bank Data Guide](https://www.fdic.gov/system/files/2024-06/guide.pdf)
  lists the History application as weekly with an application start of 1999;
  its footnote states that underlying bank structure changes reach back to
  1970. These statements do not prove complete event coverage, historical
  revisions, or public availability for every 1984–2025 quarter.

## Bounded observations

The following GET requests used `limit=1` and JSON responses. The filter
string was URL encoded. They report current index totals only, not acquired
full datasets. Retrieval date was 2026-11-15.

Reproduce the bounded totals in Windows PowerShell without fetching the full
source:

```powershell
$base = 'https://api.fdic.gov/banks/history?limit=1&format=json'
$all = (Invoke-WebRequest -Uri $base -TimeoutSec 25).Content | ConvertFrom-Json
$all.meta | Select-Object total,index
foreach ($filter in @(
  'ORG_ROLE_CDE:FI AND EFFDATE:[1984-01-01 TO 1984-12-31]',
  'ORG_ROLE_CDE:FI AND EFFDATE:[2025-01-01 TO 2025-12-31]'
)) {
  $url = 'https://api.fdic.gov/banks/history?limit=1&filters=' + [uri]::EscapeDataString($filter)
  $result = (Invoke-WebRequest -Uri $url -TimeoutSec 25).Content | ConvertFrom-Json
  [pscustomobject]@{Filter=$filter; Total=$result.meta.total; Index=$result.meta.index.name}
}
```

| Query | API total | Source index | Pages downloaded |
| --- | ---: | --- | ---: |
| `/banks/history?limit=1&format=json` | 585,705 | `history_20261113100007` | 1 sample page |
| `ORG_ROLE_CDE:FI AND EFFDATE:[1984-01-01 TO 1984-12-31]` | 4,126 | `history_20261113100007` | 1 sample page |
| `ORG_ROLE_CDE:FI AND EFFDATE:[2025-01-01 TO 2025-12-31]` | 704 | `history_20261113100007` | 1 sample page |

The unfiltered sample was a **branch closing** with `ORG_ROLE_CDE=BR`, so
endpoint totals cannot be interpreted as bank entry or exit counts. The two
filtered probes showed both date fields and institution-role records, but
retrieved one row each. No full-page completeness, key uniqueness, field
absence, conflicting-code, or exact 1984–2025 event coverage audit was run.
No `PROCDATE` evidence showed when the record first became publicly known.

## Decision

No History event enters the real prediction-membership decision. `EFFDATE`
alone is retrospective. `PROCDATE` is an internal processing date according
to the official definition, not a verified historical public-release date.
The accepted Institutions table is a later snapshot. The Failures endpoint
has dated outcomes but no verified historical publication timestamp. Thus
the accepted sources and bounded History probes do not support an exact
active-bank roster at every prediction `q`.

The implemented output is explicitly a **recent-reporter proxy** with
activity-uncertain stale candidates retained. Before claiming an active-bank
modelling population, a future task must acquire and reconcile complete
institution-level entry/exit records, event identity and codes, publication
vintages or a defensible knowability proxy, and conflicts across the whole
1984–2025 horizon. That research is outside this bounded Part 2 repair.
