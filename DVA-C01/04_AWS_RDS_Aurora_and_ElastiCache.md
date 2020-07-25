AWS RDS, Aurora and ElasticCache
================================

AWS Relational Database Service (RDS)
-------------------------------------

It is a managed DB service using SQL as a query language; it supports
following:

- postgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- AWS Aurora

RDS -- Why use it instead of deploying DB on EC2?
-------------------------------------------------

**RDS is a managed service**; as such it offers folowing features:

- automated provisioning; OS patching.
- continuous backups and restore to specific timestamp if needed.
- monitoring dashboards.
- **Read Replicas** are available to improve read performance.
- Multi-AZ setup for disaster recovery.
- Maintenance windows for upgrade.
- Scaling capability (both vertical and horizontal).
- Storage backed by EBS volumes (GP2 or IO1).

RDS -- Backups
--------------

Backups are automatically enabled in RDS.

- Daily full backup of the database during maint window.
- Transaction logs are backed up by RDS every 5 minutes.
- You can restore database back to any point in time.
- Default retention period of 7 days (upto 35 days).

Database snapshots are also available.

- It is triggered manually by the user.
- Retention period is indefinite.

RDS -- Read Replicas for read scalability
-----------------------------------------

Suppose that a single RDS DB instance is not enough to serve all the read/write
operations performed by many applications; in this case, we can create upto
5 **Read Replicas** of the DB. _These can be within AZ, cross AZ, or cross
Region_.

The replication process will be an ASYNC; hence, the reads will be eventually
consistent (the read data maybe is out of order).

Replicas can be promoted to become its own DBs.

RDS -- Read Replicas and Use Cases
----------------------------------

Suppose that you wish to run an analytics on the database; but performing
anayltics on the production database will place extra unwanted workloads
- impacting customer experience. To prevent this, we first create a read
  replica and have it serve the analytics application.

RDS -- Read Replicas and Network Cost
-------------------------------------

There is a network cost when data moves across AZs; to reduce this, you may
create a read replica within the same AZ.

RDS -- Multi-AZ (Disaster Recovery)
-----------------------------------

This is a SYNC replication that creates a replica on another AZ that is in
standby, ready to take over in case the original database goes down.

This is possible since there is a single DNS name to account for automatic
failover; hence, if master db goes down, standby takes over automatically. The
applications and users will not be affected by it.

**Read Replicas can be set-up as Multi-AZ for disaster recovery**.

RDS -- Security Encryption
--------------------------

**At rest** :

- You can encrypt the master and read replicas with AWS KMS and CMK key
  (AES-256).
- Encryption must be defined at creation. If it is not, then read replicas also
  cannot be encrypted.
- Transparent Data Encryption (TDE) is available for Oracle and SQL Server.

**In flight** :

- SSL certificate is used to encrypt the data sent to RDS.
- Provide SSL options with trust certificate when connecting to the RDS.
- To enforce SSL, you must
    - PostgreSQL: `rds.force_ssl=1` in the AWS RDS Console.
    - MySQL: within the DB, apply `GRANT USAGE ON *.* TO "mysqluser'@%'REQUIRE
      SSL;"`.

RDS -- Encryption Operations
----------------------------

**Encrypting the RDS backups**

- Snapshots of un-encrypted RDS is also un-encrypted.
- Thus, copy a snapshot of encrypted one.

**Encrypting an un-encrypted RDS DB**

- Create a snapshot of the un-encrypted db.
- Copy the snapshot and enable encryption for the snapshot.
- Restore the db from the encrypted snapshot.
- Migrate applications to the new encrypted DB and delete the old one.

RDS -- Network Security and IAM
-------------------------------

**RDS Network Security**

- Usually, RDS will be deployed within **private subnet**.
- RDS security works with security groups, controlling which source (IP, or
  other security group) can access the RDS.

**RDS Access Management**

- IAM policies help control who can manage AWS RDS through the RDS API.
- Can use tranditional database username and password as well.
- For MySQL and PostgreSQL, IAM-based authentication can be used.

RDS -- IAM Authentication
-------------------------

- **Note: IAM Authentication works with only MySQL and PostgreSQL**.
- Access via obtaining auth token obtained through IAM and RDS API calls; token
  lasts for 16 mins.

- The network traffic in and out are encrypted using SSL.
- IAM is used to manage the users instead of traditional DB.
- Can leverage IAM roles and EC2 instance profiles to integrate.

RDS -- Security Summary
----------------------

**Encryption at rest**

- Must enable it when you first create the RDS DB instance.
- Or, if un-encrypted, you must go through the process of taking the snapshot,
  encrypting it, and restoring the db.

**Your responsibility**

- Check the port, IP and Security Group inbound rules in DB's security group.
- Use the provided database user creation and permissions, or have it managed
  by IAM instead.
- Ensure parameter groups, or DB is configured to only allow SSL connections.

**AWS responsibility**

- No SSH access.
- No munal DB or OS patching.
- No way to audit the underlying instance.

---

Amazon Aurora
-------------

It is a prorietary tech from AWS (not open-source) and it supports Postgres and
MySQL - the drivers for both and work with Aurorra db as well. It is made with
AWS Cloud optimization - it promises 5 times improvement over MySQL on RDS and
3 times improvmenet over postgres on RDS.

- **Storage grows automatically** in increments of 10 GB (upto 64 TB).
- Aurora can have upto 15 Read Replicas (compared to 5 RDS); and has faster
  replication process (~10 ms lag).
