# 📊 Vendor Performance Analysis — Inventory & Liquor Distribution Data

> An end-to-end data analytics project that ingests raw inventory and sales data into a SQLite database, builds a consolidated vendor summary, and performs deep analysis on purchasing efficiency, profitability, stock turnover, and capital lock-in — all using Python.

---

## 🔍 Project Overview

This project analyzes the vendor performance of a **liquor distribution business** using real transactional data spanning purchases, sales, invoices, inventory, and pricing. The raw CSVs (spanning millions of rows) are ingested into a local SQLite database, transformed into a consolidated summary table, and then analyzed across multiple dimensions to surface actionable business insights.

The entire pipeline is built in Python using Jupyter Notebooks, with a modular design separating ingestion, transformation, and analysis responsibilities.

---

## 🎯 Objectives

- Ingest and store 7 million+ raw records from CSV files into a structured SQLite database
- Build a consolidated `vendor_sales_summary` table by joining purchases, sales, invoice, and pricing data
- Engineer key performance metrics: Gross Profit, Profit Margin, Stock Turnover, Sales-to-Purchase Ratio
- Identify top and low-performing vendors by sales and purchase contribution
- Apply the **Pareto (80/20) Principle** to determine procurement dependency on key vendors
- Detect slow-moving inventory and estimate **capital locked in unsold stock**
- Analyze whether **bulk purchasing reduces unit cost**
- Identify brands with **high profit margin but low sales** (candidates for promotion/pricing adjustment)
- Perform **statistical analysis** using confidence intervals and hypothesis testing
- Visualize all findings with clean, well-labeled charts

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Python 3.10 | Core language |
| Pandas | Data manipulation and SQL querying |
| NumPy | Numerical operations |
| Matplotlib / Seaborn | Visualizations |
| SciPy | Statistical analysis (t-tests, confidence intervals) |
| SQLite3 | Local database storage |
| SQLAlchemy | Database engine for ingestion |
| Jupyter Notebook | Interactive development |
| Logging | Execution tracking across pipeline stages |

---

## 📁 Project Structure

```
vendor-performance-analysis/
│
├── data/                              # Raw CSV files (not committed — too large)
│   ├── begin_inventory.csv
│   ├── end_inventory.csv
│   ├── purchases.csv
│   ├── purchase_prices.csv
│   ├── sales.csv
│   └── vendor_invoice.csv
│
├── logs/                              # Auto-generated logs
│   ├── ingestion_db.logs
│   └── get_vendor_summary.log
│
├── inventory.db                       # SQLite database (auto-generated)
│
├── ingestion_db.ipynb                 # Step 1: Load CSVs → SQLite database
├── Exploratory_data_analysis.ipynb    # Step 2: EDA — explore all 6 tables
├── get_vendor_summary.ipynb           # Step 3: Build & store vendor_sales_summary
├── Vendor_Performance_Analysis.ipynb  # Step 4: Full analysis & visualizations
│
└── README.md
```

---

## 🗄️ Database Schema

The SQLite database (`inventory.db`) is built from 6 raw tables:

| Table | Rows | Description |
|---|---|---|
| `begin_inventory` | 206,529 | Stock on hand at the start of the period (Jan 1, 2024) |
| `end_inventory` | 224,489 | Stock on hand at the end of the period (Dec 31, 2024) |
| `purchases` | 2,372,474 | All purchase transactions — vendor, brand, quantity, price paid |
| `purchase_prices` | 12,261 | Reference table — product-wise actual retail and purchase prices |
| `sales` | 4,555,004 | All sales transactions — brand, quantity sold, sales price, excise tax |
| `vendor_invoice` | 5,543 | Invoice-level summaries with freight cost per vendor per PO |

The consolidated output table `vendor_sales_summary` (10,692 rows) merges all of the above into a single analysis-ready view using a 3-way CTE SQL join.

---

## ⚙️ Pipeline: How It Works

### Step 1 — `ingestion_db.ipynb`
Reads every CSV from the `data/` folder and loads it into the SQLite database using `pandas.to_sql()`. Includes a `logs/` directory check, timing, and full logging of each file ingested.

### Step 2 — `Exploratory_data_analysis.ipynb`
Connects to the database and previews all 6 tables — row counts, column structures, and sample data. Also builds intermediate SQL aggregations (freight summary, purchase summary, sales summary) to validate data before merging.

### Step 3 — `get_vendor_summary.ipynb`
Runs a **3-CTE SQL query** to produce the final `vendor_sales_summary` table:
- `FreightSummary` — total freight cost per vendor from `vendor_invoice`
- `PurchaseSummary` — total quantity and spend per vendor-brand from `purchases` joined with `purchase_prices`
- `SalesSummary` — total revenue, quantity sold, and excise tax per vendor-brand from `sales`

After the join, the following **feature-engineered columns** are added:

