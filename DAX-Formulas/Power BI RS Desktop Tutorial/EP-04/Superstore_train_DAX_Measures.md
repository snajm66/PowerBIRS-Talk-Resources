# DAX Measure Library — train.csv (Superstore Sales)

Dataset profile: 9,800 rows | 2015-01-03 to 2018-12-30 | 793 customers | 4,922 orders | 49 states | 4 regions | 3 categories | 3 segments | 4 ship modes. Columns: Row ID, Order ID, Order Date, Ship Date, Ship Mode, Customer ID, Customer Name, Segment, Country, City, State, Postal Code, Region, Product ID, Category, Sub-Category, Product Name, Sales.

Table name used below: `'train'` — rename to match your actual table name in the model.

## ⚠️ Data prep gotcha before writing any measure

`Order Date` and `Ship Date` are stored as **DD/MM/YYYY** (e.g. `12/06/2017` → 16/06/2017 ship date proves it, day=16 can't be a month). If your Power BI Desktop locale is US, Power Query will silently misparse these as MM/DD/YYYY for the first 12 days of every month and corrupt the whole date model.

Fix in Power Query: select both date columns → **Transform → Data Type → Using Locale → English (United Kingdom)**. Do this before anything else.

---

## 1. Date table (required for all time intelligence below)

```DAX
Calendar = 
ADDCOLUMNS(
    CALENDAR(DATE(2015,1,1), DATE(2018,12,31)),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMM"),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "Year-Month", FORMAT([Date], "MMM YYYY")
)
```
Mark as Date Table (Table tools → Mark as Date Table → `Date` column), then relate `'Calendar'[Date]` → `'train'[Order Date]`.

---

## 2. Core measures

```DAX
Total Sales = SUM('train'[Sales])

Total Orders = DISTINCTCOUNT('train'[Order ID])

Total Customers = DISTINCTCOUNT('train'[Customer ID])

Total Products Sold = DISTINCTCOUNT('train'[Product ID])

Avg Order Value = DIVIDE([Total Sales], [Total Orders])

Avg Sales per Customer = DIVIDE([Total Sales], [Total Customers])

Avg Delivery Days = 
AVERAGEX('train', DATEDIFF('train'[Order Date], 'train'[Ship Date], DAY))
```

---

## 3. Time intelligence

```DAX
Sales PY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))

Sales YoY % = DIVIDE([Total Sales] - [Sales PY], [Sales PY])

Sales YTD = TOTALYTD([Total Sales], 'Calendar'[Date])

Sales QTD = TOTALQTD([Total Sales], 'Calendar'[Date])

Sales MTD = TOTALMTD([Total Sales], 'Calendar'[Date])

Sales Prior Month = CALCULATE([Total Sales], DATEADD('Calendar'[Date], -1, MONTH))

Sales MoM % = DIVIDE([Total Sales] - [Sales Prior Month], [Sales Prior Month])
```

---

## 4. Category / Region / Segment analysis

```DAX
% of Total Sales = 
DIVIDE([Total Sales], CALCULATE([Total Sales], ALL('train')))

Category Rank = 
RANKX(ALL('train'[Category]), [Total Sales])

Top Sub-Category by Sales = 
CALCULATE(
    SELECTEDVALUE('train'[Sub-Category]),
    TOPN(1, ALL('train'[Sub-Category]), [Total Sales])
)

Sales vs Region Avg % = 
DIVIDE([Total Sales], AVERAGEX(ALL('train'[Region]), [Total Sales])) - 1
```

---

## 5. Multi-color measure (bars colored by rule — plug into Format → Colors → fx → Field value)

Colors each Region/Category bar green / amber / red based on performance vs the average of its own field — same visual, 3+ colors.

```DAX
Region Color = 
VAR _RegionSales = [Total Sales]
VAR _AvgRegionSales = AVERAGEX(ALL('train'[Region]), [Total Sales])
RETURN
SWITCH(
    TRUE(),
    _RegionSales >= _AvgRegionSales * 1.1, "#4C9F70",   -- above avg: good
    _RegionSales >= _AvgRegionSales * 0.9, "#F2A65A",   -- near avg: neutral
    "#C4423D"                                            -- below avg: bad
)
```

---

## 6. Gradient-look measure (dark → light shading by value)

Use on a "Sales by State" or "Sales by Sub-Category" bar chart. Plug into Format → Colors → fx → Field value.

```DAX
Sales Gradient Color = 
VAR _Value = [Total Sales]
VAR _Min = MINX(ALLSELECTED('train'[State]), [Total Sales])
VAR _Max = MAXX(ALLSELECTED('train'[State]), [Total Sales])
VAR _Pct = DIVIDE(_Value - _Min, _Max - _Min, 0)

VAR _StartR = 46   VAR _StartG = 94   VAR _StartB = 78    -- #2E5E4E (dark, low value)
VAR _EndR   = 143  VAR _EndG   = 191  VAR _EndB   = 159   -- #8FBF9F (light, high value)

VAR _R = ROUND(_StartR + (_EndR - _StartR) * _Pct, 0)
VAR _G = ROUND(_StartG + (_EndG - _StartG) * _Pct, 0)
VAR _B = ROUND(_StartB + (_EndB - _StartB) * _Pct, 0)

VAR _Hex  = "0123456789ABCDEF"
VAR _RHex = MID(_Hex, INT(_R/16)+1, 1) & MID(_Hex, MOD(_R,16)+1, 1)
VAR _GHex = MID(_Hex, INT(_G/16)+1, 1) & MID(_Hex, MOD(_G,16)+1, 1)
VAR _BHex = MID(_Hex, INT(_B/16)+1, 1) & MID(_Hex, MOD(_B,16)+1, 1)

RETURN "#" & _RHex & _GHex & _BHex
```

Swap `'train'[State]` for `'train'[Sub-Category]` (or any category field) to match whatever the chart's axis is grouped by — the min/max must iterate the same field that's on the chart, otherwise the gradient will look flat.

---

## Notes for the PBIRS demo

- All measures above are standard DAX — fully supported on PBIRS 2.150.1926.0 (Jan 2026), no Power BI Service-only functions used.
- Time intelligence needs the `Calendar` table marked as a Date Table; without it `SAMEPERIODLASTYEAR`/`TOTALYTD` will error.
- `DATEDIFF` requires Order Date/Ship Date to actually be Date type (see the locale gotcha above) — if they're still text, `Avg Delivery Days` will fail.
