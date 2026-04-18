# E-Commerce Data Platform - Data Catalog

**Version:** 1.0  
**Last Updated:** February 15, 2025  
**Owner:** Data Engineering Team

---

## 1. Overview

### Purpose

This data catalog provides comprehensive documentation for all data assets in the e-commerce analytics platform. It serves as the single source of truth for understanding table structures, column definitions, data lineage, and business logic.

### Target Audience

* **Data Analysts** - Understanding available datasets and how to query them
* **Business Intelligence Developers** - Building dashboards and reports
* **Data Scientists** - Extracting features for ML models
* **Data Engineers** - Maintaining and enhancing the data pipeline
* **Business Stakeholders** - Understanding data definitions and metrics

### How to Use This Catalog

1. **Find a table** - See Table Inventory (Section 2) for complete list
2. **Understand structure** - Read Detailed Table Documentation (Section 3)
3. **Look up columns** - Check Column Dictionary (Section 4) for definitions
4. **Calculate metrics** - Reference Business Metrics Glossary (Section 5)
5. **Get started** - Use Sample Queries (Section 6) for common patterns

### Medallion Architecture

Our data platform follows the medallion architecture with three layers:

* **🥉 Bronze** - Raw data ingestion, immutable historical archive
* **🥈 Silver** - Cleaned, validated, standardized data
* **🥇 Gold** - Business-ready, denormalized, pre-aggregated data

---

## 2. Table Inventory

### Summary Statistics

| Layer | Dimension Tables | Fact Tables | Total Tables |
|-------|-----------------|-------------|--------------|
| Bronze | 5 | 1 | 6 |
| Silver | 5 | 1 | 6 |
| Gold | 3 | 1 | 4 |
| **Total** | **13** | **3** | **16** |

---

### Bronze Layer Tables

| Table Name | Type | Description | Approx Rows | Update Frequency |
|------------|------|-------------|-------------|------------------|
| `brz_brand` | Dimension | Brand master data with category associations | ~100 | Weekly |
| `brz_category` | Dimension | Product category taxonomy | ~50 | Monthly |
| `brz_products` | Dimension | Product catalog with attributes and ratings | ~5,000 | Daily |
| `brz_customers` | Dimension | Customer master with geographic data | ~50,000 | Daily |
| `brz_calendar` | Dimension | Date dimension with calendar attributes | ~3,650 | Yearly |
| `brz_order_items` | Fact | Order line items with transactional details | ~500,000+ | Real-time |

---

### Silver Layer Tables

| Table Name | Type | Description | Approx Rows | Update Frequency |
|------------|------|-------------|-------------|------------------|
| `slv_brands` | Dimension | Cleaned brands with standardized codes | ~100 | Weekly |
| `slv_category` | Dimension | Cleaned categories with deduplicated codes | ~50 | Monthly |
| `slv_products` | Dimension | Cleaned products with type-safe attributes | ~5,000 | Daily |
| `slv_customers` | Dimension | Cleaned customers with standardized geography | ~50,000 | Daily |
| `slv_calendar` | Dimension | Cleaned calendar with validated temporal data | ~3,650 | Yearly |
| `slv_order_items` | Fact | Cleaned transactions with parsed timestamps | ~500,000+ | Real-time |

---

### Gold Layer Tables

| Table Name | Type | Description | Approx Rows | Update Frequency |
|------------|------|-------------|-------------|------------------|
| `gld_dim_products` | Dimension | Business-ready products enriched with brand/category names | ~5,000 | Daily |
| `gld_dim_customers` | Dimension | Business-ready customers with regional mapping | ~50,000 | Daily |
| `gld_dim_date` | Dimension | Business calendar with weekend flags and date surrogate keys | ~3,650 | Yearly |
| `gld_fact_order_items` | Fact | Business-ready transactions with pre-calculated metrics and currency conversion | ~500,000+ | Real-time |

---

## 3. Detailed Table Documentation

---

### 3.1 Bronze Layer

#### brz_brand

**Business Definition:**  
Brand master data containing manufacturer and brand identities with their associated product categories.

**Primary Key:** `brand_code`

**Key Columns:**

| Column | Data Type | Description | Sample Values |
|--------|-----------|-------------|---------------|
| `brand_code` | String | Unique brand identifier | "NIKE-01", "ADIDAS-02" |
| `brand_name` | String | Brand display name | "Nike", "Adidas" |
| `category_code` | String | Foreign key to category | "SHOES", "APPAREL" |
| `_source_file` | String | Source file path (lineage) | "/Volumes/.../brands/brands.csv" |
| `ingested_at` | Timestamp | Ingestion timestamp | "2025-02-15 08:00:00" |

**Source System:** CSV files in `/Volumes/ecommerce/source_data/raw/brands/`

**Transformation Logic:** Direct load from source CSV with metadata enrichment (_source_file, ingested_at)

---

#### brz_category

**Business Definition:**  
Product category taxonomy defining the merchandising hierarchy.

**Primary Key:** `category_code`

**Key Columns:**

| Column | Data Type | Description | Sample Values |
|--------|-----------|-------------|---------------|
| `category_code` | String | Unique category identifier | "SHOES", "ELECTRONICS" |
| `category_name` | String | Category display name | "Shoes", "Electronics" |
| `_source_file` | String | Source file path (lineage) | "/Volumes/.../category/category.csv" |
| `_ingested_at` | Timestamp | Ingestion timestamp | "2025-02-15 08:00:00" |

