AWS DynamoDB -- NoSQL Serverless Database
=========================================

**Traditional Architecture involves:**

Client interacting with the API Layer (consists of EC2, ASG and ELB).
API Layer interacting with the Database Layer such as RDS (MySQL, PostgreSQL,
...).

- This leverages on the RDBMS databases; which uses SQL query language.
- This type of databases enforce strict requirements about how the object and
  data should be modeled.
- Advantage is that we can manipulate the databases such as perform join,
  aggregations, computations.
- But in order to scale, we would require more powerful resources (more CPU,
  RAM, IO).

**NoSQL databases involve:**

NoSQL databases are _non-relational_ and _distributed_ - enabling horizontal
scaling. Popular databases include MongoDB, DynamoDB, etc.

NoSQL dbs do not support join operation - as all data needed for a query is in
a single row.

NoSQL dbs do not perform aggregations such as SUM.

NoSQL dbs scale horizontally.

DynamoDB
--------

It is NoSQL database that is fully managed, highly available with replication
across 3 AZs. It scales to massive workloads, distributed database.

_Millions of requests per seconds, trillions of row, 100s of TB of storage_.

Fast and consistent in performance (low latency on retrieval).

Integrated with IAM for security, authorization and administration.

Enables event driven programming with DynamoDB Streams.

Low cost and auto scaling capabilities.

DynamoDB -- Basics
------------------

DynamoDB is made of **tables**.

Each table has a **primary key** (which is required at creation time).

Each table can have an infinite number of items (= rows).

Each item has **attributes** which are added over time (can be Null).

Maximum size of an item is 400 KB.

Data types supported are:

- Scalar Types: String, Number, Binary, Boolean, Null
- Document Types: List, Map
- Set Types: String Set, number Set, Binary Set

DynamoDB -- Primary Keys
------------------------

**Option.1 : Partition key only (HASH)**

- Partition key must be unique for each item.
- Partition key must be "diverse" so that the data is evenly distributed
- i.e. `user_id` for users table.

| user\_id | First name | Age |
| --- | --- | --- |
| 1299x9 | Batman | 99 |
| ddi2f8 | Superman | 66 |

**Option.2 : Partition key + Sort key**

- combination must be unique.
- data will be grouped by partition key.
- sort key == range key.
- i.e. users-games table:
    - `user_id` for the partition key.
    - `game_id` for the sort key.

| user\_id | game\_id | result |
| --- | --- | --- |
| ddi2f8 | 1243 | loss |
| 1299x9 | 8922 | win |
| 1299x9 | 8923 | loss |

DynamoDB -- Partition Keys exercise
-----------------------------------

In exam, prepare for choosing the optimal partition key for given cases.

i.e. we are building a movie database; what would be the best partition key to
maximize the data distribution among these choices: `movie_id`,
`producer_name`, `leader_actor_name`, `movie_language`.

In this case, we would pick `move_id` due to high cardinality. For example,
`movie_language` will be skwed since many movies will be in English, so it is
not a good candidate for partition key.

DynamoDB -- Provisioned Throughput
----------------------------------

- Table must have provisioned read and write capacity units.
- **Read Capcacity Units (RCU)** : throughput for reads
- **Write Capacity Units (WCU)** : throughput for writes
- option to setup auto-scaling of throughput
- throughput can be _bursted_ using the burst credit
- if burst credit is empty, you will get `ProvisionedThroughputException`.
- thus, use exponential backoffs for retries.

DynamoDB -- Write Capacity Units
--------------------------------

- One _write capacity unit_ represents one write per second for an item upto
  1 KB in size.
- If the items are larger than 1 KB, more WCU are consumed.

- i.e. Write 10 objects per seconds of 2 KB each.
    - Then we will require 2 * 10 = 20 WCU
- i.e. Write 6 objects per second of 4.5 KB each.
    - This time, 6 * 5 = 30 WCU (4.5 gets rounded up).
