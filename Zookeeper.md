# What is Zookeeper
- Zookeeper provides multiple features for distributed applications
    -   Distributed configuration management
    -   Self election / consensus building
    -   Coordination and locks
    -   Key value store
- Zookeeper is used in many distributes systems, such as Hadoop, Kafka
- It's an Apache Project that's proven to be very stable and hasn't had major relase in many years

- Zookeeper internal data structure is like a tree.
- Each node is called a zNode
- Each zNode has a path
- zNode can be persistent or ephemeral
- Each zNode can store data
- No renaming of zNode
- Each zNode can be WATCHed for changes. 

# Role of Zookeeper in Kafka
-   Brokers registration, with heartbeats mechanism to keep the list current
-   Maintaining a list of topics along with
    -   Their configuration ( partitions, replication factor, additional configurations...)
    -   The list of ISRs for partitions.
-   Performing leader elections in case some brokers go down
-   Storing the kafka cluster id ( randomly generated at 1st startup of cluster )
-   Storing ACLs ( Access Control Lists ) if security is enabled
    -   Topics
    -   Consumer Groups
    -   Users
-   Quotas configuration if enabled.

# Zookeeper Architecture Quorum sizing
-   Zookeeper needs to have a strict majority of servers up to form a strict majority when votes happen
-   Therefore Zookeeper quorums have 1,3,5,7,9,(2N+1) servers
-   This allows for 0,1,2,3,4,N Servers to go down.

### Size:1
Pros:
-   Fast to put in place
-   Fast to make decisions
Cons:
-   Not resilient
-   Not distributed

### Size:3
Pros:
-   Distributed
-   Preferred for small Kafka deployments
Cons:
-   Only one server can go down

### Size:5
Pros:
-   Used for big Kafka deployments ( Likendln, Netflix...)
-   Two servers can go down
Cons:
-   Need performant machines as lot of network calls includes.Zookeeper is very sensitive to network lags.

### Size:7
Pros:
-   Not much more than 5 servers setup except 3 servers can go down
Cons:
-   Network overhead and lag may affect Zookeeper performance