**Source System:** CSV files in `/Volumes/ecommerce/source_data/raw/category/`

**Transformation Logic:** Direct load from source CSV with metadata enrichment

**Known Issues:** Contains duplicate category_code values (resolved in Silver)

---

#### brz_products

**Business Definition:**  
Complete product catalog with physical attributes, ratings, and category/brand associations.

**Primary Key:** `product_id`

**Key Columns:**

| Column | Data Type | Description | Sample Values |
|--------|-----------|-------------|---------------|
| `product_id` | String | Unique product identifier | "PROD-12345" |
| `sku` | String | Stock keeping unit | "SKU-ABC-123" |
| `category_code` | String | Foreign key to category | "ELECTRONICS" |
| `brand_code` | String | Foreign key to brand | "SONY-01" |
| `color` | String | Product color | "Blue", "Red" |
| `size` | String | Product size | "M", "L", "XL" |
| `material` | String | Product material | "Cotton", "Aluminum" |
| `weight_grams` | String | Weight in grams (stored as string) | "500g", "1200" |
| `length_cm` | String | Length in cm (stored as string) | "25.5", "N/A" |
| `width_cm` | Float | Width in cm | 15.2 |
| `height_cm` | Float | Height in cm | 10.5 |
| `rating_count` | Integer | Number of customer reviews | 42 |
| `file_name` | String | Source file name (lineage) | "products.csv" |
| `ingest_timestamp` | Timestamp | Ingestion timestamp | "2025-02-15 08:00:00" |

**Source System:** CSV files in `/Volumes/ecommerce/source_data/raw/products/`

**Transformation Logic:** Direct load with metadata. Note: weight_grams and length_cm stored as strings due to inconsistent source formats ("500g", "N/A", mixed units).

---

#### brz_customers

**Business Definition:**  
Customer master data with contact information and geographic attributes.

**Primary Key:** `customer_id`

**Key Columns:**

| Column | Data Type | Description | Sample Values |
|--------|-----------|-------------|---------------|
| `customer_id` | String | Unique customer identifier | "CUST-001234" |
| `phone` | String | Customer phone number (PII) | "+1-555-1234" |
| `country_code` | String | ISO country code | "US", "IN" |
| `country` | String | Country name | "United States", "India" |
| `state` | String | State/province code | "CA", "MH" |
| `file_name` | String | Source file name (lineage) | "customers.csv" |
| `ingest_timestamp` | Timestamp | Ingestion timestamp | "2025-02-15 08:00:00" |

**Source System:** CSV files in `/Volumes/ecommerce/source_data/raw/customers/`

**Transformation Logic:** Direct load with metadata

**Privacy Note:** Contains PII (phone numbers) - access restricted via Unity Catalog RBAC

---

#### brz_calendar

**Business Definition:**  
Date dimension with calendar attributes for temporal analysis.

**Primary Key:** `date`

**Key Columns:**

| Column | Data Type | Description | Sample Values |
|--------|-----------|-------------|---------------|
| `date` | String | Date value (raw string) | "2025-01-15" |
| `year` | Integer | Year | 2025 |
| `day_name` | String | Day of week | "Monday", "Tuesday" |
| `quarter` | Integer | Fiscal quarter | 1, 2, 3, 4 |
| `week_of_year` | Integer | Week number (may be negative) | 3, -1 |
| `_source_file` | String | Source file path (lineage) | "/Volumes/.../date/calendar.csv" |
| `_ingested_at` | Timestamp | Ingestion timestamp | "2025-02-15 08:00:00" |

**Source System:** CSV files in `/Volumes/ecommerce/source_data/raw/date/`

**Transformation Logic:** Direct load with metadata

**Known Issues:** Negative week_of_year values and mixed case day_name (resolved in Silver)

---

#### brz_order_items

**Business Definition:**  
Transactional fact table capturing every line item within customer orders.

**Primary Key:** Composite (`order_id`, `item_seq`)

**Key Columns:**

| Column | Data Type | Description | Sample Values |
|--------|-----------|-------------|---------------|
| `order_id` | String | Unique order identifier | "ORD-2025-001234" |
| `customer_id` | String | Foreign key to customers | "CUST-001234" |
| `product_id` | String | Foreign key to products | "PROD-12345" |
| `item_seq` | String | Line item sequence (stored as string) | "1", "2", "3" |
| `dt` | String | Order date (raw string) | "2025-08-01" |
| `order_ts` | String | Order timestamp (raw string) | "2025-08-01 22:53:52" |
| `quantity` | String | Quantity ordered (stored as string) | "2", "Two" |
| `unit_price` | String | Price per unit (stored as string) | "$49.99", "49.99" |
| `discount_pct` | String | Discount percentage (stored as string) | "15%", "10" |
| `tax_amount` | String | Tax amount (stored as string) | "5.00" |
| `channel` | String | Sales channel | "web", "app", "Website" |
| `coupon_code` | String | Coupon identifier | "SAVE20", "FREESHIP" |
| `unit_price_currency` | String | Transaction currency | "USD", "INR", "GBP" |
| `file_name` | String | Source file name (lineage) | "order_items.csv" |
| `ingest_timestamp` | Timestamp | Ingestion timestamp | "2025-02-15 08:00:00" |

**Source System:** CSV files in `/Volumes/ecommerce/source_data/raw/order_items/landing/`