- i.e. Write 120 objects per min of 2 KB each.
    - (120 / 60) * 2 = 4 WCU (since writing 2 objects per second).

Strongly Consistent Read vs Eventually Consistent Read
------------------------------------------------------

With DynamoDB you get an option to choose how the read behavior would be like:

**Eventually Consistent Read** - if we read just after a write, it is possible
that we may get unexpected response because of replication.

**Strongly Consistent Read** - if we read just after a write, we will always
get the correct data.

By default, DynamoDB uses eventually consistent reads, but `GetItem` query and
scan provides a `ConsistentRead` parameter you can set to True.

i.e. Remember that DynamoDB is replicated over 3 AZs. So, if an application
does a write to one DynamoDB server, it will be replicated to other two. This
is the problem; if an application attempts to read from the one of the server
that is being replicated, we are in _eventual consistent pattern_.

This Consistent read model impacts the Read Capacity Units.

DynamoDB -- Read Capacity Units
-------------------------------

- One _read capacity unit_ represents one strongly consistent read per second,
  or two eventually consistent reads per second, for an item up to 4 KB in
  size.

- If the items are larger than 4 KB, more RCU are consumed.

- i.e. 10 strongly consistent reads per seconds of 4 KB each?
    - 10 * 4 KB / 4 KB = 10 RCU required.

- i.e. 16 eventually consistent reads per seconds of 12 KB each?
    - (16 / 2) * (12 / 4) = 24 RCU.

DynamoDB -- Partitions Internal
-------------------------------

- Data is divided in partitions.
- Partition keys go through a hashing algorithm to know to which partition to
  go.
- To compute the number of partitions:
    - by capacity : (TOTAL\_RCU / 3000) + (TOTAL\_WCU / 1000)
    - by size : TOTAL\_SIZE / 10 GB
    - Total partitions = `ceil(max(Capacity.Size))`

- **WCU and RCU** are spread evenly between partitions.

DynamoDB -- Throttling
----------------------

- If we exceed our RCU or WCU, we receive `ProvisionedThroughputExceededExceptions`.

- Reasons:
    - Hot Keys : one partition key is being read too many times.
    - Hot partitions
    - Very large items : RCU and WCU are DEPENDENT on the size of items.

- Solutions:
    - Exponentional backoff strategy (included in SDK).
    - Distribute partition keys as much as possible.
    - If RCU issue, we can use DynamoDB Accelerator (DAX) - a cache.

---

Need to know the operations of DynamoDB API.

DynamoDB -- Writing Data
------------------------

- `PutItem` 
    - write data to DynamoDB (create data or full replace). 
    - consumes WCU.

- `UpdateItem`
    - "update" data in DynamoDB (partial update of attributes)
    - possibility to use Atomic Counters and increase them

- Conditional Writes:
    - Accept a write / update only if the conditions are met; otherwise,
      reject.
    - _Helps with concurrent access to items._
    - No performance impact.

DynamoDB -- Deleting Data
-------------------------

- `DeleteItem`
    - delete an individual row.
    - ability o perform a conditional delete.

- `DeleteTable`
    - delete a whole table and all its items.
    - much quicker deletion than calling `DeleteItem` on all items.

DynamoDB -- Batching Writes
---------------------------

- `BatchWriteItem`
    - upto 25 `PutItem` and / or `DeleteItem` in one call.
    - upto 16 MB of data written.
    - upto 400 KB of data per item.

- Batching allows you to save in latency by reducing the number of API calls to
  the DynamoDB.

- Batch operations are done in parallel for better efficiency.

- It is possible for part of the batch to fail, in which we have to retry the
  failed items (using exponential back-off algorithm).

DynamoDB -- Reading Data
------------------------

- `GetItem`
    - read based on Primary key = HASH or HASH-RANGE
    - eventually consistent read by default
    - option to use strongly consistents reads but it will consume more RCU
      & take longer.
    - `ProjectonExpression` can be specified to include only certain
      attributes.

