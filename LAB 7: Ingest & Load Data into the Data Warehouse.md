# LAB 7: Ingest & Load Data into the Data Warehouse

## 📖 Overview

In this lab, you will implement a complete data ingestion pipeline by landing files in a **Microsoft Fabric Lakehouse**, bulk-loading data into a **Fabric Data Warehouse** using `COPY INTO`, and validating the loaded data using row counts and constraint checks.

## 🎯 Objective

Implement a full ingestion pipeline: land files in Lakehouse, use `COPY INTO` to bulk-load a Fabric DW table, then validate with row counts and constraint checks.

## 📚 Key Concepts

| Concept            | Description                                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------------------------- |
| **COPY INTO**      | T-SQL bulk load command and the fastest method to ingest files from Lakehouse or ADLS into Data Warehouse. |
| **External Table** | Virtual table pointing to files stored in the Lakehouse; no physical data movement is required.            |
| **Staging Table**  | Temporary landing table where data is validated before being promoted to production tables.                |

## 🛠️ Lab Steps

1. Upload **`orders_2024.parquet`** to the Lakehouse → **Files/raw/**.
2. In the **Data Warehouse SQL Endpoint**, create a staging table matching the Parquet schema.
3. Run `COPY INTO` from the OneLake path into the staging table.
4. Validate the loaded data:

   * Row count verification
   * Null check on primary key
   * Date range validation
5. Merge staging data into the production table using the **UPSERT pattern**.
6. Truncate the staging table after a successful merge.

## 💻 Code Reference

### COPY INTO Staging Table

```sql id="a7n2kx"
-- Create Staging Table
CREATE TABLE dbo.stg_orders (
    order_id INT,
    customer VARCHAR(200),
    amount DECIMAL(18,2),
    order_date DATE
);

-- Bulk Load from OneLake
COPY INTO dbo.stg_orders
FROM 'https://onelake.dfs.fabric.microsoft.com//.Lakehouse/Files/raw/orders_2024.parquet'
WITH (
    FILE_TYPE = 'PARQUET'
);

-- Validate Data
SELECT COUNT(*) AS total_rows
FROM dbo.stg_orders;

SELECT COUNT(*) AS nulls
FROM dbo.stg_orders
WHERE order_id IS NULL;
```

### UPSERT to Production Table

```sql id="m4q8vz"
MERGE dbo.orders AS tgt
USING dbo.stg_orders AS src
ON tgt.order_id = src.order_id

WHEN MATCHED THEN
    UPDATE SET
        tgt.amount = src.amount,
        tgt.order_date = src.order_date

WHEN NOT MATCHED THEN
    INSERT (
        order_id,
        customer,
        amount,
        order_date
    )
    VALUES (
        src.order_id,
        src.customer,
        src.amount,
        src.order_date
    );

TRUNCATE TABLE dbo.stg_orders;
```

> 📝 **Note**
> Run `UPDATE STATISTICS` after every major data load to keep the query optimizer accurate and improve query performance.
