# 7. Transactions

### Overview

- Transactions are helpful in overcoming issues faced while designing data systems.
- Transaction ⇒ way for an application to group several reads and writes together into single logical unit.
- All reads and writes are executed as one single operation ⇒ either entire transaction succeeds or fails.
- NoSQL databases started gaining popularity in the late 2000s and aimed to improve the status quo of relational databases by offering new data models, and including replication and partitioning by default. However, many of these new models didn't implement transactions or watered down the meaning of the word to describe a weaker set of guarantees than had previously been understood.
- As these distributed databases started to emerge, the belief that transactions oppose scalability became popular; that a system would have to abandon transactions in order to maintain good performance and high availability. This is not true though.
- Like every technical design choice, there are advantages and disadvantages of using transactions. It's a tradeoff.

### Slippery Concept of Transaction

- NoSQL databases started gaining popularity in the late 2000s because they aimed to improve the relational status quo by offering the choice of new data models and by including replication and partitioning by default. Transactions were the primary reason behind this movement ⇒ Many of the new generation databases abandoned transactions entirely or redefined the word to describe a much weaker set of guarantees than before.
- With the rise of distributed databases, there emerged a belief that transactions were the antithesis of scalability. And that any large-scale system would have to abandon transactions to maintain good performance and high availability. On the other hand, transaction guarantees are presented by different database vendors as an essential requirement for applications with valuable data.

### **Meaning of ACID**

- One database's ACID implementation doesn't equal another database's implementation. The meaning of ACID highly varies different databases in practice. There is a lot of ambiguity in the term isolation. The high level idea is very sound, but the devil is in the details. When a system claims to be ACID complaint, its unclear what kind of guarantees you can actually expect.
- **Atomicity:**
    - In general, atomic means something that is in its most simpler form or something that can't be broken into smaller components. It describes, what happens if the client wants to make several writes, but there is a fault in some of the writes that have been processed.
    - If the writes are grouped together into the atomic transactions and it can't be completed due to fault; the transaction is aborted and the database must discard or undo any transaction it had made so far for this transaction.
    - The ability to abort a transaction on the error and have all writes from that transaction
    discarded is the defining feature of ACID atomicity
- **Consistency -**
    - The idea of ACID consistency is that you have certain statements about your data
    (invariants) that must always be true.
    - It's the responsibility of the application to define transactions that preserve consistency and not the databases's.
    - Atomicity, isolation and durability are properties of databases whereas consistency is property of appication.
- **Isolation -**
    - Most databases are accessed by several clients at the same time. That is no problem if
    they are reading and writing different parts of the database, but if they are accessing
    the same database records, you can run into concurrency problems
    - Isolation means concurrently executing transactions are isolated from each other that is they can't interrupt each other.
    - Many databases formalize the definition of isolation as serializability which means that each transaction can pretend that its only transaction running the entire database. That is, the database has the responsibility of ensuring that when transactions have committed, the result is the same as if it had run serially. However, in practice, serializable isolation is rarely used, because it carries a performance penalty.
- **Durability -**
    - Durability is the promise that once a transaction has committed successfully, any data it has written will not be forgotten, even if there is a hardware fault or the database crashes.
    - In a single-node database, durability typically means that the data has been written to
    nonvolatile storage such as a hard drive or SSD. This usually involves a write-ahead log or similar which allows recovery in the event that data structures on disk are corrupted.
    - In a replicated database, durability can mean that data has been successfully copied to other nodes.