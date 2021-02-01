# 2. Data models and query languages

### Overview

- Most applications are built by layering one data model on top of another. For each layer the key question is: *How it is represented in terms of next-lower layer?*
- An app developer models data in terms of objects, data structures and API's. To store these data structures, we use a general purpose data model alike JSON / XML. The engineers who build the app decide a way to represent JSON like data model in form of bytes in memory or disk. On more lower level, hardware engineers figure out how to represent bytes in form of electrical currents, magnetic fields, etc.
- **Each layer hides complexity of the layers below providing a clean data model.**

### Relational Model VS Document Model

- Relational - *Data organized into relations where each relation is unordered collections of tuples.*
- **Goal of relational model was to hide the implementation details behind the cleaner interface.**
- Driving forces behind NoSQL databases -
    - Achieve better scalability than their relational counterparts.
    - Free and opensource.
    - Specialized query operations not well-supported in relational databases.
    - Making database schemas adapt more dynamic and expressive data models.
- NoSQL Databases are in two forms:
    - **Document databases**: Data is organized by self-contained documents and the relationship inter-between documents are rare.
    - **Graph databases:** They are originated by core concepts of graph theory where each element is associated with everything else(and potentially vice-a-versa)
- *Impedance mismatch* - OOP languages contribute to most of the application development today. If the data is stored in relational tables, a translation layer is needed between objects in the application and the database models. For this, we have the ORM frameworks to rescue.
- **JSON has a better locality of reference than the multi-table schema**. In a way it allows us to structure all the relevant information in one single place. If we use relational models, we will need to apply multiple joins to the database. Also, JSON reduces impedance mismatch.
- One to many relationship's can be easily visualized in a tree fashion and JSON makes this tree structure explicit.
- In relational databases, it's normal to refer to rows in other tables by ID, because joins are easy. In document databases, joins are not needed for one-to-many tree structures, and support for joins is often weak.
- For many to (one / many)-to-many relationships, document databases might not be the best option because such relationships involve data normalization. Document databases don't have good support for joins. Also, if the database inherently doesn't support joins, then you have to emulate a join in application code by making multiple queries to the database.
- The most popular database for business processing systems was IBM's IMS developed in 1970. It had striking similarities with JSON documents(hierarchical model) by representing documents as a tree of records nested within records. IMS worked well for one to many relationships but for many-to-many relationships, developers had to decide whether to duplicate data or manually resolve references.
- Network model - a generalization of the hierarchical model. In the tree structure of the hierarchical model, every record had exactly one parent whereas in the network model every record can have multiple parents.
- Relational model - a way to layout the data in open and store data in form of table and tuples
- *Foreign* key in SQL world and *document reference* in NoSQL world.
- **Relational VS Document today -**
    - Document model advantages - **schema flexibility, better performance due to locality, closer to data structures used by the application.**
    - Relational model advantages - **better support for joins, many-to-(one/many) relationships.**
    - *If data in the application has a document-like structure, then it might be a good idea to use a document model. This avoids the problem of cumbersome schemas and complicated application code because of shredding(splitting document-like structure into multiple tables)*
    - As long as documents are not nested to extreme depths, its fine because you cannot refer directly to nested item within the document. You have to traverse from the root to reach the item down the sub-tree.
    - **Long story short - For highly interconnected data, the document model is awkward, the relational model is acceptable and the graph model is the most natural.**
- **Schema flexibility in document model**
    - Document databases are schemaless and this is misleading. The code that reads the data has an implicit schema but it is not enforced by the database: Also known as *schema-on-read -* The structure of the data is implicit, and only interpreted when the data is read. This is contrasted with *schema-on-write:* The schema is explicit and the database ensures all written data conforms to it.
    - The difference between schema-on-write and schema-on-read is noticeable in situations when an application wants to change the data format. For document databases, we can simply start writing new documents with new fields and have the application handle case when new documents are read. But for relational schema, we need to perform migration in the database structure first before adding a new field. Schema changes require downtime and they are likely very slow.
- **Data locality for queries**
    - A document stored as a single continuous encoded as JSON or XML or a binary variant like BSON. If the application needs to access the entire document very frequently; then document storage has a significant performance advantage.
    - If the data is split across multiple tables, multiple index lookups might be needed to retrieve it all which requires more disk seeks and takes more time.
    - Document databases have the advantage of having a storage locality. This advantage holds true only if you need large parts of a document at the same time. You don't need to load the entire document in memory. Also, updates to document entire document need to be rewritten.  If documents are fairly small and if they avoid writes which increase the size of the document, then there are significant advantages to the NoSQL world.
    - Maintaining locality/grouping data together is also present in Google's Spanner Database which is a relational DB. It allows the schema to declare that a table's rows should be interleaves within the parent table.
- **Query languages for data**
    - ***An imperative language*** tells the computer to perform certain operations in a certain order. You can imagine stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time.
    - ***In a declarative query language***, like SQL or relational algebra, you just specify the pattern of the data you want—what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal.
    - A declarative query language is **attractive** because it is **typically more concise and easier to work with** than an imperative API. But more importantly, it also **hides implementation details** of the database engine, which makes it possible for the database system to introduce performance improvements without requiring any changes to queries.
    - Declarative languages often lend themselves to parallel execution while imperative code is very hard to parallelise across multiple cores because it specifies instructions that must be performed in a particular order. Declarative languages specify only the pattern of the results, not the algorithm that is used to determine results.