AWS DynamoDB
============

DynamoDB is a NoSQL, serverless database that is fully managed, highly
available with replication across three AZs. It scales to massive workloads,
distributed database. It can perform millions of requests per seconds and 100s
TB of storage.

DynamoDB
--------

DynamoDB is made of _tables_ which has **Primay Key** and indefinite amount of
rows. Each item has **attributes** which are added over time, and max size is
400 KB. It supports following data types:

- Scalar Types : String, Number, Binary, Boolean, Null.
- Document Types: List, Map.
- Set Types: String Set, Number Set, Binary Set.

DynamoDB -- Primary Key
-----------------------

**Option.1 Partition Key Only (HASH)**

- Partition key must be unique for each item.
- Partition key must be "diverse" so that the data is evenly distributed.
- For example, "user\_id" is great for Users table:

| user\_id | First Name | Last Name | Age |
| --- | --- | --- | --- |
| 123abc | Batman | Null | Null |
| 321cba | Superman | Clarke | 32 |

**Option.2 Partition Key + Sort Key**

- Combination must be unique and data will be grouped by partition key.
- Sort key works as a range key.
- i.e. Users-Game table:
    - "user\_id" as a partition key.
    - "game\_id" as a sort key.

DynamoDB -- Partition Keys Exercises
------------------------------------

In the exam, prepare for choosing the most optimal partition key for given
cases - remember that choose a key that will distribute evenly, not skwed.

DynamoDB -- Parovisioned Throughput
-----------------------------------

- Table must have provisioned READ and WRITE capcity units.
- **READ Capacity Units (RCU)**
- **WRITE Capacity Units (WCU)**
- Throughput can be auto-scaled.
- Througput can _burst_; and when empty, `ProvisionedThroughputException` will
  be raised. Retry with exponential back-off strategy.

DynamoDB -- WRITE Capacity Units
--------------------------------

**One WCU represnets one write per second for an item up to 1 KB**. If the item
size exceeds 1 KB, then extra WCU is consumed.

i.e. Write 10 items per seconds of 2 KB each will consume 2 * 10 = 20 WCUs.

i.e. Write 120 items per min of 2 KB each will consume (120 / 60) * 2 = 4 WCUs.

DynamoDB -- Strongly Consistent READ vs Eventually Consistent READ
------------------------------------------------------------------

**Eventually Consistent READ** may return a stale object if we READ just after
a WRITE.

**Strongly Consistent READ** will always return a correct data.

By default, eventual consistent READs are used, but for `GetItem` query and
scan provides `ConsistenRead` parameter that you may set to true.

i.e. Remember that DynamoDB is replicated over three AZs. SO, if an application
does WRITE to a server, it will be replicated to other two.

This consistency model will impact the Read Capacity Units.

DynamoDB -- READ Capacity Units
-------------------------------

**One RCU represents one strongly consistent READ per second, or two eventually
consistent READ per second for items up to 4 KB**.

i.e. 10 strongly consistent reads per second of 4 KB consumes 10 * (4 / 4) = 10
RCUs.

i.e. 16 eventually consistent reads per second of 12 KB consumes (16 / 2) * (12
/ 4) = 24 RCUs.

DynamoDB -- Partitions Internal
-------------------------------

- Data is partitioned.
- Partition keys go through a hashing algorithm to determine which partition to
  place.
- To compute the number of partitions:
    - by capcacity : (Total RCU / 3,000) + (Total WCU / 1,000)
    - by szie : Total Size / 10 GB
    - Total partitions = ceil(max(Capacity.Size))

- **WCU and RCU are spread evenly between partitions**.

DynamoDB -- Throttling
----------------------

- If RCU or WCU is exceeded, `ProvisionedThroughputExceededExceptions` is
  raised.

- Possble reasons:
    - Hot Key : one partition key is used too many times.
    - Hot partitions.
    - Very large items : RCU and WCU are DEPENDENT on the item size.

- Solutions:
    - Retry with exponential back-off strategy.
    - Distribute partition keys as much as possible.
    - If RCU issue, use DynamoDB Accelerator (DAX) which is a cache.

DynamoDB -- Writing Data
------------------------

