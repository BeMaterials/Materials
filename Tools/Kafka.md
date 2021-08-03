# Introduction

## Topics, Partitions, and Offsets

- It is the same as `channel` or `subject` in NATS.
- A topic has a `name`.
- A topic is a particular `stream of data`.
- Topics are split in `partitions`: From partition 0 to ...
- You have to define the number of partitions for each topic when defining a topic (like 10 partitions).
- Each message within a partition gets an incremental id, called `offset`.
- Each partition is independent. So we need to specify message 3 in partition 2 in some topic. In other word, offsets only have a meaning for a specific partition.
- An example of a topic is `truck_gps` (topic is like a table with messages as records).
- `Order` is guaranteed only within a partition (we know that message with offset 2 in a specific partition is later than the message with offset 1) but we cannot say anything about across partitions.
- Data (Messages) is kept only for a limited time (default is one week). But the offsets will be incrementing even after the data has been deleted.
- Once the data is written to a partition, it can't be changed (`immutability`). So it will be appended only.
- Data is assigned `randomly` to a partition unless a `key` is provided.

## Brokers

- A Kafka `cluster` is composed of multiple brokers (servers).
- The default port is `9092`.
- Each broker is identifies with an ID which is an integer (broker 101, broker 102, and broker 103).
- Each broker contains certain topic partitions.
- After connecting to any broker (called a `bootstrap` broker), you will be connected to the entire cluster.
- A good number to get started is `3` brokers, but some big clusters have over 100 brokers.

![](/md/167.jpg)

- When you create a topic with a number of partitions, Kafka will distribute the partitions in different brokers.

## Topic Replication Factor

- Because Kafka is distributed (different brokers in a cluster), we need to replicate each topic (with replication factor usually 2 or 3), to make sure that if a broker goes down, another broker can serve the data.

![](/md/168.jpg)
![](/md/169.jpg)

- At any time, only one broker can be a `leader` for `a given partition`.
- Only the leader, can receive and serve data for a partition.
- The other brokers will synchronize the data.
- Therefore each partition has one leader and multiple ISR (in-sync replicas).
- The leader and the ISR are determined by `Zookeeper` in the background.
- When a leader goes down, the other broker will be the leader and when it comes back, it re-gains the leadership for a specific partition.

![](/md/170.jpg)

## Producers

- Producers write data to topics (which are made of partitions).
- Producers automatically know (so in case of broker failure, producers will automatically recover) to which broker and partition to write to (producer is like a load balancer and will send data to different partitions of a topic in a round-robin way if we send data without a key).
- Producers can choose to receive acknowledgement of data writes:
  - `acks=0`: Producer won't wait for acknowledgement (possible data loss).
  - `acks=1`: Producer will wait for leader acknowledgement (limited data loss).
  - `acks=all`: Producer will wait for leader + replicas acknowledgement (no data loss).

### Message Keys

- Producers can choose to send a `key` with the message (string, number, etc).
- If `key=null`, data is sent round robin to different partitions (and so different brokers).
- If a key is sent, then all messages with the same key go to the `same partition`. Note that, we don't specify the partition.
- Key is used when you need `message ordering` for a specific field (like truck_id):
  > key -> same partition -> message ordering

## Consumers

- Consumers read data from a topic (identified by name). It polls data (long polling).
- Consumers know which broker to read from (it is automatic even in case of broker failure).
- Data is read `in order within each partitions` but `parallel across different partitions`.

### Consumer Groups

- kafka is a mixture of pub/sub (publish once, consume from many consumers) system and a queue system (publish once, consume once -> great for tasks) with the help of consumer group. If you want pure queue, just put all consumers in one group. If you want pure pub/sub, put each consumer in a different consumer group.
- Consumers read data in consumer groups.
- A consumer group is in reality an application like notification service.
- Each partition belongs to only one consumer in a consumer group. So when a message comes to a partition, it will be read only by one consumer in a consumer group.
- If the number of consumers are fewer than the number of partitions, multiple partitions can be assigned to one consumer.
- If you have more consumers than partitions, some consumers will receive no messages. So we don't want that. So if we want more consumers, we should have more partitions because number of consumers is better to be as high as number of partitions.

### Consumer Offsets

- We know that offsets are for partitions and each partition belongs to one consumer in a consumer group; When a consumer in a group has processed data received from Kafka, it should be committing the offsets (which is usually automatic and goes to a topic named `__consumer_offsets`).
- If a consumer dies, it will be read back from where it left off thanks to committed consumer offsets. Note that the committed message is not deleted. So other consumer groups can poll it too.
- Consumers choose when to commit offsets. There are 3 delivery semantics:
  - `At most once`
    - Offsets are committed as soon as the message is received.
    - If the processing goes wrong, the message will be lost.
  - `At least once` (usually preferred)
    - Offsets are committed after the message is processed.
    - If the processing goes wrong, the message will be read again.
    - This can result in duplicate processing of messages. So we have to write our application so that processing again won't impact the system -> `idempotent consumer`.
  - `Exactly once`
    - Can be achieved for Kafka -> Kafka workflows suing Kafka Streams API.

## Kafka Broker Discovery

- You only need to connect to one broker and behind the scene, the client (producer or consumer) will get the list of all brokers, topics and partitions (metadata) and then it can send it to the desired broker.

![](/md/171.jpg)

## Zookeeper

- Kafka can't work without Zookeeper. It manages brokers (keeps a list of them), sends notification to Kafka in case of changes (new topic, delete topic, broker dies, broker comes up, etc.), and elects leader for partitions.
- Zookeeper operates with an odd number of servers.
- Clients do not write or read to/from Zookeeper, they write or read to/from Kafka. kafka uses Zookeeper to manages metadata.
- If we have a cluster of 3 servers Zookeeper, one of them is leader and the rest will be followers. Each broker will be connected to one Zookeeper server. The leader handles writes (for metadata about brokers not messages) and the followers handle reads.

## Kafka Guarantees

- Messages are appended to a topic-partition in the order they are sent.
- Consumers read messages in the order stored in a topic-partition.
- With a replication factor of N, producers and consumers can tolerate up to N-1 brokers being down.
- This is why a replication factor of 3 is a good idea. Because one can be down for maintenance and the other might get down unexpectedly.
- As long as the number of partitions remains constant for a topic (no new partition), the same key will always go to the same partition.

## Roundup

![](/md/172.jpg)

# Setup

- We want to have one Kafka cluster with one broker and one zookeeper in our local. So `docker-compose.yml`:

```yml
version: "3.7"
services:
  registration-svc:
    depends_on:
      - kafka-svc
    build:
      dockerfile: Dockerfile.dev
      context: ./registration
    volumes:
      - ./registration:/app
      - /app/node_modules
    container_name: registration_api

  zookeeper-svc:
    image: bitnami/zookeeper:latest
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    ports:
      - "2181:2181"
    container_name: zookeeper

  kafka-svc:
    depends_on:
      - zookeeper-svc
    image: bitnami/kafka:latest
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper-svc:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
    ports:
      - "9092:9092"
    container_name: kafka
```
