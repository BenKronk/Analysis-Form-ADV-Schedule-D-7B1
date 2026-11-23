# VC Project — Form ADV Schedule D 7.B.1 Private Fund Dashboard

*Status as of November 23, 2026*

**Repository:** https://github.com/BenKronk/Analysis-Form-ADV-Schedule-D-7B1 (public)

## Purpose

A Power BI dashboard summarizing private funds reported by **exempt reporting advisers (ERAs)** on **SEC Form ADV, Schedule D, Section 7.B.1**, covering filings from **January–October 2026**.

The dashboard summarizes the filings so far this year. It has no month filter, by design.

## Files

| File | Role |
|---|---|
| `Dashboard Schedule D7B1 - Jan-Oct 2026.pbix` | The Power BI report |
| `*_Schedule_D_7B1_2026MMDD_2026MMDD.csv` (10 files) | Monthly source files, January–October 2026 |
| `ERA_Schedule_D_7B1_Jan-Oct 2026.csv` | Raw extract: the monthly source files combined, one row per fund per filing (76,615 rows, 39 columns) |
| `ERA_Schedule_D_7B1_Jan-Oct_2026_deduped.csv` | Deduplicated extract (one row per fund) that the report loads |
| `Fund_Family_Names.csv` | Fund ID → Family Name / Family ID lookup that the report loads |

## Data preparation

### Deduplication

The raw extract has **76,615 rows** covering **68,181 distinct Fund IDs**. A fund appears more than once when it was reported in several filings during the period. The deduplicated file keeps **one row per Fund ID**.

The deduplicated data was checked against the raw file:

- **No funds lost.** All 68,181 Fund IDs from the raw file are in the model, with none missing or extra.
- **Rule for the kept row.** For every fund, the row from its **latest filing month** was kept. When a fund had more than one filing in that month (2,636 funds), the row with the **highest FilingID** was kept, in every case. FilingIDs rise from month to month; there's no filing date in the data to confirm that a higher ID is the later filing.
- **What was removed:**
  - 8,434 earlier rows for funds reported again later
  - older Gross Asset Values for the 1,490 funds whose value changed between filings
  - 1,198 FilingIDs, all of whose funds appear in later filings
- **Why removing them is correct.** Including the removed rows would count funds twice in fund counts and total Gross Asset Value.

### Helper columns added during preparation

These columns are in the deduplicated file:

- `Filings In Period`
- `Earlier Filing Months`
- `Had Earlier Filing` (7,133 funds = Y)
- `Name Shared Across Fund IDs` (207 funds = Y)

### Precision of large Gross Asset Values (accepted)

Some of the largest Gross Asset Values lost precision along the way:

- In the raw CSV, 18 values are in scientific notation (e.g., `1.94916E+11`), which rounds them to 6 significant digits.
- In the deduplicated file, 17 of them were rounded again, to 3 significant digits (e.g., 194,916,000,000 → 195,000,000,000).

The impact is small:

- All 17 affected funds are in the $1B+ category, so no fund lands in the wrong size category.
- Net effect on total Gross Asset Value: **−$295 million** out of $14.35 trillion (about 0.002%). Each affected fund's own figure is off by less than 0.5%.

**The discrepancy was reviewed and accepted.**

## Data model

**Tables**
- `Schedule_D_7B1`: 68,181 rows (one per fund), 43 columns from the deduplicated file (the raw file's 39 plus 4 helper columns) and 3 calculated columns
- `Fund_Family_Names`: Fund ID, Fund Name, Family Name, Family ID

**Relationship**
- `Fund_Family_Names[Fund ID]` ↔ `Schedule_D_7B1[Fund ID]`: one-to-one, active, filtering in both directions

**Calculated columns** (in `Schedule_D_7B1`)
- `GAV Category`: nine size bands: <$250K, $250K–$500K, $500K–$1M, $1M–$5M, $5M–$25M, $25M–$100M, $100M–$500M, $500M–$1B, $1B+
- `GAV Helper Column`: numbers 1–9 matching the `GAV Category` bands in size order
- `Gross Asset Value (bins)`: eight equal-width bins

## Report

The report has one page, **Summary**, with 15 visuals.

### Summary page visuals

- **Slicers:** Country, Fund Type, Fund Family, Individual Fund, Search by Fund(s)
- **Cards:**
  - **Total Fund Filings:** count of Fund IDs (68,181, one per unique fund)
  - **Total Fund Families:** distinct count of Family IDs
  - **Gross Asset Value (GAV): All Funds:** sum of Gross Asset Value
  - **Average GAV per Fund**
  - **Median GAV per Fund**
- **Bar chart:** Count of Funds by Type
- **Pie chart:** Funds by Gross Asset Value Category (fund count per size band)
- **Map:** Gross Asset Value by country
- **Treemap:** Gross Asset Value by Fund Type
- **Table:** Fund Name, Fund Type, Gross Asset Value, Owners, Minimum Investment

## Review findings

| Item | Finding | Outcome |
|---|---|---|
| Fund-count bar chart | Counts unique funds (68,181), not filings (6,680 distinct FilingIDs remain in the deduplicated data) | Retitled from "Total Filings by Fund Type" to "Count of Funds by Type" |
| "Funds by Gross Asset Value Category" chart | Counts non-blank Gross Asset Values; one row per fund, so it equals the distinct fund count | Correct as built (changed from a column chart to a pie chart) |
| Fund Family slicer | Filters fund visuals through the one-to-one relationship | Correct as built |
| Internal labels on visuals | 12 visuals still show the old table name (`ERA_Schedule_D_7B1_Jan-Oct_2026_deduped`), but every visual is bound to `Schedule_D_7B1` | Cosmetic only |
| No month filter | Year-to-date summary | By design |
| Precision loss on 17 large funds | −$295M net, about 0.002% | Accepted |
