# Relational Database Service

## What is relational database?

- Relational databases is basically a spreadsheet.
- Think in terms of rows and columns; a table.

## What are different Relational Databases available?

- In Amazon RDS, you can provision following:
    - SQL Server
    - Oracle
    - MySQL Server
    - PostgreSQL
    - Aurora
    - MariaDB

## Non-Relational Databases?

- Database:
    - Collection = Table
    - Document = Row
    - Key-Value Pairs = Fields

- i.e. JSON/NoSQL

    ```json
    {
        "id": "1234567890abcdefg",
        "firstname": "Rick",
        "surname": "James",
        "age": "11"
    }
    ```

- Amazon DynamoDB.

## What is Data Warehousing?

- Used for business intelligence. Tools like Cognos, Jaspersoft, SQL Server
    Reporting Services, Oracle Hyperion, and SAP NetWeaver.
- Used to pull in very large and complex data sets; used by management to do
    queries on data (i.e. performance vs target).

## OLTP vs OLAP?

- Online Transaction Processing (OLTP) differs from Online Analytics Processing
    (OLAP) in terms of types of queries you will run.

- OLTP Example:
    - order number 111111
    - pulls up a row of data such as name, data, addr and so on.

- OLAP Example (much more complex):
    - Net Profit for EMEA and Pacific for the Digital Radio Product.
    - Pulls in large numbers of records
    - Sum of Radios Sold in EMEA
    - Sum of Radios Sold in Pacific
    - Unit Cost of Radio in each region
    - Sales price of each radio
    - Sales price - unit cost...

- Basically, simpler query vs complex query.

## Elasticache?

- ElastiCache is a web service that makes it easy to deploy, operate, and scale
    an in-memory cache in the cloud. The service improves the performance of web
    apps by allowing you to retrieve information from fast, managed, in-memory
    caches, instead of relying entirely on slower disk-based databases.

- i.e. online store may want to cache the frequently sold items instead of
    fetching from the database each time.

- It supports two open-source in-memory caching engines:
    - Memcached
    - Redis

---

# AWS Database Types - Summary

- RDS is a OLTP involving either:
    - SQL; mySQL; postgreSQL; Oracle; Aurora; MariaDB

- DynamoDB is a NoSQL.

- RedShift - OLAP.

- Elasticache - in-memory caching used with:
    - memcached; redis.

---

# RDS follow-along

- we will provision RDS instance, and connect to web app.

- Head over to RDS Console.
- Let's create a new database.
    - We can choose different engine options. Choose MySQL.
    - On templates, choose Free Tier (Aurora is unavailable with Free Tier).
    - Then we can set the database credentials.
    - Because of Free Tier, we cannot select Availability & Durability.
- We should be able to see database created in RDS dashboard.

- Now, headover to the EC2 Console.
- Let's create a new EC2 instance.
    - leave everything in default.
    - but under advanced details, we will run a bootstrap script.
        - bascially, make EC2 instance run the script when running.
    
    ```sh
    #!/bin/bash
    yum install httpd php php-mysql -y
    yum update -y
    chkconfig httpd on
    service httpd start
    echo "<?php phpinfo();?>" > /var/www/html/index.php
    cd /var/www/html
    wget https://s3.eu-west-2.amazonaws.com/acloudguru-example/connect.php
    ```

    ```php
    <?php
    $surname = "abcdefg";
    $password = "abcdefg";
    $hostname = "YOURHOSTNAME";
    $dbname = "YOURDBNAME";

    //connection to the database
    $dbhandle = mysql_connect($hostname, $username, $password) or die("Unable to
    connect to MySQL");
    echo "Connected to MySQL using username - $username, password - $password,
    host - $hostname<br>";
    $selected = mysql_select_db("dbname", $dbhandle) or die("Unable to connect
    to MySQL DB - check dbname");
    ?>
    ```

- Once EC2 instance and RDS has been provisioned, we can now configure database
    information in EC2.
- SSH into the EC2 instance.
- After bootstrap script has been ran, there should be two files under
    '/var/www/html' directory: index.php, connect.php.
- edit connect.php to update hostname; hostname is the one that is used to
    connect to RDS instance and can be seen under RDS tab (Connect endpoint).
- go to webpage by EC2 instance IP/connect.php.
- This will result in error.
    - When RDS is created, there is a security group separate from EC2 instance.
    - Two separate cloud; security groups won't talk to each other.
