# 4 RDS, Aurora and ElastiCache

## RDS

- It is a **relational database service**.
- Managed DB that uses SQL as its query language.
- Supports Postgres, MySQL, MariaDB, Oracle, SQL Server, Aurora.

### Why use RDS over DB in EC2?

- RDS is a _managed_ service.

- It automatically provisions, performs updats, and patches OS.
- Continuously backup and can restore to specific point in time.
- Provides monitoring dashboards.
- Can create **read replicas** to improve read performance.
- Muti-AZ can be set up for high availability.
- Scales both vertically and horizontally.
- Storage is backed by EBS (GP2 or IO1).

- Meaning, you cannot SSH into the running RDS instance.

### RDS Backups

- Backups are enabled automatically.
- Backups occur daily during maint window.
- Transaction logs are backed-up by RDS every 5 mins.
- Has 7 days retention period (upto 35 days).

- DB Snapshots are triggered manually.

### **Read Replicas**

- We can create upto 5 Read Replicas within AZ, Cross-AZ, or Cross-Region.
- Read Replicas will be async'd with the main RDS DB instance (eventually
  consistent).
- Read replicas can be promoted to become its own DB.
- Apps must update their connection string to read from new read replicas.

- One use case would be running an analytics tool:
    - We do not want to place an additional load on the production DB.
    - Instead, we create a read replicas and run our queries there.

- Note that it is **read** replica; it supports `SELECT` kind of queries.

- It costs extra when data is transfered from one AZ to another.

- Multi-AZ RDS allows for SYNC replication and used for disaster recovery.
    - There will be a single DNS name for automatic failover.
    - Read Replica will be in stand-by in case main DB fails.

### RDS Encryption

- At rest encryption:
    - Possible to encrypt the master & read replicas with AWS KMS (AES-256).
    - Encryption has to be specified at launch time.
    - If master is not encrypted, nor read replicas would be.
    - TDE is available for Oracle and SQL Server.

- In-flight:
    - SSL certificates to encrypt data to RDS in-flight.
    - Provide SSL options with trust certificate when connecting to db.
    - To enforce SSL:
        - PostgreSQL: `rds.force_ssl=1` in the console.
        - MySQL: `GRANT USAGE ON *.* TO 'mysqluser'@%' REQUIRE SSL;`

### RDS Encryption Operations

- Encryption RDS Backups:
    - Snapshots of unencrypted RDS is also unencrypted (and vice versa).
    - Can copy a snapshot into an encrypted one.

- To encrypt an unencrypted RDS database:
    - Snapshot the unencrypted db.
    - Copy the snapshot and enable encryption.
    - Restore the db from thn encrypted snapshot.
    - Migrate apps to the new encrypted db and delete the old.

### RDS Security (Network and IAM)

- Network Security
    - RDS db are usually deployed within a private subnet.
    - RDS uses security groups.

- Access Management
    - IAM policies help control who can manage AWS RDS.
    - Traditional username/password can be used to login to the db.
    - IAM-based authentication can be used to login into RDS Mysql and
      PostgreSQL.

### IAM Security

- IAM db authentication works with MySQL and PostgreSQL.
- You do not need a password; just an authentication token obtained through IAM
  and RDS API calls.
- Can leberage IAM roles and EC2 Instance profiles for easy integration.

## Aurora

- It is a proprietary tech from AWS (not open source - boo).
- Postgre and MySQL are both supported as Aurora DB (application drivers also
  work with Aurora).
- It is cloud optimized and claims higher performance over RDS.
- Automaitcally scaled in increments of 10GB upto 64 TB.
- It can have upto 15 Read Replicas (and faster).
- Failover in Aurora is instantaneous.

### Aurora High Availability and Read Scaling

- 6 Copies of your data across 3 AZs:
    - 4 copies out of 6 needed for writes.
    - 3 copies out of 6 need for reads.
    - Storage spreaded across 100s of volumes.

- One Aurora Instance takes writes (master).
- Automatic failover for master in less than 30 seconds.

### Aurora Security

- Similar to RDS because uses the same engines.
- Encryption at rest using KMS.
- Automated backups, snapshots and replicas are also encrytped.
- Encrytuin in flight using SSL.

### Aurora Serverless

- Automated db instantiation and autoscaling based on usage.
- Good for infrequent, intermittent or unpredictable workloads.
- No capacity planning required.
- Pay per second, can be more cost-effective.

### Global Aurora

- **Aurora Cross Region Read Replicas**:
    - useful for diaster recovery.

- **Aurora Global Database**:
    - 1 Primary Region
    - Upto 5 secondary regions, replication lag is less than 1 second.
    - Upto 16 Read Replicas per secondary region.
    - Promoting another region takes < 1 min.

## ElastiCache

- It is a managed Redis or Memcached.
- Write scaling using sharding.
- Read Scaling using Read Replicas.
- Multi AZ with Failover Capability.

### Redis v Memcached

- **Redis**
    - Multi-AZ with Auto-failover.
    - Read Replicas to scale reads and have high availability.
    - Data Durability using AOF persistence.
    - Backup and restore features.

- **Memcached**
    - Multi-node for partitioning of data (sharding).
    - Non-persistent.
    - No backup and restore.
    - Multithreaded architecture (scales vertically).

### Cache Strategies
#### Lazy Loading

- Cache miss? Go to DB and read from DB. Then store the result to the cache.
- Cache hit? Read from cache.

- There is no cache-churning (no unused data).
- Cache miss means 3 round trips to the DB and to cache.
- Data can be out of date with DB.

#### Write-Through

- When we write to cache, we also write to the cache as well.
- If cache hit, we read from cache.
- Reads are improved, but the cache may suffer from a lot of unused data.

### Cache Eviction and TTL

- Cache eviction can occur by:
    - Delete explicitly.
    - Cache is full and LRU item is evicted.
    - Set an item for TTL.

- TTL can range from seconds to days.



