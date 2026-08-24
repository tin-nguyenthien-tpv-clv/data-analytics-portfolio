# Pharmacy Sales & Profitability Analysis

A Power BI report over 62,139 daily sales transactions from 120 pharmacies across 8 European countries, covering 220 products over 24 months (2024–2025).

The report is built around one question: **where does margin actually come from?** The answer is nowhere near where revenue comes from. Revenue is a geography story. Margin is not — it lives entirely in product mix and promotional pricing, and the five pages are sequenced to prove that.

> **This dataset was created for the Onyx Data DNA challenge and does not reflect real pharmacy operations.** Every figure below comes from a modelling exercise. Read [Data Limits and Assumptions](#3-data-limits-and-assumptions) before treating any of it as a business KPI — in particular the margin *rate* in this data is effectively a constant across geography, which changes what the geographic pages can and cannot tell you.

> **Working notes:** [`docs/business-questions-to-visuals.md`](docs/business-questions-to-visuals.md) maps each challenge question to the visuals that answer it; [`docs/report-storyline.md`](docs/report-storyline.md) records the page-by-page narrative and the analysis behind it.

![Dashboard overview](assets/00_dashboard.png)

---

## 1. Business Problem and Objectives

Understand how sales and profitability vary across a European pharmacy distribution network, and quantify which levers actually move margin.

* **Primary objective:** locate the levers that move margin, and put a number on each.
* **Core objectives:**
  * **Geographic performance** — how revenue and margin distribute across country → region → city → pharmacy, and whether any market is structurally more profitable.
  * **Store benchmarking** — compare each pharmacy against peers *in its own region*, on a basis not distorted by store age or store size.
  * **Product & promotion economics** — which categories and brands generate revenue versus margin, which products are high-volume/low-margin, and whether promotional pricing pays for itself.
* **Data source:** [Onyx Data DNA Challenge (Jan–Feb 2025)](https://onyxdata.co.uk/data-dna-dataset-challenge/)

---

## 2. Dataset & Architecture

Star schema: three dimensions around one sales fact. Power Query reads the workbook in [`data/`](data/) directly from this repository over HTTPS, so **Refresh works without editing a single path**.

| Table | Source sheet | Rows | Grain |
|---|---|---|---|
| **FactSales** | `T_FactSales` | 62,139 | one row per pharmacy × product × day |
| **DimPharmacy** | `T_DimPharmacy` | 120 | one row per pharmacy |
| **DimProduct** | `T_DimProduct` | 220 | one row per product |
| **DimDate** | `T_DimDate` | 731 | one row per day, 2024-01-01 → 2025-12-31 |

Three relationships, all single-direction one-to-many from dimension to fact: `DimDate[DateKey]`, `DimPharmacy[PharmacyID]`, `DimProduct[ProductID]`.

### 2.1 Columns

| Table | Source columns | Added in the model |
|---|---|---|
| `FactSales` | `SalesID`, `DateKey`, `PharmacyID`, `ProductID`, `UnitsSold`, `RevenueEUR`, `CostEUR`, `MarginEUR`, `PromoFlag` | — |
| `DimDate` | `DateKey`, `Date`, `Year`, `Quarter`, `MonthNumber`, `MonthName`, `YearMonth` | `DayOfWeek`, `DayName`, `DayType` *(Power Query)* |
| `DimPharmacy` | `PharmacyID`, `PharmacyName`, `Country`, `Region`, `City`, `PharmacyType`, `StoreSizeBand`, `Latitude`, `Longitude`, `OpenDate` | `Store Cohort`, `Pharmacy Name`, `Latitude Band` *(DAX)* |
| `DimProduct` | `ProductID`, `ProductName`, `Category`, `Brand`, `IsGeneric`, `PackSize`, `ListPriceEUR`, `StandardCostEUR`, `LaunchDate`, `IsDiscontinued`, `DiscontinuedDate` | — |

**Why each added column exists**

* `DayOfWeek` / `DayName` / `DayType` — the source calendar has no weekday grain, and the strongest time pattern in the whole dataset turned out to live there. `DayName` is sorted by `DayOfWeek`.
* `Store Cohort` — splits *Established* from *Opened in window*, so the 11 stores that opened mid-period can be excluded from rankings.
* `Pharmacy Name` — strips the ` #NNN` suffix off `PharmacyName` for readable axis labels. It collapses 120 stores into 38 city-level names, so it is used **only** for labelling, never for per-store comparison.
* `Latitude Band` — four 4°-tall bands, used to test for a north–south gradient.

### 2.2 Hierarchies

| Table | Hierarchy | Levels |
|---|---|---|
| `DimPharmacy` | **Geography** | Country → Region → City → PharmacyName |
| `DimProduct` | **Product** | Category → Brand → ProductName |
| `DimDate` | **Calendar** | Year → Quarter → MonthName |

### 2.3 Model conventions

* `DimDate` is **marked as a date table** and Power BI's **Auto date/time is off**, so the model carries no hidden `LocalDateTable_*` tables and every time-intelligence measure resolves against one authoritative calendar.
* `Latitude` / `Longitude` are decimals with the Latitude/Longitude data categories and **Don't summarize**, so the map plots positions rather than summing coordinates.
* `Country` → Country, `Region` → State or Province, `City` → City data categories, for geocoding.
* 47 measures live in a dedicated `_Measures` table.

### 2.4 Measures

**Base**

| Measure | Definition |
|---|---|
| `Total Revenue` | `SUM(FactSales[RevenueEUR])` |
| `Total Margin` | `SUM(FactSales[MarginEUR])` |
| `Units Sold` | `SUM(FactSales[UnitsSold])` |
| `Margin %` | `DIVIDE([Total Margin],[Total Revenue])` |
| `Active Pharmacies` | `DISTINCTCOUNT(FactSales[PharmacyID])` |
| `Active Days` | `DISTINCTCOUNT(FactSales[DateKey])` |
| `Avg Unit Price` | `DIVIDE([Total Revenue],[Units Sold])` |
| `Units per Sale` | `DIVIDE([Units Sold],COUNTROWS(FactSales))` |

**Normalised — each one removes a specific confound**

| Measure | Removes |
|---|---|
| `Revenue per Pharmacy` | store-count differences between groups |
| `Revenue per Active Day` | store-age differences between stores |
| `Avg Daily Revenue` | month-length differences between months |

**Time** — `Revenue LY`, `Revenue YoY %`, `Revenue MoM %`, `Revenue Growth %`, `Margin % YoY (pp)`, `Seasonality Index`

**Contribution** — `Revenue Contribution %`, `Margin Contribution %`, `Cumulative Revenue %`, `Margin % vs Group (pp)`

**Peer benchmark** — `Region Avg Margin %`, `Region Avg Revenue per Day`, `Margin % vs Region (pp)`, `Revenue per Day vs Region %`, `Rank in Region`

**Product quadrant** — `Avg Margin % (All Products)`, `Avg Units per Product`, `Product Segment`

**Promotion** — `Revenue (Promo)`, `Revenue (Non-Promo)`, `Units (Promo)`, `Units (Non-Promo)`, `Promo Revenue Share %`, `Promo Units Share %`, `Margin % (Promo)`, `Margin % (Non-Promo)`, `Promo Margin Gap (pp)`, `Avg Price (Promo)`, `Avg Price (Non-Promo)`, `Promo Discount Depth %`, `Promo Units Uplift %`, `Margin Foregone`

**Geography** — `Store Latitude`, `Store Longitude`, `Store Country`, `Store City`

> `Revenue Growth %` exists because `Revenue YoY %` is only correct inside a time-sliced context. On an unsliced KPI card `SAMEPERIODLASTYEAR` returns 2024 as the prior period while `[Total Revenue]` returns both years, giving a meaningless **+104.4%**. `Revenue Growth %` pins the comparison to *latest year vs prior year* and returns the correct **+4.4%** regardless of selection.

---

## 3. Data Limits and Assumptions

Five properties of the raw files change how these findings should be read.

**1. The margin rate is effectively a constant across geography.**
Margin % runs 27.84%–28.16% across all 8 countries — a spread of **0.32pp**. Across 38 regions, 27.69%–28.61%. Across Urban / Suburban / Rural, 27.98%–28.10%. Across S / M / L store sizes, 28.03%–28.06%. Real pharmacy chains vary widely by country because reimbursement regimes and price regulation differ.
*Consequence:* the geographic pages answer "who is big", not "who is efficient". The report says so explicitly rather than manufacturing a difference — and it deliberately avoids colour-scaling any map or heat map by `Margin %`, because auto-scaling a 0.32pp spread renders as eight distinct colours and invents a pattern that does not exist.

**2. Category mix is uniform across every country, which is what makes margin flat.**
Prescription is 30.8%–33.6% of revenue in all 8 countries; OTC 20.0%–21.5%; Wellness 19.2%–21.2%. The same holds across store types (Prescription 31.7% / 32.5% / 32.6% for Rural / Suburban / Urban). Margin is a property of the mix, and every market sells the same mix, so every market lands on the same margin. Generator artifact, not market insight.

**3. Eleven of 120 stores opened inside the observation window, so raw revenue rankings rank store age.**
Vienna HealthPoint #115 opened 2025-12-13 and has **10 trading days**. On `Total Revenue` the network spread is **102.6×**; restricted to established stores and normalised to revenue per trading day it is **3.33×**. Every store comparison here uses the normalised basis, and the `Store Cohort` slicer sits on every page so the distinction stays visible.

**4. Promotion produces no volume response, which is commercially implausible.**
Units per transaction on promo (7.023) is *lower* than off promo (7.195) while price drops 11.0%. The promo flag lands at a near-identical 10.16%–11.73% of revenue in every country, and costs a near-identical 8.36–9.18pp of margin in every category. Together these read as a randomly assigned flag with a fixed discount, not a modelled campaign.
*Consequence:* treat the €82,709 margin-foregone figure as a demonstration of the method, not a recoverable number.

**5. Monthly seasonality is largely a calendar-length artifact.**
On raw revenue February appears ~6% below trend — but February has 28 days. Normalised to `Avg Daily Revenue`, February's index moves from 0.93 to 0.98 and the full-year range is only 0.92–1.09. Margin % by month varies 0.41pp. The genuine time signal is the weekday/weekend split, invisible in the source data until `DayName` was derived.

**Geographic position also fails to predict performance.** Across the 109 established stores, latitude correlates with revenue per trading day at r = +0.10 (r = +0.28 excluding Poland) and longitude at r = −0.15. There is a mild north–south gradient of about +11% from the southernmost band to the northernmost, and no east–west gradient at all. Poland — the weakest market — is a country effect, not a spatial one. *This is a data-shape note; no page in the current report visualises it.*

Minor notes: `DiscontinuedDate` is blank for 185 of 220 products; `OpenDate` reaches back to 2010 while the fact table covers only 2024–2025.

---

## 4. Executive Summary & Key Insights

Every insight names the visual and page it is read from.

### 4.1 The network is stable and time explains almost nothing

| | |
|---|---|
| **Scale** | €8,633,977 revenue · €2,421,141 margin · **28.04%** margin · 445,793 units · 120 pharmacies · 731 days |
| **Growth** | 2024 €4,223,414 → 2025 €4,410,563 = **+4.43%**, margin rate essentially unchanged (27.97% → 28.11%, **+0.13pp**) |
| **Seasonality** | day-normalised index **0.92–1.09**; margin % by month varies **0.41pp** |
| **Weekly rhythm** | weekdays **€12,471/day** vs weekends **€10,153/day** — an **18.6%** gap |

The weekday/weekend gap is roughly **five times larger** than the entire month-to-month seasonal range. It is the only strong time effect in the dataset.

> **Read from — Overview:** KPI card row (`Total Revenue`, `Revenue Growth %`, `Margin %`, `Units Sold`, `Active Pharmacies`) · *Total Revenue, Total Margin and Margin % by YearMonth* · *Avg Daily Revenue by MonthName and Year* · *Avg Daily Revenue by DayName and DayType*

![Overview page](assets/01_overview.png)

### 4.2 Geography is a story about scale, not efficiency

* **Concentration:** Germany 18.2%, France 16.3%, Italy 15.4% — the top three carry **49.9%** of revenue. The top 10 of 38 regions carry **46.9%**.
* **Profitability does not vary:** all 8 countries land between 27.84% and 28.16% margin.
* **Scale does vary:** revenue per store runs from Belgium €89,036 down to Poland €54,941 — a **1.62×** spread that is entirely volume.

> **Read from — Geography:** *Revenue by country* · *Margin % by country – identical everywhere* · *Pareto – top 10 of 38 regions* (`Cumulative Revenue %` against an 80% line) · *Country > region detail* matrix · *Where the stores are* map

![Geography page](assets/02_geography.png)

### 4.3 Store rankings only mean something after normalisation

* **Lifecycle bias is severe.** On raw revenue the worst store is Vienna #115 — which had simply been open 10 days. Excluding in-window openings and dividing by trading days collapses the spread from **102.6× to 3.33×**.
* **The genuine laggards are Polish.** Bottom five by revenue per trading day: Poznań #068 (€95), Warsaw #030 (€100), Katowice #080 (€110), Gdańsk #064 (€117), Poznań #025 (€127) — against a network median of **€193**. Poland's country median (€143) sits **33%** below the Netherlands (€215).
* **Store format drives volume only.** Urban averages €82,506 per store, Suburban €66,155, Rural €60,843 — all three at 27.98%–28.10% margin, because all three sell the same category mix. Size band L averages €126,898 against S at €38,713, a **3.28×** spread, again at identical margin.

> **Read from — Store Performance:** *Peer map – normalised to revenue per trading day* scatter · *Revenue per trading day – sort ascending to see laggards* · *Margin gap vs own region* · *League table* (`Rank in Region`, `Revenue per Day vs Region %`) · *Urban / Suburban / Rural – revenue per store*

![Store Performance page](assets/03_store_performance.png)

### 4.4 Margin comes from the product mix

| Category | Revenue | % of revenue | Margin % | Avg price |
|---|---|---|---|---|
| Prescription | €2,797,016 | 32.4% | **21.92%** | €44.20 |
| OTC | €1,797,330 | 20.8% | 29.39% | €10.12 |
| Wellness | €1,712,457 | 19.8% | **33.58%** | €19.15 |
| Personal Care | €1,454,603 | 16.9% | **33.47%** | €14.33 |
| Medical Devices | €872,572 | 10.1% | **24.96%** | €62.79 |

* **The ranking inverts.** Prescription is the largest revenue line and the least profitable — an **11.66pp** margin gap to Wellness.
* **Brands spread wider than categories:** **20.37%–35.31%** across 32 brands. AntiBioX leads revenue (€726,405) at 23.39% margin; ZenHealth earns **35.31%** on €285,040 — 39% of AntiBioX's revenue at 1.5× the margin rate.
* **Generics underperform:** 14.6% of revenue at 25.48% margin against 28.48% for branded.
* **Quadrant split** (against averages of 2,026 units and 28.04% margin per product): 109 Stars, 18 Volume drivers, 21 Niche premium, 72 Low priority. Low priority still carries €3.61M of revenue — that is where the high-price, low-volume Prescription and Medical Devices lines land.

> **Read from — Product Mix:** *Revenue vs margin by category – the ranks invert* · *Volume vs margin – 220 products, four quadrants* scatter (constant lines at `Avg Units per Product` and `Avg Margin % (All Products)`) · *Revenue by brand* · *Generic vs branded margin* · *Product action list* table (`Product Segment`)

![Product Mix page](assets/04_product_mix.png)

### 4.5 Promotion is where margin is lost

| Metric | Non-Promo | Promo | Delta |
|---|---|---|---|
| Revenue | €7,722,676 (89.4%) | €911,301 (10.6%) | — |
| **Margin %** | **29.00%** | **19.92%** | **−9.08pp** |
| Avg unit price | €19.62 | €17.46 | −11.00% |
| **Units per transaction** | **7.195** | **7.023** | **−2.39%** |

* **There is no volume uplift at all** — units per transaction is marginally *lower* on promotion. An 11% discount buys nothing.
* The sacrifice is near-identical everywhere: Prescription −9.18pp, Medical Devices −8.92pp, OTC −8.73pp, Wellness −8.67pp, Personal Care −8.36pp. Promo share of revenue is 10.16%–11.73% in every country — blanket application, not targeting.
* **Margin foregone: €82,709** over 24 months (≈ **€41,355/year**), computed as promo revenue valued at the non-promo margin rate, less actual promo margin.

> **Read from — Promotion:** KPI card row (`Promo Revenue Share %`, `Margin % (Non-Promo)`, `Margin % (Promo)`, `Promo Margin Gap (pp)`, `Margin Foregone`) · *Promo costs ~8.5pp of margin in every single category* · *Units per sale – promo buys no extra volume* · *Promo revenue over time and its share* · *Category > brand, promo vs non-promo* matrix

![Promotion page](assets/05_promotion.png)

### 4.6 Insight → visual traceability

| # | Insight | Page | Visual |
|---|---|---|---|
| 1 | +4.43% revenue growth, margin rate flat | Overview | KPI cards · combo trend by `YearMonth` |
| 2 | Monthly seasonality is a day-count artifact | Overview | *Avg Daily Revenue by MonthName and Year* |
| 3 | Weekends run 18.6% below weekdays | Overview | *Avg Daily Revenue by DayName and DayType* |
| 4 | Country contribution to the 2024→2025 change | Overview | waterfall, `Year` broken down by `Country` |
| 5 | Revenue composition holds steady over time | Overview | stacked area, `YearMonth` × `Country` |
| 6 | Top 3 countries = 49.9% of revenue | Geography | *Revenue by country* |
| 7 | Top 10 of 38 regions = 46.9% | Geography | Pareto with `Cumulative Revenue %` |
| 8 | Margin % identical in all 8 countries | Geography | *Margin % by country* on a fixed 0–35% axis |
| 9 | Where the stores physically are | Geography | Azure Maps bubble layer |
| 10 | Performance spread inside a country | Geography | drill-down Country → Region → City → Pharmacy · contribution matrix |
| 11 | 11 in-window openings distort rankings | Store Performance | `Store Cohort` slicer · *Revenue per trading day* |
| 12 | Bottom 5 stores per trading day are all Polish | Store Performance | *Revenue per trading day* · league table |
| 13 | Urban/Suburban/Rural differ in volume only | Store Performance | *Urban / Suburban / Rural – revenue per store* |
| 14 | Per-store margin gap against its own region | Store Performance | *Margin gap vs own region* · `Rank in Region` |
| 15 | Prescription: most revenue, least margin | Product Mix | *Revenue vs margin by category* |
| 16 | High-volume/low-margin vs low-volume/high-margin | Product Mix | four-quadrant scatter · `Product Segment` table |
| 17 | Brand margin spreads 20.4%–35.3% | Product Mix | *Revenue by brand* |
| 18 | Generics run 3pp below branded | Product Mix | *Generic vs branded margin* |
| 19 | Promo costs 9.08pp of margin | Promotion | KPI cards · *Promo costs ~8.5pp…* |
| 20 | Promo generates zero volume uplift | Promotion | *Units per sale – promo buys no extra volume* |
| 21 | Promo is untargeted | Promotion | promo share over time · category × brand matrix |

---

## 5. Actionable Recommendations

1. **Stop untargeted promotion — ≈ €41,355/year.** The flag is spread evenly across every country, store type and category and buys no incremental volume. Withdrawing blanket promotion recovers the full margin sacrifice. *(See limitation 4: in this dataset the absence of uplift is a generator property.)*
2. **If promotion continues, restrict it to Wellness and Personal Care.** Their 33.5% base margin absorbs a 9pp discount; the same discount on Prescription (21.92% base) consumes 42% of the product's profit.
3. **Shift mix toward Wellness and Personal Care — ≈ €25,173/year.** Moving 5 percentage points of revenue out of Prescription into Wellness captures the 11.66pp gap, worth €50,345 across the two-year window.
4. **Fix Poland before touching store formats.** The five weakest stores per trading day are all Polish and the country median trails the network by 33%. Store format is not the lever — margin is identical across Urban, Suburban and Rural.
5. **Reconsider weekend staffing and hours.** Weekends run 18.6% below weekdays — five times the size of any month-to-month seasonal effect.

---

## 6. Dashboard Overview

Five report pages, a fixed 220px navigation rail carrying five slicers (`Year`, `Country`, `Category`, `PharmacyType`, `Store Cohort`), and a page navigator across the top of every page. Two exploratory *Data Mining* pages remain in the file but are hidden from view mode.

### Page 1 — Overview
KPI row, revenue and margin trend with `Margin %` on a fixed axis, day-count-normalised seasonality, the weekday/weekend rhythm, a 2024→2025 waterfall broken down by country, and revenue composition over time.

![Overview page](assets/01_overview.png)

### Page 2 — Geography
Azure Maps bubble layer over all 120 stores, a Pareto of the 38 regions against cumulative revenue share, revenue by country, the deliberate "margin is flat everywhere" bar chart that sets up Page 4, a Country → Region → City → Pharmacy drill-down, and a contribution matrix.

> The Azure Maps visual needs two tenant switches under **Admin portal → Tenant settings → Integration settings**: *Users can use the Azure Maps visual*, and — for tenants outside the US/EU — *Data sent to Azure Maps can be processed outside your tenant's geographic region, compliance boundary, or national cloud instance*, which is **disabled by default**.

![Geography page](assets/02_geography.png)

### Page 3 — Store Performance
Peer scatter on revenue per trading day against margin, the laggard bar chart, margin gap against each store's own region, a league table ranked within region, and the Urban/Suburban/Rural comparison.

![Store Performance page](assets/03_store_performance.png)

### Page 4 — Product Mix
Four-quadrant scatter of all 220 products, revenue-versus-margin by category showing the rank inversion, brand ranking, generic versus branded, and the product action list.

![Product Mix page](assets/04_product_mix.png)

### Page 5 — Promotion
Promo KPI row including margin foregone, the per-category margin gap, the units-per-sale evidence that no volume uplift exists, promo share over time, and a category → brand promo matrix.

![Promotion page](assets/05_promotion.png)

### Selected charts

![Weekday vs weekend rhythm](assets/06_dayofweek.png)

![Four-quadrant product scatter](assets/07_product_quadrant.png)

![Promo margin gap by category](assets/08_promo_gap.png)

---

## 7. Key DAX Logic

Peer comparison is the analytical core. A store is judged against its own region, per trading day, so neither store age nor regional scale distorts the ranking.

```dax
-- Trading days, so a store open 10 days is not called a laggard
Active Days = DISTINCTCOUNT( FactSales[DateKey] )

Revenue per Active Day = DIVIDE( [Total Revenue], [Active Days] )

-- Peer benchmark: same country and region, all stores, own filter removed
Region Avg Revenue per Day =
VAR _RegionRevenue =
    CALCULATE( [Total Revenue], ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] ) )
VAR _RegionDays =
    CALCULATE( [Active Days], ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] ) )
RETURN
    DIVIDE( _RegionRevenue, _RegionDays )

-- Rank within the store's own region, not the whole network
Rank in Region =
RANKX(
    CALCULATETABLE(
        VALUES( DimPharmacy[PharmacyName] ),
        ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] )
    ),
    [Revenue per Active Day], , DESC, DENSE
)
```

Product quadrant classification, driven by dynamic averages so it re-bases under any slicer selection:

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

Quantifying the promotion decision — promo revenue valued at the non-promo rate, less what it actually earned:

```dax
Margin Foregone =
VAR _PromoRev = [Revenue (Promo)]
VAR _BaseRate = [Margin % (Non-Promo)]
VAR _PromoMar = CALCULATE( [Total Margin], FactSales[PromoFlag] = "Yes" )
RETURN
    _PromoRev * _BaseRate - _PromoMar
```

A KPI-card growth measure that stays correct when no year is selected:

```dax
Revenue Growth % =
VAR _MaxYear = MAX( DimDate[Year] )
VAR _Cur   = CALCULATE( [Total Revenue], DimDate[Year] = _MaxYear,     REMOVEFILTERS(DimDate) )
VAR _Prior = CALCULATE( [Total Revenue], DimDate[Year] = _MaxYear - 1, REMOVEFILTERS(DimDate) )
RETURN
    DIVIDE( _Cur - _Prior, _Prior )
```

---

## 8. Repository Structure

```
├── assets/   # Dashboard and chart screenshots
├── data/     # Source workbook, read over HTTPS by Power Query
├── docs/     # Challenge brief, business-question map, report storyline
├── powerbi/  # Power BI project source: .pbip, report layout & semantic model
└── README.md
```

**To open the report:** launch `powerbi/TinNguyen_Jan_Feb2025_Challenge.pbip` in Power BI Desktop, then **Refresh** — the derived calendar columns populate on refresh.

---

## 9. Acknowledgments & References

* **Challenge host:** [Onyx Data DNA Dataset Challenge](https://onyxdata.co.uk/data-dna-dataset-challenge/)
* **Mini challenge:** ZoomCharts Drill Down visuals — see [`docs/`](docs/)