- To resolve this, head over to the RDS console; and look into security group
    rules.
    - within security groups console, we add in new inbound rules.
    - whitelist our EC2 instance.
- Should see our EC2 instance connected to RDS instance.

---

# RDS - Back Ups, Multi-AZ & Read Replicas

## Automated Backups?

- Two types of backups for AWS: Automated and DB Snapshots.
- Automated Backups allow you to recover your db to any point in time within a
    'retention period'. This period can be between 1 ~ 35 days.
- Automated Backups will take a full daily snapshot and will also store
    transaction logs throughout the day.
- When you do a recovery, AWS will first choose the most recent daily back up,
    and then apply transaction logs relevant to that day. This allows you to do
    a point in time recovery down to a second within the retention period.

- It is enabled by default, and stored in S3 where you get free storage space
    equal to the size of your db. If RDS instance has 10 Gb, you also have 10 Gb
    storage.
- Backups are taken within a defined window. During backup window, storage I/O
    may be suspended while your data is being backed up and you may experience
    elevated latency.

## Snapshots

- DB Snapshots are done manually and stored even after you delete the original
    RDS instance; unlike automated backups.

## Restoring Backups

- Whenever you restore either an Automatic Bacup or a manual Snapshot, the
    restore version of the database will be a new RDS instance wit a new DNS
    endpoint.
- i.e. original.aws-region.rds.amazonaws.com would have changed to differnet
    endpoint such as restored.aws-region.rds.amazonaws.com

## Encryption

- Encryption at rest is supported for MySQl, Oracle, SQL Server, PostgreSQL,
    MariaDB & Aurora.
- It is done using the AWS Key Management Service (KMS).
- Once RDS instance is encrypted, the data stored at rest in the underlying
    storage is encrypted, as are its automated backups, read replicas, and
    snapshots.
- At present time, encrypting an existing DB Instance is not supported; to use
    Amazon RDS encryption for an existing db, you must first create a snapshot,
    make a copy of that snapshot and encrypt the copy.

- Goto RDS console; find the RDS instance.
- Here, we can choose various Instance Actions such as create read replica, take
    snapshot, restore to point in time, and etc.
    - we can restore db back to specific date and time down to seconds.

## What is Multi-AZ?

- We have ELB that routes traffic to multiple EC2 instances that has a single
    database behind (RDS instance).
- Suppose that RDS instance is based in us-west-1a region.
- When a change has been made to the database, it will synchronize to db in
    another region say us-west-1b, which is the Multi-AZ copy of db.
    - So, when one goes down, another one takes over automatically.
- Previously, we had to provide RDS instance endpoint (hostname) in order for
    EC2 instance to access to it.
- If this primary db goes down, Amazon detects the failure, and changes DNS to
    point to the Multi-AZ copy of the db in another region.
    - remember that we don't use IP directly with RDS; DNS endpoint.

- So, Multi-AZ allows you to have an exact copy of your production database in
    another Availability Zone. AWS handles the replication for you, so wher your
    production database is wrtten to, this write will automatically be
    synchronized to the stand by database as well.
- In event of planned database maintenance, DB Instance failure, or an AZ
    failure, Amazon RDS will automatically failover to the standby so that
    database operations can resume quickly without admin intervention.

- **Multi-AZ is Disaster only**.
- It is not primaily used for improving performance; for that, need 'Read
    Replicas'.

## What is Read Replica?

- Let's take the previous example: ELB routing traffic to multiple EC2 instances
    performing I/O to RDS database. 
- When read replicas are created, it pushes out copies of the original db.
- Why do this? It could be that most of the traffic is for reading from the
    database.
- Hence, each EC2 instance will instead of reading from that single production
    RDS database, they can choose to read from the read replicas instead.

- In short, Read replicas allow you to have a read-only copy of your production
    databases. This is achieved by using Asynchronous replication from the
    primary RDS instance to the read replica. You use read replicas primarily
    for very read-heavy database workloads.
- Currently, Read Replica databases are MySQL, postgreSQL, MariaDB, Aurora.

- Used for scaling; never disaster recovery.
- Must have automatic backups turned on to deploy a read replica.
- Can have upto 5 read replica copies of any db.
- Can have read replica of read replica.
- Each read replica has own DNS endpoint.
- Can have read replicas that have multi-AZ.
- Can create read replicas of Multi-AZ source db.
- Can be promoted to become own db; but breaks replication.
- Not tied to a single region.