`PutItem`

- Write data to DynamoDB (replace); consumes WCU.

`UpdateItem`

- Update data in DynamoB (partial update of attributes).

Conditional Writes

- Accept a WRITE or UPDATE only if the conditions are met.
- Helps with concurrent access to items.

DynamoDB -- Deleting Data
-------------------------

`DeleteItem`

- Deletes an individual row.

`DeleteTable`

- Drops entire table.
- Much faster than repeatedly calling `DeleteItem` on all items in the table.

DynamoDB -- Batching WRTIEs
---------------------------

Batching allows you to save in latency by reducing the number of API calls
t othe DynamoDB and it is done in parallel. It is possible that part of the
batch to fail, in which we can retry.

`BatchWriteItem`

- Up to 25 `PutItem` and/or `DeleteItem` in one call.
- Up to 16 MB of data written.
- Up to 400 KB of data per item.

DynamoDB -- Reading Data
-----------------------

`GetItem`

- READ based on the primary key.
- Eventuallu consistent by default.
- `ProjectionExpression` can be used to include only certain attributes.

`Batch GetItem`

- Up to 100 items to read.
- Up to 16 MB of data.

DynamoDB -- Query
-----------------

`Query` return item based on following:

- PartitionKey value (must be ==).
- SortKey value (==, <, <=, >, >=, Between, Begin) is optional.
- FilterExpression to further filter. This is client-side filtering and will
  still receive the full data.

It will return up to 1 MB of data or number of items specified in limit.

It is possible to perform pagination on the results.

It can query table, a local secondy index or a global secondary index.

DynamoDB -- Scan
---------------

`Scan` will "scan" entire table and then filter out the data which is highly
inefficient. It will return up to 1 MB of data - use pagination to continue
reading.

For faster performance, parallel scans may be used.

- multiple instances scan multiple partitions at the same time.
- increases the throughput and RCUs.

DynamoDB -- Local Secondary Index (LSI)
---------------------------------------

It is a alternative range key for a table which is local to the hash key. There
can be up to 5 LSIs for a table. It must be defined at the table creation time.

i.e. Suppose we have a table with `user_id` as a partition key and `game_id` as
a sort key. If we wanted to query based on other attributes on this table such
as `timestamp`, we would have that attributes as LSI.

DynamoDB -- Global Secondary Index (GSI)
----------------------------------------

It is used to speed up queries on non-key atributes and is composed of
partition key and optional sort key.

Basically, this creates an additional table with a new sort key.

- Partition Key and Sort Key of the original table are always projects.
- Can specify extra attributes to project (INCLUDE).
- Can use all attributes from the main table (ALL).

Must define RCU and WCU for index; and we can modify GSIs but not LSI which
needs to be defined at creation of the table.

DynamoDB -- Index and Throttling
--------------------------------

**GSI**

- **If WRITE is throttled on the GSI, then the main table will also be
  throttled!**
- It is true even when WCU on the main tables are fine.
- So, we need to assign WCU capacity carefully to GSIs.

**LSI**

- LSI uses the same WCU and RCU of the main table.
- Thus, no speical throttling considerations.

DynamoDB -- Concurrency
-----------------------

DynamoDB has a feature of "Conditional Update and Delete", which enssure that
an item has not been changed before altering it. This makes the DynamoDB an
**Optimistic Locking and Concurrency** database.

i.e. If two WRITE occurs between two clients, we do not know which update will
go through.

---

DynamoDB -- DAX
---------------

DynamoDB Accelerator is a cache service for DynamoDB, which the data is
uploaded to DAX first, then onto the DynamoDB.

It solves the Hot Key problem where few items are very popular and is being
accessed much more than others.

By default every item is cached for 5 mins (TTL); and have up to 10 nodes in the
cluster. It is multi-AZ and secured through encryption at rest with KMS, VPC,
CloudTrail, etc.

DynamoDB -- DAX vs ElastiCache
------------------------------

If client does some individual objects query caches or query and scan cache,
DAX is the correct usage since it can retrieve the items quickly. But
ElastiCache is also possible and serves different needs.

---

DynamoDB -- Streams
-------------------

