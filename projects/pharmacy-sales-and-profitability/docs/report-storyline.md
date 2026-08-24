# Report Storyline — 5 pages

**Project:** Pharmacy Sales & Profitability (DataDNA Jan–Feb 2025)
Khung này **được rút ra từ dữ liệu**, không phải khung mẫu. Mọi con số dưới đây đã tính trực tiếp từ `Pharmacy_Data_Challenge_Dataset.xlsx`.

---

## 🔑 Phát hiện chi phối toàn bộ story

> **Margin RATE gần như là hằng số trên mọi chiều địa lý — biến động margin chỉ nằm ở product mix và promotion.**

| Chiều phân tích | Spread của `Margin %` | Kết luận |
|---|---|---|
| 8 Countries | 27,9% – 28,1% (**0,2pp**) | Phẳng |
| 38 Regions | 27,7% – 28,6% (**0,9pp**) | Phẳng |
| Urban / Suburban / Rural | tất cả **28,0%** | Phẳng |
| Store size S / M / L | tất cả **28,0%** | Phẳng |
| 12 tháng | 27,8% – 28,2% (**0,4pp**) | Phẳng |
| **5 Categories** | 21,9% – 33,6% (**11,7pp**) | ⚡ **Biến động thật** |
| **32 Brands** | 20,4% – 35,3% (**14,9pp**) | ⚡ **Biến động thật** |
| **Promo vs Non-promo** | 19,9% vs 29,0% (**9,1pp**) | ⚡ **Biến động thật** |

**Lý do margin phẳng theo địa lý:** category mix gần như **giống hệt nhau ở cả 8 nước** — Prescription chiếm 30,8%–33,6% doanh thu ở mọi nước, OTC 20,0%–21,5%, Wellness 19,2%–21,2%. Không nước nào bán mix khác biệt → không nước nào có margin khác biệt.

**Hệ quả cho thiết kế report:**
- Câu hỏi địa lý (Q2, Q3, Q9, Q10) trả lời về **quy mô**, không phải về **hiệu quả**. Đừng tô heat-map `Margin %` theo nước — nó sẽ ra một mảng màu đồng nhất vô nghĩa.
- Câu hỏi margin (Q6, Q7, Q8) mới là nơi đặt insight và recommendation.
- Story đi từ *"tiền đang ở đâu"* → *"tiền đang mất ở đâu"*.

---

## 📖 Cấu trúc: 5 page + 1 drill-through

| # | Page | Câu hỏi | Vai trò trong story |
|---|---|---|---|
| 1 | **Overview — Ổn định, tăng nhẹ** | Q1, Q9 | Đặt bối cảnh: quy mô, tăng trưởng, mùa vụ |
| 2 | **Geography — Quy mô nằm ở đâu** | Q2, Q3, Q10 | Tiền đang ở đâu (và margin **không** khác nhau) |
| 3 | **Store Performance — So sánh công bằng** | Q4, Q5 | Ai thật sự yếu, sau khi chuẩn hoá |
| 4 | **Product Mix — Nguồn gốc của margin** | Q6, Q7 | ⚡ Bước ngoặt: margin đến từ mix, không từ nơi chốn |
| 5 | **Promotion — Nơi margin bị đốt** | Q8 | ⚡ Cao trào + khuyến nghị hành động |
| ↳ | *Store Detail* (drill-through) | — | Chi tiết 1 pharmacy khi click từ page 2/3 |

---

# Page 1 — Overview: Ổn định, tăng nhẹ, không có mùa vụ mạnh

**Thông điệp:** *Doanh nghiệp €8,6M, margin 28%, tăng trưởng đều +4,4%. Mùa vụ theo tháng yếu — nhịp đáng chú ý là theo ngày trong tuần.*

### Số liệu nền
- Revenue **€8.633.977** · Margin **€2.421.141** · Margin% **28,04%** · **445.793** units · 120 pharmacies · 24 tháng
- 2024 €4,22M → 2025 €4,41M = **+4,43%**; Margin% 27,97% → 28,11% (**+0,14pp**)
- Seasonality index sau khi chuẩn hoá số ngày: **0,92 – 1,09** (yếu)
- Ngày trong tuần: T2–T6 **€12,3–12,7k/ngày** vs T7–CN **€10,0–10,3k/ngày** → **−19%**