**Transformation Logic:** Direct load with metadata. All fields stored as strings to handle data quality issues in source (text quantities like "Two", currency symbols like "$49.99").

---

### 3.2 Silver Layer

#### slv_brands

**Business Definition:**  
Cleaned brand data with standardized codes and normalized text.

**Primary Key:** `brand_code`

**Key Columns:**

| Column | Data Type | Description | Transformation from Bronze |
|--------|-----------|-------------|---------------------------|
| `brand_code` | String | Sanitized brand identifier | Removed special characters: "NIKE-01" → "NIKE01" |
| `brand_name` | String | Trimmed brand name | Whitespace removed: "Nike " → "Nike" |
| `category_code` | String | Standardized category FK | Mapped to codes: "GROCERY" → "GRCY" |
| `_source_file` | String | Preserved lineage | Inherited from Bronze |
| `ingested_at` | Timestamp | Preserved audit timestamp | Inherited from Bronze |

**Source:** `ecommerce.bronze.brz_brand`

**Transformation Logic:**
1. Trim whitespace from brand_name
2. Remove special characters from brand_code
3. Standardize category_code (full words → 3-4 char codes)

---

#### slv_category

**Business Definition:**  
Deduplicated and standardized product categories.

**Primary Key:** `category_code`

**Key Columns:**

| Column | Data Type | Description | Transformation from Bronze |
|--------|-----------|-------------|---------------------------|
| `category_code` | String | Uppercase category identifier | "elec" → "ELEC" |
| `category_name` | String | Category display name | Deduplicated |
| `_source_file` | String | Preserved lineage | Inherited from Bronze |
| `_ingested_at` | Timestamp | Preserved audit timestamp | Inherited from Bronze |

**Source:** `ecommerce.bronze.brz_category`

**Transformation Logic:**
1. Detect and remove duplicates (keep first occurrence)
2. Uppercase category_code for consistency

---

#### slv_products

**Business Definition:**  
Type-safe product catalog with cleaned attributes and corrected spelling.

**Primary Key:** `product_id`

**Key Columns:**

| Column | Data Type | Description | Transformation from Bronze |
|--------|-----------|-------------|---------------------------|
| `product_id` | String | Product identifier | No change |
| `sku` | String | Stock keeping unit | No change |
| `category_code` | String | Uppercase category FK | "elec" → "ELEC" |
| `brand_code` | String | Uppercase brand FK | "nike-01" → "NIKE-01" |
| `weight_grams` | Integer | Weight (converted to integer) | "500g" → 500, "N/A" → null |
| `length_cm` | Float | Length (converted to float) | "15,5" → 15.5 (European decimal) |
| `material` | String | Corrected material spelling | "Coton" → "Cotton", "Alumium" → "Aluminum" |

**Source:** `ecommerce.bronze.brz_products`

**Transformation Logic:**
1. Remove "g" suffix from weight_grams, cast to Integer
2. Replace European decimal comma with dot in length_cm, cast to Float
3. Uppercase category_code and brand_code for join compatibility
4. Fix known spelling errors in material field

---

#### slv_customers

**Business Definition:**  
Cleaned customer data with standardized phone numbers and geographic normalization.

**Primary Key:** `customer_id`

**Key Columns:**

| Column | Data Type | Description | Transformation from Bronze |
|--------|-----------|-------------|---------------------------|
| `customer_id` | String | Customer identifier | No change |
| `phone` | String | Standardized phone format | Removed non-numeric chars except "+" |
| `country_code` | String | Uppercase country code | "us" → "US" |
| `country` | String | Country name | No change |
| `state` | String | Uppercase state code | "ca" → "CA" |

**Source:** `ecommerce.bronze.brz_customers`

**Transformation Logic:**
1. Remove all non-numeric characters from phone (except "+")
2. Uppercase country_code and state
3. Trim whitespace from all fields

---

#### slv_calendar

**Business Definition:**  
Validated date dimension with corrected week numbers and standardized day names.

**Primary Key:** `date`

**Key Columns:**

| Column | Data Type | Description | Transformation from Bronze |
|--------|-----------|-------------|---------------------------|
| `date` | Date | Parsed date | "2025-01-15" (string) → Date type |
| `year` | Integer | Year | No change |
| `day_name` | String | Capitalized day name | "MONDAY" → "Monday" |
| `quarter` | Integer | Fiscal quarter | No change |
| `week_of_year` | Integer | Corrected week number | Negative values recalculated using weekofyear() |

**Source:** `ecommerce.bronze.brz_calendar`

**Transformation Logic:**
1. Parse date string to Date type
2. Recalculate negative week_of_year using Spark's weekofyear()
3. Standardize day_name to InitCap format

---

#### slv_order_items

**Business Definition:**  
Deduplicated transactions with type-safe metrics and standardized categorical values.

**Primary Key:** Composite (`order_id`, `item_seq`)

**Key Columns:**

