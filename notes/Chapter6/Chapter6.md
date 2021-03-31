# 6. Partitioning

### Overview

- **Partitioning** - **For very large datasets, or very high query throughput, that is
not sufficient: we need to break the data up into partitions, also known as sharding.**
- **Partitioning == shard == region == tablet == vnode == vbucket**
- Each partition is a small database of its own as each piece of data belongs to exactly one partition.
- Why? ⇒ To achieve scalability and distribute query load across multiple processors.

### Partitioning and Replication

- Partitionining + replication ⇒ Copies of each partition are stored on multiple nodes.
- One node can store multiple partitions. In a master-slave architecture, we can have each partition's leader assigned to one node and its followers assigned to other nodes. A node can be a leader for one partition and a follower for some other.

    ![C601](../../assets/C601.png)

- **Goal of partitioning ⇒ spread data & query load evenly across nodes**. If every node takes a fair share, in theory, 10 nodes can handle 10x times data and 10x times read and write throughput in comparison to a single node.
- Unfair partitioning results in skewing of either data and queries towards certain partitions. Skewed partition makes the idea of partitioning less effective. In the worst case, all the load of the application can end up on a single node, so 9/10 nodes are idle which can be a performance bottleneck. **Partition with disproportionately high load ⇒ hotspot**
- A really simple approach to avoid hotspots is to assign records to nodes randomly. This creates a good balance between partitions in terms of the distribution of data. However, while trying to access an item, it becomes really difficult to figure out in which partition a particular node lies.
- **Partitioning by key-range:**
    - Assign continous range of keys to each partition ⇒ similar to volumes of paper encyclopedia.
    - If we know the boundaries between ranges, it becomes really easy to query data/key from respective partitions. It also becomes really easy to query an appropriate node, if we know the mapping between a respective partition to a node.
    - The distribution of key ranges is not evenly spread out as the data is not evenly distributed. To distribute data evenly, partition boundaries need to adapt to the data.
    - Used in RethinkDB, MongoDB as the partitioning strategy.
    - Within each partition, keys can be kept in sorted order using SSTables and LSM-Trees. This brings various advantages like easy range scans, allows fetching several related records in a single query with the help of treating key as a concatenated index. Range scans are useful in cases where applications deal with timestamp data. (application having data from a network of sensors)
    - **Advantage: Range scans are easy keeping in mind that keys are evenly distributed.**
    - **Disadvantage: Access patterns can lead to hotspots.**
        - If key is timestamp, then each partition corresponds to each range of time. One partition / day.  Since we write data to the database with primary key as timestamp, all writes will end up going to same partition for the respective day, thereby creating a load on the respective write partition while others sit ideal.
- **Partitioning by hash of key:**
    - key-range partitioning is not a great idea because of skews and hotspots ⇒ Using a hash function to determine partition for key.
    - For partitioning ⇒ hash function needn't be strong. Cassandra and MongoDB use MD5.

        ![C602](../../assets/C602.png)

    - Partition boundaries can be evenly spaced or they can be choosen pseudorandomly(consistent hashing ⇒ a way of evenly distributing load across an internet-wide system of caches such as a content delivery network (CDN). It uses randomly chosen partition boundaries to avoid the need for central control or distributed consensus.)
    - Disadvantage: Not possible to do efficient range queries since keys are scattered because the partitioning is done according to hashes.
    - In real implementations:
        - Range queries on the primary key are not supported by Riak, Couchbase, or Voldermort.
        - MongoDB sends range queries to all the partitions.
        - **Cassandra** has a nice compromise for this. A table in Cassandra can be declared with a compound primary key consisting of several columns. Only the first part of the key is hashed, but the other columns act as a concatenated index. Thus, a query cannot search for a range of values within the first column of a compound key, but if it specifies a fixed value for the first column, it can perform an efficient range scan over the other columns of the key.
- **Skewed workloads and relieving hotspots:**
    - Using hashing on the key to determining partition can avoid hotspots but it can't avoid them entirely. In the worst case, if reads and writes are for the same key; we can end up with all requests on the same partition.
    - **Most data systems are not able to automatically compensate for such a highly skewed workload, so it’s the responsibility of the application to reduce the skew.** For example - if one key is known to be very hot, a simple technique is to add a random number to the beginning or end of the key. **Just a two-digit decimal random number would split the writes to the key evenly across 100 different keys,** **allowing those keys to be distributed to different partitions.**
    - But if we split writes across different keys, we need additional work with reads to read data from all 100 keys and combine it. This needs additional bookkeeping - it only makes sense to append a random number for a small number of hotkeys.
    - For the vast majority of keys with low write throughput, using decimal numbers will be adding unnecessary overhead. There needs to be a way to keep track of which keys are being split.

### **Partitioning and Secondary Indexes**

- Secondary indexes are usually used for searching for occurrences of a particular value rather than uniquely identifying a record. The problem with secondary indexes is that they don’t map neatly to partitions. Two approaches to achieve partitioning a database with secondary indexes.
- **Partitioning Secondary Index by Document:**
    - Here, each partition is completely separate: **each partition maintains its own secondary indexes, covering only the documents in that partition**. [Also known as **scatter/gather**]
    - Whenever you need to write to the database—to add, remove, or update a document—you only need to deal with the partition that contains the document ID that you are writing.

        ![C603](../../assets/C603.png)

    - Since we deal with documents only in the respective partition, the document-partitioned index is called the **local index.**
    - **Challenges**: Reading from a document-partitioned index needs to be done carefully ⇒ You need to query all the partitions and combine all the results. This technique makes read queries on secondary indexes really expensive.
    - Scatter/gather is prone to **Tail latency amplification.** But it is widely used in MongoDB, Riak, Cassandra, Elasticsearch, etc.
