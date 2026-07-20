# LAB 9: Real-Time Stream Processing with Azure Stream Analytics

## 📖 Overview

In this lab, you will ingest live IoT and click-stream data using **Azure Event Hubs**, process the incoming streams using **Azure Stream Analytics SQL**, and send real-time outputs to **Azure SQL Database** and **Power BI** for monitoring and visualization.

## 🎯 Objective

Ingest live IoT/click-stream data via Event Hub, process with Stream Analytics SQL (tumbling windows), and output alerts to Azure SQL and Power BI.

## 📚 Key Concepts

| Concept              | Description                                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Event Hub**        | Managed Kafka-compatible ingestion service capable of processing millions of events per second.                  |
| **Stream Analytics** | Fully managed real-time SQL analytics engine for processing streaming inputs.                                    |
| **Tumbling Window**  | Non-overlapping fixed-duration time window used for aggregations, such as counting events per 1-minute interval. |
| **Hopping Window**   | Overlapping time window useful for rolling averages and continuous analytics.                                    |

## 🛠️ Lab Steps

1. Create an **Event Hub Namespace** and an Event Hub named **`telemetry-hub`**.
2. Create an **Azure Stream Analytics Job**:

   * Input = Event Hub
   * Output = Azure SQL Database + Power BI Dataset
3. Write a Stream Analytics query using:

   * Tumbling window aggregation
   * Alert conditions
4. Send sample JSON events using:

   * Event Hub Data Generator
   * Azure SDK
5. Start the Stream Analytics job and verify:

   * Output records in SQL Database
   * Live Power BI dashboard tile
6. Scale **Streaming Units (SU)** based on throughput requirements and monitor watermark delay.

## 💻 Code Reference

### Stream Analytics SQL

```sql id="p8w4qa"
-- Tumbling window: events per device per minute

SELECT
    deviceId,
    System.Timestamp() AS window_end,
    COUNT(*) AS event_count,
    AVG(temperature) AS avg_temp,
    MAX(temperature) AS max_temp
INTO [sql-output]
FROM [eventhub-input]
TIMESTAMP BY EventEnqueuedUtcTime
GROUP BY
    deviceId,
    TumblingWindow(minute, 1);


-- Alert: high temperature

SELECT
    deviceId,
    temperature,
    EventEnqueuedUtcTime
INTO [alert-output]
FROM [eventhub-input]
WHERE temperature > 85;
```

### Python Producer — Send Test Events

```python id="m7z2kp"
from azure.eventhub import EventHubProducerClient, EventData
import json
import random
import time

client = EventHubProducerClient.from_connection_string(
    conn_str="",
    eventhub_name="telemetry-hub"
)

with client:
    for _ in range(100):
        batch = client.create_batch()

        batch.add(EventData(json.dumps({
            "deviceId": f"device-{random.randint(1,5)}",
            "temperature": round(random.uniform(60, 95), 1),
            "ts": time.time()
        })))

        client.send_batch(batch)

        time.sleep(0.5)
```

> 📝 **Note**
> One Streaming Unit (SU) provides approximately **1 MB/s throughput**. Start with **3 SUs** and enable auto-scale for workloads with variable traffic patterns.