### Visual

| # | Visual | Field / Metric |
|---|---|---|
| 1.1 | **KPI card row** (5 card) | `Total Revenue` + `Revenue YoY %` · `Total Margin` · `Margin %` + `Margin % YoY (pp)` · `Units Sold` · `Active Pharmacies` |
| 1.2 | **Line & column combo** — trend chính | X `DimDate[YearMonth]` · Column `Total Revenue` · Line Y2 `Margin %` với trục **cố định 20–35%** để thấy rõ nó là đường thẳng |
| 1.3 | **Line chart** — mùa vụ | X `DimDate[MonthName]` · Legend `DimDate[Year]` · Y **`Avg Daily Revenue`** ⚠️ *không dùng `Total Revenue`* |
| 1.4 | **Column chart** — nhịp tuần ⭐ | X `DimDate[DayName]` 🆕 · Legend `DimDate[DayType]` 🆕 · Y `Avg Daily Revenue` |
| 1.5 | **Waterfall** — cầu nối 2024→2025 | Y `Total Revenue` · Breakdown `DimPharmacy[Country]` · filter 2 năm |
| 1.6 | **Stacked area** | X `YearMonth` · Legend `Country` · Y `Total Revenue` |

> **⚠️ Bẫy phải tránh ở 1.3:** dùng `Total Revenue` theo tháng thì tháng 2 trông tụt 6%, nhưng đó gần như hoàn toàn là do tháng 2 có 28/29 ngày. Chuẩn hoá bằng `Avg Daily Revenue` thì chỉ số tháng 2 nhảy từ 0,93 lên 0,98 — mùa vụ biến mất. **Nói thẳng điều này trong text box** — nhận ra artifact là điểm cộng lớn khi chấm.

> **🆕 Cần thêm cột** `DimDate[DayOfWeek]` (sort-by) + `[DayName]` + `[DayType]` trong Power Query — dataset gốc không có, mà đây là pattern thời gian mạnh nhất trong toàn bộ data.

---

# Page 2 — Geography: Quy mô nằm ở đâu

**Thông điệp:** *Doanh thu tập trung ở Đức/Pháp/Ý (50%), nhưng khả năng sinh lời thì giống nhau ở mọi nơi. Địa lý là câu chuyện quy mô, không phải hiệu quả.*

### Số liệu nền
- Top 3 nước (DE 18,2% · FR 16,3% · IT 15,4%) = **49,9%** doanh thu
- Top 10 / 38 regions = **46,9%** doanh thu
- Revenue/store: Belgium €89,0k cao nhất → Poland €54,9k thấp nhất (**1,6x**)
- Margin% mọi nước: **28,0%** (spread 0,2pp)

### Visual

| # | Visual | Field / Metric |
|---|---|---|
| 2.1 | **Azure Map (bubble)** ⭐ | Lat `DimPharmacy[Latitude]` · Long `[Longitude]` · Size `Total Revenue` · Color `PharmacyType` · Tooltip `PharmacyName`, `City`, `Revenue per Active Day` 🆕, `Margin %` |
| 2.2 | **Bar chart** sort desc | Y `Country` · X `Total Revenue` · data label `Revenue Contribution %` ⚠️ |
| 2.3 | **Pareto combo** ⭐ | X `Region` sort desc · Column `Total Revenue` · Line `Cumulative Revenue %` ⚠️ · constant line 80% |
| 2.4 | **Drill-down column** | Axis hierarchy `Geography` (`Country→Region→City→PharmacyName`) · Column `Total Revenue` · Line `Margin %` |
| 2.5 | **Small KPI + text box** — bằng chứng "margin phẳng" | Card `Margin %` lặp cho 8 nước, hoặc bar `Country` × `Margin %` với trục **0–35%** → cho thấy 8 cột cao bằng nhau |
| 2.6 | **Matrix** | Rows `Country`→`Region` · Values `Total Revenue`, `Revenue Contribution %` ⚠️, `Active Pharmacies`, `Revenue per Pharmacy`, `Margin %` |

