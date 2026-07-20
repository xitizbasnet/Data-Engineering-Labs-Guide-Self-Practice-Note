# LAB 4: OneLake — Unified Storage in Microsoft Fabric

## 📖 Overview

In this lab, you will explore **OneLake** as the single logical data lake for Microsoft Fabric workspaces. You will create shortcuts to external storage sources and query cross-workspace data without duplicating data.

## 🎯 Objective

Explore OneLake as the single logical data lake for Fabric workspaces; create shortcuts to external sources and query cross-workspace data.

## 📚 Key Concepts

| Concept        | Description                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **OneLake**    | Organization-wide single data lake built into every Microsoft Fabric tenant (similar to OneDrive for data).                                |
| **Shortcut**   | Virtual link that maps external storage sources such as ADLS, Amazon S3, and Google Cloud Storage (GCS) into OneLake without copying data. |
| **Delta Lake** | Open table format used natively by OneLake for ACID transactions, versioning, and schema evolution.                                        |

## 🛠️ Lab Steps

1. Open **Microsoft Fabric** → navigate to your **Workspace** → open **Lakehouse**.
2. Navigate to the **OneLake Data Hub** to view data across all workspaces.
3. Create a shortcut: **Files** → **New shortcut** → **Azure Data Lake Storage Gen2**.
4. Enter the **ADLS account URL** and **container path**, then authenticate.
5. Query the shortcut data using a Fabric Notebook without copying the data.
6. Verify Delta table metadata using `DESCRIBE DETAIL`.

## 💻 Code Reference

### Query via Shortcut

```python id="5n8q6a"
# Shortcut appears as a folder under Files/

df = spark.read.format("delta").load("Files/shortcut_folder/sales_delta/")

df.show(10)


# Cross-workspace query using OneLake path

onelake_path = "abfss://workspace@onelake.dfs.fabric.microsoft.com/lakehouse.Lakehouse/Tables/sales_clean"

df2 = spark.read.format("delta").load(onelake_path)

df2.groupBy("status_flag").count().show()
```

### Inspect Delta Metadata

```sql id="0k3z5c"
-- In Fabric SQL Analytics Endpoint

DESCRIBE DETAIL sales_clean;
```

The command displays metadata information including:

* `numFiles`
* `sizeInBytes`
* `partitionColumns`
* `lastModified`

> 📝 **Note**
> Shortcuts eliminate data duplication. All workspaces can read from the same physical data stored in ADLS Gen2.
