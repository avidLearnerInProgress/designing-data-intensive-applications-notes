# 5. Replication

### Overview

- Replication - Keeping a copy of the same data on multiple machines connected via a network.
    - Why? - Keeping data geographically close to users(reduce latency). Allow the system to continue working even if some parts have failed(high availability).  Scale-out a number of machines that serve read queries. (increased throughput)
    - Here we assume that the database is so small that each machine can hold it. If the data that you are replicating doesn't change over time, we just have to replicate/copy data to every node once.
    - When data is changing, it becomes difficult to implement replication. There are 3 popular algorithms for replication to be looked into - single leader, multi leader, and leaderless.
    - Many trade-offs need to be considered for replication - synchronous vs asynchronous replication; how to handle failed replicas, etc.

### Leaders and Followers

- Each node stores a copy of the database called a replica. With multiple replicas, how to ensure that there is consistency between both the databases is a primary concern. Every write must be followed/processed by a replica.
- **Leader-based replication(master-slave replication) -**
    - One replica ⇒ leader.  Whenever clients want to write to the database, they send requests to the leader which first writes data to its local storage.
    - Other replicas ⇒ followers. The leader also sends the changed data to all of its followers other than writing it to its local storage. (This is called change stream/replication log) Each follower takes the log from the leader and updates its local copy of the database accordingly.
    - When the client wants to read the database it can either query the leader or any of its followers. Writes have to go through the leader.

        ![C501.png](../../assets/C501.png)

    - This mode of replication is a built-in feature for most relational databases like PostgreSQL, MySQL, SQL Server, etc. It is also used in MongoDB, RethinkDB, Espresso.
    - Leader Based replication is also employed in Distributed Message Brokers like Kafka and RabbitMQ.

### Synchronous vs Asynchronous replication

- Scenario - When a user wants to update profile picture in the database. A leader with a synchronous and an asynchronous follower -

    ![C502.png](../../assets/C502.png)

- The replication to follower 1 is synchronous. Leader waits until follower 1 has confirmed that it received the write before reporting success to the user.  The replication to follower 2 is asynchronous.  The leader sends a message but doesn't wait for a response from the follower.
- Normally replication is very fast and most database systems apply changes in less than a second.
- Synchronous replication guarantees that follower has an up-to-date copy of data consistent with the leader. Disadvantage is that if the follower doesn't respond back, the write cant be processed.
- **Semi-synchronous** - In general scenarios, it is impossible for all the followers to be synchronous. Even a single node outage would cause the whole system to halt. In practice ⇒ synchronous replication on a db ⇒ one of the followers is synchronous.  The above configuration guarantees that you have at least two machines that are always up to date(leader and one sync follower).
- **Often leader-based replication is configured asynchronously. The write is not guaranteed to be durable.**  However, a fully asynchronous configuration has the advantage that the leader can continue processing writes, even if all of its followers have fallen behind.
- **Weakening durability may sound like a bad trade-off, but asynchronous replication is nevertheless widely used.**

### Setting up new followers

- Setting up new followers without downtime can be done as -
    - Take a consistent snapshot of the leader's database at some point in time (if possible) without taking a lock on the entire database.
    - Copy snapshot to a new follower node.
    - **Follower connects to the leader and requests all data changes that have happened since the snapshot was taken. The snapshot has to be associated with the exact position in the leader's replication log. (log sequence numbers/ binlog coordinates)**
    - When the follower has processed a backlog of data changes since the snapshot, it means that it is on the same page as that of the leader.

### Handling new outages

- Achieving high availability with leader-based replication -
    - **Follower failure: Catch-up recovery -** On the local disk, each follower keeps a log of data changes it has received from the leader. When the follower crashes or the n/w connection between the follower and leader crashes; the follower recovers quite easily. **It knows the last transaction from the log that was processed before the fault occurred.**
    - **Leader failure: Failover -**
        - When a leader fails, another follower needs to promoted to the new leader. Clients have to be reconfigured and other followers need to start consuming data changes from the new leader. This is called failover. Failover ⇒ automatic/manually
        - Automatic failover - Determine the failed leader ⇒ Choose new leader ⇒ Reconfigure sysstem to use new leader.
        - Things going wrong in the failover process -
            - For asynchronous systems, we may have to discard some writes if they have not been processed on a follower at the time of the leader failure. This violates clients' durability expectations.
            - Discarding writes is especially dangerous if other storage systems are coordinated with the database contents. For example, say an autoincrementing counter is used as a MySQL primary key and a redis store, if the old leader fails and some writes have not been processed, the new leader could begin using some primary keys which have already been assigned in redis. This will lead to inconsistency in the data, and it's what happened to Github ([https://github.blog/2012-09-14-github-availability-this-week/](https://github.blog/2012-09-14-github-availability-this-week/)).
            - In fault scenarios, we could have two nodes both believe that they are the leader: *split brain.* Data is likely to be lost/corrupted if both leaders accept writes and there's no process for resolving conflicts. Some systems have a mechanism to shut down one node if two leaders are detected. This mechanism needs to be designed properly though, or what happened at Github can happen again( [https://github.blog/2012-12-26-downtime-last-saturday/](https://github.blog/2012-12-26-downtime-last-saturday/))
            - It's difficult to determine the right timeout before the leader is declared dead. If it's too long, it means a longer time to recovery in the case where the leader fails. If it's too short, we can have unnecessary failovers, since a temporary load spike could cause a node's response time to increase above the timeout, or a network glitch could cause delayed packets. If the system is already struggling with high load or network problems, unnecessary failover can make the situation worse.