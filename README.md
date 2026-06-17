# SalesFlow Data Lakehouse
### Databricks Lakehouse Architecture | Bronze · Silver · Gold

A end-to-end data engineering project implementing a **Medallion Architecture** on Databricks,
migrating a transactional sales database from SQL Server on-premise to a modern Lakehouse
with full governance, data quality, and a dimensional model ready for analytics.

---

## Architecture Overview

```
SQL Server On-Premise (CSV Exports)
            │
            ▼
┌───────────────────────┐
│      BRONZE LAYER     │  Raw ingestion — data as-is + control metadata
│   salesflow_dev.bronze│  8 Delta tables
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│      SILVER LAYER     │  Cleaned, validated, standardized
│   salesflow_dev.silver│  8 Delta tables + data quality flags
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐
│       GOLD LAYER      │  Dimensional model + aggregated views
│   salesflow_dev.gold  │  4 dimensions + 1 fact + 3 views
└───────────────────────┘
```

---

## Tech Stack

| Tool | Purpose |
|---|---|
| **Databricks Community Edition** | Compute and notebook environment |
| **Delta Lake** | Storage format across all layers |
| **Unity Catalog** | Data governance and discoverability |
| **PySpark** | Data transformation and ingestion |
| **SQL** | DDL, aggregated views, and validation |
| **Databricks Workflows** | Pipeline orchestration and scheduling |

---

## Project Structure

```
sales_data_lakehouse_ls_mentorship/
├── 00_Setup/
│   ├── 01_create_catalog_structure.sql   # Catalog, schemas, and volume
│   └── 02_create_volume_and_upload.py    # CSV upload validation
│
├── 01_Bronze/                            # Raw ingestion notebooks (1 per table)
│   ├── bronze_categories.ipynb
│   ├── bronze_customers.ipynb
│   ├── bronze_employees.ipynb
│   ├── bronze_orderdetails.ipynb
│   ├── bronze_orders.ipynb
│   ├── bronze_products.ipynb
│   ├── bronze_shippers.ipynb
│   └── bronze_suppliers.ipynb
│
├── 02_Silver/                            # Transformation notebooks (1 per table)
│   ├── silver_categories.ipynb
│   ├── silver_customers.ipynb
│   ├── silver_employees.ipynb
│   ├── silver_orderdetails.ipynb
│   ├── silver_orders.ipynb
│   ├── silver_products.ipynb
│   ├── silver_shippers.ipynb
│   └── silver_suppliers.ipynb
│
├── 03_Gold/                              # Dimensional model and aggregations
│   ├── gold_dim_customer.ipynb
│   ├── gold_dim_customer_scd2.ipynb      # SCD Type 2 implementation
│   ├── gold_dim_date.ipynb               # Programmatically generated date dimension
│   ├── gold_dim_employee.ipynb
│   ├── gold_dim_product.ipynb
│   ├── gold_fact_sales.ipynb
│   └── gold_aggregated_metrics.ipynb     # Pre-aggregated views
│
├── 04_Utils/
│   └── common_functions.ipynb            # Shared utility functions
│
├── job_configuration.md                  # Databricks Job setup and monitoring guide
└── README.md
```

---

## Data Sources

8 CSV files exported from SQL Server on-premise (semicolon-delimited),
loaded via Databricks Volume at `/Volumes/salesflow_dev/bronze/sales_data/`:

| File | Description | Key Column |
|---|---|---|
| `categories.csv` | Product categories | `CategoryID` |
| `customers.csv` | Customer master data | `CustomerID` |
| `employees.csv` | Employee records | `EmployeeID` |
| `products.csv` | Product catalog | `ProductID` |
| `suppliers.csv` | Supplier master data | `SupplierID` |
| `shippers.csv` | Shipping companies | `ShipperID` |
| `orders.csv` | Order headers | `OrderID` |
| `orderdetails.csv` | Order line items | `OrderID` + `ProductID` |

---

## Medallion Architecture

### Bronze — Raw Ingestion
- Data stored **exactly as received** from source CSV files
- No transformations applied
- Three control metadata columns added to every table:

| Column | Description |
|---|---|
| `ingestion_timestamp` | When the record was loaded |
| `source_system` | Always `"SQL_Server_OnPremise"` |
| `file_name` | Source CSV filename for traceability |