- Changes in the DynamoDB (Create, Update, Delete) can end up in a DynamoDB
  Stream.
- This sream can be read by Lambda function or EC2 instances to:
    - react to changes real time (i.e. send welcome e-mail to registered user).
    - perform analytics.
    - create derivative tables and views.
    - insert into ElastiCache.
- Could implement Cross-region replication using streams.
- Stream has 2 hours of data retention.

DynamoDB Sreams are made of shards just like Kinesis Data Streams, but we do
not have to provision them (automated).

Records are not retroactively populated in a stream after enabling it.

DynamoDB -- Streams and Lambda
------------------------------

You need to define an **Event Source Mapping** in Lambda in order to read from
the DynamoDB streams just like Kinesis. It will facilitate polling and
retrieveing the batch from the stream and invoke the function SYNC.

DynamoDB -- Time-To-Live (TTL)
------------------------------

TTL is used to automatically delete an item after set date, and deletions will
not cost RCU nor WCU. It is a background task that performs regularly and helps
to minimize the storage and table sizes.

TTL is enabled per row.

DynamoDB -- CLI
---------------

`--projection-expression` is used to define atributes to retrieve.

`--filter-expression` is used to filter the results.

General CLI pagination options including DynamoDB and S3:

- Optimization:
    - `--page-size` will require full dataset, but each API call will request
      less data to avoid timeouts.
- Pagination:
    - `--max-items` specify max number of results; returns `NextToken`.
    - `--starting-token` specify with last received `NextToken` to continue.

DynamoDB -- Transactions
------------------------

Transactions are ability to create, update and delete multiple rows in
different tables at the same time which is an all-or-nothing opeations.

- Write Modes : Standard, or Transactional.
- Read Modes : Eventual Consistency, Strong COnsistency, or Transactional.

DynamoDB -- as a Session State Cache
------------------------------------

One of common use case for DynamoDB is to store session cache.

- compared to ElastiCache:
    - ElastiCache is in-memory but DynamoDB is serverless and auto-scales.
    - Both are key-value storages.
- compared to EFS (disk storage):
    - EFS must be attached to EC2 instance as a network drive.
- compared to EBS and Instance Store:
    - EBs and Instance Store can only be used for local caching (not shared).
- compared to S3:
    - Too high of latency.

DynamoDB -- Write Types
-----------------------

**Concurrent Writes**

- Two users updating a same time with different values; in this case, second
  write will overwrite the first.

**Conditional Writes**

- A way to mitigate the above problem; add conditions to write such that update
  only iff the value is the same as last time it was known to the clients.

**Atomic Writes**

- Another way to avoid concurrenct problem; instead of overwriting values, the
  users instead INCREMENT or DECREMENT the value in the table. Then, the
  overall changes will be their total.

**Batch Writes**

- Simply group all the changes into a single transaction.

DynamoDB -- Large Objects Pattern
---------------------------------

DynamoDB can support items size up to 400 KB; to work around this, we can
instead put the large objects in the S3 first. Then, we insert the metadata
into the DynamoDB instead.

DynamoDB -- Indexing S3 Objects Meatadata
-----------------------------------------

After writing a file to S3 bucket, we cannot index it in meaningful way such as
sort by size, date or etc. To solve this, we store the metadata on the DynamoDB
which is triggered by the Lambda functions.

DynamoDB -- Operations
----------------------

**Table Cleanup**

- Drop the table; easier than scan and delete every item.

**Copying a DynamoDB Table**

- Use AWS DataPipeline (EMR); which writes to S3 bucket first, then restores
  back to the new DynamoDB table.
- Or, create a backup and restore the backup into the new table.
- Or, do manual scan and write.

DynamoDB -- Security and Other Features
---------------------------------------

**Security**

- VPC endpoints are available to access DyanmoDB without www connection.
- IAM controls the access.
- Encryption at rest using KMS.
- Encryption in transit using HTTPS.

**Backup and Restore Feature**

- Allows for point in time restore.
- No performance impact.

**Global Tables**

- Multi-region, fully replicated and high in performance.

You may launch a local DynamoDB on local machine for development.
