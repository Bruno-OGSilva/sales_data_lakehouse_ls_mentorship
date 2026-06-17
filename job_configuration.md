# Job Configuration — Sales_Lakehouse_Daily_Pipeline
## SalesFlow Data Lakehouse | Phase 6: Orchestration

This document describes the Databricks Job configuration for the daily pipeline
that orchestrates the full Bronze → Silver → Gold execution.

---

## Job Overview

| Setting | Value |
|---|---|
| **Job Name** | `Sales_Lakehouse_Daily_Pipeline` |
| **Platform** | Databricks Community Edition |
| **Schedule** | Daily at 02:00 AM UTC |
| **Retries** | 2 attempts, 5-minute interval |
| **Total Tasks** | 24 notebooks |
| **Typical Duration** | ~15 minutes |

---

## Task Dependency Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    BRONZE LAYER                         │
│                  (8 tasks, parallel)                    │
│                                                         │
│  bronze_layer_categories    bronze_layer_orders         │
│  bronze_layer_customers     bronze_layer_orderdetails   │
│  bronze_layer_employees     bronze_layer_shippers       │
│  bronze_layer_products      bronze_layer_suppliers      │
└────────────────────────┬────────────────────────────────┘
                         │ depends_on
┌────────────────────────▼────────────────────────────────┐
│                    SILVER LAYER                         │
│                  (8 tasks, parallel)                    │
│                                                         │
│  silver_layer_categories    silver_layer_orders         │
│  silver_layer_customers     silver_layer_orderdetails   │
│  silver_layer_employees     silver_layer_shippers       │
│  silver_layer_products      silver_layer_suppliers      │
└────────────────────────┬────────────────────────────────┘
                         │ depends_on
┌────────────────────────▼────────────────────────────────┐
│                   GOLD DIMENSIONS                       │
│                  (5 tasks, parallel)                    │
│                                                         │
│  gold_layer_dim_customer    gold_layer_dim_date         │
│  gold_layer_dim_customer_scd2                           │
│  gold_layer_dim_employee    gold_layer_dim_product      │
└────────────────────────┬────────────────────────────────┘
                         │ depends_on
┌────────────────────────▼────────────────────────────────┐
│                    GOLD FACTS                           │
│                   (1 task)                              │
│                                                         │
│                gold_layer_fact_sales                    │
└────────────────────────┬────────────────────────────────┘
                         │ depends_on
