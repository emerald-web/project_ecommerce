# E-Commerce Data Platform - Naming Convention

**Version:** 1.0  
**Last Updated:** April 18, 2026  
**Owner:** Data Engineering Team

---

## Overview

This document defines the standardized naming conventions for all data assets in the E-Commerce Analytics Platform. Consistent naming improves discoverability, maintainability, and team collaboration across the medallion architecture (Bronze → Silver → Gold).

---

## Core Principles

1. **Clarity over Brevity** - Names should be self-explanatory
2. **Consistency** - Apply patterns uniformly across all assets
3. **Case Standards** - Use snake_case for technical assets
4. **Layer Identification** - Prefix tables with layer indicators (brz_, slv_, gld_)
5. **No Reserved Words** - Avoid SQL/Python reserved keywords
6. **Descriptive Suffixes** - Add context (e.g., _id, _date, _amount, _flag)

---

## Unity Catalog Structure

### Catalog Naming

**Pattern:** `<domain_name>`

**Example:** `ecommerce`

**Rules:**
- Lowercase, single word or snake_case
- Represents business domain
- No special characters except underscore

### Schema Naming

**Pattern:** `<layer_name>`

**Layers:**
- `bronze` - Raw, immutable data
- `silver` - Cleaned, validated data  
- `gold` - Business-ready, denormalized data
- `source_data` - Raw file storage

---

## Table Naming

### Bronze Layer
**Pattern:** `brz_<entity_name>`

**Examples:**
```
brz_brand, brz_category, brz_products, brz_customers, brz_calendar
brz_order_items
```

### Silver Layer  
**Pattern:** `slv_<entity_name>`

**Examples:**
```
slv_brands, slv_category, slv_products, slv_customers, slv_calendar
slv_order_items
```

### Gold Layer
**Pattern:** `gld_dim_<entity>` or `gld_fact_<entity>`

**Examples:**
```
gld_dim_products, gld_dim_customers, gld_dim_date
gld_fact_order_items
```

---

## Column Naming

### Primary Keys
**Pattern:** `<entity>_id` or `<entity>_code`
```
customer_id, product_id, order_id
brand_code, category_code
```

### Temporal Columns
```
order_date, transaction_date (dates)
order_ts, transaction_ts (timestamps)
ingested_at, processed_time (audit)
date_id (integer surrogate key: YYYYMMDD)
```

### Metrics
```
quantity, unit_price, gross_amount, discount_amount, net_amount
discount_pct (Bronze/Silver), discount_percent (Gold)
weight_grams, length_cm, width_cm
sale_amount_inr (currency-specific)
```

### Flags
```
is_weekend, is_holiday, has_discount, coupon_flag
```

### Audit Columns
```
_source_file, _ingested_at (Bronze)
file_name, ingest_timestamp (alternative)
processed_time (Silver)
```

---

## Notebook Naming

### Folder Structure
```
1_setup/
2_medallion_processing_dim/
3_medallion_processing_fact/
Docs/
```

### Notebook Files
**Pattern:** `<sequence>_<layer>_<purpose>.ipynb`

**Examples:**
```
1_dim_bronze.ipynb
2_dim_silver.ipynb
3_dim_gold.ipynb
1_fact_bronze.ipynb
2_fact_silver.ipynb
3_fact_gold.ipynb
E-commerce setup.ipynb
```

---

## Variable Naming (Python/PySpark)

### DataFrames
```python
df                  # Generic
df_bronze, df_silver, df_gold
orders_df, customers_df
orders_gold_df
```

### Configuration
```python
catalog_name = 'ecommerce'
raw_data_path = '/Volumes/ecommerce/source_data/raw/'
```

### Mappings
```python
fx_rates = {"USD": 88.29}
country_state_map = {"India": {"MH": "West"}}
```

---

## Quick Reference

**Layer Prefixes:**
```
Bronze:  brz_
Silver:  slv_
Gold:    gld_dim_ (dimensions), gld_fact_ (facts)
```

**Common Suffixes:**
```
_id, _code          (identifiers)
_date, _ts, _at     (temporal)
_amount, _pct       (metrics)
_flag, is_, has_    (boolean)
_grams, _cm, _inr   (units)
_name               (vs. _id)
```

**Case Standards:** snake_case for everything

---

## Version History

| Version | Date | Author | Changes |
|---------|------|--------|--------|
| 1.0 | 2026-04-18 | Data Engineering | Initial release |