| Column | Data Type | Description | Transformation from Bronze |
|--------|-----------|-------------|---------------------------|
| `order_id` | String | Order identifier | No change |
| `customer_id` | String | Customer FK | No change |
| `product_id` | String | Product FK | No change |
| `item_seq` | Integer | Line item sequence | "1" (string) → 1 (integer) |
| `dt` | Date | Order date | "2025-08-01" (string) → Date type |
| `order_ts` | Timestamp | Order timestamp | Parsed with multiple format fallbacks |
| `quantity` | Integer | Quantity ordered | "Two" → 2, "2" → 2 |
| `unit_price` | Double | Price per unit | "$49.99" → 49.99 (removed "$") |
| `discount_pct` | Double | Discount percentage | "15%" → 15.0 (removed "%") |
| `tax_amount` | Double | Tax amount | "5.00" → 5.0 |
| `channel` | String | Standardized channel | "web" → "Website", "app" → "Mobile" |
| `coupon_code` | String | Lowercase coupon | "SAVE20" → "save20" |
| `processed_time` | Timestamp | Silver processing timestamp | Current timestamp |

**Source:** `ecommerce.bronze.brz_order_items`

**Transformation Logic:**
1. Deduplicate on (order_id, item_seq)
2. Convert text quantities ("Two") to integers
3. Remove currency symbols from unit_price, cast to Double
4. Remove "%" from discount_pct, cast to Double
5. Standardize channel names: "web"→"Website", "app"→"Mobile"
6. Lowercase and trim coupon_code
7. Parse order_ts with multiple format fallbacks
8. Add processed_time audit column

---

### 3.3 Gold Layer

#### gld_dim_products

**Business Definition:**  
Business-ready product dimension enriched with brand and category names (pre-joined).

**Primary Key:** `product_id`

**Key Columns:**

| Column | Data Type | Description | Enrichment Logic |
|--------|-----------|-------------|------------------|
| `product_id` | String | Product identifier | From slv_products |
| `sku` | String | Stock keeping unit | From slv_products |
| `brand_name` | String | Brand display name | Joined from slv_brands |
| `category_name` | String | Category display name | Joined from slv_category via slv_brands |
| `color` | String | Product color | From slv_products |
| `size` | String | Product size | From slv_products |
| `material` | String | Product material | From slv_products |
| `weight_grams` | Integer | Weight in grams | From slv_products |
| `length_cm` | Float | Length in cm | From slv_products |
| `rating_count` | Integer | Number of reviews | From slv_products |

**Source Tables:**
* `ecommerce.silver.slv_products`
* `ecommerce.silver.slv_brands`
* `ecommerce.silver.slv_category`

**Transformation Logic:**
1. Join slv_brands with slv_category on category_code
2. Left join slv_products with enriched brands_categories on brand_code
3. COALESCE brand_name and category_name to 'Not Available' (handles orphaned products)

**Business Value:** Eliminates 3-table joins for product analysis

---

#### gld_dim_customers

**Business Definition:**  
Business-ready customer dimension with pre-calculated regional mapping.

**Primary Key:** `customer_id`

**Key Columns:**

| Column | Data Type | Description | Enrichment Logic |
|--------|-----------|-------------|------------------|
| `customer_id` | String | Customer identifier | From slv_customers |
| `phone` | String | Customer phone | From slv_customers |
| `country` | String | Country name | From slv_customers |
| `country_code` | String | ISO country code | From slv_customers |
| `state` | String | State/province code | From slv_customers |
| `region` | String | Business region mapping | Mapped via country_state_map dictionary |

**Source Tables:**
* `ecommerce.silver.slv_customers`
* Python dictionary: `country_state_map` (7 countries, 50+ states)

**Transformation Logic:**
1. Define regional mapping dictionary: {"India": {"MH": "West", "KA": "South", ...}}
2. Convert dictionary to Spark DataFrame
3. Left join slv_customers with regional mapping on (country, state)
4. fillna with 'Other' for unmapped territories

**Business Value:** Eliminates manual state-to-region mapping in every report

---

#### gld_dim_date

**Business Definition:**  
Business calendar with integer surrogate keys and weekend flags.

**Primary Key:** `date_id`

**Key Columns:**

| Column | Data Type | Description | Enrichment Logic |
|--------|-----------|-------------|------------------|
| `date_id` | Integer | Integer surrogate key (YYYYMMDD) | date_format(date, "yyyyMMdd").cast(Integer) |
| `date` | Date | Date value | From slv_calendar |
| `year` | Integer | Year | From slv_calendar |
| `month_name` | String | Full month name | date_format(date, "MMMM") |
| `day_name` | String | Day of week | From slv_calendar |
| `quarter` | Integer | Fiscal quarter | From slv_calendar |
| `week_of_year` | Integer | Week number | From slv_calendar |
| `is_weekend` | Integer | Weekend flag (1=yes, 0=no) | CASE WHEN day_name IN ('Saturday','Sunday') THEN 1 ELSE 0 |

**Source Tables:**
* `ecommerce.silver.slv_calendar`

**Transformation Logic:**
1. Create date_id as integer (YYYYMMDD format)
2. Add month_name from date
3. Add is_weekend flag (Saturday/Sunday = 1)
4. Reorder columns for usability

**Business Value:** Integer surrogate keys for 3-5x faster BI joins, instant weekend filtering

---

#### gld_fact_order_items

**Business Definition:**  
Business-ready transaction fact with pre-calculated metrics and multi-currency standardization.

**Primary Key:** Composite (`transaction_id`, `seq_no`)

**Key Columns:**

