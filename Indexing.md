Indexing: "Indexing is a database optimization technique that uses an additional data structure to improve data retrieval speed by 
reducing the need for full table scans. However, it requires additional storage and can increase the cost of write operations."

### Problem 
<img width="886" height="483" alt="Screenshot 2026-11-25 at 11 33 51 AM" src="https://github.com/user-attachments/assets/14c78a0f-e9f2-4928-9d2a-2399c559be16" />

why Indexing : We need indexing in a database because it makes data retrieval faster. 
Instead of checking every row in a table to find the required record, the database uses an index to locate the data much more quickly.

type Of indexing :
 1: Primary Indexing : A Primary Index is an index created on the column (or columns) that uniquely identifies every record in a table.
    . Properties of a Primary Index
    1. A table can have only one primary key constraint.
    2. Primary key values must be unique.
    3. Primary key columns cannot contain NULL values.
    4. PostgreSQL automatically creates a unique index for the primary key.
    5. A primary key can consist of multiple columns (composite primary key).

How to create indexing: 
    CREATE INDEX idx_users_email
    ON users(email);

Is NoSql db has indexing : yes NoSQL databases use indexes to speed up data retrieval, just like SQL databases. 
However, their indexing mechanisms can differ depending on the database and its data model.
  Without an index, MongoDB may perform a Collection Scan (COLLSCAN).
  With an index, it can perform an Index Scan (IXSCAN) to find matching documents efficiently.
  ```JS
  db.users.createIndex({ email: 1 });
  Here:
- email = field to index.
- 1 = ascending index.
- -1 = descending index
```
```
Indexing in other NoSQL databases
Database	Indexing support
MongoDB	Single-field, compound, text, multikey, geospatial
Redis	Secondary indexing through Redis Search
Cassandra	Primary indexes, secondary indexes, SAI
DynamoDB	Local Secondary Index (LSI), Global Secondary Index (GSI)
Couchbase	Primary, secondary, composite indexes
```
```
How to check wheather mongodb has indexing or not db.users.find({
  email: "utkarsh@gmail.com"
}).explain("executionStats");


```