- **Partitioning Secondary index by Term:**
    - Rather than each partition having its own secondary index (a local index), we can construct a global index that covers data in all partitions. We can't store the global index on a single node because it can become a bottleneck. So the global index also must be partitioned.

        ![C604](../../assets/C604.png)

    - Here in the above diagram, red cars from all partitions appear under color: red in the index, but the index is partitioned so that colors starting with the letters a to r appears in partition 0 and colors starting with s to z appear in partition The index on the make of car is partitioned similarly (with the partition boundary being between f and h).
    - This kind of indexing scheme is called a term-partitioned index because here the term we are looking for determines the partition of the index. Again, we can partition the index by the term itself or by the hash of the term. Partitioning by the term is useful for range scans and partitioning by the hash of term is useful for even distribution of load.
    - **Advantage: This technique makes reads more efficient as compared to the document-partitioned index.**
    - **Disadvantage: Here, the writes are slower and more complicated because a write to a single document may now affect multiple partitions of the index.**


### **Rebalancing Partitions**

- Process of moving load from one node to cluster ⇒ **rebalancing**
- We need rebalancing when - Query throughput increases, Dataset size increases, Machine fails
- Minimum requirements for rebalancing with any partitioning scheme:
    - Load should be failry shared/distributed between nodes after rebalancing.
    - During rebalancing, the database should continue accepting reads and writes(without any downtime)
    - No more data than necessary should be moved between nodes to make rebalancing fast and minimize network and disk I/O load.
- **How not to do rebalancing: Hash mod n**
    - The problem with rebalancing with **mod n**  technique is the **integer n**. Whenever n changes, all the data has to be rebalanced/moved to its correct node depending on the new n.
- **Fixed number of partitions -**
    - Create many more partitions than nodes and assign several paritions to each node.
    - When a node is added to the cluster, the new node can steal a few partitions from every existing node until partitions are fairly distributed again.

        ![C605](../../assets/C605.png)

    - In principle, you can even account for mismatched hardware in your cluster: by assigning more partitions to nodes that are more powerful, you can force those nodes to take a greater share of the load.
    - Used in Riak, Elasticsearch, Couchbase, Voldemort.
    - Choosing the right number of partitions is difficult if the total size of the dataset is
    highly variable. If the dataset size keeps varying or growing; the partitions have to be rebalanced which is really expensive.
- **Dynamic partitions -**
    - For databases using key-range parititoning; fixed number of partitions with fixed boundaries is really inconvenient ⇒ If we get the boundaries wrong, we end up with all data in one partitions resulting in hotspots.
    - Dynamic partitions used in HBase and RethinkDB. It is well-suited for key-based and hash-based partitioning schemes.
    - Here, each partition is assigned to one node and each node can handle multiple partitions.(Similar to a fixed number of partitions)
    - When the dataset size is varying, dynamic partitioning works really well as the number of partitions can be quickly adapted to total data volume.
    - To mitigate the issue of starting with a single node, few databases like MongoDB and HBase allow pre-splitting.
- **Partitioning proportionally to nodes -**
    - For dynamic partitions ⇒ Number of partitions proportional to the size of dataset. ⇒ splitting and merging processes keep size of each partition between some fixed maxima and minima.
    - For fixed partitions ⇒ Size of each partition proportional to the size of dataset.
    - **In both the above cases, number of partitions is independent to number of nodes.**
    - **Another option is to make number of partitons proportional to number of nodes. Here, each size of partition grows proportionally to dataset size while number of nodes remain unchanged. But when we increase the number of nodes, partitions become smaller again.** This is used by Cassandra and Ketama.
    - When a new node joins the cluster ⇒ it randomly chooses a fixed number of existing partitions to split. Amongst the choosen partitions; the new node takes ownership of one half of split partitions while leaving other half in place.(This is the reason why partitions become smaller when we increase number of nodes) This randomization of partitions can produce unfair splits but when averaged over larger number of partitions, the new node takes a fair share of load from existing nodes.

### Request routing

- When a client wants to make request, how does it know which node it has to connect with? ⇒ As partitions are rebalanced, the assignment of partitions to nodes changes. There has to be a central service that determines the changes in partition to node mapping and someone who can answer the question "from which node ⇒ partition can I read a key `foo` and write a key `bar` to?"
- **This is done by service discovery and it isn't just limited to databases. Any software accessible over a network has this problem(especially if we are aiming for high availability)**
- On a high-level there are few different approaches to this problem -
    - Allow clients to contact any node. If that particular node has the partition to which the request applies; it can handle the request itself and send the response to client'; else - forward the request to appropriate node.
    - Send all requests from client to a routing tier first which determines which node has the appropriate partition from where the request can be handled. (Similar to load balancer)
    - Require that clients be aware of partitioning and mapping between partitions and nodes.

        ![C606](../../assets/C606.png)

- In all the afore-mentioned cases, the key problem is how does the component make routing decisions. (keeping in mind changes in the assignment of partitions)
- Zookeeper has become a de-facto standard coordination service to keep track of mapping between node and partition.