# RDS, Aurora, and ElasticCache Overview

----

## RDS - Relational Database Service

- It is a managed DB service for DB using SQL as a query language.
- It allows you to create databases in the cloud that are managed by AWS.
    - Postres
    - MySQL
    - MariaDB
    - Oracle
    - Microsoft SQL Server
    - Aurora (AWS Proprietary database)

## Why use RDS when we can deploy DB on EC2?

- RDS is a managed service:
    - Automated provisioning, OS patching.
    - Continuous backups and restore to specific timestamp.
    - Monitoring dashboards.
    - Read replicas to improve read performance.
    - Multi AZ set up for DR (Disaster Recovery).
    - Maintenance windows for upgrades.
    - Scaling capability (vertical and horizontal).
    - Storage backed by EBS (GP2 or IO1).

- You cannot SSH into your RDS instance.

## RDS Back-ups

- Backups are automatically enabled in RDS.
- Automated Backups:
    - Daily full backup of the database (during maintenance window).
    - Transaction logs are backed-up by RDS every 5 minutes.
    - This allows you to restore db back to any point in time.
    - Default retention period is 7 days (upto 35 days).

- DB Snapshots:
    - Backups manually triggered by the user.
    - Retention period is indefinite.

---

- *Important* : know the difference between read replicas and multi-AZ; and
    their use cases!

## RDS Read Replicas for read scalability

- A RDS DB instance is used by Application that performs read/write.
- Suppose that a single RDS DB instance is not enough; we want to scale out.
- In this case, we can create up to 5 **Read Replicas**.
    - These can be within AZ, Cross AZ, or Cross Region!

- The replication is a ASYNC process; hence, the reads are eventually
    consistent (When READ, the data may be is older).

- Replicas can be promoted to become its own DBs.

- Applications must update the connection string in order to leverage read
    replicas.

## Read Replicas -- Use Cases

- Suppose you have a production database under normal load.
- We want to run a reporting application to run some analytics.
- However, this will create an extra load on the RDS DB instance that is
    currently serving the Production; and will degrade in performance.
- To prevent this, we would create a Read Replica of the RDS instance serving
    the Production Application. Then, use replica to serve the reporting
    application.

- Remember that **Read Replica** is for the purpose of **SELECT** kind of SQL
    query; not INSERT, UPDATE, or DELETE.

## Read Replicas -- Network Cost

- In AWS, there is a network cost when data moves between AZs.
- i.e. Suppose we have an RDS DB instance in one AZ, and created its Read
    Replica in another AZ. This will incurr network cost.

- To reduce this cost, you can have your Read Replica in same AZ.

---

## RDS -- Multi AZ (Disaster Recovery)

- SYNC Replication!
- Suppose we have a master RDS DB instance in one AZ.
- Via SYNC replication, there will be a replica instance on standby in another
    AZ.
- Hence, there is a single DNS name - to account for automatic failover.
- In case master goes down, automatically standby takes over; the app using the
    database via DNS name is unaffected.

- Increases availability - the concept of high availability.
- No manual intervention in apps.

- **Read Replicas be setup as Multi-AZ for Disaster Recovery.**

## Creating RDS

- RDS databases has its own service section.
- We choose our Engine options: Aurora, MySQL, MariaDB, PostgreSQL, Oracle,
    Microsoft SQL Server.
- Choose your template (Prod/Dev/Free).
- Make Credentials to use with DB instance.
- We can choose Multi-AZ deployment; creating the standby instance.
- VPC options; Security groups; AZs.
- Able to configure to access DB using IAM DB users/roles.
- Choose Backup retention period.
- Monitoring.
- Logs.

- Provisioned RDS has a DNS endpoint and port to connect to.
- Security group will determine inbound/outbound traffics.

## RDS -- Security Encryption

- At rest encryption:
    - Possibility to encrypt the master & read replicas with AWS KMS - AES-256.
    - Encryption has to be defined at launch time.
    - If the master is not encrypted, then the read replicas cannot be encrypted!
    - Transparent Data Encryption (TDE) available for Oracle and SQL Server.