| Column | Data Type | Description | Enrichment Logic |
|--------|-----------|-------------|------------------|
| `transaction_id` | String | Order identifier (renamed from order_id) | From slv_order_items |
| `customer_id` | String | Customer FK | From slv_order_items |
| `product_id` | String | Product FK | From slv_order_items |
| `seq_no` | Integer | Line item sequence (renamed from item_seq) | From slv_order_items |
| `date_id` | Integer | Integer date surrogate key | date_format(transaction_date, "yyyyMMdd").cast(Integer) |
| `transaction_date` | Date | Order date (renamed from dt) | From slv_order_items |
| `transaction_ts` | Timestamp | Order timestamp (renamed from order_ts) | From slv_order_items |
| `quantity` | Integer | Quantity ordered | From slv_order_items |
| `unit_price` | Double | Price per unit | From slv_order_items |
| `gross_amount` | Double | **Pre-calculated metric** | quantity × unit_price |
| `discount_percent` | Double | Discount percentage (renamed) | From slv_order_items |
| `discount_amount` | Double | **Pre-calculated metric** | CEIL(gross_amount × (discount_percent / 100)) |
| `tax_amount` | Double | Tax amount | From slv_order_items |
| `net_amount` | Double | **Pre-calculated metric** (renamed from sale_amount) | gross_amount - discount_amount + tax_amount |
| `net_amount_inr` | Double | **Currency-converted metric** | net_amount × currency_rate (FX rates: USD=88.29, GBP=117.98, etc.) |
| `channel` | String | Sales channel | From slv_order_items |
| `coupon_code` | String | Coupon identifier | From slv_order_items |
| `coupon_flag` | Integer | **Derived flag** | CASE WHEN coupon_code IS NOT NULL THEN 1 ELSE 0 |
| `unit_price_currency` | String | Original transaction currency | From slv_order_items |

**Source Tables:**
* `ecommerce.silver.slv_order_items`
* FX rates table (7 currencies to INR)

**Transformation Logic:**
1. Calculate derived metrics: gross_amount, discount_amount, net_amount
2. Add date_id surrogate key
3. Add coupon_flag binary indicator
4. Join with FX rates to calculate net_amount_inr
5. Rename columns for business clarity (order_id → transaction_id, etc.)

**Business Value:** Pre-calculated metrics eliminate dashboard calculations, multi-currency standardization for global reporting

---

## 4. Column Dictionary

### Key Business Columns

---

#### Identifiers

**`product_id`**
* **Type:** String
* **Definition:** Unique identifier for each product in the catalog
* **Format:** "PROD-" prefix followed by numeric ID
* **Sample Values:** "PROD-12345", "PROD-67890"
* **Used In:** brz_products, slv_products, gld_dim_products, fact tables
* **Business Rule:** Primary key, non-nullable, immutable

**`customer_id`**
* **Type:** String
* **Definition:** Unique identifier for each customer
* **Format:** "CUST-" prefix followed by numeric ID
* **Sample Values:** "CUST-001234", "CUST-567890"
* **Used In:** brz_customers, slv_customers, gld_dim_customers, fact tables
* **Business Rule:** Primary key, non-nullable, assigned on first purchase

**`order_id` / `transaction_id`**
* **Type:** String
* **Definition:** Unique identifier for each order
* **Format:** "ORD-" prefix, year, and sequence
* **Sample Values:** "ORD-2025-001234"
* **Used In:** brz_order_items (order_id), slv_order_items (order_id), gld_fact_order_items (transaction_id)
* **Business Rule:** Primary key component, non-nullable, renamed to transaction_id in Gold for business alignment

**`brand_code`**
* **Type:** String
* **Definition:** Business key for brand identification
* **Format:** Brand name abbreviation with optional numeric suffix
* **Sample Values:** "NIKE01", "ADIDAS02", "SONY-01"
* **Used In:** brz_brand, slv_brands, products tables
* **Business Rule:** Primary key in brand tables, foreign key in products

**`category_code`**
* **Type:** String
* **Definition:** Business key for product category
* **Format:** 3-4 character uppercase abbreviation
* **Sample Values:** "ELEC", "GRCY", "SHOES"
* **Used In:** brz_category, slv_category, brand/product tables
* **Business Rule:** Primary key in category tables, foreign key in brands/products

---

#### Temporal Columns

**`dt` / `transaction_date`**
* **Type:** Date
* **Definition:** Date when the order was placed
* **Format:** YYYY-MM-DD
* **Sample Values:** "2025-08-01"
* **Used In:** Bronze/Silver (dt), Gold (transaction_date)
* **Business Rule:** Used for time-series analysis, partitioning, date joins

**`order_ts` / `transaction_ts`**
* **Type:** Timestamp
* **Definition:** Exact timestamp when order was placed
* **Format:** YYYY-MM-DD HH:MM:SS
* **Sample Values:** "2025-08-01 22:53:52"
* **Used In:** Bronze/Silver (order_ts), Gold (transaction_ts)
* **Business Rule:** Used for hourly analysis, order sequencing

**`date_id`**
* **Type:** Integer
* **Definition:** Integer surrogate key for date dimension (YYYYMMDD format)
* **Format:** 8-digit integer
* **Sample Values:** 20250801, 20251231
* **Used In:** gld_dim_date (PK), gld_fact_order_items (FK)
* **Business Rule:** Optimized for BI tool joins (3-5x faster than date joins)

**`ingested_at` / `_ingested_at`**
* **Type:** Timestamp
* **Definition:** When data was ingested into Bronze layer
* **Format:** YYYY-MM-DD HH:MM:SS
* **Sample Values:** "2025-02-15 08:00:00"
* **Used In:** All Bronze tables, preserved through Silver/Gold
* **Business Rule:** Audit field for data lineage and freshness tracking

