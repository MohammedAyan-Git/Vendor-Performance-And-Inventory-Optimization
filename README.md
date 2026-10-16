# 📦 Vendor Performance and Inventory Optimization

An end-to-end data analysis project that turns millions of raw purchase, sales, invoice and inventory records into one decision-ready view of **vendor performance, procurement risk and locked inventory capital**, built with **SQL, Python (Pandas, Seaborn, SciPy) and Power BI**.

📄 **Full project report (PDF):** [report/Vendor_Performance_Project_Report.pdf](report/Vendor_Performance_Project_Report.pdf)

![Dashboard preview](images/dashboard_preview.png)

---

## 📌 Table of Contents
- [Business Problem](#-business-problem)
- [Business Questions](#-business-questions)
- [Dataset](#-dataset)
- [Tech Stack](#-tech-stack)
- [Project Workflow](#-project-workflow)
- [Key Metrics](#-key-metrics)
- [Key Findings](#-key-findings)
- [Visualizations](#-visualizations)
- [Recommendations](#-recommendations)
- [Repository Structure](#-repository-structure)
- [How to Run](#-how-to-run)
- [Limitations](#-limitations)
- [Future Work](#-future-work)
- [Author](#-author)

---

## 🎯 Business Problem

A retailer that buys from hundreds of vendors needs to know which vendor relationships are profitable, which create risk, and where cash is tied up in unsold stock. Purchases, invoices, prices and sales sit in separate tables, so there was no single view of vendor performance. This project builds that view and uses it to guide purchasing, pricing and inventory decisions.

## ❓ Business Questions

1. Which brands need **promotional or pricing adjustments** (low sales but high margin)?
2. Which **vendors and brands** deliver the highest sales?
3. How **dependent is procurement** on the top vendors?
4. Does **bulk purchasing** reduce unit price?
5. Which vendors have **low inventory turnover**, and how much capital is locked in unsold stock?
6. Do top and low performers differ **significantly in profit margin**?

## 🗃 Dataset

Raw data was loaded into a SQLite database (`inventory.db`):

| Table | Records | Contents |
|---|---|---|
| `sales` | 12,825,363 | Store-level sales: quantity, dollars, price, date, excise tax, vendor |
| `purchases` | 2,372,474 | Purchase lines: quantity, dollars, purchase price, vendor |
| `purchase_prices` | 12,261 | Brand price, purchase price, volume, classification |
| `vendor_invoice` | 5,543 | Invoices: PO and pay dates, quantity, dollars, freight, approval |
| `begin_inventory` | 206,529 | Opening stock by store and brand |
| `end_inventory` | 224,489 | Closing stock by store and brand |
| `vendor_sales_summary` | 10,692 | **Created in this project**: one row per vendor and brand |

> The database and raw files are not included in this repository because of their size. **Dataset source:** _add the link here_.

## 🛠 Tech Stack

| Area | Tools |
|---|---|
| Storage and querying | SQLite, SQL (joins, CTEs, aggregations) |
| Analysis | Python, Pandas, NumPy, SciPy |
| Visualization | Matplotlib, Seaborn, Power BI |
| Workflow | Jupyter Notebook, Python logging, Git and GitHub |

## 🔄 Project Workflow

```
Raw tables ─► SQLite ─► SQL CTE summary ─► Pandas cleaning & metrics ─► EDA & statistics ─► Power BI dashboard ─► Recommendations
```

1. **Ingest** raw files into SQLite with a small ingestion module (with logging).
2. **Consolidate** with SQL: CTEs aggregate freight per vendor, purchases per vendor-brand and sales per vendor-brand, then join them into `vendor_sales_summary`. Pre-aggregating avoids re-running heavy joins on millions of rows each time the dashboard refreshes.
3. **Clean**: fill 178 missing sales rows with 0, trim vendor names, convert data types, then filter to records with positive gross profit, margin and sales quantity (**8,564 of 10,692 records**).
4. **Explore** distributions, outliers and correlations.
5. **Analyse** the six business questions, including confidence intervals and a hypothesis test.
6. **Visualize** in a Power BI dashboard.

## 📐 Key Metrics

| Metric | Definition |
|---|---|
| Gross Profit | Total Sales Dollars - Total Purchase Dollars |
| Profit Margin (%) | Gross Profit / Total Purchase Dollars x 100 |
| Stock Turnover | Total Sales Quantity / Total Purchase Quantity |
| Sales-to-Purchase Ratio | Total Sales Dollars / Total Purchase Dollars |
| Purchase Contribution (%) | Vendor purchase dollars / total purchase dollars x 100 |
| Unsold Inventory Value | (Purchase Quantity - Sales Quantity) x Purchase Price |

## 🔍 Key Findings

| KPI | Value |
|---|---|
| Total sales | **$441.41M** |
| Total purchases | **$307.34M** |
| Gross profit | **$134.07M** |
| Unsold capital (net) | **$2.71M** |
| Top-10 vendor share of purchases | **65.69%** |

- **Purchasing is concentrated.** The top 10 vendors account for 65.69% of purchase spend; **Diageo North America alone is 16.30%** ($50.10M).
- **Top sellers.** Diageo leads vendor sales ($67.99M), followed by Martignetti ($39.33M) and Pernod Ricard ($32.06M). Top brands are Jack Daniels No 7 Black ($7.96M), Tito's Handmade Vodka ($7.40M) and Grey Goose Vodka ($7.21M).
- **Slow movers.** Ten vendors have stock turnover below 1; the lowest is **Alisa Carr Beverages (0.615)**, then Highland Wine Merchants (0.708) and Park Street Imports (0.751).
- **Locked capital.** The largest unsold balances are Diageo ($722K), Jim Beam ($555K) and Pernod Ricard ($471K). The $2.71M total is a net figure.
- **Bulk buying lowers unit cost.** Average unit price is $10.78 for large orders versus $39.07 for small orders (about 72% lower), partly reflecting product mix.
- **Low sellers earn higher margins.** Mean margin is **41.57%** (95% CI 40.50% to 42.64%) for the lowest-selling quartile versus **31.18%** (30.74% to 31.61%) for the top quartile. A Welch t-test gives **t = -17.67, p < 0.0001**, so the difference is statistically significant.
- **Pricing and promotion candidates.** Brands below **$560** in sales (15th percentile) but above **64.97%** margin (85th percentile).
- **Data quality.** Gross profit goes as low as -$52,002.78, so some items sell below cost, and freight cost ranges from $0.09 to $257,032.07.

## 📊 Visualizations

### Top vendors and brands by sales
![Top vendors and brands](<img width="985" height="652" alt="image" src="https://github.com/user-attachments/assets/3851feb8-6521-4afc-9c03-ef55cbabdfd2" />
)

### Vendor purchase contribution (Pareto)
![Pareto](images/pareto_vendor_purchase_contribution.png)

### Brands for promotional or pricing adjustment
![Low sales high margin](images/brands_low_sales_high_margin.png)

### Bulk purchasing and unit price
![Bulk purchase](images/bulk_purchase_unit_price.png)

### Profit margin: top vs low performers
![Confidence intervals](images/profit_margin_confidence_intervals.png)

### Correlation heatmap
![Correlation heatmap](images/correlation_heatmap.png)

<details>
<summary>More charts (EDA)</summary>

![Distributions before filtering](images/eda_distributions_raw.png)
![Outlier box plots](images/eda_boxplots_outliers.png)
![Distributions after filtering](images/eda_distributions_clean.png)
![Count plots](images/eda_countplots_top_categories.png)
![Purchase share donut](images/vendor_purchase_share_donut.png)

</details>

## 💡 Recommendations

1. **Reduce vendor dependence:** qualify alternative suppliers for top categories and negotiate volume terms with the largest vendors.
2. **Right-size inventory:** cut reorder quantities for vendors with turnover below 1 and run promotions on their excess stock.
3. **Promote or re-price** low-sales, high-margin brands to lift volume.
4. **Use bulk purchasing selectively** where demand is steady, and avoid over-buying slow movers.
5. **Review loss-making items** and consider selective price increases or bundles for high-volume vendors with thinner margins.
6. **Investigate freight cost outliers** by vendor and shipment size.

## 🗂 Repository Structure

```
.
├── README.md
├── notebooks/
│   ├── 01_EDA_Vendor_Summary.ipynb          # table exploration, SQL summary table creation
│   └── 02_Vendor_Performance_Analysis.ipynb # EDA, business questions, statistics
├── scripts/
│   ├── ingestion_db.py                      # loads raw files into SQLite
│   └── get_vendor_summary.py                # builds vendor_sales_summary
├── dashboard/
│   └── Vendor_Performance_Dashboard.pbix    # Power BI file (add yours)
├── images/                                  # charts used in this README
└── report/
    └── Vendor_Performance_Project_Report.pdf
```

## ▶️ How to Run

1. **Clone the repository**
   ```bash
   git clone https://github.com/MohammedAyan-Git/Vendor-Performance-and-Inventory-Optimization.git
   cd Vendor-Performance-and-Inventory-Optimization
   ```
2. **Install dependencies**
   ```bash
   pip install pandas numpy matplotlib seaborn scipy jupyter
   ```
3. **Add the raw data** to a `data/` folder and build the database by running the ingestion script.
4. **Run `01_EDA_Vendor_Summary.ipynb`** to create the `vendor_sales_summary` table in `inventory.db`.
5. **Run `02_Vendor_Performance_Analysis.ipynb`** for the cleaning, charts and statistics.
6. **Open the `.pbix` file** in Power BI Desktop to explore the dashboard.

## ⚠️ Limitations

- Single-period snapshot, so seasonality and trends are not captured.
- Several analyses use only items with positive profit and sales, so averages describe profitable, selling products.
- Profit margin is calculated on purchase cost, which gives higher values than margin on sales.
- The bulk-price gap mixes volume discounts with product mix.
- Unsold capital is a net figure; vendors that sold down opening stock offset others' positive balances.
- Results show patterns, not proven causes.

## 🚀 Future Work

- Add time-series analysis and demand forecasting for reorder decisions.
- Predict slow-moving items before they accumulate.
- Segment vendors and brands with ABC analysis or clustering.
- Automate the pipeline so the dashboard refreshes on a schedule.

## 👤 Author

**Mohammed Ayan**
B.Tech in Artificial Intelligence and Machine Learning

[LinkedIn](https://www.linkedin.com/in/mohammedayan-in/) · [GitHub](https://github.com/MohammedAyan-Git)