- In-flight encryption:
    - SSL certificates to encrypt data to RDS in flight.
    - Provide SSL options with trust certificate whe nconnecting to database.
    - To enforce SSL:
        - PostgreSQL: rds.force_ssl=1 in the AWS RDS Console (parameter groups).
        - MySQL: Within the DB, `GRANT USAGE ON *.* TO 'mysqluser'@%'REQUIRE SSL;'`

## RDS Encryption Operations

- How to encrypt the RDS Backups?
    - Snapshots of un-encrypted RDS databases are also un-encrpyted.
    - Vice versa.
    - So, copy a sanpshotinto an encrypted one.

- To encrypt an un-encrypted RDS DB,
    - Create an snapshot of the un-encrypted database.
    - Copy the snapshot and enable encryption for the snapshot.
    - Restore the database from the encrypted snapshot.
    - Migrate applications to the new encrpyted DB and delete the old DB.

## RDS Security - Network Security & IAM

- Network Security
    - RDS Databases are usually deployed within a private subnet, not in a public
        one.
    - RDS security works by leveraging security groups (same as EC2 instances).
        - controls which IP / Security group can communicate with RDS.

- Access management
    - IAM policies help control who can amange AWS RDS (through the RDS API).
    - Can use traditional master username/password to connect to the database.
    - For MySQL and PostgreSQL, IAM-based authentication can be used.

## RDS - IAM Authentication

- IAM database authentication works with MySQL and postgreSQL.
- Do not need a password; just an authentication token obtained through IAM &
    RDS API calls.
- Token lasts for 15 mins.

- i.e. EC2 instance is running under its own security group and IAM role. The
    IAM role granted to this EC2 instance is able to make an API call to get
    auth token from RDS Service. Using this token, it can access the RDS
    database (traffic would be also encrypted - SSL).

- Benefits:
    - Network in/out must be encrypted using SSL.
    - IAM to centrallu manage users instead of DB.
    - Can leverage IAM Roles and EC2 instance profiles to easy integration.

---

## RDS Security -- Summary

- Encryption at rest?
    - Is done only when you first create the DB instance.
    - Or, if the DB is unencrypted, first take the snapshot; copy the snapshot
        as encrypted; create DB from the snapshot.

- Your responsibility:
    - Check the port / IP / Security group inbound rules in DB's security group.
    - In-database user creation and permissions or managed through IAM.
    - Creating a DB with or without public access (VPC).
    - Ensure parameter groups or DB is configured to only allow SSL connections.

- AWS responsibility:
    - No SSH access.
    - No manual DB or OS patching.
    - No way to audit the underlying instance.

---

# Amazon Aurora

- Do not need an in-depth understanding; but just know what it is.

- Aurora is proprietary technology from AWS (not open sourced).
- postgres and MySQL are both supported as Aurora DB which means your drivers
    will work as if Aurora was a postgres or MySQL db.

- Aurora is built with AWS Cloud optimization in mind;
    - 5 times improvement over MySQL on RDS.
    - 3 times improvement over postgres on RDS.

- Storage automatically grows in increments of 10GB (upto 64 TB).
- Aurora can have upto 15 Replicas while MySQL has 5; and much faster
    replication process (sub 10 ms replica lag).
- Failover in Aurora is instantaneous; it is high availability.
- Aurora costs about 20% more.

## Aurora High Availability and Read Scaling

- 6 copies of your data across 3 AZ:
    - 4 copies out of 6 needed for writes.
    - 3 copies out of 6 needed for reads.
    - **Self-healing** with peer-to-peer replication.
    - Storage is striped across 100s of volumes.

- One Aurora Instance takes writes (master).
- Automated failover for master in less than 30 seconds.
- On top of master, upto 15 read replicas can be used to serve reads.
- Support Cross Region Replication.

## Aurora DB Cluster

- i.e. We have a shared storage volume that spans over multi-AZ; it is
    auto-expanding from 10 G to 64 TB.
- Here, we have a master Aurora that writes to the volume. Since master may fail
    and be replaced, Aurora provides the **Write Endpoint** (DNS name).
- Also, we may have several read replicas that will read from the shared volume;
    this can be autoscaled.
- It would be hard to manage all the read replicas being scaled in and out;
    thus, Aurora provides **Reader Endpoint** (DNS name), which is connection to
    Load Balancing.

## Features of Aurora

