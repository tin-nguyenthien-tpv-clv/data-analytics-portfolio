# DAX Measures & Model Documentation

Full reference for the semantic model behind the Pharmacy Sales & Profitability report: tables, relationships, calculated columns, and all 47 measures.

The report README keeps only the single measure that carries the analytical argument. Everything else lives here.

---

## 1. Model Structure

### 1.1 Tables

| Table | Source sheet | Rows | Grain |
|---|---|---|---|
| **FactSales** | `T_FactSales` | 62,139 | one row per pharmacy x product x day |
| **DimPharmacy** | `T_DimPharmacy` | 120 | one row per pharmacy |
| **DimProduct** | `T_DimProduct` | 220 | one row per product |
| **DimDate** | `T_DimDate` | 731 | one row per day, 2024-01-01 -> 2025-12-31 |
| `_Measures` | - | 1 (dummy) | measure host table, hidden |

### 1.2 Relationships

Three relationships, all single-direction one-to-many from dimension to fact.

| From | To | Cardinality | Direction |
|---|---|---|---|
| `DimDate[DateKey]` | `FactSales[DateKey]` | 1 : * | single |
| `DimPharmacy[PharmacyID]` | `FactSales[PharmacyID]` | 1 : * | single |
| `DimProduct[ProductID]` | `FactSales[ProductID]` | 1 : * | single |

No bidirectional filters and no inactive relationships, so every measure resolves in one predictable direction and no `USERELATIONSHIP` is needed anywhere.

### 1.3 Hierarchies

| Table | Hierarchy | Levels |
|---|---|---|
| `DimPharmacy` | **Geography** | Country -> Region -> City -> PharmacyName |
| `DimProduct` | **Product** | Category -> Brand -> ProductName |
| `DimDate` | **Calendar** | Year -> Quarter -> MonthName |

### 1.4 Model conventions

* `DimDate` is **marked as a date table** (`dataCategory: Time`) and Power BI's **Auto date/time is off**, so the model carries no hidden `LocalDateTable_*` tables and every time-intelligence measure resolves against one authoritative calendar.
* `Latitude` / `Longitude` are decimals with the Latitude / Longitude data categories and **Don't summarize**, so the map plots positions rather than summing coordinates.
* `Country` -> Country, `Region` -> State or Province, `City` -> City data categories, for geocoding.
* `MonthName` is sorted by `MonthNumber`, `DayName` is sorted by `DayOfWeek`.
* All 47 measures live in a dedicated `_Measures` table so the field list separates logic from data.

---

## 2. Source Columns

| Table | Columns as delivered |
|---|---|
| `FactSales` | `SalesID`, `DateKey`, `PharmacyID`, `ProductID`, `UnitsSold`, `RevenueEUR`, `CostEUR`, `MarginEUR`, `PromoFlag` |
| `DimDate` | `DateKey`, `Date`, `Year`, `Quarter`, `MonthNumber`, `MonthName`, `YearMonth` |
| `DimPharmacy` | `PharmacyID`, `PharmacyName`, `Country`, `Region`, `City`, `PharmacyType`, `StoreSizeBand`, `Latitude`, `Longitude`, `OpenDate` |
| `DimProduct` | `ProductID`, `ProductName`, `Category`, `Brand`, `IsGeneric`, `PackSize`, `ListPriceEUR`, `StandardCostEUR`, `LaunchDate`, `IsDiscontinued`, `DiscontinuedDate` |

`MarginEUR` arrives pre-computed in the source, so margin is summed rather than derived from `RevenueEUR - CostEUR`.

---

## 3. Calculated Columns

### 3.1 `DimDate` - added in Power Query

The source calendar has no weekday grain, and the strongest time pattern in the whole dataset turned out to live there.

```m
DayOfWeek = Table.AddColumn(#"Changed Type", "DayOfWeek",
                each Date.DayOfWeek([Date], Day.Monday), Int64.Type)

DayName   = Table.AddColumn(#"Added DayOfWeek", "DayName",
                each Date.DayOfWeekName([Date], "en-US"), type text)

DayType   = Table.AddColumn(#"Added DayName", "DayType",
                each if [DayOfWeek] >= 5 then "Weekend" else "Weekday", type text)
```

`DayOfWeek` is 0-6 starting Monday and exists purely as the sort-by column for `DayName`, which would otherwise sort alphabetically. `DayType` collapses the week into the two buckets that actually differ: weekdays run 18.6% ahead of weekends.

