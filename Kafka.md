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