- Automatic fail-over (replacing master).
- Backup and Recovery.
- Isolation and Security.
- Industry Compliance.
- Push-button Scaling.
- Automated Patching with Zero Downtime.
- Advanced Monitoring.
- Routine Maintenance.
- Backtrack: restore data to any point of time without using backups.

## Aurora Security

- Similar to RDS because uses the same engines.
- Encryption at rest using KMS.
- Automated backups, snapshots, and replicas are also encrypted.
- Encryption in flight using SSL (same process as MySQL or postgre).
- Possibility to authenticate using IAM token (same method; roles assigned can
    grab auth token from RDS service).
- You are responsible for protecting the instance with security groups.
- Same you cannot SSH.

## Aurora Serverless

- Automated database instantiation and autoscaling based on actual usage.
- Good for infrequent, itermittent or unpredictable workloads.
- No capacity planning needed.
- Pay per second, can be more cost-effective.

- Shared storage volume has Amazon Aurora on top which can be auto-scaled. Proxy
    Fleet (managed by Aurora) is where Client is connecting to.

## Global Aurora

- Aurora Cross Region Read Replicas:
    - useful for disaster recovery.
    - simple to put in place.

- Aurora Global Database (recommended):
    - 1 Primary Region (read / write).
    - Up to 5 secondary (read-only) regions, replication lag is less than a second.
    - Up to 16 Read Replicas per seconday region, helps to decrease latency.
    - Promoting another region (for diaster recovery) has an RTO (Recovery Time
        Objective) of less than a minute.

- i.e. We may have a Aurora DB where our application read / write from at one
    primary region (say, us-west-1). We can set up a seconday region (i.e.
    eu-west-1) by replication. And apps set up in the eu-west-1 can read from
    the replica.

## Creating an Aurora instance

- Aurora is created under RDS -> Create database.
- There is an Aurora option amongst the Engine options.
- We can choose to create Aurora database location: Regional or Global.
    - Global is where Aurora database is provisioned in multiple AWS regions.
    - Writes in the primary AWS Region are replicated with typical latency of
        less than 1 sec to seconday Regions.

- We can select various database features:
    - One writer and multiple readers.
        - Support multiple readers reading from same storage volume as writer.
        - General purpose.
    - One writer and multiple readers - Parallel query.
        - Analyze and improve performance of queries made.
    - Multiple writers.
        - Supports multiple writer instances to same storage volume.
    - Serverless.
        - Specifiy min/max amount of resources required; Aurora scales capacity
            based on database load.
        - Good for unpredictable workloads.

- Choose your credentials.
- Select DB instance size - performance.

- High Availability is made via chooseing Multi-AZ depolyment option.
    - Creates an Aurora replica or reader node in a different AZ.

---

# AWS ElastiCache

- If RDS was to get managed Relational Databases...
- Then, ElastiCache is to get managed Redis or Memcached.
- Caches are in-memory databases with really high performance, low latency.
- They help to reduce the load off the databases for read-intensive workloads.
- Helps to make your application stateless.
- Write Scaling using sharding.
- Read Scaling using Read Replicas.
- Multi-AZ with Failover Capability.
- AWS takes care of OS maintenance / patching, optimizations, setup,
    configuration, monitoring, failure recovery and backups.

## How does ElastiCache help us? Where in Solutions Architecture it fits in?

### 1. Releving Load off main database.

- Our application will first queries the ElastiCache; if it is not available, it
    will get from RDS and store in ElastiCache.
- **Cache hit** grabs directly from ElastiCache.
- **Cache miss** reads from DB; app should be coded to save the data to cache.
- Helps relieve load in RDS.
- Cache must have an invalidation strategy to make sure only the most freqeuntly
    accessed data is stored.

### 2. User Session Store

- User interacts with our 'stateless' applications; meaning that application is
    not aware of who it is dealing with nor what state is it in.
- This can be done by allowing application to write the session data into the
    ElastiCache.
- Suppose user request now hits the another application in our manage
    auto-scaling group; the application will query the ElastiCache for user
    information to realize what state that user is in - in this case, user's
    session data is restored.
- The instance retreives the data and realize that the user is already logged in.

## ElastiCache -- Redis vs Memcached