### 3.2 `DimPharmacy` - added in DAX

#### `Store Cohort`

Separates stores that traded for the full window from the 11 that opened inside it. Without this split, raw revenue rankings rank store age.

```dax
Store Cohort =
IF( DimPharmacy[OpenDate] < DATE(2024,1,1), "Established", "Opened in window" )
```

#### `Pharmacy Name`

Strips the ` #NNN` suffix off `PharmacyName` for readable axis labels. The final `LEN(...) + 1` argument makes `SEARCH` return a fallback position instead of erroring when no suffix is present.

> Used **only** for labelling. It collapses 120 stores into 38 city-level names, so it must never be used for per-store comparison.

```dax
Pharmacy Name =
LEFT(
    DimPharmacy[PharmacyName],
    SEARCH( " #", DimPharmacy[PharmacyName], 1, LEN( DimPharmacy[PharmacyName] ) + 1 ) - 1
)
```

#### `Latitude Band`

Four 4-degree bands, built to test for a north-south performance gradient. The test came back weak (r = +0.10), which is why no page visualises it.

```dax
Latitude Band =
SWITCH( TRUE(),
    DimPharmacy[Latitude] < 44, "37-44 N (South)",
    DimPharmacy[Latitude] < 48, "44-48 N",
    DimPharmacy[Latitude] < 52, "48-52 N",
    "52-56 N (North)" )
```

---

## 4. Base Measures

Eight measures every other measure is built from.

| Measure | Definition | Format |
|---|---|---|
| `Total Revenue` | `SUM( FactSales[RevenueEUR] )` | `#,0` |
| `Total Margin` | `SUM( FactSales[MarginEUR] )` | `#,0` |
| `Units Sold` | `SUM( FactSales[UnitsSold] )` | `#,0` |
| `Margin %` | `DIVIDE( [Total Margin], [Total Revenue] )` | `0.00%` |
| `Active Pharmacies` | `DISTINCTCOUNT( FactSales[PharmacyID] )` | `0` |
| `Active Days` | `DISTINCTCOUNT( FactSales[DateKey] )` | `#,0` |
| `Avg Unit Price` | `DIVIDE( [Total Revenue], [Units Sold] )` | `0.00` |
| `Units per Sale` | `DIVIDE( [Units Sold], COUNTROWS( FactSales ) )` | general |

`Active Days` counts days on which a store actually transacted, taken from the fact table rather than the calendar. That distinction is what makes it safe to divide by: a store open 10 days gets 10, not 731.

`Units per Sale` divides by `COUNTROWS(FactSales)` rather than a transaction ID, because the fact grain is already one row per pharmacy x product x day.

---

## 5. Normalised Measures

Each one removes a specific confound. These three are the reason the analysis holds up.

| Measure | Definition | Removes |
|---|---|---|
| `Revenue per Pharmacy` | `DIVIDE( [Total Revenue], [Active Pharmacies] )` | store-count differences between groups |
| `Revenue per Active Day` | `DIVIDE( [Total Revenue], [Active Days] )` | store-age differences between stores |
| `Avg Daily Revenue` | `DIVIDE( [Total Revenue], DISTINCTCOUNT( DimDate[Date] ) )` | month-length differences between months |

**Why `Revenue per Active Day` matters.** On raw `Total Revenue` the network spread between best and worst store is 102.6x, and the "worst" store is one that had been open for 10 days. Restricted to established stores and divided by trading days, the spread collapses to 3.33x and the genuine laggards surface.

**Why `Avg Daily Revenue` matters.** February appears about 6% below trend on raw revenue, but February has 28 days. Normalised, February's seasonality index moves from 0.93 to 0.98 and the apparent seasonal dip disappears.

Note the denominators differ on purpose: `Revenue per Active Day` uses `FactSales[DateKey]` (days a store traded) while `Avg Daily Revenue` uses `DimDate[Date]` (calendar days in the period).

---

## 6. Time Intelligence

### `Revenue LY`

```dax
Revenue LY =
CALCULATE( [Total Revenue], SAMEPERIODLASTYEAR( DimDate[Date] ) )
```

### `Revenue YoY %`

Correct inside a time-sliced context. See `Revenue Growth %` for the unsliced case.

```dax
Revenue YoY % =
VAR _Prior = CALCULATE( [Total Revenue], SAMEPERIODLASTYEAR( DimDate[Date] ) )
RETURN
    DIVIDE( [Total Revenue] - _Prior, _Prior )
```

