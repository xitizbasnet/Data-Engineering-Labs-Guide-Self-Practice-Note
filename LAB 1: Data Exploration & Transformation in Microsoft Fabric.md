# LAB 1: Data Exploration & Transformation in Microsoft Fabric

## 📖 Overview

In this lab, you will use **Microsoft Fabric** to explore, clean, and transform raw sales data using a **Lakehouse** and a **Notebook** with **PySpark**.

---

## 🎯 Objective

Use Microsoft Fabric's **Lakehouse** and **Notebook** to explore, clean, and transform raw datasets using **PySpark**.

---

## 📚 Key Concepts

| Concept              | Description                                                                    |
| -------------------- | ------------------------------------------------------------------------------ |
| **Microsoft Fabric** | Unified analytics SaaS platform combining Data Factory, Synapse, and Power BI. |
| **Lakehouse**        | Storage layer merging data lake flexibility with data warehouse query power.   |
| **PySpark**          | Python API for Apache Spark — the compute engine in Fabric notebooks.          |

---

## 🛠️ Lab Steps

Follow the steps below to complete the lab.

1. Create a **Fabric Workspace** → **New Lakehouse** → name it **SalesLakehouse**.
2. Upload **`sales_raw.csv`** to the **Files** section of the Lakehouse.
3. Open a new **Notebook** and attach the Lakehouse as the default data source.
4. Load the data into a DataFrame and inspect the schema.
5. Clean null values, cast data types, and add a derived column.
6. Save the cleaned table back to the Lakehouse.

---

## 💻 Code Reference

### Load & Inspect Data

```python
df = spark.read.csv("Files/sales_raw.csv", header=True, inferSchema=True)

df.printSchema()

df.show(5)
```

---

### Clean & Transform Data

```python
from pyspark.sql.functions import col, to_date, when

df_clean = (
    df
    .dropna(subset=["order_id", "amount"])
    .withColumn("order_date", to_date(col("order_date"), "yyyy-MM-dd"))
    .withColumn("amount", col("amount").cast("double"))
    .withColumn(
        "status_flag",
        when(col("amount") > 1000, "High").otherwise("Normal")
    )
)

df_clean.write.format("delta").mode("overwrite").saveAsTable("sales_clean")
```

---

> [!NOTE]
> **Delta format** enables **ACID transactions** and **time-travel queries** on your Lakehouse table.

---