- REDIS:
    - Multi AZ with Auto-Failover.
    - Read Replicas to scale reads and have high availability.
    - Data Durability using AOF persistence.
    - Backup and restore features.
    - In many ways, very much similar to RDS storages.

- MEMCACHED:
    - Multi-node for partitioning of data (sharding).
    - Non-persistent.
    - No backup and restore.
    - Multi-threaded Architecture.
    - Sharding is the key here; part of cache is on one shard, and another on
        other shard, where each shard is the memcached node.
    - In a sense, this is more pure cache.

## Creating ElastiCache

- ElastiCache is available under its own service.
- Creating ElastiCache cluster first begins with selecting the Cluster engine
    option: Redis or Memcached.
    - Redis for Multi-AZ with Auto-Failover; enhanced robustness. Can also be
        used as a database if needed.
    - Memcached for high-performance, distributed memory object caching system.
        Mostly intended for dynamic web applications.

- Various security options available.
    - It has its own security group.
    - Encryption at rest?
    - Encryption in-transit?
        - Redis AUTH?
        - Redis Auth token requires for access.

- Backup options with Redis available.

## Caching Implementation Considerations

- Read more at https://aws.amazon.com/caching/implementation-considerations

- Is it safe to cache data?
    - data may be out of data; eventually consistent.
    - i.e. data stored in cache is out of sync with database.
- Is cahcing effective for that data?
    - Pattern: data changing slowly; few keys are frequently needed.
    - Anti-Pattern: data changing rapidly, all large key space frequently
        needed.
- Is data structured well for chacing?
    - i.e. key-value caching, or caching of aggregations results.

- **Which caching design pattern is the most appropriate?**

## Lazy Loading / Cache-Aside / Lazy Population

- Suppose we have our application, ElastiCache, and RDS.
- If application has a cache hit, it can retrieve the data from ElastiCache.
- In case of cache miss, application makes READ from DB.
    - Then, it will write to cache, so that any other app that is requesting the
        same data will access it faster with subsequent cache hits.

- Pros:
    - Only requestd data is cached (the cache isn't filled with unused data).
    - Node failures are not fatal (just increased latency to warm the cache).

- Cons:
    - Cache miss results in 3 round trips, which causes delays.
    - Stale data; it is possible that cache'd data is out of date.

- **Note** that you are expected to be able to read pseudocode.

- Following is Lazy-Loading strategy:

    ```Python
    def get_user(user_id):
        # check the cache
        record = cache.get(user_id)
        
        if record is None:
            # Run a DB query
            record = db.query("select * from users where id = ?", user_id)
            # Populate the cache
            cache.set(user_id, record)
            return record
        else:
            return record

    # Application code
    user = get_user(17)
    ```

## Write Through -- Add or Update cache when database is updated

- Suppose the same set up.
- When cache hit, app can use the data from ElastiCache.
- When app WRITES to DB, it will also write to cache as well.

- Pros:
    - Data in cache is never stale; reads are quick!
    - Write penalty vs Read penalty (each write is a 2 step process).

- Cons:
    - Missing Data until it is added / updated in the DB. Mitigation is to
        implement Lazy Loading strategy as well.
    - Cache churn - a lot of the data will never be read.

    ```Python
    def save_user(user_id, values):
        # save to DB
        record = db.query("update users ... where id = ?", user_id, values)
        # save to Cache
        cache.set(user_id, record)
        return record

    user = save_user(17, {'name' : 'Jason Loyd'})
    ```

## Cache Evictions and TTL

- Cache Eviction occurs in 3 ways:
    - You manually delete the item.
    - Cache is full and needs a space (LRU).
    - Item has met is Time-To-Live (TTL).

- TTL is helpful for any kind of data:
    - leaderboards; comments; activity streams.

- TTL can range from seconds to days.

- If too many evictions happen, it is a sign that we need to scale up or out.

---

## Cache Use Cases Summary

- Lazy Loading / Cache aside is easy to implement and works for many situations.

- Write-through is usually combined with Lazy Loading as targeted for the
    queries or workloads that benefit from this optimization.

- Setting a TTL is usually not a bad idea, except when you are using
    Write-through. Set it to a sensible value for your application.

- Only cache the data that makes sense (user profiles, blogs, etc...).

