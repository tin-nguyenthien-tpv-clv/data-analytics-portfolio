# Business Questions → Visual Spec

**Project:** Pharmacy Sales & Profitability (DataDNA Jan–Feb 2025)
**Model hiện tại:** `FactSales` ← `DimDate` / `DimPharmacy` / `DimProduct`, measures nằm ở `_Measures`
**Data scope:** 2024-01-01 → 2025-12-31 (2 năm đủ → YoY & seasonality chạy được) · 8 countries · 38 regions/cities · 120 pharmacies · 220 products · 5 categories · 32 brands · `PromoFlag = Yes` chiếm 12% số dòng

---

## 0. Field inventory (những gì đang có sẵn)

| Bảng | Cột dùng được cho visual |
|---|---|
| `DimDate` | `Date`, `Year`, `Quarter`, `MonthNumber`, `MonthName` (đã sort by MonthNumber ✅), `YearMonth` |
| `DimPharmacy` | `PharmacyName`, `Country`, `Region`, `City`, `PharmacyType` (Urban/Suburban/Rural), `StoreSizeBand` (S/M/L), `Latitude` ⚠️ text, `Longitude` ⚠️ text, `OpenDate`, `Store Cohort` |
| `DimProduct` | `ProductName`, `Category`, `Brand`, `IsGeneric`, `PackSize`, `ListPriceEUR`, `StandardCostEUR`, `IsDiscontinued`, `LaunchDate` |
| `FactSales` | `UnitsSold`, `RevenueEUR`, `CostEUR`, `MarginEUR`, `PromoFlag` |

**Measures đã có:** `Total Revenue`, `Total Margin`, `Units Sold`, `Margin %`, `Active Pharmacies`, `Revenue per Pharmacy`, `Avg Unit Price`, `Margin % vs Group (pp)`, `Revenue YoY %`, `Margin % YoY (pp)`, `Seasonality Index`, `Units per Sale`, `Avg Daily Revenue`

Ký hiệu ⚠️ = measure/field chưa có, cần tạo (DAX ở cuối file).

---

## Q1 — Revenue / Units / Margin theo thời gian + seasonality

**Trang:** *Executive Overview*

| # | Visual | Field / Metric |
|---|---|---|
| 1.1 | **KPI card row** (5 cards) | `Total Revenue` (+ callout `Revenue YoY %`) · `Total Margin` · `Margin %` (+ `Margin % YoY (pp)`) · `Units Sold` · `Avg Unit Price` |
| 1.2 | **Line & clustered column combo** — trend chính | X: `DimDate[YearMonth]` · Column Y: `Total Revenue`, `Total Margin` · Line Y2: `Margin %` |
| 1.3 | **Line chart** — seasonal overlay | X: `DimDate[MonthName]` · Legend: `DimDate[Year]` · Y: `Total Revenue` → 2024 vs 2025 chồng nhau, mùa vụ lộ ngay |
| 1.4 | **Matrix heat-map** — seasonality | Rows: `DimDate[MonthName]` · Columns: `DimDate[Year]` · Values: `Seasonality Index` + conditional formatting background (diverging quanh 1.00) |
| 1.5 | **Line chart** — tách price vs volume | X: `DimDate[YearMonth]` · Y: `Units Sold` · Y2: `Avg Unit Price` → trả lời "revenue tăng do bán nhiều hơn hay bán đắt hơn" |
| 1.6 | *(tùy chọn)* **Combo** — Q-o-Q | Axis: `DimDate[Year]` + `DimDate[Quarter]` · Column: `Total Revenue` · Line: `Revenue MoM %` ⚠️ |

> Visual `Seasonality Index / Avg Daily Revenue` đang nằm ở page **Data Mining 2** — chuyển sang đây, đổi trục sang `MonthName` + legend `Year`.

---

## Q2 — Country & Region nào đóng góp nhiều Revenue / Margin nhất

**Trang:** *Geography*