---

#### Metrics

**`quantity`**
* **Type:** Integer
* **Definition:** Number of items purchased in this line item
* **Range:** 1 to 999 (typical)
* **Sample Values:** 1, 2, 5
* **Used In:** All fact tables
* **Business Rule:** Non-negative, used in revenue calculations

**`unit_price`**
* **Type:** Double
* **Definition:** Price per unit in original transaction currency
* **Range:** 0.01 to 10,000.00 (typical)
* **Sample Values:** 49.99, 1299.00
* **Used In:** All fact tables
* **Business Rule:** Non-negative, currency-specific (see unit_price_currency)

**`gross_amount`**
* **Type:** Double
* **Definition:** Total revenue before discounts and taxes
* **Calculation:** quantity × unit_price
* **Sample Values:** 99.98, 2598.00
* **Used In:** gld_fact_order_items (pre-calculated)
* **Business Rule:** Always ≥ 0, basis for discount and net calculations

**`discount_pct` / `discount_percent`**
* **Type:** Double
* **Definition:** Discount percentage applied to this line item
* **Range:** 0 to 100 (percentage)
* **Sample Values:** 0, 10.0, 15.0, 20.0
* **Used In:** Bronze/Silver (discount_pct), Gold (discount_percent)
* **Business Rule:** 0 = no discount, 100 = free (rare)

**`discount_amount`**
* **Type:** Double
* **Definition:** Dollar value of discount applied
* **Calculation:** CEIL(gross_amount × (discount_percent / 100))
* **Sample Values:** 0, 5.00, 15.00
* **Used In:** gld_fact_order_items (pre-calculated)
* **Business Rule:** Always ≤ gross_amount

**`tax_amount`**
* **Type:** Double
* **Definition:** Tax charged on this line item
* **Range:** 0 to gross_amount (typical 0-20%)
* **Sample Values:** 0, 3.50, 8.99
* **Used In:** All fact tables
* **Business Rule:** Varies by jurisdiction, may be 0 for tax-exempt items

**`net_amount` / `sale_amount`**
* **Type:** Double
* **Definition:** Final amount customer pays (net revenue)
* **Calculation:** gross_amount - discount_amount + tax_amount
* **Sample Values:** 89.98, 2203.00
* **Used In:** Silver (sale_amount), Gold (net_amount)
* **Business Rule:** This is the recognized revenue for finance

**`net_amount_inr`**
* **Type:** Double
* **Definition:** Net amount converted to INR (base currency)
* **Calculation:** net_amount × currency_rate
* **Sample Values:** 7489.94 (for $89.98 × 83.29)
* **Used In:** gld_fact_order_items
* **Business Rule:** Enables global revenue consolidation, FX rates updated quarterly

---

#### Flags

**`is_weekend`**
* **Type:** Integer (binary)
* **Definition:** Indicates if date falls on weekend
* **Values:** 1 = Saturday or Sunday, 0 = Weekday
* **Used In:** gld_dim_date
* **Business Rule:** Used for weekday vs. weekend analysis

**`coupon_flag`**
* **Type:** Integer (binary)
* **Definition:** Indicates if coupon was used in this transaction
* **Values:** 1 = coupon applied, 0 = no coupon
* **Used In:** gld_fact_order_items
* **Business Rule:** Faster filtering than checking coupon_code IS NOT NULL

---

#### Descriptive Attributes

**`brand_name`**
* **Type:** String
* **Definition:** Display name of the brand
* **Sample Values:** "Nike", "Adidas", "Sony"
* **Used In:** brz_brand, slv_brands, gld_dim_products (enriched)
* **Business Rule:** User-facing display value, may contain spaces

**`category_name`**
* **Type:** String
* **Definition:** Display name of the product category
* **Sample Values:** "Electronics", "Shoes", "Groceries"
* **Used In:** brz_category, slv_category, gld_dim_products (enriched)
* **Business Rule:** User-facing display value, corresponds to category_code

**`channel`**
* **Type:** String
* **Definition:** Sales channel where order was placed
* **Valid Values:** "Website", "Mobile"
* **Used In:** All fact tables
* **Business Rule:** Standardized in Silver ("web"→"Website", "app"→"Mobile")

**`region`**
* **Type:** String
* **Definition:** Geographic business region
* **Valid Values:** "North", "South", "East", "West", "NorthEast", "SouthEast", "Other"
* **Sample Values:** "West", "South"
* **Used In:** gld_dim_customers (derived)
* **Business Rule:** Mapped from (country, state) pairs, aligned with business org structure

---

## 5. Business Metrics Glossary

### Revenue Metrics

---

**Gross Amount**
* **Definition:** Total revenue before any discounts or taxes
* **Formula:** `quantity × unit_price`
* **When to Use:** Product performance analysis, inventory valuation, pricing analysis
* **Example:** 2 units × $49.99 = $99.98 gross_amount
* **Available In:** gld_fact_order_items (pre-calculated)
* **Business Context:** Shows the "sticker price" revenue before promotional discounts

---

**Discount Amount**
* **Definition:** Dollar value of discount applied to the line item
* **Formula:** `CEIL(gross_amount × (discount_percent / 100))`
* **When to Use:** Coupon effectiveness, margin analysis, promotional ROI
* **Example:** $99.98 × (15 / 100) = $14.997, rounded up to $15.00
* **Available In:** gld_fact_order_items (pre-calculated)
* **Business Context:** Quantifies the promotional investment per transaction

