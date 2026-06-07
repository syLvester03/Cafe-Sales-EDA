# Cafe Sales EDA

Exploratory data analysis of a deliberately "dirty" cafe sales dataset. The project covers end-to-end data cleaning followed by a revenue-focused analysis to surface actionable recommendations for 2024.

**Dataset source:** [Cafe Sales — Dirty Data for Cleaning Training](https://www.kaggle.com/datasets/ahmedmohamed2003/cafe-sales-dirty-data-for-cleaning-training) (Kaggle)

---

## Dataset

10,000 rows of synthetic cafe transaction data with intentional quality issues: missing values, `ERROR`/`UNKNOWN` placeholders, mixed data types, and inconsistent entries.

| Column | Type | Description |
|---|---|---|
| `Transaction ID` | string | Unique transaction identifier (format: `TXN_XXXXXXX`) |
| `Item` | string | Product purchased (Coffee, Tea, Juice, Smoothie, Cake, Cookie, Sandwich, Salad) |
| `Quantity` | int | Number of units sold |
| `Price Per Unit` | float | Unit price (£) |
| `Total Spent` | float | Transaction total — equals `Quantity × Price Per Unit` |
| `Payment Method` | string | Cash, Credit Card, or Digital Wallet |
| `Location` | string | In-Store or Takeaway |
| `Transaction Date` | datetime | Date of transaction (all within 2023) |

---

## Project structure

```
Cafe-Sales-EDA/
├── Cafe_sales.csv                        # Raw dataset (as downloaded)
├── Clean_Cafe_sales.csv                  # Cleaned dataset (all rows retained, unusable flagged)
├── Clean_Cafe_sales_without_unusable.csv # Cleaned dataset (26 unusable rows removed)
├── Cleaning.ipynb                        # Notebook 1: data cleaning pipeline
└── Analysis.ipynb                        # Notebook 2: EDA and business analysis
```

---

## Notebook 1 — Data cleaning (`Cleaning.ipynb`)

**Goal:** Resolve all data quality issues while preserving as many rows as possible.

**Steps covered:**

1. **Type conversion** — `Quantity` → `Int64`, `Price Per Unit` / `Total Spent` → `float64`, `Transaction Date` → `datetime`
2. **Invalid value handling** — replace `ERROR` and `UNKNOWN` strings with `NaN` across all columns
3. **Duplicate check** — no duplicate `Transaction ID`s; apparent row duplicates deemed valid (no time or customer data to confirm)
4. **String normalisation** — strip whitespace, apply `.title()` to categorical columns
5. **Price Per Unit imputation** — looked up each item's fixed price and filled missing values; used `Total Spent / Quantity` where item was unknown
6. **Item imputation** — resolved missing/invalid items by matching on `Price Per Unit`; where two items share a price, filled with the higher-selling one
7. **Quantity / Total Spent imputation** — calculated from the other two numeric columns where one was missing
8. **Unusable row flagging** — 26 rows missing two or more of the three numeric columns were flagged `IsUnusable` rather than dropped, preserving them for non-numeric analyses
9. **Payment Method / Location** — no recoverable pattern in missing values; replaced with `"Unavailable"`
10. **Consistency assertion** — validated `Total Spent ≈ Quantity × Price Per Unit` (within 1% tolerance) across all non-unusable rows

**Null reduction summary:**

| Column | Before | After (usable rows) |
|---|---|---|
| Item | 333 | 0 |
| Quantity | 479 | 0 |
| Price Per Unit | 533 | 0 |
| Total Spent | 502 | 0 |
| Payment Method | 2,579 | 0 (marked Unavailable) |
| Location | 3,265 | 0 (marked Unavailable) |
| Transaction Date | 460 | 460 (unresolvable) |

---

## Notebook 2 — EDA & analysis (`Analysis.ipynb`)

**Central question:** *What should the cafe prioritise to grow revenue in 2024?*

**Questions answered:**

1. **Which items drive the most revenue vs. the most transactions — are they the same?**
   - Revenue is almost entirely determined by unit price, not transaction volume.
   - Top 4 items (Salad, Sandwich, Smoothie, Juice) generate ~70% of total revenue.

2. **Are there items that sell well in volume but contribute little to revenue?**
   - Coffee ranks 3rd in transactions but 6th in revenue (8.7% contribution).
   - Cookie and Tea show the same pattern — high volume, low revenue per transaction.

3. **Does purchasing behaviour differ between In-Store and Takeaway customers?**
   - No meaningful difference in item mix, quantity per transaction, or average spend.
   - The only marginal difference: Coffee is ordered ~0.7% more in Takeaway.

4. **Is there a day-of-week or monthly pattern in sales?**
   - Monthly revenue is stable throughout the year; February's dip is a calendar artifact (28 days, not a demand signal).
   - Daily transaction volume is consistent across the week (~1,350/day); Tuesday is marginally lower but not significantly so.

5. **Which payment method is associated with higher average spend?**
   - No difference — average spend is ~£9 regardless of payment method.

**Key recommendation:** Price revision on Coffee, Cookie, and Tea is the clearest lever to grow revenue. These items each have transaction counts comparable to the top earners but contribute disproportionately less because of their lower unit prices.


## Limitations

- Payment Method and Location are missing for ~31% of transactions (marked `"Unavailable"`) — conclusions drawn from those columns are directional at best.
- No customer identifiers in the dataset — repeat purchase behaviour and customer segmentation are not possible.
- Transaction Date is missing for 460 rows (~4.6%) and cannot be recovered.
- The dataset is synthetic; price elasticity or real-world demand signals should not be inferred from it.
