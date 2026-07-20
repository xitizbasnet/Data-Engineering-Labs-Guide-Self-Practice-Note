# LAB 5: Fabric Data Factory — Pipeline Design & Automation

## 📖 Overview

In this lab, you will build an end-to-end **ELT pipeline** using **Microsoft Fabric Data Factory**. The pipeline will copy data from an HTTP/REST source, apply transformations using **Dataflow Gen2**, and load clean data into a **Lakehouse**.

## 🎯 Objective

Build an end-to-end ELT pipeline in Fabric Data Factory that copies data from an HTTP/REST source, applies a dataflow transformation, and lands clean data in a Lakehouse.

## 📚 Key Concepts

| Concept           | Description                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------- |
| **Pipeline**      | Orchestration unit containing activities such as Copy, Dataflow, and Notebook activities. |
| **Copy Activity** | Moves data between 90+ connectors using zero-code configuration.                          |
| **Dataflow Gen2** | Low-code, Power Query-based transformation layer inside Fabric pipelines.                 |
| **Trigger**       | Schedule- or event-based mechanism used to run pipelines automatically.                   |

## 🛠️ Lab Steps

1. Open **Fabric Workspace** → **Data Factory** → **New Pipeline**.
2. Add a **Copy Data** activity:

   * Source = **HTTP connector** (public CSV URL)
   * Sink = **Lakehouse Files**
3. Add a **Dataflow Gen2** activity downstream and apply:

   * Column rename
   * Data type casting
   * Data filtering
4. Connect activities using **success dependency arrows**.
5. Add a **Schedule Trigger**:

   * Frequency: Daily
   * Execution Time: 6:00 AM IST
6. Run the pipeline manually and inspect:

   * Run history
   * Activity execution logs

## 💻 Code Reference

### Pipeline JSON Snippet (Copy Activity)

```json id="m8r4q2"
{
  // Source
  "source": {
    "type": "HttpSource",
    "requestMethod": "GET"
  },

  "sourceDataset": {
    "referenceName": "HttpCSV_Dataset",
    "parameters": {
      "url": "https://example.com/data/orders.csv"
    }
  },

  // Sink
  "sink": {
    "type": "LakehouseTableSink",
    "tableActionOption": "Overwrite"
  }
}
```

### Trigger Definition

```json id="2k8p6v"
{
  "name": "DailyTrigger",
  "type": "ScheduleTrigger",
  "recurrence": {
    "frequency": "Day",
    "interval": 1,
    "startTime": "2025-01-01T00:30:00Z"
  }
}
```

> 📝 **Note**
> Use pipeline parameters to make source URLs and table names dynamic across different environments such as **Dev**, **QA**, and **Prod**.
