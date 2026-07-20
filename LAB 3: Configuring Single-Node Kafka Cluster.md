# LAB 3: Configuring Single-Node Kafka Cluster

## 📖 Overview

In this lab, you will install Apache Kafka on a single Linux node, create topics, produce and consume messages, and verify consumer offset tracking.

## 🎯 Objective

Install Kafka on a single Linux node, create topics, produce and consume messages, and verify offset tracking.

## 📚 Key Concepts

| Concept       | Description                                                                                                           |
| ------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Broker**    | Kafka server that stores and serves topic partitions.                                                                 |
| **Topic**     | Named category or feed to which records are published.                                                                |
| **Offset**    | Sequential identifier assigned to each record within a partition. Consumers use offsets to track message consumption. |
| **ZooKeeper** | Coordinates broker metadata. (Being replaced by **KRaft** in Kafka 3.x and later.)                                    |

## 🛠️ Lab Steps

1. Install **Java 11** and download **Apache Kafka 3.x** binaries.
2. Start **ZooKeeper**, followed by the **Kafka Broker**.
3. Create a topic named **`orders-topic`** with **3 partitions** and a **replication factor of 1**.
4. Run the console producer to send sample JSON messages.
5. Run the console consumer from **offset 0** to verify the messages.
6. Inspect consumer offsets using **`kafka-consumer-groups.sh`**.

## 💻 Code Reference

### Install & Start

```bash
# Install Java
sudo apt-get install -y openjdk-11-jdk

# Download & Extract Kafka
wget https://downloads.apache.org/kafka/3.7.0/kafka_2.13-3.7.0.tgz

tar -xzf kafka_2.13-3.7.0.tgz
cd kafka_2.13-3.7.0

# Start ZooKeeper & Kafka Broker
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties

bin/kafka-server-start.sh -daemon config/server.properties
```

### Create Topic, Produce & Consume Messages

```bash
# Create Topic
bin/kafka-topics.sh \
  --create \
  --topic orders-topic \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1

# Produce Messages
echo '{"order_id":1,"amount":250}' | \
bin/kafka-console-producer.sh \
  --broker-list localhost:9092 \
  --topic orders-topic

# Consume Messages from Beginning
bin/kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 \
  --topic orders-topic \
  --from-beginning
```

> 📝 **Note**
> For production environments, configure a **replication factor of 3 or greater** across multiple Kafka brokers to provide fault tolerance and high availability.