- Failover in Aurora is instananeous - High availability.
- Aurora costs ~20% more than RDS.

Aurora -- High Availability and Read Scaling
--------------------------------------------

It spreads 6 copies of your data across 3 AZs.

- 4 copies out of 6 is needed for WRITE.
- 3 copies out of 6 is needed for READ.

A single Aurora instance takes WRITE (master), and when master goes down,
automated failover takes over in less than 30 seconds. Aside from master, there
can be 15 Read Replicas to serve READs. It supports cross-region replication.

Aurora -- DB Cluster
--------------------

i.e. We have a shared storage volume that spans over multi-AZ; it is
auto-expanding from 10 GB to 64 TB. Here, we have a master Aurora that WRITE to
the volume. Since master may fail or be replaced, Aurora provides the **WRITE
Endpoint** which is a DNS name. Also, we have may read replicas which are
auto-scaled. To manage the automatically scaling read replicas, Aurora also
provides the **READ Endpoint** which is a connection point for load balancers.

Aurora -- Security
------------------

- It is very similar to RDS since it uses same engines.
- Encryption at rest using KMS.
- Automated backups, snapshots and replicas are also encrypted.
- Encryption in flight using SSL.
- Possible to authenticate using IAM tokens.
- Your responsibility includes protecting the instance with security groups.

Aurora -- Serverless
--------------------

- Automated database instantiation; autoscales based on usage.
- Good for infrequent, intermittent, or unpredictable workloads.
- Pay-per-second.

Aurora -- Global
----------------

- Aurora Cross Region Read Replicas are useful for disaster recovery.

- Aurora Global Daabase:
    - 1 Primary Region (READ/WRITE).
    - Up to 5 secondar (READ ONLY) regions, replication lag is < 1 second.
    - Upto 16 Read Replicas per secondary region.
    - Promoting another region has an RTO of < 1 minutes.

---

AWS ElastiCache
---------------

Elasticache is a managed Redis / Memcahced service. Cache is a in-memory
databases with high-performance which helps to reduce the load off the
databases for read-intensive workloads.

- WRITE scaling uses sharding.
- READ scaling using Read Replicas.
- Multi-AZ with failover capability.
- It is managed by AWS (OS maintenance, patching, optimizations, setup, backups...).

ElastiCache -- Use Cases
------------------------

**1 Relieving Load from Main DB**

- The application will first query the ElastiCache; if it is _cache miss_, the
  app will instead go to the origin (i.e. RDS) and then store the result in the
  cache.
- So, next time application needs the same data, it does not have to go all the
  way to the origin. Instead, it can get it fast from the ElastiCache.

**2 User Session Store**

- This enables making our application "stateless" by storing the session data
  into the ElastiCache instead. By doing so, it does not have to care who nor
  what state it is in.
- Suppose that user request is forwarded to another application in the
  instances managed by ASG. The application will query the ElastiCache to check
  user information and restore its state.

ElastiCache -- Redis and Memcached
----------------------------------

**REDIS**

- Multi-AZ with automatic failover.
- Read Replicas to scale reads and have high availability.
- **Data durability using AOF persistence**.
- **Backup and restore features**.

**MEMCACHED**

- Multi-node for partitioning of data (Sharding).
- **Non-persistent**.
- **No Backup, nor restore**.
- Multi-threaded Architecture.
- **Sharding** - a part of cache is on one shard where each shard is the
  memcached node.

ElastiCache -- Caching Implementation Considerations
----------------------------------------------------

**Is it safe to cache data?**

- data can go out of date, resulting in eventual consistentcy.
- in other words, data in cache is out of sync with the origin database.

**Is caching effective for that data?**

- Pattern : data mutates slowly
- Anti-Pattern : data changes rapidly

**Is data structured well for caching?**

- Key-Value caching
- Aggregate result caching

More at <https://aws.amazon.com/caching/implementation-considerations>.

Cache Strategy -- Lazy Loading, Cache-Aside and Lazy Population
---------------------------------------------------------------

When the application has _cache hit_ it can retrieve from the cache; if missed,
then it READs from the database. Then, it will write to cache so that
subsequent request for same object will not be _cache miss_.

Benefits:

- Only requested data is cached (the cache is not filled with unused data).
- Node failures are not fatal.

Cons:

- _Cache miss_ results in 3 round-trips.
- Can result in "stale" data.

```Python
def get_user(user_id):
    # check cache
    record = cache.get(user_id)

    if record is None:
        # run a DB query
        record = db.query("select * from users where id = ?", user_id)
        # populate the cache
        cache.set(user_id, record)
        return record
    return record
```

Cache Strategy -- Write Through
-------------------------------

If _cache hit_, the app can use the data from ElastiCache. When the app WRITEs
to DB, it will also write to cache as well.

Benefits:

- Data in cache never goes "stale".
- Write penalty vs Read penalty (each write is 2-step process).

Cons:

- Missing data until it is added or updated in the DB. Mitigate by implementing
  lazy-loading strategy as well.
- Cache churn : a lot of data in the cache will never be used.



```Python
def save_user(user_id, values):
    # save to DB
    record = db.query("update users ... where id = ?", user_id, values)
    # save to cache
    cache.set(user_id, record)
    return record

user = save_user(17, {'name' : 'Jason Bourne'})
```

Cache Evictions (TTL)
---------------------

Cache Eviction occurs in 3 different ways:

- You manually delete the item.
- Cache is full and needs a space (LRU).
- Item has met is Time-To-Live (TTL).

TTL ranges from seconds to days.
