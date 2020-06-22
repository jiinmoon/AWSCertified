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