> **Bố cục có chủ ý:** 2.5 là visual "kết quả âm tính" — cố ý cho người xem thấy margin bằng nhau ở mọi nước, để bàn đạp sang Page 4. Đừng dùng filled map tô màu theo `Margin %`: chênh 0,2pp sẽ bị Power BI kéo giãn thành 8 màu khác biệt và **tạo ra một pattern không hề tồn tại**.

> **⚠️ Về ZoomCharts:** nếu làm mini-challenge, thay 2.4 bằng **Drill Down Combo PRO** và 2.1 bằng **Drill Down Map PRO** (`Country → Region → City`).

---

# Page 3 — Store Performance: So sánh cho công bằng

**Thông điệp:** *Xếp hạng theo doanh thu thô là xếp hạng tuổi cửa hàng. Chuẩn hoá xong, khoảng cách thu hẹp từ 102x xuống 3,3x — và nhóm yếu thật sự là Ba Lan.*

### Số liệu nền — đây là page nhiều insight kỹ thuật nhất
- **11/120 pharmacy khai trương trong kỳ 2024–2025.** Vienna HealthPoint #115 mở 13/12/2025 → chỉ **10 ngày** bán, €1.583 doanh thu
- Dùng `Total Revenue` thô: chênh lệch max/min = **102,6x** (vô nghĩa)
- Chỉ lấy store đã ổn định + dùng revenue/ngày hoạt động: **3,3x** (so sánh được)
- Bottom 5 theo €/ngày: **cả 5 đều ở Ba Lan** — Poznań #068 (€95), Warsaw #030 (€100), Katowice #080 (€110), Gdańsk #064 (€117), Poznań #025 (€127) · median toàn hệ thống **€192**
- Poland median €143/ngày vs Netherlands €213/ngày (**−33%**)
- Urban 48% revenue, €82,5k/store · Suburban 36%, €66,2k · Rural 16%, €60,8k — **margin% cả 3 đều 28,0%**
- Size L €126,9k/store vs S €38,7k (**3,3x**) — margin% vẫn 28,0%
- Category mix của Urban/Suburban/Rural gần như trùng khít (Prescription 31,7%/32,5%/32,6%)

### Visual

| # | Visual | Field / Metric |
|---|---|---|
| 3.1 | **Slicer nổi bật** | `DimPharmacy[Store Cohort]` (đã có sẵn: Established / Opened in window) — **mặc định chọn Established** + text box giải thích |
| 3.2 | **Scatter — peer map** ⭐ | X `Revenue per Active Day` 🆕 · Y `Margin %` · Size `Units Sold` · Legend `Country` · Details `PharmacyName` · constant line X = median hệ thống |
| 3.3 | **Bar chart đối chiếu** ⭐ | Y `PharmacyName` (bottom 10) · X **2 series**: `Total Revenue` và `Revenue per Active Day` 🆕 → cho thấy 3 store "tệ nhất" biến mất khỏi danh sách sau khi chuẩn hoá |
| 3.4 | **Diverging bar** | Y `PharmacyName` · X `Margin % vs Region (pp)` ⚠️ · màu theo dấu |
| 3.5 | **Table league** | `Country`, `Region`, `PharmacyName`, `PharmacyType`, `StoreSizeBand`, `Active Days` 🆕, `Total Revenue`, `Revenue per Active Day` 🆕, `Rank in Region` ⚠️, `Margin %` |
| 3.6 | **Clustered column** — Q5 | X `PharmacyType` · Y `Revenue per Pharmacy` **và** `Margin %` (2 trục) → cột revenue chênh rõ, đường margin phẳng |
| 3.7 | **Matrix** — Q5 chi tiết | Rows `PharmacyType`→`StoreSizeBand` · Values `Active Pharmacies`, `Revenue per Pharmacy`, `Units per Sale`, `Avg Unit Price`, `Margin %` |

