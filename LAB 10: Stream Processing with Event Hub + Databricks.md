# LAB 10: Stream Processing with Event Hub + Databricks

## 📖 Overview

In this lab, you will use **Azure Databricks Structured Streaming** to consume messages from **Azure Event Hubs**, apply real-time transformations, and write the processed results into **Delta Lake** with exactly-once processing semantics.

## 🎯 Objective

Use Azure Databricks Structured Streaming to consume Event Hub messages, apply transformations, and write results to Delta Lake with exactly-once semantics.

## 📚 Key Concepts

| Concept                  | Description                                                                                                           |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| **Structured Streaming** | Spark's scalable and fault-tolerant stream processing API built on micro-batch execution.                             |
| **Delta Lake**           | ACID-compliant storage layer that supports streaming writes with checkpointing.                                       |
| **Checkpointing**        | Databricks feature that persists streaming offsets to ADLS, allowing jobs to recover exactly from where they stopped. |
| **Trigger Interval**     | Controls micro-batch execution frequency, such as `ProcessingTime('30 seconds')` or `Trigger.Once()`.                 |

## 🛠️ Lab Steps

1. Create an **Azure Databricks workspace** and attach an Event Hub-compatible cluster library:

   * `azure-eventhubs-spark`
2. Store the Event Hub connection string in a **Databricks Secret Scope**.
3. Write a Structured Streaming notebook:

   * Read streaming data
   * Apply transformations
   * Write results to Delta Lake
4. Enable checkpointing using an **ADLS Gen2** path.
5. Query the streaming Delta table in real time.
6. Monitor the streaming process in:

   * Databricks Spark UI
   * Streaming tab

## 💻 Code Reference

### Read from Event Hub

```python id="n8w3kf"
# Install:
# com.microsoft.azure:azure-eventhubs-spark_2.12:2.3.22

eh_conf = {
    "eventhubs.connectionString": sc._jvm.org.apache.spark.eventhubs \
        .EventHubsUtils.encrypt(
            dbutils.secrets.get("kv-scope", "eh-conn-str")
        ),

    "eventhubs.consumerGroup": "$Default",

    "eventhubs.startingPosition":
        '{"offset":"-1","seqNo":-1,"enqueuedTime":null,"isInclusive":true}'
}

raw_stream = (
    spark.readStream
    .format("eventhubs")
    .options(**eh_conf)
    .load()
)
```

### Transform & Write to Delta

```python id="q5m2vz"
from pyspark.sql.functions import from_json, col, current_timestamp
from pyspark.sql.types import StructType, StringType, DoubleType

schema = (
    StructType()
    .add("deviceId", StringType())
    .add("temperature", DoubleType())
    .add("ts", DoubleType())
)

parsed = (
    raw_stream
    .withColumn(
        "body",
        from_json(col("body").cast("string"), schema)
    )
    .select(
        "body.*",
        current_timestamp().alias("ingest_time")
    )
    .filter(col("temperature").isNotNull())
)

query = (
    parsed.writeStream
    .format("delta")
    .outputMode("append")
    .option(
        "checkpointLocation",
        "abfss://data@.dfs.core.windows.net/checkpoints/telemetry"
    )
    .trigger(processingTime="30 seconds")
    .table("telemetry_stream")
)

query.awaitTermination()
```

> 📝 **Note**
> Use Delta Lake **OPTIMIZE** and **ZORDER** on the `deviceId` column after batch loads to improve device-level query performance.
