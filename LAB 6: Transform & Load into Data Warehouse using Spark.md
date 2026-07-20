# LAB 6: Transform & Load into Data Warehouse using Spark

## 📖 Overview

In this lab, you will read raw Delta files from a **Microsoft Fabric Lakehouse**, apply business transformations using **PySpark**, and write the processed results into **Fabric SQL Data Warehouse (Synapse) tables**.

## 🎯 Objective

Read raw Delta files from Lakehouse, apply business transformations via PySpark, and write the results into Fabric's SQL Data Warehouse (Synapse) tables.

## 📚 Key Concepts

| Concept                    | Description                                                                                        |
| -------------------------- | -------------------------------------------------------------------------------------------------- |
| **Fabric Data Warehouse**  | Fully managed T-SQL-compatible warehouse inside Microsoft Fabric with no infrastructure to manage. |
| **Spark → Data Warehouse** | Use Fabric's built-in connector to write Spark DataFrames directly into Data Warehouse tables.     |
| **Partitioning**           | Distributes data across nodes to improve parallel query execution and data loading performance.    |

## 🛠️ Lab Steps

1. Read the cleaned Delta table from the Lakehouse into a Spark DataFrame.
2. Apply window functions:

   * Running total
   * Rank by revenue
3. Aggregate data to monthly grain and filter top customers.
4. Write the transformed results to Fabric Data Warehouse using the `synapsesql` connector.
5. Validate record counts in Data Warehouse using the SQL Analytics Endpoint.

## 💻 Code Reference

### Transform with Window Functions

```python id="7h2k9p"
from pyspark.sql.functions import sum, rank, col, date_trunc
from pyspark.sql.window import Window

df = spark.read.format("delta").load("Tables/sales_clean")

w = Window.partitionBy("customer").orderBy("order_date")

df_ranked = df.withColumn(
    "running_total",
    sum("amount").over(w)
)

monthly = (
    df
    .withColumn("month", date_trunc("month", col("order_date")))
    .groupBy("month", "customer")
    .agg(sum("amount").alias("monthly_revenue"))
)

w2 = Window.partitionBy("month").orderBy(
    col("monthly_revenue").desc()
)

top_customers = (
    monthly
    .withColumn("rank", rank().over(w2))
    .filter(col("rank") <= 10)
)
```

### Write to Fabric Data Warehouse

```python id="x4m7qa"
top_customers.write \
.format("com.microsoft.synapse.spark.sql.DefaultSource") \
.option("server", "your-workspace.datawarehouse.fabric.microsoft.com") \
.option("database", "SalesDW") \
.option("dbtable", "dbo.top_customers_monthly") \
.mode("overwrite") \
.save()
```

> 📝 **Note**
> Always run **statistics updates** on Data Warehouse tables after bulk loading data to enable the query optimizer to generate efficient execution plans.
