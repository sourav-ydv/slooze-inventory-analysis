# Slooze Inventory & Sales Analysis

Submission for the Slooze take-home challenge.

The data is from a retail wine & spirits company — around 3.5 million rows across sales, purchases, inventory and pricing for 2016. Took me a bit to get through all of it but managed to cover everything in the task list plus a few extra things.

---

## Getting the data

Data is available at the Kaggle link shared in the challenge. Download and place all 6 CSV files in the same folder as the notebooks before running anything.

Expected files:
- `SalesFINAL12312016.csv`
- `PurchasesFINAL12312016.csv`
- `BegInvFINAL12312016.csv`
- `EndInvFINAL12312016.csv`
- `InvoicePurchases12312016.csv`
- `2017PurchasePricesDec.csv`

---

## Setup

```bash
pip install pandas numpy matplotlib seaborn prophet statsmodels scikit-learn jupyter
```

Ran this on Python 3.10. Prophet can be a bit annoying to install on Windows — if it throws errors try installing through conda instead of pip.

---

## Notebooks

Run them in order. Task 2 exports a CSV that Task 3 through 6 depend on.

**Task_1.ipynb**
Data loading, cleaning and exploratory analysis. Fixed some null cities in the ending inventory file, stripped whitespace from vendor names, parsed the dates (sales uses M/D/YYYY, everything else is YYYY-MM-DD). Then looked at monthly revenue trends, top products, top stores, vendor spend distribution, day-of-week patterns and gross margin distribution. Also merged purchase prices into the sales data to get per-transaction profit estimates.

**Task_2.ipynb**
ABC classification — sorted all products by total revenue and split into A (top 70%), B (next 20%), C (bottom 10%). Plotted the Pareto curve and broke down the tiers by store and product category. Also checked which A-tier items ended 2016 with zero stock since those are direct missed sales. Saves `abc_classified_products.csv` at the end.

**Task_3.ipynb**
Economic Order Quantity and reorder points for each product. Used 95% service level for safety stock, factoring in both demand variability and lead time variability. Then compared current ending stock against the calculated reorder points to see which A-tier items are already understocked going into 2017. Saves `eoq_reorder_table.csv`.

**Task_4.ipynb**
Analysed how long it takes from placing a PO to actually receiving goods, broken down by vendor. Looked at variability (std dev) not just averages since a vendor with inconsistent lead times is harder to plan around than a slow but predictable one. Also plotted monthly lead time trend to see if things got better or worse over the year.

**Task_5.ipynb**
Quick note — the sales file caps out at 1,048,575 rows which is the Excel limit, so it only has January and February data. Caught this when the date range looked wrong. Switched to using purchase receipts (ReceivingDate) as the demand signal instead since that file has the full year.

Used Prophet for daily forecasting and Holt-Winters on the monthly aggregates. Also ran individual forecasts for the top 5 A-tier products to give a concrete Q1 2017 number. Compared both models against a 30-day rolling average baseline.

**Task_6.ipynb**
A few extra things that felt worth looking into — gross margin breakdown by product and vendor, inventory shrinkage (beginning stock + purchases - sales - ending stock), vendor concentration risk, store efficiency measured as revenue per unique SKU stocked, and excise tax as a percentage of revenue by category. Summary of recommendations at the end.

---

## Outputs

Running the notebooks generates:

- `outputs/abc_classified_products.csv` — full product list with ABC tiers
- `outputs/eoq_reorder_table.csv` — EOQ, safety stock and reorder point per product
- `plots/` — all charts saved as PNGs

---

## Assumptions

EOQ calculation uses ordering cost of $50 per order and holding cost of 20% of unit cost per year. These are standard retail ballpark figures — actual values would need input from the business to be accurate, but the model works the same way once you plug in real numbers.

The `Approval` column in the invoices file is about 93% null so I ignored it. Didn't seem useful.
