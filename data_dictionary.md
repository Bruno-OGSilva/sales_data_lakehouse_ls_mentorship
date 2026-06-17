# Data Dictionary — SalesFlow Data Lakehouse
## SalesFlow Data Lakehouse | Phase 7: Documentation

This document describes all tables across the Bronze, Silver, and Gold layers of the
SalesFlow Lakehouse, including column definitions, data types, transformations, and relationships.

**Catalog:** `salesflow_dev`
**Last updated:** 2026-06-16

---

## Table of Contents

- [Bronze Layer](#bronze-layer)
  - [bronze.categories](#bronzecategories)
  - [bronze.customers](#bronzecustomers)
  - [bronze.employees](#bronzeemployees)
  - [bronze.products](#bronzeproducts)
  - [bronze.suppliers](#bronzesuppliers)
  - [bronze.shippers](#bronzeshippers)
  - [bronze.orders](#bronzeorders)
  - [bronze.orderdetails](#bronzeorderdetails)
- [Silver Layer](#silver-layer)
  - [silver.categories](#silvercategories)
  - [silver.customers](#silvercustomers)
  - [silver.employees](#silveremployees)
  - [silver.products](#silverproducts)
  - [silver.suppliers](#silversuppliers)
  - [silver.shippers](#silvershippers)
  - [silver.orders](#silverorders)
  - [silver.orderdetails](#silverorderdetails)
- [Gold Layer](#gold-layer)
  - [gold.dim_customer](#golddim_customer)
  - [gold.dim_customer_scd2](#golddim_customer_scd2)
  - [gold.dim_product](#golddim_product)
  - [gold.dim_employee](#golddim_employee)
  - [gold.dim_date](#golddim_date)
  - [gold.fact_sales](#goldfact_sales)
  - [gold.sales_by_month](#goldsales_by_month)
  - [gold.sales_by_category](#goldsales_by_category)
  - [gold.sales_by_country](#goldsales_by_country)

---

# Bronze Layer

> Raw ingestion layer. Tables store data exactly as received from the SQL Server
> on-premise CSV extracts. No transformations applied. Three control metadata
> columns are added to every table for traceability.

**Control metadata columns (present in all Bronze tables):**

| Column | Type | Description |
|---|---|---|
| `ingestion_timestamp` | timestamp | When this record was loaded into Bronze |
| `source_system` | string | Always `"SQL_Server_OnPremise"` |
| `file_name` | string | Name of the source CSV file |

---

## bronze.categories

**Description:** Product categories used to classify items in the product catalog.
Sourced from `categories.csv`.

| Column | Type | Description |
|---|---|---|
| `CategoryID` | integer | Primary key — unique category identifier |
| `CategoryName` | string | Name of the category (e.g. `Beverages`, `Dairy Products`) |
| `EnglishDescription` | string | Plain-text description of the category |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

## bronze.customers

**Description:** Customer master data including contact details and location.
Sourced from `customers.csv`.

| Column | Type | Description |
|---|---|---|
| `CustomerID` | string | Primary key — unique customer code (e.g. `ALFKI`) |
| `CompanyName` | string | Customer company name |
| `ContactName` | string | Name of the primary contact person |
| `ContactTitle` | string | Job title of the contact person |
| `Address` | string | Street address |
| `City` | string | City |
| `Region` | string | Region or state (nullable) |
| `PostalCode` | string | Postal or ZIP code (nullable) |
| `Country` | string | Country |
| `Phone` | string | Phone number (may include special characters) |
| `Fax` | string | Fax number (nullable) |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

## bronze.employees

**Description:** Employee records including personal details and location.
Sourced from `employees.csv`.

| Column | Type | Description |
|---|---|---|
| `EmployeeID` | integer | Primary key — unique employee identifier |
| `LastName` | string | Employee last name |
| `FirstName` | string | Employee first name |
| `Address` | string | Street address |
| `City` | string | City |
| `Province` | string | Province or region |
| `PostalCode` | string | Postal code |
| `Phone` | long | Phone number (ingested as numeric by inferSchema) |
| `BirthDate` | string | Date of birth in `yyyy/MM/dd HH:mm:ss` format |
| `CreateDate` | string | Record creation date |
| `UpdateDate` | string | Record last update date |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

## bronze.products

**Description:** Product catalog with pricing, stock levels, and category reference.
Sourced from `products.csv`.

| Column | Type | Description |
|---|---|---|
| `ProductID` | integer | Primary key — unique product identifier |
| `ProductName` | string | Product name |
| `SupplierID` | integer | Foreign key to `suppliers` |
| `CategoryID` | integer | Foreign key to `categories` |
| `QuantityPerUnit` | string | Packaging description (e.g. `12 - 550 ml bottles`) |
| `UnitPrice` | double | Price per unit |
| `UnitsInStock` | integer | Current stock quantity (nullable) |
| `UnitsOnOrder` | integer | Units currently on order (nullable) |
| `ReorderLevel` | integer | Stock level that triggers reorder (nullable) |
| `Discontinued` | integer | `1` if product is discontinued, `0` if active |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

## bronze.suppliers

**Description:** Supplier master data including company details and contact information.
Sourced from `suppliers.csv`.

| Column | Type | Description |
|---|---|---|
| `SupplierID` | integer | Primary key — unique supplier identifier |
| `CompanyName` | string | Supplier company name |
| `ContactName` | string | Name of the primary contact person |
| `ContactTitle` | string | Job title of the contact person |
| `Address` | string | Street address |
| `City` | string | City |
| `Region` | string | Region (nullable) |
| `PostalCode` | string | Postal code (nullable) |
| `Country` | string | Country |
| `Phone` | string | Phone number |
| `Fax` | string | Fax number (nullable) |
| `HomePage` | string | Supplier website URL (nullable) |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

## bronze.shippers

**Description:** Shipping company reference data.
Sourced from `shippers.csv`.

| Column | Type | Description |
|---|---|---|
| `ShipperID` | integer | Primary key — unique shipper identifier |
| `CompanyName` | string | Shipping company name |
| `Phone` | string | Contact phone number |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

## bronze.orders

**Description:** Order header records linking customers, employees, and shippers.
Sourced from `orders.csv`.

| Column | Type | Description |
|---|---|---|
| `OrderID` | integer | Primary key — unique order identifier |
| `CustomerID` | string | Foreign key to `customers` |
| `EmployeeID` | integer | Foreign key to `employees` — who handled the order |
| `OrderDate` | string | Date order was placed (`yyyy/MM/dd HH:mm:ss`) |
| `RequiredDate` | string | Date order was required by customer |
| `ShippedDate` | string | Date order was shipped (nullable if not yet shipped) |
| `ShipVia` | integer | Foreign key to `shippers` |
| `Freight` | double | Shipping cost |
| `ShipName` | string | Recipient name |
| `ShipAddress` | string | Shipping street address |
| `ShipCity` | string | Shipping city |
| `ShipRegion` | string | Shipping region (nullable) |
| `ShipPostalCode` | string | Shipping postal code (nullable) |
| `ShipCountry` | string | Shipping country |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

## bronze.orderdetails

**Description:** Order line items — one row per product per order.
Sourced from `orderdetails.csv`.

| Column | Type | Description |
|---|---|---|
| `OrderID` | integer | Foreign key to `orders` — composite PK |
| `ProductID` | integer | Foreign key to `products` — composite PK |
| `UnitPrice` | double | Price per unit at time of order |
| `Quantity` | integer | Number of units ordered |
| `Discount` | double | Discount applied as a decimal (e.g. `0.05` = 5%) |
| `ingestion_timestamp` | timestamp | *(control metadata)* |
| `source_system` | string | *(control metadata)* |
| `file_name` | string | *(control metadata)* |

---

# Silver Layer

> Curated layer. Each table is sourced from its Bronze counterpart and applies
> deduplication, cleaning, standardization, and quality validation.
> Only columns that are **added or transformed** relative to Bronze are documented here.
> All Silver tables inherit the original Bronze columns (unchanged) plus the columns below.

**Processing metadata columns (present in all Silver tables):**

| Column | Type | Description |
|---|---|---|
| `data_quality_status` | string | `VALID` or `INVALID` based on business rules |
| `processing_timestamp` | timestamp | When this record was processed in Silver |

---

## silver.categories

**Source:** `bronze.categories`

**Transformations:**
- Deduplicated by `CategoryID`
- `CategoryName` cleaned (trim + title case)
- `EnglishDescription` cleaned (trim + title case); nulls filled with `"N/A"`

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `CategoryName` | string | Trimmed, title case applied |
| `EnglishDescription` | string | Trimmed, title case; `NULL` → `"N/A"` |
| `data_quality_status` | string | `INVALID` if `CategoryID` or `CategoryName` is null |
| `processing_timestamp` | timestamp | Silver processing time |

---

## silver.customers

**Source:** `bronze.customers`

**Transformations:**
- Deduplicated by `CustomerID`
- String columns cleaned (trim + title case)
- `Phone` standardized — special characters removed, digits and `+` only; null if fewer than 7 digits
- Nulls filled with business defaults

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `CompanyName` | string | Trimmed, title case applied |
| `ContactName` | string | Trimmed, title case; `NULL` → `"Unknown"` |
| `City` | string | Trimmed, title case applied |
| `Country` | string | Trimmed, title case applied |
| `Region` | string | `NULL` → `"N/A"` |
| `PostalCode` | string | `NULL` → `"00000"` |
| `Phone` | string | Special characters removed; `NULL` if < 7 digits |
| `data_quality_status` | string | `INVALID` if `CompanyName` or `Country` is null |
| `processing_timestamp` | timestamp | Silver processing time |

---

## silver.employees

**Source:** `bronze.employees`

**Transformations:**
- Deduplicated by `EmployeeID`
- Name and location columns cleaned (trim + title case)
- Optional fields filled with defaults

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `FirstName` | string | Trimmed, title case applied |
| `LastName` | string | Trimmed, title case applied |
| `City` | string | Trimmed, title case applied |
| `Country` | string | Trimmed, title case applied |
| `Region` | string | `NULL` → `"N/A"` |
| `Notes` | string | `NULL` → `"N/A"` |
| `data_quality_status` | string | `INVALID` if `EmployeeID`, `FirstName`, or `LastName` is null |
| `processing_timestamp` | timestamp | Silver processing time |

---

## silver.products

**Source:** `bronze.products`

**Transformations:**
- Deduplicated by `ProductID`
- `ProductName` cleaned (trim + title case)
- Numeric nulls filled with `0`
- Business logic columns derived

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `ProductName` | string | Trimmed, title case applied |
| `UnitsInStock` | integer | `NULL` → `0` |
| `UnitsOnOrder` | integer | `NULL` → `0` |
| `ReorderLevel` | integer | `NULL` → `0` |
| `is_available` | boolean | `TRUE` if `UnitsInStock > 0` |
| `price_category` | string | `"Low"` if `UnitPrice < 10`; `"Medium"` if `10 ≤ UnitPrice ≤ 50`; `"High"` if `UnitPrice > 50`; `NULL` if price is null |
| `data_quality_status` | string | `INVALID` if `ProductName` is null OR `UnitPrice < 0` |
| `processing_timestamp` | timestamp | Silver processing time |

---

## silver.suppliers

**Source:** `bronze.suppliers`

**Transformations:**
- Deduplicated by `SupplierID`
- String columns cleaned (trim + title case)
- `Phone` standardized
- Optional fields filled with defaults

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `CompanyName` | string | Trimmed, title case applied |
| `ContactName` | string | Trimmed, title case applied |
| `City` | string | Trimmed, title case applied |
| `Country` | string | Trimmed, title case applied |
| `Phone` | string | Special characters removed; `NULL` if < 7 digits |
| `Region` | string | `NULL` → `"N/A"` |
| `Fax` | string | `NULL` → `"N/A"` |
| `HomePage` | string | `NULL` → `"N/A"` |
| `data_quality_status` | string | `INVALID` if `SupplierID` or `CompanyName` is null |
| `processing_timestamp` | timestamp | Silver processing time |

---

## silver.shippers

**Source:** `bronze.shippers`

**Transformations:**
- Deduplicated by `ShipperID`
- `CompanyName` cleaned (trim + title case)
- `Phone` standardized

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `CompanyName` | string | Trimmed, title case applied |
| `Phone` | string | Special characters removed; `NULL` if < 7 digits |
| `data_quality_status` | string | `INVALID` if `ShipperID` or `CompanyName` is null |
| `processing_timestamp` | timestamp | Silver processing time |

---

## silver.orders

**Source:** `bronze.orders`

**Transformations:**
- Deduplicated by `OrderID`
- Date columns parsed from `yyyy/MM/dd HH:mm:ss` string to `DateType`
  using `substring(col, 1, 10)` + explicit format — required due to SQL Server export format
- Business logic columns derived from shipping data
- Date integrity validated

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `OrderDate` | date | Parsed from string: `substring(1,10)` + `to_date("yyyy/MM/dd")` |
| `ShippedDate` | date | Parsed from string: same pattern; `NULL` if not yet shipped |
| `RequiredDate` | date | Parsed from string: same pattern |
| `is_shipped` | boolean | `TRUE` if `ShippedDate` is not null |
| `days_to_ship` | integer | `datediff(ShippedDate, OrderDate)`; `NULL` for unshipped orders |
| `data_quality_status` | string | `INVALID` if `OrderID` or `CustomerID` is null, `OrderDate` is null or future, or `days_to_ship < 0` |
| `processing_timestamp` | timestamp | Silver processing time |

---

## silver.orderdetails

**Source:** `bronze.orderdetails`

**Transformations:**
- Numeric columns explicitly cast (`Quantity` → integer, `UnitPrice` → double, `Discount` → double)
- Business rules validated per column
- `line_total` computed only for records passing all validations

| Added/Transformed Column | Type | Transformation |
|---|---|---|
| `Quantity` | integer | Explicitly cast from inferred type |
| `UnitPrice` | double | Explicitly cast from inferred type |
| `Discount` | double | Explicitly cast from inferred type |
| `line_total` | double | `round(Quantity × UnitPrice × (1 − Discount), 2)`; `NULL` if any validation fails |
| `data_quality_status` | string | `INVALID` if `OrderID` or `ProductID` is null, `Quantity ≤ 0`, `UnitPrice < 0`, or `Discount` outside `[0, 1]` |
| `processing_timestamp` | timestamp | Silver processing time |

---

# Gold Layer

> Analytical layer. Contains a star schema dimensional model built from the
> Silver curated tables. Only `VALID` records from Silver are promoted to Gold.
> All dimension tables use MD5 hash surrogate keys for stable cross-layer joins.

---

## gold.dim_customer

**Description:** Customer dimension containing cleaned and standardized customer
attributes. Source: `silver.customers` (VALID records only).

**Grain:** One row per unique customer.

| Column | Type | PK/FK | Description |
|---|---|---|---|
| `customer_key` | string | PK | MD5 surrogate key derived from `customer_id` |
| `customer_id` | string | NK | Natural key — original `CustomerID` from source |
| `company_name` | string | | Cleaned company name (title case) |
| `contact_name` | string | | Primary contact name (`"Unknown"` if null) |
| `country` | string | | Country (title case) |
| `city` | string | | City (title case) |
| `region` | string | | Region (`"N/A"` if not applicable) |
| `phone` | string | | Standardized phone number (digits and `+` only) |
| `effective_date` | date | | Date this record was loaded into Gold |

**Relationships:**
- `customer_key` → `fact_sales.customer_key`

---

## gold.dim_customer_scd2

**Description:** Customer dimension with full history tracking using
**Slowly Changing Dimension Type 2**. Preserves all versions of a customer's
attributes over time. Source: `silver.customers` (VALID records only).

**Grain:** One row per customer per version (a customer may have multiple rows).

| Column | Type | PK/FK | Description |
|---|---|---|---|
| `customer_key` | string | PK | MD5 surrogate key (unique per version) |
| `customer_id` | string | NK | Natural key — original `CustomerID` from source |
| `company_name` | string | | Company name for this version |
| `contact_name` | string | | Contact name for this version |
| `country` | string | | Country for this version |
| `city` | string | | City for this version |
| `region` | string | | Region for this version |
| `phone` | string | | Phone number for this version |
| `effective_date` | date | | When this version became active |
| `end_date` | date | | When this version was superseded (`NULL` if current) |
| `is_current` | boolean | | `TRUE` if this is the active version |

**SCD2 Rules:**
- New customer → INSERT with `is_current = TRUE`, `end_date = NULL`
- Changed customer → CLOSE old (`is_current = FALSE`, `end_date = today - 1`) → INSERT new (`is_current = TRUE`)
- Unchanged customer → no action

---

## gold.dim_product

**Description:** Product dimension enriched with category name via JOIN with
`silver.categories`. Source: `silver.products` + `silver.categories` (VALID records only).

**Grain:** One row per unique product.

| Column | Type | PK/FK | Description |
|---|---|---|---|
| `product_key` | string | PK | MD5 surrogate key derived from `product_id` |
| `product_id` | integer | NK | Natural key — original `ProductID` from source |
| `product_name` | string | | Cleaned product name (title case) |
| `category_name` | string | | Category name from JOIN with `silver.categories` (`NULL` if unmatched) |
| `unit_price` | double | | Unit price at time of dimension load |
| `price_category` | string | | Price classification: `Low` (< $10), `Medium` ($10–$50), `High` (> $50) |
| `is_available` | boolean | | `TRUE` if `UnitsInStock > 0` |
| `effective_date` | date | | Date this record was loaded into Gold |

**Relationships:**
- `product_key` → `fact_sales.product_key`

---

## gold.dim_employee

**Description:** Employee dimension with computed full name.
Source: `silver.employees` (VALID records only).

**Grain:** One row per unique employee.

| Column | Type | PK/FK | Description |
|---|---|---|---|
| `employee_key` | string | PK | MD5 surrogate key derived from `employee_id` |
| `employee_id` | integer | NK | Natural key — original `EmployeeID` from source |
| `full_name` | string | | Concatenation of `FirstName` + `LastName` via `concat_ws` (nulls skipped) |
| `title` | string | | Job title |
| `city` | string | | City |
| `province` | string | | Province |
| `postal_code` | string | | Postal code |
| `phone` | string | | Phone number (cast from `Long` to `string`) |
| `birth_date` | date | | Date of birth (parsed from `yyyy/MM/dd`) |
| `effective_date` | date | | Date this record was loaded into Gold |

**Relationships:**
- `employee_key` → `fact_sales.employee_key`

---

## gold.dim_date

**Description:** Date dimension generated programmatically — not sourced from
any table. Covers 2020-01-01 to 2030-12-31 (4,018 dates).

**Grain:** One row per calendar date.

| Column | Type | PK/FK | Description |
|---|---|---|---|
| `date_key` | integer | PK | Integer in `YYYYMMDD` format (e.g. `20260616`) |
| `full_date` | date | | Full date value |
| `year` | integer | | Calendar year (e.g. `2026`) |
| `quarter` | integer | | Quarter of the year (1–4) |
| `month` | integer | | Month number (1–12) |
| `month_name` | string | | Month name (e.g. `January`) |
| `day` | integer | | Day of the month (1–31) |
| `day_of_week` | integer | | Day of week (1 = Sunday, 7 = Saturday) |
| `day_name` | string | | Day name (e.g. `Monday`) |
| `week_of_year` | integer | | Week number within the year (1–53) |
| `is_weekend` | boolean | | `TRUE` if Saturday or Sunday |

**Relationships:**
- `date_key` → `fact_sales.date_key`

---

## gold.fact_sales

**Description:** Central fact table at order line grain, linking all four dimensions.
Sourced from `silver.orderdetails` + `silver.orders` (VALID records only).
Natural keys replaced by surrogate keys via dimension lookups.

**Grain:** One row per order line (`OrderID` + `ProductID`).

| Column | Type | PK/FK | Description |
|---|---|---|---|
| `sales_key` | string | PK | MD5 surrogate key derived from `order_id` + `product_key` |
| `date_key` | integer | FK → `dim_date` | Date the order was placed |
| `customer_key` | string | FK → `dim_customer` | Customer who placed the order |
| `product_key` | string | FK → `dim_product` | Product ordered |
| `employee_key` | string | FK → `dim_employee` | Employee who handled the order |
| `order_id` | integer | DD | Degenerate dimension — natural order key from source |
| `quantity` | integer | | Number of units ordered |
| `unit_price` | double | | Price per unit at time of order |
| `discount` | double | | Discount applied (0–1, e.g. `0.05` = 5%) |
| `line_total` | double | | `round(Quantity × UnitPrice × (1 − Discount), 2)`; `NULL` for invalid records |
| `load_timestamp` | timestamp | | When this record was written to Gold |

**Relationships:**
- `date_key` → `dim_date.date_key`
- `customer_key` → `dim_customer.customer_key`
- `product_key` → `dim_product.product_key`
- `employee_key` → `dim_employee.employee_key`

**Notes:**
- All dimension joins use **left joins** to preserve sales records even when a dimension member is missing
- `order_id` is a degenerate dimension — it carries no descriptive attributes but enables drill-through to the order header
- `line_total` is `NULL` for records that failed Silver quality validation

---

## gold.sales_by_month

**Description:** Pre-aggregated view. Revenue, orders, and quantity grouped by
year and month. Joins `fact_sales` with `dim_date`.

**Grain:** One row per calendar year + month combination.

| Column | Type | Description |
|---|---|---|
| `year` | integer | Calendar year |
| `month` | integer | Month number (1–12) |
| `month_name` | string | Month name (e.g. `January`) |
| `total_orders` | long | Count of distinct `order_id` |
| `total_quantity` | long | Sum of `quantity` |
| `total_revenue` | double | Sum of `line_total`, rounded to 2 decimals |
| `avg_line_value` | double | Average `line_total` per row, rounded to 2 decimals |

---

## gold.sales_by_category

**Description:** Pre-aggregated view. Revenue and quantity grouped by product
category. Joins `fact_sales` with `dim_product`.

**Grain:** One row per product category.

| Column | Type | Description |
|---|---|---|
| `category_name` | string | Product category name |
| `total_orders` | long | Count of distinct `order_id` |
| `total_quantity` | long | Sum of `quantity` |
| `total_revenue` | double | Sum of `line_total`, rounded to 2 decimals |
| `avg_unit_price` | double | Average `unit_price`, rounded to 2 decimals |
| `avg_discount` | double | Average `discount`, rounded to 4 decimals |

---

## gold.sales_by_country

**Description:** Pre-aggregated view. Revenue, orders, and customers grouped by
customer country. Joins `fact_sales` with `dim_customer` (left join to preserve
unmatched records under `"Unknown"`).

**Grain:** One row per customer country.

| Column | Type | Description |
|---|---|---|
| `country` | string | Customer country (`"Unknown"` if no dimension match) |
| `total_orders` | long | Count of distinct `order_id` |
| `total_customers` | long | Count of distinct `customer_key` |
| `total_quantity` | long | Sum of `quantity` |
| `total_revenue` | double | Sum of `line_total`, rounded to 2 decimals |
| `avg_order_value` | double | `total_revenue / total_orders`, rounded to 2 decimals |

---

*Generated for the SalesFlow Data Lakehouse project — Bruno Silva*
