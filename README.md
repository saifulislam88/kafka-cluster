# kafka-cluster



- Kafka Streams
- Kafka REST Proxy 
- Kafka Consumer Group and Offset Management


### Kafka Cluster Configuration Methods: ZooKeeper vs KRaft

Kafka offers two main ways to configure a cluster: **ZooKeeper** and **KRaft** (Kafka Raft). Here’s a simplified explanation of each:

---

#### 1. **Kafka with ZooKeeper (Traditional Method)**

- **What is it?**  
  ZooKeeper is an external service used for **coordination and metadata management**. It helps Kafka brokers keep track of each other, manage topics, partitions, and Handles leader election, partition assignment, and   
  ensures fault tolerance. Keeps track of Kafka broker details like which brokers are alive, leader election, etc. and manages information about topics, partitions, and their respective leaders.
  
- **Why use it?**  
  - **Mature & Widely Used**: It’s been the default method for years.
  - **Compatibility**: Works with all Kafka tools and clients.
  - **Proven Stability**: It’s a well-tested and reliable system.

- **Downsides**:  
  - **Complexity**: Requires managing both Kafka and ZooKeeper clusters.
  - **Scalability**: As the Kafka cluster grows, managing ZooKeeper can become challenging.

---

#### 2. **Kafka with KRaft (Raft Consensus Mode)**

- **What is it?**  
  **KRaft** is Kafka’s new internal metadata management system using the **Raft consensus protocol**. KRaft stands for Kafka Raft and is the newer, simplified mode for managing Kafka clusters. It removes the need for ZooKeeper entirely by using Kafka’s built-in Raft consensus protocol to manage metadata and coordination tasks directly within Kafka itself. 

- **Why use it?**  
  - **Simpler Setup**: No need to manage a separate ZooKeeper instance.
  - **Scalability**: Kafka handles its own metadata, making it easier to scale.
  - **Better Performance**: Faster leader election and recovery compared to ZooKeeper.

- **Downsides**:  
  - **Newer**: It’s still evolving and might not yet support all Kafka features.
  - **Less Compatibility**: Older Kafka tools may rely on ZooKeeper.

---

#### **Which to Choose?**

- **Use ZooKeeper** if you're working with **existing Kafka clusters** or need **compatibility** with older tools.
- **Use KRaft** for **new Kafka deployments** to simplify your architecture and take advantage of better scalability and performance.


---
### Asking Confused Questions |💡| BraninStromning❓

- Q. Can a Developer Create Topics from the Application Code?

    **Yes,** developers can create topics from their application code using KafkaAdminClient in Java or other client libraries depending on your language, although it’s not always recommended for production environments 
             due to potential for mismanagement or accidental topic creation.
  
- Q. Understanding whether Kafka, ZooKeeper, and Schema Registry are stateful or stateless, and which of these components require a quorum for high availability ?

    ## Kafka Brokers
    - **Stateful**: Kafka brokers store messages and offsets on disk.
    - **Requires Quorum**: Kafka requires quorum for replication (to ensure fault tolerance) and leader election during partition management.
    
    ## ZooKeeper
    - **Stateful**: ZooKeeper stores coordination metadata like broker information and leader election details.
    - **Requires Quorum**: ZooKeeper requires a quorum for consistency, distributed consensus, and fault tolerance.
    
    ## Schema Registry
    - **Stateful | Stateless**: Schema Registry is stateless in terms of handling requests but stores schemas in Kafka (which is stateful).
    - **Does Not Require Quorum**: While it relies on Kafka for storage, Schema Registry itself does not require a quorum for operations.


    | Component       | Stateful or Stateless?                       | Requires Quorum?                                                         |
    |-----------------|----------------------------------------------|--------------------------------------------------------------------------|
    | Kafka Brokers   | Stateful (stores message data)               | Requires quorum for replication and leader election.                     |
    | ZooKeeper       | Stateful (stores coordination metadata)      | Requires quorum for distributed consensus and fault tolerance.           |
    | Schema Registry | Stateless (depends on Kafka for storage)      | Does not require quorum but depends on the Kafka cluster's availability. |