| Column | Formula | Meaning |
|---|---|---|
| `GrossProfit` | `TotalSalesDollars - TotalPurchaseDollars` | Profit after cost of goods |
| `ProfitMargin` | `(GrossProfit / TotalSalesDollars) × 100` | % of revenue retained as profit |
| `StockTurnover` | `TotalSalesQuantity / TotalPurchaseQuantity` | How quickly purchased stock was sold |
| `SalesToPurchaseRatio` | `TotalSalesDollars / TotalPurchaseDollars` | Revenue generated per $ spent |

Missing values (from products with no sales) are filled with 0. The result is written back to the database.

### Step 4 — `Vendor_Performance_Analysis.ipynb`
Full analysis with the following sections:

---

## 📈 Analysis Sections & Key Findings

### 1. Descriptive Statistics & Outlier Detection
- Distribution plots and box plots for all numerical columns
- Some `GrossProfit` values as low as **−$52,002** — products sold below cost
- `FreightCost` ranges from **$0.09 to $257,032** — massive logistics variance
- `StockTurnover` ranges from 0 to **274.5** — extreme range between dead stock and fast movers
- Inconsistent records filtered: only rows with `GrossProfit > 0`, `ProfitMargin > 0`, and `TotalSalesQuantity > 0` are retained for analysis

### 2. Correlation Analysis
- Near-perfect correlation (0.999) between `TotalPurchaseQuantity` and `TotalSalesQuantity` — inventory is well-managed overall
- Weak correlation between `PurchasePrice` and `GrossProfit` (-0.016) — price alone doesn't drive profit
- Negative correlation between `ProfitMargin` and `TotalSalesPrice` (-0.179) — higher-priced products face margin pressure

### 3. Top Vendors & Brands by Sales
- Identified the **Top 10 vendors and brands** by total sales dollars using grouped bar charts
- Dollar values formatted dynamically (e.g., `$1.56M`, `$450K`) for readability

### 4. Vendor Procurement Concentration (Pareto Analysis)
- Calculated each vendor's **Purchase Contribution %** to total procurement spend
- **Top 10 vendors account for ~72% of total purchases** — significant procurement dependency
- Visualized as a **Pareto chart** (bar + cumulative line) and a **donut chart** showing top-10 vs. rest-of-market share

### 5. Bulk Purchasing & Unit Cost Analysis
- Purchase quantities binned into `small`, `medium`, `large` order sizes using `pd.qcut()`
- **Large orders have meaningfully lower unit purchase prices** — bulk discounts are real and quantifiable

### 6. Slow-Moving Inventory Detection
- Vendors with `StockTurnover < 1` identified — these vendors' products are selling slower than the purchase rate
- Top 10 slowest-turning vendors displayed — candidates for reduced ordering or markdown strategies

### 7. Capital Locked in Unsold Inventory
- Calculated `UnsoldQuantity = (TotalPurchaseQuantity - TotalSalesQuantity).clip(lower=0)`
- `InventoryValue = UnsoldQuantity × UnitPurchasePrice`
- Identified vendors with the **highest capital tied up in unsold stock** — directly informing cash flow decisions

### 8. Brands Needing Promotion — High Margin, Low Sales
- Brands in the **bottom 15th percentile of sales** but **top 85th percentile of profit margin** flagged
- These are high-potential products underperforming in revenue — prime candidates for pricing adjustments or marketing campaigns
- Visualized on a scatter plot with threshold lines

### 9. Statistical Analysis — Confidence Intervals & Hypothesis Testing
- **95% Confidence Intervals** computed for profit margins of top vs. low-performing vendors
- **t-test** (`ttest_ind`) used to determine if the difference in profit margins is statistically significant
- Results interpreted to validate whether performance differences are real or due to chance

---

## 🚀 How to Run

**1. Clone the repository:**
```bash
git clone https://github.com/your-username/vendor-performance-analysis.git
cd vendor-performance-analysis
```

**2. Install dependencies:**
```bash
pip install pandas numpy matplotlib seaborn scipy sqlalchemy jupyter
```

**3. Add your data:**  
Place the raw CSV files inside a `data/` folder in the project root. The expected files are:
`begin_inventory.csv`, `end_inventory.csv`, `purchases.csv`, `purchase_prices.csv`, `sales.csv`, `vendor_invoice.csv`

**4. Run the notebooks in order:**
```
1. ingestion_db.ipynb           → builds inventory.db
2. Exploratory_data_analysis.ipynb → explore the raw tables
3. get_vendor_summary.ipynb     → creates vendor_sales_summary table
4. Vendor_Performance_Analysis.ipynb → full analysis & charts
```

**5. Open Jupyter:**
```bash
jupyter notebook
```

Logs will be auto-written to `logs/ingestion_db.logs` and `logs/get_vendor_summary.log`.

---

## 📌 Future Improvements

- [ ] Build an interactive dashboard using **Streamlit** or **Power BI**
- [ ] Automate the full pipeline using a task scheduler (cron / Airflow)
- [ ] Add a **vendor scoring system** combining profit margin, turnover, and procurement share
- [ ] Extend the analysis to **store-level** performance using the `Store` and `City` fields
- [ ] Integrate real-time data feeds to replace static CSV ingestion

---

## 👨‍💻 Author
Sumran Harchirkar
sumranharchirkar58@gmail.com

