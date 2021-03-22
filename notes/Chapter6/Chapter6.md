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