### `Revenue MoM %`

```dax
Revenue MoM % =
VAR _Prev = CALCULATE( [Total Revenue], DATEADD( DimDate[Date], -1, MONTH ) )
RETURN
    DIVIDE( [Total Revenue] - _Prev, _Prev )
```

### `Margin % YoY (pp)`

Returned in percentage points, not as a percentage change, because a rate moving 27.97% -> 28.11% is a +0.13pp move and reporting it as "+0.5%" would be misleading.

```dax
Margin % YoY (pp) =
VAR _Prior = CALCULATE( [Margin %], SAMEPERIODLASTYEAR( DimDate[Date] ) )
RETURN
    [Margin %] - _Prior
```

### `Revenue Growth %`

The KPI-card variant, and the reason it exists is worth spelling out.

On an unsliced card, `SAMEPERIODLASTYEAR` returns 2024 as the prior period while `[Total Revenue]` returns both years, producing a meaningless **+104.4%**. `Revenue Growth %` pins the comparison to latest year against prior year and returns the correct **+4.43%** regardless of selection.

```dax
Revenue Growth % =
VAR _MaxYear = MAX( DimDate[Year] )
VAR _Cur   = CALCULATE( [Total Revenue], DimDate[Year] = _MaxYear,     REMOVEFILTERS( DimDate ) )
VAR _Prior = CALCULATE( [Total Revenue], DimDate[Year] = _MaxYear - 1, REMOVEFILTERS( DimDate ) )
RETURN
    DIVIDE( _Cur - _Prior, _Prior )
```

### `Seasonality Index`

Average monthly revenue in the current context against average monthly revenue across the whole selection. A value of 1.00 is an average month.

```dax
Seasonality Index =
VAR _ThisMonthAvg = AVERAGEX( VALUES( DimDate[YearMonth] ), [Total Revenue] )
VAR _AllMonthAvg =
    CALCULATE(
        AVERAGEX( VALUES( DimDate[YearMonth] ), [Total Revenue] ),
        ALLSELECTED( DimDate )
    )
RETURN
    DIVIDE( _ThisMonthAvg, _AllMonthAvg )
```

> Pair this with `Avg Daily Revenue`, not `Total Revenue`, or it will report calendar length as seasonality.

---

## 7. Contribution & Concentration

### `Revenue Contribution %` and `Margin Contribution %`

Share of the visible total. `ALLSELECTED` means the denominator respects slicers but ignores the row's own category, so the column sums to 100% inside any filter state.

```dax
Revenue Contribution % =
DIVIDE( [Total Revenue], CALCULATE( [Total Revenue], ALLSELECTED() ) )

Margin Contribution % =
DIVIDE( [Total Margin], CALCULATE( [Total Margin], ALLSELECTED() ) )
```

### `Margin % vs Group (pp)`

Generic gap-to-peer-group measure in percentage points, where the group is whatever is currently visible.

```dax
Margin % vs Group (pp) =
VAR _Group = CALCULATE( [Margin %], ALLSELECTED() )
RETURN
    ( [Margin %] - _Group ) * 100
```

### `Cumulative Revenue %`

The Pareto line on the Geography page. Running share of revenue across regions, ordered by revenue descending, computed without relying on visual sort order.

The `>=` comparison is what builds the running total: for the current region it sums every region whose revenue is at least as large.

```dax
Cumulative Revenue % =
VAR _Regions = CALCULATETABLE( VALUES( DimPharmacy[Region] ), ALLSELECTED( DimPharmacy ) )
VAR _Current = [Total Revenue]
VAR _Running =
    SUMX(
        FILTER( _Regions, CALCULATE( [Total Revenue] ) >= _Current ),
        CALCULATE( [Total Revenue] )
    )
RETURN
    DIVIDE( _Running, CALCULATE( [Total Revenue], ALLSELECTED( DimPharmacy ) ) )
```

> Ties are counted together, so two regions on identical revenue both report the higher cumulative value. With continuous euro amounts across 38 regions this does not occur in practice.

---

## 8. Peer Benchmark

The analytical core of the Store Performance page. A store is judged against stores in its own region, on a per-trading-day basis, so neither store age nor regional scale distorts the ranking.

`ALLEXCEPT( DimPharmacy, Country, Region )` is the shared mechanism: it clears the current store's own filters while keeping the region it belongs to, which turns a per-store measure into a per-region peer average.

### `Region Avg Margin %`