---

**Net Amount (Revenue)**
* **Definition:** Final amount customer pays; recognized revenue
* **Formula:** `gross_amount - discount_amount + tax_amount`
* **When to Use:** Revenue reporting, financial reconciliation, executive KPIs
* **Example:** $99.98 - $15.00 + $6.80 = $91.78 net_amount
* **Available In:** slv_order_items (as sale_amount), gld_fact_order_items (as net_amount)
* **Business Context:** This is the revenue number reported to Finance and Executives

---

**Net Amount INR**
* **Definition:** Net amount converted to base currency (INR) for global consolidation
* **Formula:** `net_amount × currency_rate`
* **When to Use:** Multi-currency reporting, global revenue aggregation, investor relations
* **Example:** $91.78 × 88.29 (USD→INR rate) = ₹8,103.20
* **Available In:** gld_fact_order_items
* **Business Context:** Enables apples-to-apples comparison across 7 currencies

**Currency Rates (as of 2025-10-15):**
| Currency | Rate to INR |
|----------|-------------|
| INR | 1.00 |
| USD | 88.29 |
| GBP | 117.98 |
| AUD | 57.55 |
| CAD | 62.93 |
| AED | 24.18 |
| SGD | 68.18 |

---

### Operational Metrics

**Order Volume**
* **Definition:** Count of distinct order_id values
* **Formula:** `COUNT(DISTINCT order_id)`
* **When to Use:** Operations capacity planning, order fulfillment tracking
* **Example:** 1,250 orders placed on 2025-08-01

**Average Order Value (AOV)**
* **Definition:** Average net revenue per order
* **Formula:** `SUM(net_amount) / COUNT(DISTINCT order_id)`
* **When to Use:** Customer value analysis, pricing strategy
* **Example:** $45,000 revenue / 1,250 orders = $36.00 AOV

**Items Per Order**
* **Definition:** Average line items per order
* **Formula:** `COUNT(*) / COUNT(DISTINCT order_id)`
* **When to Use:** Basket analysis, cross-sell effectiveness
* **Example:** 2,500 line items / 1,250 orders = 2.0 items per order

---

### Customer Metrics

**Customer Lifetime Value (CLV)**
* **Definition:** Total net revenue from a customer over all time
* **Formula:** `SUM(net_amount) GROUP BY customer_id`
* **When to Use:** Customer segmentation, marketing spend allocation
* **Example:** Customer CUST-001234 has CLV of $2,450 (sum of all their orders)

**Purchase Frequency**
* **Definition:** Number of orders placed by a customer
* **Formula:** `COUNT(DISTINCT order_id) GROUP BY customer_id`
* **When to Use:** Customer loyalty analysis, retention modeling
* **Example:** Customer has placed 12 orders over 2 years

---

### Product Metrics

**Best Sellers (by Volume)**
* **Definition:** Products with highest total quantity sold
* **Formula:** `SUM(quantity) GROUP BY product_id ORDER BY SUM(quantity) DESC`
* **When to Use:** Inventory planning, merchandising decisions
* **Example:** Product PROD-12345 sold 5,432 units this quarter

**Best Sellers (by Revenue)**
* **Definition:** Products with highest net revenue
* **Formula:** `SUM(net_amount) GROUP BY product_id ORDER BY SUM(net_amount) DESC`
* **When to Use:** Product portfolio optimization, pricing decisions
* **Example:** Product PROD-67890 generated $127,450 revenue this quarter

---

## 6. Sample Queries

### Revenue Analysis

---

**Daily Revenue (Multi-Currency Consolidated)**
```sql
SELECT 
    transaction_date,
    SUM(net_amount_inr) AS total_revenue_inr,
    COUNT(DISTINCT transaction_id) AS order_count,
    SUM(net_amount_inr) / COUNT(DISTINCT transaction_id) AS avg_order_value_inr
FROM ecommerce.gold.gld_fact_order_items
GROUP BY transaction_date
ORDER BY transaction_date DESC
```

**Monthly Revenue Trends with YoY Comparison**
```sql
SELECT 
    YEAR(transaction_date) AS year,
    MONTH(transaction_date) AS month,
    SUM(net_amount_inr) AS monthly_revenue,
    LAG(SUM(net_amount_inr), 12) OVER (ORDER BY YEAR(transaction_date), MONTH(transaction_date)) AS prev_year_revenue,
    (SUM(net_amount_inr) - LAG(SUM(net_amount_inr), 12) OVER (ORDER BY YEAR(transaction_date), MONTH(transaction_date))) / 
        LAG(SUM(net_amount_inr), 12) OVER (ORDER BY YEAR(transaction_date), MONTH(transaction_date)) * 100 AS yoy_growth_pct
FROM ecommerce.gold.gld_fact_order_items
GROUP BY YEAR(transaction_date), MONTH(transaction_date)
ORDER BY year DESC, month DESC
```

**Revenue by Channel**
```sql
SELECT 
    channel,
    SUM(net_amount_inr) AS total_revenue,
    COUNT(DISTINCT transaction_id) AS order_count,
    SUM(quantity) AS total_items_sold
FROM ecommerce.gold.gld_fact_order_items
GROUP BY channel
ORDER BY total_revenue DESC
```

