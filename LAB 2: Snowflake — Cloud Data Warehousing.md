# LAB 2: Snowflake — Cloud Data Warehousing

## Overview

In this lab, you will provision a Snowflake trial account, load data using `COPY INTO`, query the data, and implement clustering to improve query performance.

## Objective

Provision a Snowflake trial, load data via `COPY INTO`, query it, and implement clustering for performance.

## Key Concepts

| Concept               | Description                                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------- |
| **Virtual Warehouse** | Compute cluster (XS–4XL) that runs SQL queries; auto-suspends when idle.                    |
| **Stage**             | Named location (internal or Amazon S3/Azure Blob Storage) from which Snowflake loads files. |
| **Clustering Keys**   | Columns Snowflake uses to micro-partition data for faster range queries.                    |

## Lab Steps

1. Sign up at **app.snowflake.com** → select the region closest to your data.
2. Create a database, schema, warehouse, and table.
3. Create an internal stage, upload the CSV file, and run `COPY INTO`.
4. Query the data with aggregations and analyze the query profile.
5. Add a clustering key and run `SYSTEM$CLUSTERING_INFORMATION`.
6. Create a **SECURE VIEW** for role-based access.

## Code Reference

### Setup & Load

```sql
-- Setup
CREATE DATABASE SALES_DB;

CREATE SCHEMA SALES_DB.PUBLIC;

CREATE WAREHOUSE COMPUTE_WH
WITH WAREHOUSE_SIZE='X-SMALL'
AUTO_SUSPEND=60;

CREATE TABLE orders (
    order_id INT,
    customer VARCHAR(100),
    amount FLOAT,
    order_date DATE
);

-- Stage & Load
CREATE OR REPLACE STAGE my_stage;

PUT file:///local/orders.csv @my_stage;

COPY INTO orders
FROM @my_stage
FILE_FORMAT = (
    TYPE = 'CSV'
    SKIP_HEADER = 1
);
```

### Query & Cluster

```sql
-- Aggregation
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY 1
ORDER BY 1;

-- Clustering
ALTER TABLE orders
CLUSTER BY (order_date);

SELECT SYSTEM$CLUSTERING_INFORMATION('orders', '(order_date)');
```

> [!NOTE]
> Auto-suspend and auto-resume ensure that compute resources are billed only while they are actively running.
