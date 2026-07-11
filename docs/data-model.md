# Data Model

## Overview

Project Phoenix follows the Medallion Architecture to transform raw market data into analytics-ready datasets.

```
Yahoo Finance API
        │
        ▼
Bronze Layer (Raw)
        │
        ▼
Silver Layer (Validated & Standardized)
        │
        ▼
Gold Layer (Analytics)
```

---

# Bronze Layer

## Purpose

Store raw stock market data exactly as received from the source.

No business transformations are performed in this layer.

## Example Schema

| Column | Description |
|----------|-------------|
| symbol | Stock ticker |
| trade_date | Trading date |
| open | Opening price |
| high | Highest price |
| low | Lowest price |
| close | Closing price |
| volume | Trading volume |
| ingestion_timestamp | Pipeline ingestion time |

---

# Silver Layer

## Purpose

Clean and standardize the Bronze data.

## Transformations

- Remove duplicates
- Validate symbols
- Validate dates
- Standardize column names
- Convert data types
- Handle missing values
- Calculate daily returns

## Example Schema

| Column | Description |
|----------|-------------|
| symbol | Stock ticker |
| trade_date | Trading date |
| open_price | Opening price |
| high_price | Highest price |
| low_price | Lowest price |
| close_price | Closing price |
| volume | Trading volume |
| daily_return | Daily percentage return |

---

# Gold Layer

The Gold layer contains analytics-ready tables for reporting and visualization.

## Fact Tables

### FactStockPrice

| Column |
|----------|
| company_key |
| date_key |
| open_price |
| high_price |
| low_price |
| close_price |
| volume |
| daily_return |

---

## Dimension Tables

### DimCompany

| Column |
|----------|
| company_key |
| ticker |
| company_name |
| sector |
| exchange |

### DimDate

| Column |
|----------|
| date_key |
| date |
| day |
| month |
| quarter |
| year |

---

# Star Schema

```
                DimDate
                   │
                   │
DimCompany ─── FactStockPrice
```

---

# End-to-End Data Flow

```
Yahoo Finance
      │
      ▼
AWS S3
(Bronze)
      │
      ▼
PySpark
      │
      ▼
Delta Lake
(Silver)
      │
      ▼
dbt
      │
      ▼
Gold Layer
      │
      ▼
Snowflake
      │
      ▼
Streamlit Dashboard
```

---

# Data Quality Rules

- Stock symbol must not be null.
- Trading date must be valid.
- Prices must be greater than or equal to zero.
- Volume must be greater than or equal to zero.
- Duplicate records are removed.
- Invalid records are quarantined.

---

# Design Principles

- Bronze stores raw data.
- Silver applies validation and standardization.
- Gold serves analytics and reporting.
- Every transformation is reproducible.
- Data lineage is preserved.
- Pipelines are modular and maintainable.

---

# Future Enhancements

- Multiple data providers
- NASDAQ Data Link integration
- BSE integration
- Real-time streaming
- Kafka ingestion
- ML feature store