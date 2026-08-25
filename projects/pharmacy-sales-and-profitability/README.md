# Pharmacy Sales & Profitability Analysis

A Power BI report over 62,139 daily sales transactions from 120 pharmacies across 8 European countries, covering 220 products over 24 months (2024-2025).

The report is built around one question: **where does margin actually come from?** The answer is nowhere near where revenue comes from. Revenue is a geography story. Margin is not - it lives entirely in product mix and promotional pricing, and the five pages are sequenced to prove that.

> **This dataset was created for the Onyx Data DNA challenge and does not reflect real pharmacy operations.** Every figure below comes from a modelling exercise. Read [Data Limits and Assumptions](#3-data-limits-and-assumptions) before treating any of it as a business KPI - in particular the margin *rate* in this data is effectively a constant across geography, which changes what the geographic pages can and cannot tell you.

> **In a hurry?** [`docs/KEY_FINDINGS.md`](docs/KEY_FINDINGS.md) condenses the data limits and every insight below onto one screen.

> **Working notes:** [`docs/business-questions-to-visuals.md`](docs/business-questions-to-visuals.md) maps each challenge question to the visuals that answer it. [`docs/DAX_DOCUMENTATION.md`](docs/DAX_DOCUMENTATION.md) documents the model, every calculated column and all 47 measures.

<!-- ![Dashboard overview](assets/00_dashboard.png) -->

---

## 1. Business Problem and Objectives

Understand how sales and profitability vary across a European pharmacy distribution network, and quantify which levers actually move margin.

* **Primary objective:** locate the levers that move margin, and put a number on each.
* **Core objectives:**
  * **Geographic performance** - how revenue and margin distribute across country -> region -> city -> pharmacy, and whether any market is structurally more profitable.
  * **Store benchmarking** - compare each pharmacy against peers *in its own region*, on a basis not distorted by store age or store size.
  * **Product & promotion economics** - which categories and brands generate revenue versus margin, which products are high-volume/low-margin, and whether promotional pricing pays for itself.
* **Data source:** [Onyx Data DNA Challenge (Jan-Feb 2025)](https://onyxdata.co.uk/data-dna-dataset-challenge/)

---

## 2. Dataset & Architecture

Star schema: three dimensions around one sales fact. Power Query reads the workbook in [`data/`](data/) directly from this repository over HTTPS, so **Refresh works without editing a single path**.

| Table | Source sheet | Rows | Grain |
|---|---|---|---|
| **FactSales** | `T_FactSales` | 62,139 | one row per pharmacy x product x day |
| **DimPharmacy** | `T_DimPharmacy` | 120 | one row per pharmacy |
| **DimProduct** | `T_DimProduct` | 220 | one row per product |
| **DimDate** | `T_DimDate` | 731 | one row per day, 2024-01-01 -> 2025-12-31 |

### 2.1 Data model relationships

Three relationships, all single-direction one-to-many from dimension to fact: `DimDate[DateKey]`, `DimPharmacy[PharmacyID]` and `DimProduct[ProductID]` each filter `FactSales`. No bidirectional filters and no inactive relationships, so every measure resolves in one predictable direction.

`DimDate` is marked as a date table with Auto date/time off, which keeps time intelligence on one authoritative calendar. Three hierarchies sit on the dimensions - Geography (Country -> Region -> City -> Pharmacy), Product (Category -> Brand -> Product) and Calendar (Year -> Quarter -> Month) - and all 47 measures live in a separate `_Measures` table.

![Data model relationships](assets/Model_Relationship.png)

> Column definitions, the six calculated columns and every measure are documented in [`docs/DAX_DOCUMENTATION.md`](docs/DAX_DOCUMENTATION.md).

---

## 3. Data Limits and Assumptions

Five properties of the raw files change how these findings should be read.

**1. The margin rate is effectively a constant across geography.**
Margin % runs 27.84%-28.16% across all 8 countries - a spread of **0.32pp**. Across 38 regions, 27.69%-28.61%. Across Urban / Suburban / Rural, 27.98%-28.11%. Across S / M / L store sizes, 28.03%-28.06%. Real pharmacy chains vary widely by country because reimbursement regimes and price regulation differ.
*Consequence:* the geographic pages answer "who is big", not "who is efficient". The report says so explicitly rather than manufacturing a difference, and it deliberately avoids colour-scaling any map or heat map by `Margin %`, because auto-scaling a 0.32pp spread renders as eight distinct colours and invents a pattern that does not exist.

![Margin % by country on a fixed axis](assets/03_1_margin%20rate_limit.png)

**2. Category mix is uniform across every country, which is what makes margin flat.**
Prescription is 30.8%-33.6% of revenue in all 8 countries, OTC 20.0%-21.5%, Wellness 19.2%-21.2%. The same holds across store types (Prescription 31.7% / 32.5% / 32.6% for Rural / Suburban / Urban). Margin is a property of the mix, and every market sells the same mix, so every market lands on the same margin. Generator artifact, not market insight.

The category margin rates are identical country to country as well, so neither the mix nor the rates inside it move: Prescription lands at 21.75%-22.04% everywhere, Wellness at 33.48%-33.81%.

![Margin % by country and category](assets/03_2_margin%20flat_limit.png)

**3. Eleven of 120 stores opened inside the observation window, so raw revenue rankings rank store age.**
Vienna HealthPoint #115 opened 2025-12-13 and traded **10 days**. On raw `Total Revenue` the network spreads **102.6x**. Across the 109 established stores on revenue per trading day it spreads **3.33x** - EUR 94.75 to EUR 315.80.
*Consequence:* every store comparison uses the normalised basis, with a `Store Cohort` slicer on each page.

<!-- ![Lifecycle bias in raw store rankings](assets/03_3_lifecycle_limit.png) -->

**4. Promotion produces no volume response, which is commercially implausible.**
Average price falls in all five categories while units per transaction barely moves, and four of the five sell *fewer* on promotion - Personal Care 9.22 to 8.77, OTC 9.17 to 9.03, Wellness 7.18 to 7.07, Medical Devices 2.30 to 2.26, only Prescription edging up 4.82 to 4.84. Promo is 10.16%-11.73% of revenue in every country and costs 8.36-9.18pp of margin in every category.
*Consequence:* this reads as a randomly assigned flag with a fixed discount, not a modelled campaign. Treat the EUR 82,709 margin-foregone figure as a demonstration of method, not a recoverable number.

<!-- ![Promotion produces no volume uplift](assets/03_4_promo_uplift_limit.png) -->

**5. Monthly seasonality is largely a calendar-length artifact.**
Pooled over both years February earns EUR 656,408 against January EUR 693,212 and March EUR 700,991, about 6% below its neighbours. But February has 57 trading days to their 62: **per day it out-earns both**, EUR 11,516 against EUR 11,181 and EUR 11,306. Day-normalised, the twelve monthly indices sit inside **0.95-1.05** and margin % varies **0.41pp** across the year.
*Consequence:* the real time signal is the weekday/weekend split, invisible in the source until `DayName` was derived.

<!-- ![Seasonality is a day-count artifact](assets/03_5_seasonality_limit.png) -->

**Geographic position also fails to predict performance.** Across the 109 established stores latitude correlates with revenue per trading day at **r = +0.09** (+0.28 excluding Poland) and longitude at **r = -0.15**. The four latitude bands are not even monotonic: 48-52 N leads at EUR 213/day, the northernmost band sits at EUR 197 and the southernmost at EUR 186. Poland, the weakest market, is a country effect, not a spatial one. *Data-shape note, no page visualises it.*

<!-- ![Latitude versus revenue per trading day](assets/03_6_latitude_limit.png) -->

Minor notes: `DiscontinuedDate` is blank for 185 of 220 products, and `OpenDate` reaches back to 2010 while the fact table covers only 2024-2025.

---

## 4. Executive Summary & Key Insights

Section 4.6 names the page and visual behind every claim.

### 4.1 The network is stable and time explains almost nothing

EUR 8,633,977 revenue, EUR 2,421,141 margin, a **28.04%** margin rate, 445,793 units, 120 pharmacies, 731 days.

* 2024 EUR 4,223,414 to 2025 EUR 4,410,563 = **+4.43%**, with the margin rate essentially unchanged (27.97% to 28.11%, **+0.13pp**).
* Day-normalised, the twelve monthly indices span only **0.95-1.05** and margin % varies **0.41pp**.
* Weekdays earn **EUR 12,471/day** against weekends **EUR 10,153/day**, an **18.6%** gap - larger than the entire month-to-month spread of 10.9%, and the strongest time effect in the data.

<!-- ![Weekday versus weekend rhythm](assets/04_1_weekday_rhythm.png) -->

### 4.2 Geography is a story about scale, not efficiency

* Germany 18.2%, France 16.3% and Italy 15.4% carry **49.9%** of revenue. The top 10 of 38 regions carry **46.9%**.
* Profitability does not vary: all 8 countries land between **27.84%** and **28.16%** margin.
* Scale does: revenue per store runs Belgium EUR 89,036 down to Poland EUR 54,941, a **1.62x** spread that is entirely volume.

<!-- ![Revenue concentration versus flat margin by country](assets/04_2_geography_scale.png) -->

### 4.3 Store rankings only mean something after normalisation

* Raw revenue ranks store age. Excluding in-window openings and dividing by trading days collapses the spread from **102.6x to 3.33x**.
* The genuine laggards are all Polish: Poznan #068 (EUR 95/day), Warsaw #030 (EUR 100), Katowice #080 (EUR 110), Gdansk #064 (EUR 117), Poznan #025 (EUR 127), against a network median of **EUR 193**. Poland's median EUR 143 sits **33%** below the Netherlands at EUR 215.
* Format drives volume only: Urban EUR 82,506 per store, Suburban EUR 66,155, Rural EUR 60,843, all three at 27.98%-28.11% margin. Size band L EUR 126,898 against S EUR 38,713, a **3.28x** spread, again at identical margin.

<!-- ![Raw ranking versus normalised ranking](assets/04_3_store_league.png) -->

### 4.4 Margin comes from the product mix

| Category | Revenue | % of revenue | Margin % | Avg price |
|---|---|---|---|---|
| Prescription | EUR 2,797,016 | 32.4% | **21.92%** | EUR 44.20 |
| OTC | EUR 1,797,330 | 20.8% | 29.39% | EUR 10.12 |
| Wellness | EUR 1,712,457 | 19.8% | **33.58%** | EUR 19.15 |
| Personal Care | EUR 1,454,603 | 16.9% | **33.47%** | EUR 14.33 |
| Medical Devices | EUR 872,572 | 10.1% | **24.96%** | EUR 62.79 |

* The largest revenue line is the least profitable: **11.66pp** from Prescription to Wellness.
* Brands spread wider than categories, **20.37%-35.31%** across 32. AntiBioX leads revenue (EUR 726,405) at 23.39%, ZenHealth earns **35.31%** on EUR 285,040, 39% of AntiBioX's revenue at 1.5x the rate.
* Generics run 14.6% of revenue at **25.48%** against **28.48%** branded.
* Quadrants against averages of 2,026 units and 28.04% margin per product: **109 Stars, 18 Volume drivers, 21 Niche premium, 72 Low priority**. Low priority still carries EUR 3.61M, where the high-price low-volume Prescription and Medical Devices lines land.

<!-- ![Four-quadrant product scatter](assets/04_4_product_quadrant.png) -->

### 4.5 Promotion is where margin is lost

| Metric | Non-Promo | Promo | Delta |
|---|---|---|---|
| Revenue | EUR 7,722,676 (89.4%) | EUR 911,301 (10.6%) | - |
| **Margin %** | **29.00%** | **19.92%** | **-9.08pp** |
| Avg unit price | EUR 19.62 | EUR 17.46 | -11.00% |
| **Units per transaction** | **7.195** | **7.023** | **-2.39%** |

* **No volume uplift at all.** Units per transaction is marginally *lower* on promotion, so an 11% discount buys nothing.
* The sacrifice is near-identical everywhere: Prescription -9.18pp, Medical Devices -8.92pp, OTC -8.73pp, Wellness -8.67pp, Personal Care -8.36pp.
* Blanket application rather than targeting: promo is 10.16%-11.73% of revenue in every country.
* **Margin foregone EUR 82,709** over 24 months, about **EUR 41,355/year**, computed as promo revenue valued at the non-promo margin rate less actual promo margin.

<!-- ![Promo margin gap by category](assets/04_5_promo_margin_gap.png) -->

### 4.6 Insight to visual traceability

| # | Insight | Page | Visual |
|---|---|---|---|
| 1 | +4.43% revenue growth, margin rate flat | Overview | KPI cards, combo trend by `YearMonth` |
| 2 | Monthly seasonality is a day-count artifact | Overview | *Avg Daily Revenue by MonthName and Year* |
| 3 | Weekends run 18.6% below weekdays | Overview | *Avg Daily Revenue by DayName and DayType* |
| 4 | Country contribution to the 2024-2025 change | Overview | waterfall, `Year` broken down by `Country` |
| 5 | Revenue composition holds steady over time | Overview | stacked area, `YearMonth` x `Country` |
| 6 | Top 3 countries = 49.9% of revenue | Geography | *Revenue by country* |
| 7 | Top 10 of 38 regions = 46.9% | Geography | *Revenue concentration by region* with `Cumulative Revenue %` |
| 8 | Margin % identical in all 8 countries | Geography | *Margin % by country* on a fixed 0-35% axis |
| 9 | Where the stores physically are | Geography | *Where the stores are*, Azure Maps bubble layer |
| 10 | Performance spread inside a country | Geography | *Revenue by geography* drill-down, *Country and region scorecard* |
| 11 | 11 in-window openings distort rankings | Store Performance | `Store Cohort` slicer, *Revenue per trading day by store* |
| 12 | Bottom 5 stores per trading day are all Polish | Store Performance | *Revenue per trading day by store*, *Store league table* |
| 13 | Urban/Suburban/Rural differ in volume only | Store Performance | *Revenue per store by location type* |
| 14 | Per-store margin gap against its own region | Store Performance | *Margin gap vs own region*, `Rank in Region` |
| 15 | Prescription: most revenue, least margin | Product Mix | *Revenue vs margin by category* |
| 16 | High-volume/low-margin vs low-volume/high-margin | Product Mix | *Volume vs margin by product*, `Product Segment` table |
| 17 | Brand margin spreads 20.4%-35.3% | Product Mix | *Revenue by brand* |
| 18 | Generics run 3pp below branded | Product Mix | *Generic vs branded margin* |
| 19 | Promo costs 9.08pp of margin | Promotion | KPI cards, *Margin % by category, promo vs non-promo* |
| 20 | Promo generates zero volume uplift | Promotion | *Units per sale, promo vs non-promo* |
| 21 | Promo is untargeted | Promotion | *Promo revenue over time*, *Promo impact by category and brand* |

---

## 5. Actionable Recommendations

1. **Stop untargeted promotion, approximately EUR 41,355/year.** The flag is spread evenly across every country, store type and category and buys no incremental volume. Withdrawing blanket promotion recovers the full margin sacrifice. *(See limitation 4: in this dataset the absence of uplift is a generator property.)*
2. **If promotion continues, restrict it to Wellness and Personal Care.** Their 33.5% base margin absorbs a 9pp discount, the same discount on Prescription (21.92% base) consumes 42% of the product's profit.
3. **Shift mix toward Wellness and Personal Care, approximately EUR 25,173/year.** Moving 5 percentage points of revenue out of Prescription into Wellness captures the 11.66pp gap, worth EUR 50,345 across the two-year window.
4. **Fix Poland before touching store formats.** The five weakest stores per trading day are all Polish and the country median trails the network by 33%. Store format is not the lever, margin is identical across Urban, Suburban and Rural.
5. **Reconsider weekend staffing and hours.** Weekends run 18.6% below weekdays, a wider gap than the entire 10.9% month-to-month spread.

---

## 6. Dashboard Overview

Five report pages, a fixed 220px navigation rail carrying five slicers (`Year`, `Country`, `Category`, `PharmacyType`, `Store Cohort`), and a page navigator across the top of every page. Two exploratory *Data Mining* pages remain in the file but are hidden from view mode.

### Page 1 - Overview
KPI row, revenue and margin trend with `Margin %` on a fixed axis, day-count-normalised seasonality, the weekday/weekend rhythm, a 2024 to 2025 waterfall broken down by country, and revenue composition over time.

![Overview page](assets/01_Overview.png)

### Page 2 - Geography
Azure Maps bubble layer over all 120 stores, a Pareto of the 38 regions against cumulative revenue share, revenue by country, the deliberate "margin is flat everywhere" bar chart that sets up Page 4, a Country -> Region -> City -> Pharmacy drill-down, and a contribution matrix.

> The Azure Maps visual needs two tenant switches under **Admin portal -> Tenant settings -> Integration settings**: *Users can use the Azure Maps visual*, and, for tenants outside the US/EU, *Data sent to Azure Maps can be processed outside your tenant's geographic region, compliance boundary, or national cloud instance*, which is **disabled by default**.

![Geography page](assets/02_Geography.png)

### Page 3 - Store Performance
Peer scatter on revenue per trading day against margin, the laggard bar chart, margin gap against each store's own region, a league table ranked within region, and the Urban/Suburban/Rural comparison.

![Store Performance page](assets/03_Store_Performance.png)

### Page 4 - Product Mix
Four-quadrant scatter of all 220 products, revenue against margin by category, brand ranking, generic versus branded, and the product action list.

![Product Mix page](assets/04_Product_Mix.png)

### Page 5 - Promotion
Promo KPI row including margin foregone, the per-category margin gap, the units-per-sale evidence that no volume uplift exists, promo share over time, and a category to brand promo matrix.

![Promotion page](assets/05_Promotion.png)

---

## 7. Key DAX Logic

Peer comparison is the analytical core. A store is judged against stores in its own region, per trading day, so neither store age nor regional scale distorts the ranking. `ALLEXCEPT` clears the current store's own filters while keeping the region it belongs to, which turns a per-store measure into a per-region peer set.

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

This one measure is what turns a 102.6x raw revenue spread, which mostly ranks store age, into a 3.33x like-for-like comparison that surfaces the genuine laggards.

> All 47 measures, the six calculated columns and the model conventions behind them are documented in [`docs/DAX_DOCUMENTATION.md`](docs/DAX_DOCUMENTATION.md).

---

## 8. Repository Structure

```
├── assets/   # Dashboard and chart screenshots, model diagram
├── data/     # Source workbook, read over HTTPS by Power Query
├── docs/     # Challenge brief, business-question map, DAX documentation
├── powerbi/  # Power BI project source: .pbip, report layout & semantic model
└── README.md
```

**To open the report:** launch `powerbi/TinNguyen_Jan_Feb2025_Challenge.pbip` in Power BI Desktop, then **Refresh** - the derived calendar columns populate on refresh.

---

## 9. Acknowledgments & References

* **Challenge host:** [Onyx Data DNA Dataset Challenge](https://onyxdata.co.uk/data-dna-dataset-challenge/)
* **Mini challenge:** ZoomCharts Drill Down visuals - see [`docs/`](docs/)
