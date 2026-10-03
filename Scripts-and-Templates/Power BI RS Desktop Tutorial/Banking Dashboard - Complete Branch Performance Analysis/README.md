# Banking Dashboard — Complete Branch Performance Analysis

Power BI Report Server (PBIRS) tutorial in Urdu — branch deposits, network growth, same-store growth, CAGR and concentration analysis.

📺 Video: [ADD VIDEO LINK]

## Files for this video

| File | What it is |
|---|---|
| [`Branch_Performance_DAX_Measures.xlsx`](../../../DAX-Formulas/Power%20BI%20RS%20Desktop%20Tutorial/Banking%20Dashboard%20-%20Complete%20Branch%20Performance%20Analysis/Branch_Performance_DAX_Measures.xlsx) | All 25 measures with English + Urdu explanation and model setup |
| [`Branch_Performance_Measures.dax`](../../../DAX-Formulas/Power%20BI%20RS%20Desktop%20Tutorial/Banking%20Dashboard%20-%20Complete%20Branch%20Performance%20Analysis/Branch_Performance_Measures.dax) | Same measures as plain text, copy-paste ready |
| `BranchPerformance_Theme_v2.json` | Report theme (navy + gold). View → Themes → Browse for themes |

## Key concept: semi-additive measures

Deposits in this dataset are **year-end balances**. Summing them across years inflates the total about 5.5×. `Total Deposits` always returns the **latest snapshot** in the current filter context, while still summing across branches.

## Model setup

| Object | Name | Definition |
|---|---|---|
| Fact table | `Bank Branches` | `Power Query (CSV, unpivoted)` — One row per branch per year (2010–2016). Deposits = year-end balance. |
| Calculated column | `Bank Branches[Deposit Date]` | `DATE ( 'Bank Branches'[Year], 12, 31 )` — Year-end snapshot date used to join Dim Date. |
| Dimension | `Dim Branch` | `Calculated table (DISTINCT of branch attributes)` — One row per Branch Number. |
| Calculated column | `Dim Branch[Branch Type]` | Used in the Branch Type slicer. |
| Dimension | `Dim Date` | Dynamic date range. Mark as Date Table on [Date]; sort Month by Month Number. |
| Measure table | `_Measure` | `Blank table with one hidden column` — Holds all measures in display folders. |
| Relationship | `Bank Branches[Branch Number] → Dim Branch[Branch Number]` | `Many-to-one, single direction`  |
| Relationship | `Bank Branches[Deposit Date] → Dim Date[Date]` | `Many-to-one, single direction`  |
| Setting | `Auto date/time` | `OFF (File > Options > Current File > Data Load)` — Removes hidden LocalDateTable tables. |

## Measures

| Folder | Measure | Description |
|---|---|---|
| 0. Helpers | Snapshot Date | Helper: latest year-end snapshot date visible in the current filter context. All semi-additive measures anchor on this. |
| 1. Deposits | Total Deposits | Year-end deposit balance. Semi-additive: sums across branches, takes the LATEST snapshot across time (never sums years). |
| 1. Deposits | Main Office Deposits | Deposits held at Main Office branches only. |
| 1. Deposits | Deposits excl. Main Office | Deposits of the branch network excluding Main Office. Use this for true branch performance. |
| 1. Deposits | Main Office Share % | Share of deposits sitting in the Main Office. |
| 1. Deposits | Deposit Share % | Share of deposits vs the total of all branches selected in the visual/slicers. |
| 2. Branch Network | Active Branches | Branches holding a positive deposit balance at the latest snapshot. Zero/negative balances are excluded. |
| 2. Branch Network | Zero-Deposit Branches | Branches with zero or negative deposit balance at the latest snapshot (dormant / data quality). |
| 2. Branch Network | New Branches | Branches whose Established Date falls inside the selected period. |
| 2. Branch Network | Acquired Branches | Branches whose Acquired Date falls inside the selected period. |
| 2. Branch Network | Avg Branch Age (Years) | Average age (years) of branches established on or before the snapshot date. |
| 3. Branch Productivity | Avg Deposit per Branch | Average deposit per active branch (includes Main Office - skewed upward). |
| 3. Branch Productivity | Avg Deposit per Branch excl. MO | Average deposit per active branch, excluding Main Office. |
| 3. Branch Productivity | Median Deposit per Branch | Median deposit of active branches at the latest snapshot - the 'typical' branch. |
| 3. Branch Productivity | Top 10% Branch Share | Share of deposits held by the top 10% of active branches (concentration risk). |
| 4. Time Comparison | Deposits PY | Total Deposits for the same period last year. |
| 4. Time Comparison | Deposits YoY Change | Change in deposits vs previous year. |
| 4. Time Comparison | Deposits YoY % | Percentage growth in deposits vs previous year. |
| 4. Time Comparison | Same-Store Deposits YoY % | Like-for-like growth: only branches active (deposits > 0) in BOTH current and prior year-end snapshots. |
| 4. Time Comparison | Deposits CAGR | Compound annual growth rate of deposits between the first and last snapshot in the selected period. |
| 5. Ranking | Branch Rank | Rank of a branch by Total Deposits among branches in the visual. Ranks within State automatically when State is on an outer level. |
| 5. Ranking | State Rank | Rank of a State by Total Deposits. |
| 6. Report UX | Selected Period Label | Dynamic label for report titles, e.g. 'Year 2016' or 'Years 2010 - 2016'. |
| 6. Report UX | YoY Indicator | YoY % with arrow for cards/tables, e.g. '▲ 8.8%'. |
| 6. Report UX | YoY Color | Hex color for conditional formatting (Format > Font color > fx > Field value). |

## Data source

Branch deposit data (2010–2016) based on public FDIC Summary of Deposits records. Used for education only.