| # | Visual | Field / Metric |
|---|---|---|
| 2.1 | **Clustered bar chart** (sort desc) | Y-axis: `DimPharmacy[Country]` · X: `Total Revenue` · Line (combo): `Margin %` |
| 2.2 | **Bar chart Top 10 Region** | Y-axis: `DimPharmacy[Region]` · X: `Total Margin` · Filter: Top 10 by `Total Margin` |
| 2.3 | **Scatter chart** — quy mô vs khả năng sinh lời | X: `Total Revenue` · Y: `Margin %` · Size: `Units Sold` · Legend: `Country` · Details: `Region` |
| 2.4 | **Matrix** | Rows: `Country` → `Region` · Values: `Total Revenue`, `Revenue Contribution %` ⚠️, `Total Margin`, `Margin %`, `Margin % vs Group (pp)`, `Active Pharmacies`, `Revenue per Pharmacy` |
| 2.5 | **Decomposition tree** | Analyze: `Total Revenue` · Explain by: `Country` → `Region` → `PharmacyType` → `Category` |

> Matrix `Country / PromoFlag` trên page **Data Mining** dùng lại được — bỏ `PromoFlag` khỏi Rows, thêm `Region` làm cấp 2.

---

## Q3 — Drill-down Country → Region → Pharmacy

**Trang:** *Geography*

**Bước bắt buộc:** tạo **hierarchy** trong `DimPharmacy` (model hiện chưa có):
`Geography = Country → Region → City → PharmacyName`

| # | Visual | Field / Metric |
|---|---|---|
| 3.1 | **Clustered column chart** bật drill-down | Axis: hierarchy `DimPharmacy[Geography]` · Column Y: `Total Revenue` · Line Y2: `Margin %` |
| 3.2 | **ZoomCharts Drill Down Combo PRO** *(mini-challenge)* | Category level 1/2/3: `Country` / `Region` / `PharmacyName` · Column: `Total Revenue` · Line: `Margin %` — click-to-drill mượt hơn drill-down mặc định |
| 3.3 | **Matrix** stepped layout | Rows: hierarchy `Geography` · Values: `Total Revenue`, `Revenue Contribution %` ⚠️, `Margin %`, `Units Sold`, `Revenue per Pharmacy` |
| 3.4 | **Treemap** | Group: `Country` · Details: `Region` · Values: `Total Revenue` · Color saturation: `Margin %` |

---

## Q4 — Pharmacy nào outperform / underperform so với cùng Region

**Trang:** *Store Performance*

⚠️ Câu này cần nhiều measure mới nhất — so sánh phải là **peer-relative trong region**, không phải so với toàn chuỗi.

| # | Visual | Field / Metric |
|---|---|---|
| 4.1 | **Scatter chart** — peer map | X: `Total Revenue` · Y: `Margin %` · Size: `Units Sold` · Legend: `Region` · Details: `PharmacyName` · + constant line X = `Region Avg Revenue` ⚠️, constant line Y = `Region Avg Margin %` ⚠️ |
| 4.2 | **Diverging bar chart** — gap vs peers | Y-axis: `PharmacyName` · X: `Margin % vs Region (pp)` ⚠️ · Data color rule: < 0 đỏ / ≥ 0 xanh · sort desc |
| 4.3 | **Table** — league table | `Country`, `Region`, `PharmacyName`, `PharmacyType`, `StoreSizeBand`, `Total Revenue`, `Revenue vs Region Avg %` ⚠️, `Margin %`, `Margin % vs Region (pp)` ⚠️, `Rank in Region` ⚠️ · data bars trên cột revenue |
| 4.4 | **Small multiples column chart** | Small multiples: `Region` · Axis: `PharmacyName` · Y: `Total Revenue` → mỗi region 1 ô nhỏ, outlier lộ ngay |
| 4.5 | **Top/Bottom 5 bar pair** | 2 bar chart: `PharmacyName` × `Margin % vs Region (pp)` ⚠️, filter Top 5 / Bottom 5 |