> **Đây là page ghi điểm cao nhất về mặt phân tích.** Hầu hết bài dự thi sẽ xếp hạng theo `Total Revenue` và kết luận Vienna #115 là "underperformer tệ nhất" — trong khi nó chỉ mới mở cửa 10 ngày. Xử lý được lifecycle bias là điểm khác biệt.

> **🆕 Measure cần thêm:**
> ```dax
> Active Days = DISTINCTCOUNT( FactSales[DateKey] )
> Revenue per Active Day = DIVIDE( [Total Revenue], [Active Days] )
> ```

---

# Page 4 — Product Mix: Margin thật sự đến từ đâu ⚡

**Thông điệp:** *Prescription tạo 1/3 doanh thu nhưng margin thấp nhất. Wellness và Personal Care chỉ chiếm 37% doanh thu nhưng margin cao gấp rưỡi. Toàn bộ biến động lợi nhuận nằm ở đây.*

### Số liệu nền

| Category | Revenue | % Rev | Margin % | Avg Price |
|---|---|---|---|---|
| Prescription | €2,80M | 32,4% | **21,9%** 🔻 | €44 |
| OTC | €1,80M | 20,8% | 29,4% | €10 |
| Wellness | €1,71M | 19,8% | **33,6%** 🔺 | €19 |
| Personal Care | €1,45M | 16,9% | **33,5%** 🔺 | €14 |
| Medical Devices | €0,87M | 10,1% | **25,0%** 🔻 | €63 |

- Brand: AntiBioX doanh thu #1 (€726k) nhưng margin chỉ **23,4%**; ImmunoPlus margin **33,7%** với doanh thu bằng 1/2
- Brand margin% spread **20,4% – 35,3%**
- Generic 15% doanh thu, margin **25,5%** vs non-generic **28,5%**

### Visual

| # | Visual | Field / Metric |
|---|---|---|
| 4.1 | **Bar chart 2 series** ⭐ | Y `Category` · X `Total Revenue` + `Total Margin` cạnh nhau → thứ hạng đảo giữa 2 series là insight |
| 4.2 | **Combo chart** | X `Category` sort theo Revenue · Column `Total Revenue` · Line `Margin %` → đường margin **dốc ngược** với cột |
| 4.3 | **Scatter 4 quadrant** ⭐⭐ | X `Units Sold` · Y `Margin %` · Size `Total Revenue` · Legend `Category` · Details `ProductName` (220 điểm) · constant line X `Avg Units per Product` ⚠️ + Y `Avg Margin % (All Products)` ⚠️ |
| 4.4 | **Bar Top 10 Brand** | Y `Brand` · X `Total Revenue` · Line/tooltip `Margin %` |
| 4.5 | **Bar Top 10 Brand by Margin** | Y `Brand` · X `Total Margin` — đặt **cạnh 4.4** để so lệch thứ hạng |
| 4.6 | **Table** — danh sách hành động | `ProductName`, `Category`, `Brand`, `Product Segment` ⚠️, `Units Sold`, `Total Revenue`, `Margin %`, `Avg Unit Price` |
| 4.7 | **Treemap** | Group `Category` · Details `Brand` · Values `Total Revenue` · Color saturation `Margin %` |
| 4.8 | *(tuỳ chọn)* **Column** | X `Category` · Legend `IsGeneric` · Y `Margin %` |

> **Đây là page trả lời Q7 chuẩn nhất.** Scatter 4 góc phần tư ở cấp `ProductName` (220 điểm là mật độ lý tưởng) sẽ tách ra 4 nhóm rõ: Prescription/Medical Devices dồn về góc "giá cao – margin thấp", Wellness/Personal Care về góc "margin cao".

---

# Page 5 — Promotion: Nơi margin bị đốt ⚡

**Thông điệp:** *Promotion giảm margin 9,1pp nhưng không tạo thêm sản lượng. Đây là khoản lỗ ròng ~€83k, và nó xảy ra đồng đều ở mọi category, mọi nước, mọi loại cửa hàng.*