---

### Product Analysis

---

**Top 10 Products by Revenue**
```sql
SELECT 
    p.product_id,
    p.sku,
    p.brand_name,
    p.category_name,
    SUM(f.net_amount_inr) AS total_revenue,
    SUM(f.quantity) AS total_quantity_sold
FROM ecommerce.gold.gld_fact_order_items f
JOIN ecommerce.gold.gld_dim_products p ON f.product_id = p.product_id
GROUP BY p.product_id, p.sku, p.brand_name, p.category_name
ORDER BY total_revenue DESC
LIMIT 10
```

**Product Performance by Category**
```sql
SELECT 
    p.category_name,
    COUNT(DISTINCT p.product_id) AS product_count,
    SUM(f.quantity) AS total_units_sold,
    SUM(f.net_amount_inr) AS total_revenue,
    AVG(f.net_amount_inr / f.quantity) AS avg_price_per_unit
FROM ecommerce.gold.gld_fact_order_items f
JOIN ecommerce.gold.gld_dim_products p ON f.product_id = p.product_id
GROUP BY p.category_name
ORDER BY total_revenue DESC
```

---

### Customer Analysis

---

**Top 20 Customers by Lifetime Value**
```sql
SELECT 
    c.customer_id,
    c.country,
    c.region,
    COUNT(DISTINCT f.transaction_id) AS order_count,
    SUM(f.net_amount_inr) AS lifetime_value_inr,
    AVG(f.net_amount_inr) AS avg_order_value_inr
FROM ecommerce.gold.gld_fact_order_items f
JOIN ecommerce.gold.gld_dim_customers c ON f.customer_id = c.customer_id
GROUP BY c.customer_id, c.country, c.region
ORDER BY lifetime_value_inr DESC
LIMIT 20
```

**Customer Acquisition by Region and Month**
```sql
SELECT 
    c.region,
    DATE_TRUNC('month', f.transaction_date) AS month,
    COUNT(DISTINCT f.customer_id) AS new_customers
FROM ecommerce.gold.gld_fact_order_items f
JOIN ecommerce.gold.gld_dim_customers c ON f.customer_id = c.customer_id
WHERE f.transaction_date >= '2025-01-01'
GROUP BY c.region, DATE_TRUNC('month', f.transaction_date)
ORDER BY month DESC, new_customers DESC
```

---

### Promotional Analysis

---

**Coupon Effectiveness**
```sql
SELECT 
    coupon_code,
    COUNT(DISTINCT transaction_id) AS orders_with_coupon,
    SUM(gross_amount) AS gross_revenue,
    SUM(discount_amount) AS total_discounts_given,
    SUM(net_amount_inr) AS net_revenue,
    AVG(discount_percent) AS avg_discount_pct
FROM ecommerce.gold.gld_fact_order_items
WHERE coupon_flag = 1
GROUP BY coupon_code
ORDER BY total_discounts_given DESC
```

**Weekday vs Weekend Sales**
```sql
SELECT 
    d.is_weekend,
    CASE WHEN d.is_weekend = 1 THEN 'Weekend' ELSE 'Weekday' END AS day_type,
    COUNT(DISTINCT f.transaction_id) AS order_count,
    SUM(f.net_amount_inr) AS total_revenue,
    AVG(f.net_amount_inr) AS avg_order_value
FROM ecommerce.gold.gld_fact_order_items f
JOIN ecommerce.gold.gld_dim_date d ON f.date_id = d.date_id
GROUP BY d.is_weekend
ORDER BY d.is_weekend
```

---

### Data Quality Checks

---

**Check for Orphaned Products (No Brand/Category)**
```sql
SELECT 
    product_id,
    sku,
    brand_name,
    category_name
FROM ecommerce.gold.gld_dim_products
WHERE brand_name = 'Not Available' OR category_name = 'Not Available'
```

**Check for Orders with Zero or Negative Revenue**
```sql
SELECT 
    transaction_id,
    product_id,
    quantity,
    unit_price,
    gross_amount,
    discount_amount,
    net_amount
FROM ecommerce.gold.gld_fact_order_items
WHERE net_amount <= 0
ORDER BY transaction_date DESC
```

---

## 7. Appendix

### Data Refresh Schedule

| Layer | Refresh Frequency | Typical Time | Dependencies |
|-------|------------------|--------------|--------------|
| Bronze Dimensions | Daily | 02:00 AM UTC | Source CSV files |
| Bronze Facts | Real-time | Continuous | Order events |
| Silver Dimensions | Hourly | :15 past hour | Bronze dimensions |
| Silver Facts | Every 15 min | :00, :15, :30, :45 | Bronze facts |
| Gold Dimensions | Hourly | :30 past hour | Silver dimensions |
| Gold Facts | Every 15 min | :05, :20, :35, :50 | Silver facts |

### Contact Information

**Data Engineering Team**
* Email: data-engineering@company.com
* Slack: #data-engineering
* On-call: PagerDuty rotation

**Data Stewards by Domain**
* Products: Product Management Team
* Customers: Customer Success Team
* Orders: Operations Team
* Finance: Finance & Accounting Team

---

**Document Maintenance**

This catalog is maintained by the Data Engineering Team. For updates or corrections, please submit a pull request to the project repository or contact the team directly.

**Last Updated:** February 15, 2025  
**Next Review:** May 15, 2025
