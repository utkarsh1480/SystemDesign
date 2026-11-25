
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

    Clustered Primary Indexing: if Actual DB is sorted exact same as the index Stored index it is called Clustered db
    non- Clustured DB;
   2: Secondary Indexing
   3: Composite indexing: 

 <img width="875" height="639" alt="Screenshot 2026-11-25 at 11 42 04 AM" src="https://github.com/user-attachments/assets/dbfb6225-f877-4abd-a110-dd24a0c696ac" />
<img width="767" height="562" alt="Screenshot 2026-11-25 at 11 43 34 AM" src="https://github.com/user-attachments/assets/50f736e2-0b17-4497-8c0f-9db6dcfb69e6" />
<img width="1117" height="648" alt="Screenshot 2026-11-25 at 11 44 44 AM" src="https://github.com/user-attachments/assets/276b609a-48cc-4694-9b48-eb6426e623e6" />
hash Based rearly use in production because of the range of query , more efficient B tree

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
```
GeoSpetail Indexing
Imagine you are building a zomato app and want to locate near by use restorant then in that case we use Geospetail Indexing

<img width="1024" height="548" alt="Screenshot 2026-11-25 at 11 49 44 AM" src="https://github.com/user-attachments/assets/fac0a98f-9f23-4a45-9dad-3c4ac6d03592" />
<img width="991" height="556" alt="Screenshot 2026-11-25 at 11 51 46 AM" src="https://github.com/user-attachments/assets/927046f9-d34b-4ee5-819b-802c769f01d7" />

```
```
Inverted Indexing : lets suppose we want to search give document which contain kuberneties
<img width="818" height="499" alt="Screenshot 2026-11-25 at 11 57 41 AM" src="https://github.com/user-attachments/assets/232eab98-906f-4a73-9048-cbe9b5e0a00c" />
<img width="767" height="445" alt="Screenshot 2026-11-25 at 11 58 14 AM" src="https://github.com/user-attachments/assets/34cfc9dd-e373-4de8-b65a-5e3c7f5d1062" />
Elastic Search under the hood do this searchng
```
### when to use indexng
<img width="1143" height="633" alt="Screenshot 2026-11-25 at 12 00 02 PM" src="https://github.com/user-attachments/assets/a2ef83df-6afa-4592-b592-379946bee2b8" />

kis cheez par jyada queries ati hai usi par indexing use hoti hai
during system designing ask kis cheez par jyada queries ati hai
ask is it is heavy operation