> Combo chart `PharmacyName × Total Margin / Total Revenue` trên page **Data Mining** đang so cả 120 pharmacy với nhau — thay bằng 4.2 hoặc 4.4.

---

## Q5 — Urban vs Suburban vs Rural

**Trang:** *Store Performance*

| # | Visual | Field / Metric |
|---|---|---|
| 5.1 | **Clustered column chart** | Axis: `DimPharmacy[PharmacyType]` · Y: `Revenue per Pharmacy`, `Total Margin` — dùng per-pharmacy vì số store 3 nhóm không bằng nhau |
| 5.2 | **Combo chart** | Axis: `PharmacyType` · Column: `Units Sold` · Line: `Margin %` |
| 5.3 | **Matrix** | Rows: `PharmacyType` → `StoreSizeBand` · Values: `Active Pharmacies`, `Total Revenue`, `Revenue per Pharmacy`, `Units Sold`, `Units per Sale`, `Avg Unit Price`, `Margin %` |
| 5.4 | **100% stacked bar** — mix sản phẩm | Axis: `PharmacyType` · Legend: `DimProduct[Category]` · Y: `Total Revenue` → giải thích *tại sao* margin 3 nhóm khác nhau |
| 5.5 | **Line chart** | X: `DimDate[YearMonth]` · Legend: `PharmacyType` · Y: `Revenue per Pharmacy` → 3 nhóm có mùa vụ khác nhau không |
| 5.6 | *(tùy chọn)* **Matrix cross-tab** | Rows: `Country` · Columns: `PharmacyType` · Values: `Margin %` + heat-map |

---

## Q6 — Category & Brand nào tạo nhiều Revenue / Margin nhất

**Trang:** *Product & Promotion*

| # | Visual | Field / Metric |
|---|---|---|
| 6.1 | **Clustered bar chart** | Y-axis: `DimProduct[Category]` · X: `Total Revenue` + `Total Margin` (2 series cạnh nhau → thấy ngay category revenue cao nhưng margin thấp) |
| 6.2 | **Bar chart Top 10 Brand by Revenue** | Y-axis: `DimProduct[Brand]` · X: `Total Revenue` · Tooltip: `Margin %` · Top 10 by `Total Revenue` |
| 6.3 | **Bar chart Top 10 Brand by Margin** | Y-axis: `Brand` · X: `Total Margin` · Top 10 by `Total Margin` → đặt cạnh 6.2 để thấy lệch thứ hạng |
| 6.4 | **Treemap** | Group: `Category` · Details: `Brand` · Values: `Total Revenue` · Color: `Margin %` |
| 6.5 | **Matrix** | Rows: `Category` → `Brand` · Values: `Total Revenue`, `Revenue Contribution %` ⚠️, `Units Sold`, `Avg Unit Price`, `Total Margin`, `Margin %`, `Margin % vs Group (pp)` |
| 6.6 | **Matrix cross-tab** — mix theo địa lý | Rows: `Category` · Columns: `Country` · Values: `Margin %` heat-map (page Data Mining 2 đã có bản `Avg Unit Price` — thêm bản `Margin %`) |
| 6.7 | *(tùy chọn)* **Clustered column** | Axis: `Category` · Legend: `DimProduct[IsGeneric]` · Y: `Margin %` → generic vs branded |

---

## Q7 — High volume/low margin vs low volume/high margin

**Trang:** *Product & Promotion*