- `BatchGetItem`
    - upto 100 items.
    - upto 16 MB of data.
    - items are retrieved in parallel to minimize latency.

DynamoDB -- Query
-----------------

- `Query` returns items based on following:
    - PartitionKey value (must be == operator)
    - SortKey value (=, <, <=, >, >=, Between, Begin) - optional
    - FilterExpression to further filter (client side filtering; will still
      receive full data matching PartitionKey + SortKey).
- Returns:
    - upto 1 MB of data.
    - or number of items specified in Limit
- Able to do pagination on the results
- Can query table, a local secondary index, or a global secondary index.

DynamoDB - Scan
---------------

- `Scan` the entire table and then filter out data (highly inefficient).
- Returns upto 1 MB of data - use pagination to keep on reading.
- Consumes a lot of RCU;
- Limit impact using Limit or reduce the size of the result and pause.
- For faster performance, may use **parallel scans**:
    - multiple instances scan multiple partitions at the same time.
    - increases the throughput and RCU consumed.
    - limit the impact of paralle scans just like you would for Scans.
- Can use `ProjectionExpression` and `FilterExpression` (no charge on RCU).

---

DynamoDB -- LSI (Local Secondary Index)
---------------------------------------

It is an alternate range key for your table, _local to the hash key_; upto
5 LSIs can exist per table.

The sort key consists of exactly one scalar attribute - which is either scalar
String, Number, or Binary.

LSI must be defined at the table creation time.

i.e. Suppose we have a table with `user_id` as partition key and `game_id` as
sort key. If we wanted to query based on other attributes on this table (such
as `timestamp` or `result` or etc), we would have that attributes as LSI.

DynamoDB -- GSI (Global Secondary Index)
----------------------------------------

This is to speed up queries on non-key attributes.

GSI = Partition Key + Optional Sort Key

This index is practically a new table and we can project attributes on it.

- partition key and sort key of the original table are always projects
  (KEYS\_ONLY).
- Can specify extra attributes to project (INCLUDE).
- Can use all attributes from the main table (ALL).

Must define RCU / WCU for index.

We can add or modify GSI but not LSI.

i.e. Same set up as previous, but now we can use GSI to choose `game_id` as the
partition key and `timestamp` as the sort key. This will create a new table
where we can do query by `timestamp` - (which user finished game last time?)

DynamoDB Indexes and Throttling
-------------------------------

GSI:

- **If writes are throttled on the GSI, then the main table will be
  throttled**.
- It is true even when WCU on the main tables are fine!
- So, need to choose GSI partition key carefully and assign WCU capacity
  carefully.

LSI:

- LSI uses the WCU and RCU of the main table;
- Thus, no special throttling considerations.

DynamoDB Concurrency
--------------------

- DynamoDB has a feature of "Conditional Update / Delete".
- You can either ensure that an item has not changed before altering it.
- This makes DynamoDB an **optimistic locking / concurrency** database.

i.e. Suppose there is an item in DynamoDB and is being accessed by two clients.
One client requests to `update to change name to ABC only if item version == 1`.
And other client requests to `update to change name to XYZ only if item version
== 1`. Since both are trying to update, we do not know which update would have
taken place.

---

DynamoDB DAX
------------

It is an DynamoDB Accelerator that is a seamless cache for DynamoDB (no need to
rewrite the application). The writes will go through the DAX first, then onto
DynamoDB.

Microsecond latency for cached reads and queries!

DAX exists to solve _Hot Key_ problem where few items are popular and is being
accessed much more frequently than others.

By default, every item is cached for 5 minutes TTL; and have upto 10 nodes in
the cluster.

It is multi-AZ (3 nodes minimum recommended for production).

It is secured through encryption at rest with KMS, VPC, IAM, CloudTrail ...

DynamoDB DAX vs ElastiCache
---------------------------