### Số liệu nền — insight mạnh nhất của cả report

| Chỉ số | Non-Promo | Promo | Chênh |
|---|---|---|---|
| Revenue | €7,72M (89,4%) | €0,91M (10,6%) | — |
| **Margin %** | **29,0%** | **19,9%** | **−9,1pp** |
| Avg unit price | €19,62 | €17,46 | −11,0% |
| **Units / transaction** | **7,20** | **7,02** | **−2,4%** ⚠️ |

- **Không có volume uplift.** Số units trên mỗi giao dịch khi có promo còn *thấp hơn* lúc bình thường. Giảm giá 11% mà không bán được nhiều hơn.
- Margin bị hy sinh **đồng đều ở cả 5 category**: Medical Devices −8,9pp · OTC −8,7pp · Personal Care −8,4pp · Prescription −9,2pp · Wellness −8,7pp
- Promo share phẳng ở mọi nước (10,2%–11,7%) và mọi loại store (10,2%–11,2%) → promo đang được rải đại trà, không có targeting
- **Margin mất đi ≈ €82.700** (€911k doanh thu promo × 9,1pp)

### Visual

| # | Visual | Field / Metric |
|---|---|---|
| 5.1 | **KPI card row** ⭐ | `Promo Revenue Share %` ⚠️ · `Margin % (Promo)` ⚠️ · `Margin % (Non-Promo)` ⚠️ · `Promo Margin Gap (pp)` ⚠️ · `Margin Foregone` 🆕 |
| 5.2 | **Butterfly / clustered column** ⭐ | X `PromoFlag` · Y `Margin %`, `Avg Unit Price`, `Units per Sale` (3 visual nhỏ đặt hàng ngang) — trọng tâm là `Units per Sale` gần như bằng nhau |
| 5.3 | **Clustered bar** ⭐ | Y `Category` · Legend `PromoFlag` · X `Margin %` → 5 cặp cột, khoảng hở đều nhau ~8,5pp ở cả 5 |
| 5.4 | **Stacked column + line** | X `YearMonth` · Column `Revenue (Promo)` ⚠️ + `Revenue (Non-Promo)` ⚠️ · Line `Promo Revenue Share %` ⚠️ |
| 5.5 | **Scatter** — promo có đáng không | X `Promo Discount Depth %` ⚠️ · Y `Promo Units Uplift %` ⚠️ · Details `Brand` · Size `Revenue (Promo)` ⚠️ · constant line Y = 0 |
| 5.6 | **Matrix** | Rows `Category`→`Brand` · Columns `PromoFlag` · Values `Units Sold`, `Avg Unit Price`, `Margin %` |
| 5.7 | **Text box — Recommendation** | Kết luận + hành động đề xuất (xem dưới) |

> **🆕 Measure cần thêm:**
> ```dax
> Margin Foregone =
> VAR _PromoRev = [Revenue (Promo)]
> VAR _BaseRate = [Margin % (Non-Promo)]
> RETURN _PromoRev * _BaseRate - CALCULATE( [Total Margin], FactSales[PromoFlag] = "Yes" )
> ```

### Khuyến nghị đặt ở 5.7
1. **Dừng promo đại trà** — nó không tạo sản lượng. Thu hồi ước tính **€82,7k margin trên kỳ 2 năm** (≈ €41,4k/năm).
2. **Nếu vẫn chạy promo, dồn vào Wellness & Personal Care** (margin nền 33,5%) thay vì Prescription (21,9%) — nơi 9pp giảm giá ăn gần một nửa lợi nhuận.
3. **Dịch mix về phía Wellness/Personal Care.** Chuyển 5 điểm % doanh thu từ Prescription sang Wellness (chênh 11,66pp margin) ≈ **+€25,2k margin/năm** (€50,3k trên cả kỳ 2 năm).

---

# ↳ Drill-through: Store Detail

Drill-through field: `DimPharmacy[PharmacyName]` — gọi từ map (2.1), scatter (3.2), table (3.5).