| # | Visual | Field / Metric |
|---|---|---|
| 7.1 | **Scatter chart 4 góc phần tư** ⭐ | X: `Units Sold` · Y: `Margin %` · Size: `Total Revenue` · Legend: `Category` · Details: `DimProduct[ProductName]` · + X constant line = `Avg Units per Product` ⚠️ · + Y constant line = `Avg Margin % (All Products)` ⚠️ · tô nền 4 quadrant |
| 7.2 | **Table** — danh sách hành động | `ProductName`, `Category`, `Brand`, `Product Segment` ⚠️, `Units Sold`, `Total Revenue`, `Total Margin`, `Margin %`, `Avg Unit Price`, `ListPriceEUR` · sort `Units Sold` desc |
| 7.3 | **Clustered column** — quy mô từng nhóm | Axis: `Product Segment` ⚠️ · Y: `Total Revenue` + số sản phẩm (`COUNTROWS(DimProduct)`) |
| 7.4 | **Bar chart** — bán nhiều nhưng không lời | Y-axis: `ProductName` · X: `Margin %` (asc) · Filter: Top 30 by `Units Sold`, rồi lấy Bottom 15 margin |
| 7.5 | **Bar chart** — lời cao nhưng bán ít | Y-axis: `ProductName` · X: `Margin %` (desc) · Filter: Bottom 30 by `Units Sold`, lấy Top 15 margin |

> Scatter `Avg Unit Price × Units per Sale` trên page **Data Mining** đang ở cấp `Category` — phải xuống cấp `ProductName` mới trả lời được câu này (220 điểm, vừa đủ cho scatter).

---

## Q8 — Promoted vs Non-promoted

**Trang:** *Product & Promotion*

⚠️ Nhóm measure promo chưa có — hiện đang phải nhét `PromoFlag` vào Columns của matrix. Có measure riêng thì mới vẽ được card + combo.

| # | Visual | Field / Metric |
|---|---|---|
| 8.1 | **KPI card row** | `Promo Revenue Share %` ⚠️ · `Promo Units Share %` ⚠️ · `Margin % (Promo)` ⚠️ · `Margin % (Non-Promo)` ⚠️ · `Promo Margin Gap (pp)` ⚠️ |
| 8.2 | **Clustered column chart** | Axis: `FactSales[PromoFlag]` · Y: `Units per Sale`, `Avg Unit Price`, `Margin %` |
| 8.3 | **Clustered bar** — promo theo category | Axis: `Category` · Legend: `PromoFlag` · Y: `Margin %` *(đã có trên page Data Mining — giữ nguyên)* |
| 8.4 | **Stacked column + line** — promo theo thời gian | X: `DimDate[YearMonth]` · Column: `Revenue (Promo)` ⚠️ + `Revenue (Non-Promo)` ⚠️ · Line: `Promo Revenue Share %` ⚠️ |
| 8.5 | **Matrix** | Rows: `Category` → `Brand` · Columns: `PromoFlag` · Values: `Units Sold`, `Avg Unit Price`, `Margin %` *(page Data Mining 2 đã có bản Category-level)* |
| 8.6 | **Scatter** — promo có đáng tiền không | X: `Promo Discount Depth %` ⚠️ · Y: `Promo Units Uplift %` ⚠️ · Details: `Brand` · Size: `Revenue (Promo)` ⚠️ |
| 8.7 | **Clustered column** — promo theo loại store | Axis: `PharmacyType` · Legend: `PromoFlag` · Y: `Margin %` |

---

## Q9 — Regional performance đóng góp vào kết quả tổng

**Trang:** *Executive Overview*

| # | Visual | Field / Metric |
|---|---|---|
| 9.1 | **Waterfall chart** ⭐ | Category: `DimPharmacy[Country]` · Y: `Total Revenue` · Breakdown YoY: Y = `Total Revenue`, Breakdown = `Country`, so 2024 → 2025 |
| 9.2 | **Stacked area chart** | X: `DimDate[YearMonth]` · Legend: `Country` · Y: `Total Revenue` |
| 9.3 | **100% stacked column** — mix dịch chuyển | Axis: `Year` + `Quarter` · Legend: `Country` · Y: `Total Revenue` → share từng nước tăng/giảm |
| 9.4 | **Pareto combo chart** | Axis: `Region` (sort desc) · Column: `Total Revenue` · Line: `Cumulative Revenue %` ⚠️ · + constant line 80% |
| 9.5 | **Matrix contribution** | Rows: `Country` → `Region` · Values: `Total Revenue`, `Revenue Contribution %` ⚠️, `Total Margin`, `Margin Contribution %` ⚠️, `Revenue YoY %` |
| 9.6 | **Key influencers visual** | Analyze: `Margin %` · Explain by: `Country`, `Region`, `PharmacyType`, `StoreSizeBand`, `Category`, `PromoFlag`, `IsGeneric` |