If client does some individual objects query caches or query / scan cache, DAX
is the correct usage - DAX will be able to retrieve the items quickly.

But it is also possible to set-up an ElastiCache - just different needs.

---

DynamoDB Streams
----------------

- Changes in DynamoDB (Create, Update, Delete) can end up in a DynamoDB Stream.
- This stream can be read by Lambda or EC2 instances:
    - react to changes real time (i.e. send welcome email to newly registered
      user in our database).
    - analytics.
    - create derivative tables / views.
    - insert into ElasticSearch.
- Could implement cross region replication using Streams
- Stream has 2 hours of data retention

Choose the information that will be written to the steram whenever the data in
the table is modified:

- `KEYS_ONLY` : only the key attributes of the modified item
- `NEW_IMAGE` : entire item, as it appears after it was modified
- `OLD_IMAGE` : entire itme, as it appeared before it was modified
- `NEW_AND_OLD_IMAGES` : both new and old of images of the item

DynamoDB Streams are made of shards just like Kinesis Data Streams, but we do
not have to provision shards - it is automated.

Records are not retroactively populated in a stream after enabling it; only
future changes will make it.

DynamoDB Streams and Lambda
---------------------------

You need to define an **Event Source Mapping** in Lambda in order to read from
the DynamoDB Streams - which will be polling and retrieve the batch from
DynamoDB, then invoke the Lambda function.

Make sure that Lambda function has the appropriate permissions.

The Lambda function is invoked synchronously.

DynamoDB TTL (Time to live)
---------------------------

- TTL will automatically delete an item after an expiry date and time.
- TTL is provided at no extra cost, deletions do not use WCU and RCU.
- It is a background task performed periodically.
- Helps reduce storage and manage the table size over time.
- TTL is enalbed per row.
- DynamoDB typically deletes expired items within 48 hours of exirations.
- Deleted item due to TTL are also deleted from GSI and LSI.
- DynamoDB Streams can help recover expired items.

- Insert Epoch time stamp as a new attribute; under the settings, we can enable
  TTL based on a attribute.

DynamoDB CLI -- Good to know
----------------------------

- `--projection-expression` : attributes to retrieve
- `--filter-experssion` : filter results

- General CLI pagination options including DynamoDB and S3:
    - Optimization:
        - `--page-size` : full dataset is still required; but each API call
          will request less data (avoids timeouts).
    - Pagination:
        - `--max-itesm` : max number of results returned by the CLI; returns
          `NextToken`.
        - `--starting-token` : specify the last received `NextToken` to keep on
          reading.

Examples of CLI commands:

- Using projction expression:


    $ aws dynamodb scan -table-name UserPosts --projection-expression "user_id, content" --region us-west-1