```dax
Region Avg Margin % =
CALCULATE( [Margin %], ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] ) )
```

### `Region Avg Revenue per Day`

Revenue and days are aggregated to region level **before** dividing, so the result is a true regional rate rather than an average of store averages.

```dax
Region Avg Revenue per Day =
VAR _RegionRevenue =
    CALCULATE( [Total Revenue], ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] ) )
VAR _RegionDays =
    CALCULATE( [Active Days], ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] ) )
RETURN
    DIVIDE( _RegionRevenue, _RegionDays )
```

### `Margin % vs Region (pp)`

Drives the diverging bar chart. Positive means the store beats its region.

```dax
Margin % vs Region (pp) =
( [Margin %] - [Region Avg Margin %] ) * 100
```

### `Revenue per Day vs Region %`

```dax
Revenue per Day vs Region % =
VAR _Peer = [Region Avg Revenue per Day]
RETURN
    DIVIDE( [Revenue per Active Day] - _Peer, _Peer )
```

### `Rank in Region`

Rank 1 is the strongest store **within its own region**, not within the network. `DENSE` keeps ranks contiguous when stores tie.

```dax
Rank in Region =
RANKX(
    CALCULATETABLE(
        VALUES( DimPharmacy[PharmacyName] ),
        ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] )
    ),
    [Revenue per Active Day], , DESC, DENSE
)
```

> Ranks over `PharmacyName` (unique, with the `#NNN` suffix), not the label column `Pharmacy Name`, which would merge stores sharing a city.

---

## 9. Product Quadrant

Classifies all 220 products against dynamic averages, so the quadrant boundaries re-base under any slicer selection instead of being hard-coded.

### `Avg Margin % (All Products)`

The horizontal reference line on the scatter.

```dax
Avg Margin % (All Products) =
CALCULATE( [Margin %], ALLSELECTED( DimProduct ) )
```

### `Avg Units per Product`

The vertical reference line. Total selected units divided by the count of selected products, which is a per-product mean rather than a per-row mean.

```dax
Avg Units per Product =
VAR _Products = CALCULATETABLE( VALUES( DimProduct[ProductID] ), ALLSELECTED( DimProduct ) )
RETURN
    DIVIDE( CALCULATE( [Units Sold], ALLSELECTED( DimProduct ) ), COUNTROWS( _Products ) )
```

### `Product Segment`

The numeric prefixes exist so the four labels sort in reading order in slicers and tables.

```dax
Product Segment =
VAR _HighVol = [Units Sold] >= [Avg Units per Product]
VAR _HighMar = [Margin %]   >= [Avg Margin % (All Products)]
RETURN
    SWITCH( TRUE(),
        _HighVol        &&      _HighMar,   "1 - Star (high volume, high margin)",
        _HighVol        && NOT( _HighMar ), "2 - Volume driver (high volume, low margin)",
        NOT( _HighVol ) &&      _HighMar,   "3 - Niche premium (low volume, high margin)",
        "4 - Low priority (low volume, low margin)" )
```

At the full 220-product selection this splits into 109 Stars, 18 Volume drivers, 21 Niche premium and 72 Low priority.

---

## 10. Promotion

Fourteen measures. `PromoFlag` is a text column holding `"Yes"` and `"No"`, so every split is a `CALCULATE` filter rather than a separate relationship.

### Split totals

```dax
Revenue (Promo)     = CALCULATE( [Total Revenue], FactSales[PromoFlag] = "Yes" )
Revenue (Non-Promo) = CALCULATE( [Total Revenue], FactSales[PromoFlag] = "No"  )
Units (Promo)       = CALCULATE( [Units Sold],    FactSales[PromoFlag] = "Yes" )
Units (Non-Promo)   = CALCULATE( [Units Sold],    FactSales[PromoFlag] = "No"  )

Margin % (Promo)     = CALCULATE( [Margin %], FactSales[PromoFlag] = "Yes" )
Margin % (Non-Promo) = CALCULATE( [Margin %], FactSales[PromoFlag] = "No"  )

Avg Price (Promo)     = CALCULATE( [Avg Unit Price], FactSales[PromoFlag] = "Yes" )
Avg Price (Non-Promo) = CALCULATE( [Avg Unit Price], FactSales[PromoFlag] = "No"  )
```

### Shares and gaps

