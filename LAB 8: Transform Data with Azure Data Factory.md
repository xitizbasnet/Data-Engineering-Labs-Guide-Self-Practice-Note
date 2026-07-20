# LAB 8: Transform Data with Azure Data Factory

## 📖 Overview

In this lab, you will build an **Azure Data Factory (ADF)** pipeline using **Mapping Data Flows** to join data from multiple sources, apply transformations, and load the processed results into an **Azure SQL Database**.

## 🎯 Objective

Build an ADF pipeline using Mapping Data Flows to join two sources, apply transformations, and sink the result into Azure SQL Database.

## 📚 Key Concepts

| Concept                 | Description                                                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Mapping Data Flow**   | Visual ETL canvas in Azure Data Factory that provides code-free data transformation and runs on Apache Spark internally. |
| **Integration Runtime** | Compute infrastructure used by ADF activities, including Azure Integration Runtime and Self-hosted Integration Runtime.  |
| **Linked Service**      | Connection definition that can be reused across datasets and pipeline activities.                                        |

## 🛠️ Lab Steps

1. Create an **Azure Data Factory** resource in the Azure Portal and author a pipeline.

2. Create Linked Services:

   * Azure Blob Storage
   * Azure SQL Database

3. Add a **Mapping Data Flow** activity.

4. In the dataflow canvas, configure the following flow:

   **Source (Blob CSV)** → **Join (SQL Lookup Table)** → **Derived Column** → **Sink (SQL Database)**

5. Configure a filter to exclude rows where:

   ```text
   amount < 0
   ```

6. Debug the dataflow using a **50-row sample**, then publish and trigger the pipeline.

## 💻 Code Reference

### Data Flow Expression — Derived Column

```text id="c8y5zp"
// In ADF Data Flow Derived Column transformation:

status_flag = iif(amount > 1000, 'High', 'Normal')

year_month = toString(orderDate, 'yyyy-MM')

profit_margin = toDecimal((revenue - cost) / revenue * 100, 10, 2)
```

### ADF Pipeline ARM Snippet (Trigger)

```json id="v3k8sd"
{
  "name": "WeeklyETLTrigger",
  "properties": {
    "type": "ScheduleTrigger",
    "typeProperties": {
      "recurrence": {
        "frequency": "Week",
        "interval": 1,
        "schedule": {
          "weekDays": [
            "Monday"
          ]
        },
        "startTime": "2025-01-06T01:00:00Z"
      }
    },
    "pipelines": [
      {
        "pipelineReference": {
          "referenceName": "MainETLPipeline"
        }
      }
    ]
  }
}
```

> 📝 **Note**
> Use Data Flow debug clusters sparingly. They start a **4-core Spark cluster** for execution and may incur additional compute costs.