- Using filter expression: filters by all rows which have `user_id` equal to
  String value "124user01".


    $ aws dynamodb scan --table-name UserPosts --filter-expression "user_id = :u" --expression-attribute-values '{ ":u": {"S":"124user01"}}' --region us-west-1`


- Using max-items: returns only specified number of max-items (rows) at a time.
  It will return next token so that we can continue from last position in the
  row (Pagination).


    $ aws dynamodb scan --table-name UserPosts --region us-west-1 --max-items 1
    $ aws dynamodb scan --table-name UserPosts --region us-west-1 --max-items 1 --starting-token ejKHSJHFneLOJHHlnqHSLJHLKJHFJHDJF


DynamoDB Transactions
---------------------

- Transactions refer to ability to create, update and delete multiple rows in
  different tables at the same time (similar to RDMS databases).

- This is "all or nothing" operations.

- Write Modes: Standard, _Transactional_

- Read Modes: Eventual Consistency, Strong Consistency, _Transactional_

- It consumes twice the WCU and RCU.

- In short, _transcation is a write to both tables or none_.

DynamoDB as Session State Cache
-------------------------------

- It is common use case to use the DynamoDB to store the session state.

- compared to ElastiCache:
    - ElastiCache is in-memory, but DynamoDB is serverless + auto scales.
    - Both are key-value stroes.
- compared to EFS (disk storage):
    - EFS must be attached to EC2 instances as a network drive.
- compared to EBS and Instance Store:
    - EBS and Instance Store can only be used for local caching; not shared.
- compared to S3:
    - S3 has higher latency; it is not suitable for small objects.

DynamoDB Write Sharding
-----------------------

- Suppose we have a voting application with two candidates, A and B.
- If we use a partition key of candidate\_id, it will run into partitions
  issues as there are only two partitions.
- Solution is to add a suffix (usually _random suffix_).

| **Partition Key** | **Sort Key** | **Attributes** |
| --- | --- | -- |
| **CandidateID + RandomSuffix** | **VoteDate** | **VoterID** |
| CandidateA-1 | 2020-01-01-108855 | 253342 |
| CandidateA-1 | 2020-01-09-120023 | 253344 |
| CandidateA-2 | 2020-01-21-170256 | 012342 |
| CandidateB-1 | 2020-02-03-010203 | 929392 |

DynamoDB -- Write Types
----------------------

**Concurrent Writes**

- Two users updating a same item with different values; in this case, second
  write will overwrite the first.

**Conditional Writes**

- A way to mitigate the above problem: add condition to write - that write only
  if the current value is the last we have seen. The second write will fail now
  since the value would have changed and knows to avoid concurrent.

**Atomic Writes**

- Another way to avoid concurrent problem; instead of overwriting values, the
  users will either INCREASE or DECREASE the existing value in the table. The
  result would be TOTAL of changes registered.

**Batch Writes**

- Simply pool the transactions; write and update many items at a time.

DynamoDB -- Large Objects Pattern
---------------------------------

- DynamoDB can get expensive with large objects since its maximum item size is
  400 KB.

- A pattern that can deal with large objects is to first have the client send
  the large data to S3 bucket. The metadata of that large data will be inserted
  to the DynamoDB.
- Thus, another client requiring that data will read the metadata from DynamoDB
  to find where the object is in S3, and retrieve from there.

DynamoDB -- Indexing S3 objects metadata
----------------------------------------

- After writing a file to S3 bucket, we cannot index it efficiently - cannot
  sort by size, date or etc.
- A way to solve this problem is to set up S3 event notification to trigger
  Lambda function that will record the metadata to the DynamoDB such that
  clients can instead use DynamoDB to index the files, and retrieve from S3 if
  necessary.

- This will support metadata such as:
    - search by date
    - total storage used by a customer
    - list of all objects with certain attributes
    - find all objects uploaded within a date range

DynamoDB Operations
-------------------

**Table Cleanup**

- Option 1 : scan and delete every item (row); very slow, expensive, consumes
  RCU and WCU.

- Option 2 : drop table and recreate table; fast, cheap, efficient.

**Copying a DynamoDB Table**

- Option 1: use AWS DataPipeline (EMR)
    - DataPipeline launches in EMR; reads from DynamoDB Table and writes to S3
      Bucket.
    - Then, import the data stored within the S3 Bucket back to new DynamoDB
      Table.

- Option 2: create a backup and restore the backup into a new table name (may
  take some time).

- Option 3: scan and write - needs manual code.

DynamoDB -- Security and Other Features
---------------------------------------

**Security**

- VPC Endpoints available to access DynamoDB without internet.
- Access fully controlled by IAM.
- Encryption at rest using KMS.
- Encryption in transit using SSL/TLS.

**Backup and Restore feature**

- Point in time restore (like RDS).
- No performance impact.

**Global Tables**

- Multi-region, fully replicated and high in performance.

**Amazon DMS can be used to migrate to DynamoDB from Mongo, Oracle, MySQL,
S3...**

**You may launch a local DynamoDB on local machine for development purposes.**



