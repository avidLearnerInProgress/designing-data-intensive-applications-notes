# 7. Transactions

### Overview

- Transactions are helpful in overcoming issues faced while designing data systems.
- Transaction ⇒ way for an application to group several reads and writes together into single logical unit.
- All reads and writes are executed as one single operation ⇒ either entire transaction succeeds or fails.
- NoSQL databases started gaining popularity in the late 2000s and aimed to improve the status quo of relational databases by offering new data models, and including replication and partitioning by default. However, many of these new models didn't implement transactions, or watered down the meaning of the word to describe a weaker set of guarantees than had previously been understood.
- As these distributed databases started to emerge, the belief that transactions oppose scalability became popular; that a system would have to abandon transactions in order to maintain good performance and high availability. This is not true though.
- Like every technical design choice, there are advantages and disadvantages of using transactions. It's a tradeoff.