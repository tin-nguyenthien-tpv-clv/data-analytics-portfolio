# Key Findings at a Glance

Condensed version of README sections 3 (Data Limits) and 4 (Executive Summary). Full narrative, sources and screenshots live in the [README](../README.md).

---

## Data limits - read these before quoting any number

**1. Margin rate is a constant across geography.** 27.84%-28.16% across 8 countries, a 0.32pp spread. Same story across 38 regions, across Urban / Suburban / Rural, and across S / M / L store sizes.
→ Geographic pages answer "who is big", never "who is efficient". No map or heat map is colour-scaled by `Margin %`.

**2. Category mix is uniform in every country, which is what makes margin flat.** Prescription 30.8%-33.6% of revenue everywhere, OTC 20.0%-21.5%, Wellness 19.2%-21.2%. The category margin rates match country to country too (Prescription 21.75%-22.04%, Wellness 33.48%-33.62%).
→ Generator artifact, not market insight.

**3. Eleven of 120 stores opened inside the window, so raw revenue ranks store age.** Vienna HealthPoint #115 opened 2025-12-13 with 10 trading days. Raw spread 102.6x, normalised spread 3.33x.
→ Every store comparison uses `Revenue per Active Day`, with a `Store Cohort` slicer on every page.

**4. Promotion produces no volume response.** Average price falls in all five categories while units per sale barely moves, and 4 of 5 sell fewer units per transaction on promo (Personal Care 9.22 to 8.77 units at EUR 14.57 to EUR 12.44, Prescription the only exception at 4.82 to 4.84). Promo share is 10.16%-11.73% of revenue in every country and costs 8.36-9.18pp of margin in every category.
→ Reads as a random flag with a fixed discount. Treat EUR 82,709 margin foregone as a demonstration of method, not recoverable money.

**5. Monthly seasonality is a calendar-length artifact.** February earns EUR 656,408 against January EUR 693,212 and March EUR 700,991, but has 57 trading days to their 62. Per day it out-earns both (EUR 11,516 vs EUR 11,181 and EUR 11,306). Day-normalised the twelve monthly indices span only 0.95-1.05.
→ The real time signal is weekday vs weekend, which had to be derived (`DayName`, `DayType`).

**Bonus: position does not predict performance.** Across 109 established stores, latitude correlates with revenue per trading day at r = +0.09 (+0.28 excluding Poland), longitude at r = -0.15. The four latitude bands are not monotonic: 48-52 N leads at EUR 213/day, north EUR 197, south EUR 186. Poland is a country effect, not a spatial one.

**Minor:** `DiscontinuedDate` blank for 185 of 220 products. `OpenDate` reaches back to 2010 while facts cover 2024-2025 only.

---

## What the data says

### Scale and time - the network is stable

* EUR 8.63M revenue, EUR 2.42M margin, **28.04%** margin, 445,793 units, 120 pharmacies, 731 days.
* 2024 to 2025: **+4.43%** revenue, margin rate flat (27.97% to 28.11%, +0.13pp).
* Day-normalised monthly index spans only 0.95-1.05. Margin % by month varies 0.41pp.
* **Weekdays EUR 12,471/day vs weekends EUR 10,153/day, an 18.6% gap** - wider than the entire 10.9% month-to-month spread, and the strongest time effect in the data.

### Geography - scale, not efficiency

* Germany 18.2% + France 16.3% + Italy 15.4% = **49.9%** of revenue. Top 10 of 38 regions = **46.9%**.
* Margin: all 8 countries between 27.84% and 28.16%. Nothing to choose between them.
* Revenue per store: Belgium EUR 89,036 down to Poland EUR 54,941, a **1.62x** spread that is pure volume.

### Stores - rankings only mean something after normalisation

* Raw revenue ranks store age. Normalising collapses the spread from **102.6x to 3.33x**.
* Genuine laggards are Polish: Poznan #068 (EUR 95), Warsaw #030 (EUR 100), Katowice #080 (EUR 110), Gdansk #064 (EUR 117), Poznan #025 (EUR 127) against a network median of **EUR 193**. Poland median EUR 143 sits **33%** below the Netherlands.
* Store format moves volume only: Urban EUR 82,506 / Suburban EUR 66,155 / Rural EUR 60,843 per store, all three at 27.98%-28.11% margin. Size L EUR 126,898 vs S EUR 38,713, a **3.28x** gap, identical margin.

### Products - this is where margin actually lives

| Category | Revenue | % of revenue | Margin % |
|---|---|---|---|
| Prescription | EUR 2.80M | 32.4% | **21.92%** |
| OTC | EUR 1.80M | 20.8% | 29.39% |
| Wellness | EUR 1.71M | 19.8% | **33.58%** |
| Personal Care | EUR 1.45M | 16.9% | **33.47%** |
| Medical Devices | EUR 0.87M | 10.1% | **24.96%** |

* Biggest revenue line is the least profitable. **11.66pp** gap from Prescription to Wellness.
* Brands spread wider than categories: **20.37%-35.31%** across 32 brands. AntiBioX leads revenue (EUR 726,405) at 23.39%. ZenHealth earns **35.31%** on 39% of that revenue.
* Generics: 14.6% of revenue at 25.48% margin vs 28.48% branded.
* Quadrants (vs 2,026 units and 28.04% margin per product): 109 Stars, 18 Volume drivers, 21 Niche premium, 72 Low priority. Low priority still holds EUR 3.61M of revenue.

### Promotion - where margin is burned

| Metric | Non-Promo | Promo | Delta |
|---|---|---|---|
| Revenue | EUR 7.72M (89.4%) | EUR 0.91M (10.6%) | - |
| **Margin %** | **29.00%** | **19.92%** | **-9.08pp** |
| Avg unit price | EUR 19.62 | EUR 17.46 | -11.00% |
| **Units per transaction** | **7.195** | **7.023** | **-2.39%** |

* **Zero volume uplift.** An 11% discount buys nothing, units per transaction is marginally lower.
* Uniform sacrifice: Prescription -9.18pp, Medical Devices -8.92pp, OTC -8.73pp, Wellness -8.67pp, Personal Care -8.36pp.
* Blanket application, not targeting: promo is 10.16%-11.73% of revenue in every country.
* **Margin foregone EUR 82,709** over 24 months, about EUR 41,355 per year.

---

## The argument in one line

Revenue is a geography story and margin is not. Margin sits entirely in what gets sold (11.66pp between categories, 14.9pp between brands) and how it gets priced (9.08pp lost to promotion), and the five pages are sequenced to walk from the first claim to the second.