---

## Q10 — Geographic patterns trên bản đồ

**Trang:** *Geography*

🚫 **Blocker:** `DimPharmacy[Latitude]` và `[Longitude]` đang là **text** → map visual không nhận. Phải sửa trước (Fix #1).

| # | Visual | Field / Metric |
|---|---|---|
| 10.1 | **Azure Map / Map (bubble)** ⭐ | Latitude: `DimPharmacy[Latitude]` · Longitude: `[Longitude]` · Size: `Total Revenue` · Bubble color (rule-based): `Margin %` · Tooltip: `PharmacyName`, `City`, `PharmacyType`, `Units Sold`, `Margin % vs Region (pp)` ⚠️ |
| 10.2 | **Filled map (choropleth)** | Location: `Country` (Data Category = Country/Region) · Color saturation: `Margin %` · Tooltip: `Total Revenue`, `Revenue per Pharmacy` |
| 10.3 | **ZoomCharts Drill Down Map PRO** *(mini-challenge)* | Location hierarchy `Country → Region → City` · Value: `Total Revenue` · Color: `Margin %` |
| 10.4 | **Drill-through từ map** | Bấm bubble → drill-through sang trang *Store Detail*, drill-through field = `PharmacyName` |
| 10.5 | **Phương án dự phòng** *(nếu không sửa được lat/long)* | Matrix Rows `Country` · Columns `Region` · Values `Margin %` heat-map — không đẹp bằng map nhưng vẫn trả lời được |

---

# 🔧 Việc cần làm trên model TRƯỚC khi dựng visual

## Fix #1 — Latitude / Longitude đang là text (chặn Q10)

Power Query `DimPharmacy` → sửa `Table.TransformColumnTypes`, đổi từ `type text` sang `type number`:

```m
{{"Latitude", type number}, {"Longitude", type number}}
```

Sau đó trong Model view: `Latitude` → Data Category = **Latitude**, Summarize by = **Don't summarize**; tương tự `Longitude` → **Longitude**.
Cùng lúc set: `Country` → Data Category **Country/Region**, `Region` → **State or Province**, `City` → **City**.

## Fix #2 — `Margin % vs Group (pp)` đang nhân 100 hai lần

```dax
Margin % vs Group (pp) = VAR _Group = CALCULATE([Margin %], ALLSELECTED()) RETURN ([Margin %] - _Group) * 100
```

`* 100` rồi lại format `0.00%` → chênh 2pp hiển thị thành **200%**. Chọn 1 trong 2:
- bỏ `* 100`, giữ formatString `0.00%`, hoặc
- giữ `* 100`, đổi formatString thành `0.00" pp"`

## Fix #3 — Chưa có hierarchy nào (chặn Q3)

- `DimPharmacy` → hierarchy **Geography**: `Country → Region → City → PharmacyName`
- `DimProduct` → hierarchy **Product**: `Category → Brand → ProductName`
- `DimDate` → hierarchy **Calendar**: `Year → Quarter → MonthName` (`MonthName` đã sort by `MonthNumber` ✅)

## Fix #4 — Mark as Date Table + tắt auto date/time

`DimDate` chưa được mark → Model view → `DimDate` → Mark as date table → cột `Date`.
Options → Data Load → tắt **Auto date/time** (đang sinh ra 4 bảng rác `LocalDateTable_*`).

---

# 📐 Measures cần thêm (mọi mục ⚠️ ở trên)

```dax
// ---------- Time (Q1) ----------
Revenue LY = CALCULATE( [Total Revenue], SAMEPERIODLASTYEAR(DimDate[Date]) )

Revenue MoM % =
VAR _Prev = CALCULATE( [Total Revenue], DATEADD(DimDate[Date], -1, MONTH) )
RETURN DIVIDE( [Total Revenue] - _Prev, _Prev )

// ---------- Contribution (Q2, Q3, Q6, Q9) ----------
Revenue Contribution % =
DIVIDE( [Total Revenue], CALCULATE( [Total Revenue], ALLSELECTED() ) )

Margin Contribution % =
DIVIDE( [Total Margin], CALCULATE( [Total Margin], ALLSELECTED() ) )

Cumulative Revenue % =            // cho Pareto 9.4
VAR _Regions   = CALCULATETABLE( VALUES(DimPharmacy[Region]), ALLSELECTED(DimPharmacy) )
VAR _Current   = [Total Revenue]
VAR _Running   =
    SUMX(
        FILTER( _Regions, CALCULATE( [Total Revenue] ) >= _Current ),
        CALCULATE( [Total Revenue] )
    )
RETURN DIVIDE( _Running, CALCULATE( [Total Revenue], ALLSELECTED(DimPharmacy) ) )

// ---------- Peer benchmark trong region (Q4) ----------
Region Avg Margin % =
CALCULATE( [Margin %], ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] ) )

Region Avg Revenue =
VAR _RegionRevenue =
    CALCULATE( [Total Revenue], ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] ) )
VAR _StoreCount =
    CALCULATE(
        DISTINCTCOUNT( DimPharmacy[PharmacyID] ),
        ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] )
    )
RETURN DIVIDE( _RegionRevenue, _StoreCount )

Margin % vs Region (pp) = ( [Margin %] - [Region Avg Margin %] ) * 100
    // formatString: 0.00" pp"

Revenue vs Region Avg % =
DIVIDE( [Total Revenue] - [Region Avg Revenue], [Region Avg Revenue] )

Rank in Region =
RANKX(
    CALCULATETABLE(
        VALUES( DimPharmacy[PharmacyName] ),
        ALLEXCEPT( DimPharmacy, DimPharmacy[Country], DimPharmacy[Region] )
    ),
    [Total Revenue], , DESC, DENSE
)

// ---------- Product quadrant (Q7) ----------
Avg Margin % (All Products) = CALCULATE( [Margin %], ALLSELECTED(DimProduct) )

Avg Units per Product =
VAR _Products = CALCULATETABLE( VALUES(DimProduct[ProductID]), ALLSELECTED(DimProduct) )
RETURN DIVIDE( CALCULATE( [Units Sold], ALLSELECTED(DimProduct) ), COUNTROWS( _Products ) )

Product Segment =
VAR _HighVol = [Units Sold] >= [Avg Units per Product]
VAR _HighMar = [Margin %]   >= [Avg Margin % (All Products)]
RETURN
SWITCH( TRUE(),
    _HighVol      &&  _HighMar,      "Star - high volume, high margin",
    _HighVol      && NOT _HighMar,   "Volume driver - high volume, low margin",
    NOT _HighVol  &&  _HighMar,      "Niche premium - low volume, high margin",
    "Low priority - low volume, low margin" )

// ---------- Promotion (Q8) ----------
Revenue (Promo)       = CALCULATE( [Total Revenue], FactSales[PromoFlag] = "Yes" )
Revenue (Non-Promo)   = CALCULATE( [Total Revenue], FactSales[PromoFlag] = "No"  )
Units (Promo)         = CALCULATE( [Units Sold],    FactSales[PromoFlag] = "Yes" )
Units (Non-Promo)     = CALCULATE( [Units Sold],    FactSales[PromoFlag] = "No"  )

Promo Revenue Share % = DIVIDE( [Revenue (Promo)], [Total Revenue] )
Promo Units Share %   = DIVIDE( [Units (Promo)],   [Units Sold] )

Margin % (Promo)      = CALCULATE( [Margin %], FactSales[PromoFlag] = "Yes" )
Margin % (Non-Promo)  = CALCULATE( [Margin %], FactSales[PromoFlag] = "No"  )
Promo Margin Gap (pp) = ( [Margin % (Promo)] - [Margin % (Non-Promo)] ) * 100
    // formatString: 0.00" pp"

Avg Price (Promo)      = CALCULATE( [Avg Unit Price], FactSales[PromoFlag] = "Yes" )
Avg Price (Non-Promo)  = CALCULATE( [Avg Unit Price], FactSales[PromoFlag] = "No"  )
Promo Discount Depth % = 1 - DIVIDE( [Avg Price (Promo)], [Avg Price (Non-Promo)] )

Promo Units Uplift % =
VAR _PromoDays    = CALCULATE( DISTINCTCOUNT(FactSales[DateKey]), FactSales[PromoFlag] = "Yes" )
VAR _NonPromoDays = CALCULATE( DISTINCTCOUNT(FactSales[DateKey]), FactSales[PromoFlag] = "No"  )
VAR _PromoRate    = DIVIDE( [Units (Promo)],     _PromoDays )
VAR _BaseRate     = DIVIDE( [Units (Non-Promo)], _NonPromoDays )
RETURN DIVIDE( _PromoRate - _BaseRate, _BaseRate )
```

---

# 🗂 Page layout đề xuất (4 trang + 1 drill-through)

| Trang | Trả lời | Visual chính |
|---|---|---|
| **1. Executive Overview** | Q1, Q9 | KPI cards · trend combo · seasonal overlay · waterfall by Country · stacked area · key influencers |
| **2. Geography** | Q2, Q3, Q10 | Map bubble · filled map · drill-down column (hierarchy) · Pareto region · matrix Country→Region · decomposition tree |
| **3. Store Performance** | Q4, Q5 | Scatter peer map · diverging bar vs region · league table · small multiples · combo Urban/Suburban/Rural |
| **4. Product & Promotion** | Q6, Q7, Q8 | Bar Category/Brand · scatter quadrant · segment table · promo KPI row · promo matrix |
| **5. Store Detail** *(drill-through)* | — | Drill-through field `PharmacyName`: trend riêng, top product, promo mix, gap so với region |

**Slicer bar dùng chung (sync trên cả 4 trang):**
`DimDate[Year]` · `DimDate[MonthName]` · `DimPharmacy[Country]` · `DimProduct[Category]` · `FactSales[PromoFlag]` · `DimPharmacy[PharmacyType]`

---

# 📊 So với report hiện tại

| Page hiện có | Xử lý |
|---|---|
| **Page 1** — 5 card đều đo `Total Revenue` | Sửa: mỗi card 1 measure khác (Revenue / Margin / Margin % / Units / Avg Price) → thành hàng KPI của *Executive Overview* |
| **Data Mining** — matrix Category×Promo, combo Pharmacy, matrix Country×Promo, scatter Category, bar Margin% Category×Promo | Đây là visual thăm dò. Giữ `bar Margin% Category×Promo` (→ 8.3) và `matrix Country×Promo` (→ 8.5); phần còn lại nâng cấp theo spec Q4 / Q7 |
| **Data Mining 2** — seasonality combo, matrix promo, store cohort, YoY×discontinued, price by country, price by category×country, revenue by pharmacy | Giữ `seasonality combo` (→ 1.4) và `matrix price Category×Country` (→ 6.6). Các matrix chỉ có 1 measure nên gộp thành matrix nhiều measure |
| **Chưa có ở đâu** | Map (Q10) · hierarchy drill-down (Q3) · so sánh peer trong region (Q4) · `PharmacyType` Urban/Suburban/Rural (Q5) · `Brand` (Q6) · `Region` (Q2) · waterfall & Pareto contribution (Q9) |

> ⚠️ **`Brand`, `Region`, `PharmacyType`, `StoreSizeBand`, `IsGeneric` hiện chưa xuất hiện trong bất kỳ visual nào** — trong khi 4/10 câu hỏi cần đúng những field đó.