```dax
Promo Revenue Share % = DIVIDE( [Revenue (Promo)], [Total Revenue] )
Promo Units Share %   = DIVIDE( [Units (Promo)],   [Units Sold] )

Promo Margin Gap (pp) = ( [Margin % (Promo)] - [Margin % (Non-Promo)] ) * 100

Promo Discount Depth % = 1 - DIVIDE( [Avg Price (Promo)], [Avg Price (Non-Promo)] )
```

`Promo Margin Gap (pp)` returns **-9.08** across the network, and stays between -8.36 and -9.18 in every one of the five categories.

### `Promo Units Uplift %`

The measure that answers whether promotion works at all. Both sides are converted to a units-per-day rate first, because promo and non-promo cover different numbers of days and comparing raw unit totals would just compare exposure.

```dax
Promo Units Uplift % =
VAR _PromoDays    = CALCULATE( DISTINCTCOUNT( FactSales[DateKey] ), FactSales[PromoFlag] = "Yes" )
VAR _NonPromoDays = CALCULATE( DISTINCTCOUNT( FactSales[DateKey] ), FactSales[PromoFlag] = "No"  )
VAR _PromoRate    = DIVIDE( [Units (Promo)],     _PromoDays )
VAR _BaseRate     = DIVIDE( [Units (Non-Promo)], _NonPromoDays )
RETURN
    DIVIDE( _PromoRate - _BaseRate, _BaseRate )
```

### `Margin Foregone`

Promo revenue valued at the non-promo margin rate, less what promotion actually earned. This is the euro figure the recommendation rests on: **82,709 EUR** over 24 months.

```dax
Margin Foregone =
VAR _PromoRev = [Revenue (Promo)]
VAR _BaseRate = [Margin % (Non-Promo)]
VAR _PromoMar = CALCULATE( [Total Margin], FactSales[PromoFlag] = "Yes" )
RETURN
    _PromoRev * _BaseRate - _PromoMar
```

> The counterfactual assumes promo volume would have held at the non-promo price, which the near-zero uplift supports in this dataset. See limitation 4 in the README: the absence of uplift is a generator property, so treat the number as a demonstration of method rather than recoverable margin.

---

## 11. Geography Helpers

Four scalar measures feeding the drill-through card header and the map tooltip. `SELECTEDVALUE` returns blank rather than erroring when more than one store is in context.

```dax
Store Latitude  = AVERAGE( DimPharmacy[Latitude] )
Store Longitude = AVERAGE( DimPharmacy[Longitude] )
Store Country   = SELECTEDVALUE( DimPharmacy[Country] )
Store City      = SELECTEDVALUE( DimPharmacy[City] )
```

`Store Latitude` and `Store Longitude` average rather than select, so a city with several stores still resolves to a single plottable point.

---

## 12. Measure Index

| Group | Count | Measures |
|---|---|---|
| Base | 8 | `Total Revenue`, `Total Margin`, `Units Sold`, `Margin %`, `Active Pharmacies`, `Active Days`, `Avg Unit Price`, `Units per Sale` |
| Normalised | 3 | `Revenue per Pharmacy`, `Revenue per Active Day`, `Avg Daily Revenue` |
| Time | 6 | `Revenue LY`, `Revenue YoY %`, `Revenue MoM %`, `Revenue Growth %`, `Margin % YoY (pp)`, `Seasonality Index` |
| Contribution | 4 | `Revenue Contribution %`, `Margin Contribution %`, `Cumulative Revenue %`, `Margin % vs Group (pp)` |
| Peer benchmark | 5 | `Region Avg Margin %`, `Region Avg Revenue per Day`, `Margin % vs Region (pp)`, `Revenue per Day vs Region %`, `Rank in Region` |
| Product quadrant | 3 | `Avg Margin % (All Products)`, `Avg Units per Product`, `Product Segment` |
| Promotion | 14 | `Revenue (Promo)`, `Revenue (Non-Promo)`, `Units (Promo)`, `Units (Non-Promo)`, `Promo Revenue Share %`, `Promo Units Share %`, `Margin % (Promo)`, `Margin % (Non-Promo)`, `Promo Margin Gap (pp)`, `Avg Price (Promo)`, `Avg Price (Non-Promo)`, `Promo Discount Depth %`, `Promo Units Uplift %`, `Margin Foregone` |
| Geography | 4 | `Store Latitude`, `Store Longitude`, `Store Country`, `Store City` |
| **Total** | **47** | |

Calculated columns: 3 in DAX (`Store Cohort`, `Pharmacy Name`, `Latitude Band`) and 3 in Power Query (`DayOfWeek`, `DayName`, `DayType`).