### Silver — Curated Layer
Applies business rules and data quality logic on top of Bronze:

- **Deduplication** by primary key
- **String cleaning** — trim whitespace, title case normalization
- **Phone standardization** — remove special characters, validate length
- **Null filling** — business-defined defaults per column
- **Date parsing** — explicit format handling for SQL Server exports (`yyyy/MM/dd`)
- **Derived columns** — `is_available`, `price_category`, `is_shipped`, `days_to_ship`, `line_total`
- **Data quality flag** — every table gets a `data_quality_status` column (`VALID` / `INVALID`)

### Gold — Analytical Layer
Star schema dimensional model for BI consumption:

**Dimensions:**

| Table | Description | Surrogate Key |
|---|---|---|
| `dim_customer` | Customer dimension | MD5 hash of `CustomerID` |
| `dim_customer_scd2` | Customer with full history (SCD Type 2) | MD5 hash of `CustomerID` |
| `dim_product` | Product + category (JOIN) | MD5 hash of `ProductID` |
| `dim_employee` | Employee dimension | MD5 hash of `EmployeeID` |
| `dim_date` | Date dimension (2020–2030) | Integer `YYYYMMDD` |

**Fact table:**

| Table | Grain | Foreign Keys |
|---|---|---|
| `fact_sales` | One row per order line | `date_key`, `customer_key`, `product_key`, `employee_key` |

**Pre-aggregated views:**

| View | Grain |
|---|---|
| `sales_by_month` | Revenue, orders, and quantity per year/month |
| `sales_by_category` | Revenue and quantity per product category |
| `sales_by_country` | Revenue, orders, and customers per country |

---

## Utility Functions

Shared functions in `04_Utils/common_functions.ipynb`, imported via `%run` in all Silver and Gold notebooks:

| Function | Purpose | Used In |
|---|---|---|
| `clean_string_column(df, col)` | Trim whitespace + title case, preserve nulls | Silver — all tables |
| `standardize_phone(df, col)` | Remove special chars, validate 7+ digits | Silver — customers, suppliers, shippers |
| `add_quality_flag(df, required_cols)` | Add `data_quality_status` based on null checks | Silver — all tables |
| `add_surrogate_key(df, table, key_cols)` | MD5 hash surrogate key | Gold — all dimensions and fact |

---

## Pipeline Orchestration

The full pipeline runs as a Databricks Job (`Sales_Lakehouse_Daily_Pipeline`)
with 24 tasks organized in 5 sequential stages:

```
Bronze (parallel) → Silver (parallel) → Gold Dimensions (parallel) → Gold Facts → Aggregated Metrics
```

- **Schedule:** Daily at 02:00 AM UTC
- **Retry policy:** 2 retries per task, 5-minute interval
- **Typical duration:** ~15 minutes

See [`job_configuration.md`](./job_configuration.md) for full task details, dependency diagram,
and monitoring guide.

---

## Extra: SCD Type 2

`gold_dim_customer_scd2` implements **Slowly Changing Dimension Type 2** for the customer dimension,
preserving full history of attribute changes:

| Column | Description |
|---|---|
| `effective_date` | When this version became active |
| `end_date` | When this version was superseded (`NULL` if current) |
| `is_current` | `TRUE` for the active version |

The merge logic uses a two-step Python/Delta approach:
1. Classify incoming records as **new** or **changed**
2. Close outdated records (`is_current = FALSE`, `end_date = today`)
3. Insert new versions

---

## How to Run

### First time setup
1. Upload the 8 CSV files to `/Volumes/salesflow_dev/bronze/sales_data/`
2. Run `00_Setup/01_create_catalog_structure.sql` to create the catalog structure
3. Run each Bronze notebook to ingest raw data
4. Run each Silver notebook to apply transformations
5. Run Gold notebooks in order: dimensions → fact → aggregated metrics

### Daily runs
Trigger the `Sales_Lakehouse_Daily_Pipeline` job manually or let the schedule run at 02:00 AM UTC.

### Import to Databricks from GitHub
1. Go to **User Settings > Linked Accounts** and connect GitHub
2. Right-click your workspace folder → **Git** → **Add folder to Git**
3. Point to this repository and sync

---

## Author

**Bruno Oliveira**
Category Insights Executive | Data Engineer
[Dashboard Labs](https://dashboardlabs.ca)
