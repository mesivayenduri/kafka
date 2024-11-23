## Kafka Topics
- Topics are a particular stream of data in kafka cluster.
- Similar to a table in database.
- You can have as many topics you want.
- Identified by its name.
- Any kind of message format.
- Sequence of messages is called Data Stream
- Producers can Send/Write Data to Topics. Consumers can Read/Consume data from Topics

## Partitions And Offsets
- Topics are split into Partitions
- Messages within each partition are ordered
- Each message within a partition gets an incremental id, called Offset.
- Kafka Topics are IMMUTABLE. Once data is written to a partition, it cannot be changed.

## Producers:
-   Producers writes data to topics ( which are made of partitions )
-   Producers know to which partition to write to ( and which kafka broker has it)
-   In case of kafka broker failures, Producers will automatically recover

## Producers: Message Keys
-   Producers can choose to send a key with the message ( string, number, binary etc )
-   If key = null, data is sent round robin
-   If key != null, then all messages for that key will always go the same partition ( with the help of hashig )
-   A key is typically sent if you need message ordering for a specific field ( ex: sku_id )

## Consumers:
-   Consumers read data from a topic ( identified by name) - Pull model
-   Consumers automatically know which brokers to read from
-   In case of broker failures, consumers know how to recover.
-   Data is read in order from low to high offset within each partition

## Consumer Groups:
-   All the consumers in an application read data as a consumer groups.
-   Each consumer within a group reads from exclusive partitions.

## Consumer offsets:
-   Kafka stores the offsets at which a consumer groups has been reading.
-   The offsets committed are in kafka topic named __consumer_offsets
-   When a consumer in a group has processed data received from kafka, it should be periodically committing the offsets (the kafka broker will write to __consumer_offets, not the group itself)
-   If a consumer dies, it will be able to read back from where it left off thanks to the committed consumer offsets!

## Delivery semantics for consumers:
-   By default, Java consumers will automatically commits offsets ( at least once )
-   There are 3 delivery semantics if you choose to commit manually
    -   At least once (Usually preferred)
        -   Offsets are committed after the message is processed.
        -   If the processing goes wrong, the message will be read again.
        -   This can result in duplicate processing of messages. Make sure your processing is idempotent.
    -   At most once
        -   Offsets are committed as soon as messages are received.
        -   If the processing goes wrong, some messages will be lost(they won't be read again)
    -   Exactly Once
        -   For kafka -> Kafka workflow: use the transaction API (easy with kafka streams API)
        -   For Kafka -> External System workflows: use an idempotent consumer.

## Kafka Brokers
-   A kafka cluster is composed of multiple brokers (servers)
-   Each broker is identified with its ID.
-   Each broker contain certain topic partitions.
-   After connecting to a broker ( called bootstrap server ), you will be connected to the entire cluster.
-   A good number to get started is 3 brokers, but some big clusters have over 1000 brokers.

## Kafka Broker Discovery
-   Every kafka broker is also called a "Bootstrap server"
-   That means that you only need to connect to one broker, and the kafka clients will know how to be connected to the entire cluster.
-   Each broker know about all brokers, topics and partitions ( metadata )

## Topic Replication Factor
-   Topics should have a replication factor > 1 (usually between 2 and 3)
-   This way if a broker is down, another broker can server the data.

## Default Producer & Consumer behaviour with leaders
-   Kafka producers can only write to the leader broker for a partition.
-   Kafka consumers can only read from the leader broker for a partition.
-   Since Kafka 2.4, it is possible to configure consumers to read from the closed replica ( Kafka consumers replica fetching feature )
    This may help improve latency, and also decrease network costs if using in cloud.

## Producer Acknowledgements (ACKS)
-   Producers can choose to receive the acknowledgements of data writes:
    -   acks=0 : Producers won't wait for the acknowledgment ( possible data loss )
    -   acks=1 : Producers will wait for the leader acknowledgement ( limited data loss )
    -   acks=all : Producers will wait for the leader and follower(replicas) acknowledgement (No data loss)

## Kafka topic durability
-   For a topic replication factor of 3, topic data durability can withstand 2 brokers loss.
-   As a rule, for a replication factor of N, you can permanently lose up to N-1 brokers and still recover your data.

## Brokers
-   Brokers hold topic partitions
-   Brokers receive and serve data
-   Brokers are unit of parallelism of a kafka cluster
-   Brokers are the essence of the "distributed" aspect of kafka


## Sizing

### 1 broker
In case of only 1 Broker:
-   If it restarted, the kafka cluster is down.
-   The maximum replication factor for topics is 1
-   All producer and consumer requests go to the same unique broker
-   You can only scale vertically.
-   Its extremely high risk, and only useful for development purposes

### 3 brokers
In case of 3 Brokers:
-   N-1 brokers can be down, If N is your default topic replication factor.
-   Eg: If N=3, then two brokers can go down.
-   Producer and consumer requests are distributed among 3 brokers.
-   Data is spread out between brokers, which means less disk space is used per broker.
-   You can have a cluster.

### 100 brokers
In case of 100 brokers:
-   Your cluster is fully distributed and can handle tremendous volumes, even using commodity hardware.
-   Zookeeper can be under pressure because of many open connections, so you need to increase zookeeper instance performace.
-   It is not recommended to have a repication factor of 4 or more, and this would incur network communications within the brokers. So leave it as 3.
-   Scale horizontally only when a bottleneck is reached ( network, io, cpu, disk and ram)