┌────────────────────────▼────────────────────────────────┐
│               GOLD AGGREGATED METRICS                   │
│                   (1 task)                              │
│                                                         │
│             gold_layer_aggregated_metrics               │
└─────────────────────────────────────────────────────────┘
```

---

## Task Configuration

### Bronze Layer (8 tasks — run in parallel)

All Bronze tasks share the same configuration pattern:

| Setting | Value |
|---|---|
| **Depends On** | *(none — first stage)* |
| **Type** | Notebook |
| **Retry policy** | 2 retries, 5-minute interval |
| **Notebook path** | `/Workspace/Users/<email>/sales_data_lakehouse_ls_mentorship/01_Bronze/<notebook>` |

| Task Name | Notebook |
|---|---|
| `bronze_layer_categories` | `bronze_categories` |
| `bronze_layer_customers` | `bronze_customers` |
| `bronze_layer_employees` | `bronze_employees` |
| `bronze_layer_products` | `bronze_products` |
| `bronze_layer_suppliers` | `bronze_suppliers` |
| `bronze_layer_shippers` | `bronze_shippers` |
| `bronze_layer_orders` | `bronze_orders` |
| `bronze_layer_orderdetails` | `bronze_orderdetails` |

---

### Silver Layer (8 tasks — run in parallel)

| Setting | Value |
|---|---|
| **Depends On** | All 8 Bronze tasks |
| **Type** | Notebook |
| **Retry policy** | 2 retries, 5-minute interval |
| **Notebook path** | `/Workspace/Users/<email>/sales_data_lakehouse_ls_mentorship/02_Silver/<notebook>` |

| Task Name | Notebook |
|---|---|
| `silver_layer_categories` | `silver_categories` |
| `silver_layer_customers` | `silver_customers` |
| `silver_layer_employees` | `silver_employees` |
| `silver_layer_products` | `silver_products` |
| `silver_layer_suppliers` | `silver_suppliers` |
| `silver_layer_shippers` | `silver_shippers` |
| `silver_layer_orders` | `silver_orders` |
| `silver_layer_orderdetails` | `silver_orderdetails` |

---

### Gold Dimensions (5 tasks — run in parallel)

| Setting | Value |
|---|---|
| **Depends On** | All 8 Silver tasks |
| **Type** | Notebook |
| **Retry policy** | 2 retries, 5-minute interval |
| **Notebook path** | `/Workspace/Users/<email>/sales_data_lakehouse_ls_mentorship/03_Gold/<notebook>` |

| Task Name | Notebook |
|---|---|
| `gold_layer_dim_customer` | `gold_dim_customer` |
| `gold_layer_dim_customer_scd2` | `gold_dim_customer_scd2` |
| `gold_layer_dim_employee` | `gold_dim_employee` |
| `gold_layer_dim_product` | `gold_dim_product` |
| `gold_layer_dim_date` | `gold_dim_date` |

---

### Gold Facts (1 task)

| Setting | Value |
|---|---|
| **Task Name** | `gold_layer_fact_sales` |
| **Depends On** | All 5 Gold Dimension tasks |
| **Type** | Notebook |
| **Notebook path** | `/Workspace/Users/<email>/sales_data_lakehouse_ls_mentorship/03_Gold/gold_fact_sales` |
| **Retry policy** | 2 retries, 5-minute interval |

---

### Gold Aggregated Metrics (1 task)

| Setting | Value |
|---|---|
| **Task Name** | `gold_layer_aggregated_metrics` |
| **Depends On** | `gold_layer_fact_sales` |
| **Type** | Notebook |
| **Notebook path** | `/Workspace/Users/<email>/sales_data_lakehouse_ls_mentorship/03_Gold/gold_aggregated_metrics` |
| **Retry policy** | 2 retries, 5-minute interval |

---

## Schedule Configuration

| Setting | Value |
|---|---|
| **Frequency** | Daily |
| **Time** | 02:00 AM UTC |
| **Timezone** | UTC |
| **Cron expression** | `0 0 2 * * ?` |

To update the schedule in the UI: **Job > Edit Schedule > Set to 02:00 AM UTC daily**.

---

## Retry Policy

All tasks are configured with the same retry policy:

| Setting | Value |
|---|---|
| **Max retries** | 2 |
| **Retry interval** | 5 minutes |
| **On timeout** | Fail task |

Retries handle transient cluster startup failures and temporary Databricks service
interruptions without requiring manual intervention.

---

## How to Run Manually

1. Navigate to **Workflows > Jobs** in the Databricks sidebar
2. Click **Sales_Lakehouse_Daily_Pipeline**
3. Click **Run now** (top right)
4. The run will appear under the **Runs** tab with status updating in real time

To run a **single task** in isolation:
1. Go to the **Tasks** tab
2. Click the task you want to run
3. Click **Run task** — this runs only that task, ignoring dependencies

---

## How to Monitor Execution

### During a run
1. Go to **Workflows > Jobs > Sales_Lakehouse_Daily_Pipeline**
2. Click the active run under the **Runs** tab
3. The Gantt chart shows real-time progress of each task
4. Click any task box to view its notebook output and logs

### After a run
- **Succeeded** (green): all tasks completed without errors
- **Failed** (red): at least one task failed — click the task to see the error
- **Duration**: compare against the ~15 min baseline to detect slowdowns

### Email notifications
Configure under **Job > Edit > Notifications**:
- On start
- On success
- On failure (recommended minimum)

### Key metrics to check after each run
- Record counts in Bronze, Silver, and Gold should be stable day-over-day
- `data_quality_status` distribution in Silver should not degrade
- `sales_by_month` total revenue in Gold should match the previous run's cumulative total

---

## Workspace Folder Structure

```
sales_data_lakehouse_ls_mentorship/
├── 00_Setup/
│   ├── 01_create_catalog_structure.sql
│   └── 02_create_volume_and_upload.py
├── 01_Bronze/
│   ├── bronze_categories.ipynb
│   ├── bronze_customers.ipynb
│   ├── bronze_employees.ipynb
│   ├── bronze_products.ipynb
│   ├── bronze_suppliers.ipynb
│   ├── bronze_shippers.ipynb
│   ├── bronze_orders.ipynb
│   └── bronze_orderdetails.ipynb
├── 02_Silver/
│   ├── silver_categories.ipynb
│   ├── silver_customers.ipynb
│   ├── silver_employees.ipynb
│   ├── silver_products.ipynb
│   ├── silver_suppliers.ipynb
│   ├── silver_shippers.ipynb
│   ├── silver_orders.ipynb
│   └── silver_orderdetails.ipynb
├── 03_Gold/
│   ├── gold_dim_customer.ipynb
│   ├── gold_dim_customer_scd2.ipynb
│   ├── gold_dim_employee.ipynb
│   ├── gold_dim_product.ipynb
│   ├── gold_dim_date.ipynb
│   ├── gold_fact_sales.ipynb
│   └── gold_aggregated_metrics.ipynb
└── 04_Utils/
    └── common_functions.ipynb
```