| Visual | Field |
|---|---|
| Card header | `PharmacyName`, `City`, `Country`, `PharmacyType`, `StoreSizeBand`, `OpenDate`, `Active Days` 🆕 |
| Line chart | X `YearMonth` · Y `Total Revenue` + line hệ thống để so |
| Bar | Y `Category` · X `Total Revenue` + `Margin %` |
| Card | `Revenue per Active Day` 🆕 · `Rank in Region` ⚠️ · `Margin % vs Region (pp)` ⚠️ |
| Donut | `PromoFlag` × `Total Revenue` |

---

# 🧭 Mạch story khi trình bày

```
Page 1  "Doanh nghiệp €8,6M, +4,4%, margin 28% ổn định.
         Mùa vụ theo tháng gần như không có — nhịp thật nằm ở ngày trong tuần."
   ↓
Page 2  "Một nửa doanh thu đến từ Đức, Pháp, Ý.
         Nhưng margin thì giống hệt nhau ở cả 8 nước — 28,0%."
   ↓         ← câu hỏi bật ra: vậy margin khác nhau ở đâu?
Page 3  "Trước khi kết tội cửa hàng nào, phải chuẩn hoá đã.
         11 store mới mở làm sai lệch mọi bảng xếp hạng.
         Chuẩn hoá xong: nhóm yếu thật là Ba Lan, không phải Vienna."
   ↓
Page 4  "Margin không đến từ nơi chốn — nó đến từ thứ được bán.
         Prescription: 32% doanh thu, margin 21,9%.
         Wellness: 20% doanh thu, margin 33,6%."
   ↓
Page 5  "Và margin bị mất ở một chỗ duy nhất: promotion.
         −9,1pp margin, 0% volume uplift, €83k bốc hơi.
         Đây là việc cần sửa."
```

**Vì sao 5 page chứ không phải 3 hay 8:**
- Không gộp được Page 4 và 5: mỗi trang là một luận điểm độc lập, và promo cần đủ chỗ cho bằng chứng "không có uplift".
- Không tách Q5 (Urban/Rural) thành trang riêng: margin phẳng, mix giống nhau — chỉ còn chuyện quy mô, 2 visual trong Page 3 là đủ.
- Không tách Q10 (map) thành trang riêng: bản đồ là *cách trình bày* câu hỏi địa lý, không phải một câu hỏi riêng.
- Không tách Q9 (contribution) thành trang riêng: waterfall thuộc Page 1, Pareto thuộc Page 2.

---

# 🎛 Slicer bar dùng chung (sync 5 page)

`DimDate[Year]` · `DimPharmacy[Country]` · `DimProduct[Category]` · `DimPharmacy[PharmacyType]` · `DimPharmacy[Store Cohort]`

**Không** sync `FactSales[PromoFlag]` sang Page 5 — trang đó cần cả 2 giá trị để so sánh.

---

# 📋 Checklist bổ sung model trước khi dựng

**Cột mới (Power Query `DimDate`):**
```m
DayOfWeek  = Date.DayOfWeek([Date], Day.Monday)              // 0-6, dùng làm sort-by
DayName    = Date.DayOfWeekName([Date], "en-US")             // Sort by column = DayOfWeek
DayType    = if [DayOfWeek] >= 5 then "Weekend" else "Weekday"   // dùng làm Legend
```

**Measure mới ngoài danh sách trong `business-questions-to-visuals.md`:**
```dax
Active Days            = DISTINCTCOUNT( FactSales[DateKey] )
Revenue per Active Day = DIVIDE( [Total Revenue], [Active Days] )

Margin Foregone =
VAR _PromoRev = [Revenue (Promo)]
VAR _BaseRate = [Margin % (Non-Promo)]
RETURN _PromoRev * _BaseRate - CALCULATE( [Total Margin], FactSales[PromoFlag] = "Yes" )
```

**Sửa trục cố định** ở các visual `Margin %` (1.2, 2.5, 3.6): set Y-axis Start = 0, End = 0.35. Để auto-scale thì Power BI sẽ phóng đại chênh lệch 0,2pp thành một biểu đồ trông rất "có chuyện" — và đó là kết luận sai.
