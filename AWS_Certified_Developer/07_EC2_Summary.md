# EC2 Summary

- dev exam focuses more on serverless architecture; will put less stress on the
    EC2.

---

## Different Pricing Models for EC2

- **On-Demand** : Fixed rate by the hour(or by the second), no commitment.
- **Reserved** : Capacity reservation for an year or three years; discounted.
- **Spot** : Bidding for capacity; for apps with flexible start/end times.
- **Dedicated Hosts** : Physical EC2 server; reduce costs by allowing you to use
    exisiting server-bound software licenses.

- Note: If a Spot is terminated by EC2, you do not pay for partial hour usage;
    you will be charged however when you terminate yourself.

## Different EC2 Instance Types

- Different hardware requirements (i.e. graphic intensive, etc...).
- FIGHT Dr MC PX

- SSD:
    - General Purpose SSD : Balances performance/price.
    - Provisioned IOPS SSD : highest performance SSD; for mission crritical,
        low-latency or high-throughput workloads. (over 10,000 IOPS).

- Magnetic:
    - Throughput Optimized HDD : low cost; frequently accessed,
        throughput-intensive workloads.
    - Cold HDD : Lowest cost HDD.
    - Magnetic : Legacy; can be boot volume.

---

## Types of Elastic Load Balancers

- Application
- Network
- Classic

- 504 Error? App has not responded within the idle timeout period;
    - troubleshoot app; web server or database server?
- Need IPv4 addr of your request sender? Look at X-Forwarded-For header.

---

## Route53

- Amazon's DNS service.
- Allows mapping of your domain name to AWS services:
    - EC2 instances
    - Load Balancers
    - S3 Buckets

---

## CLI Tips

- **Least Privilege** always give your users the minimum access.
- Create groups; all users in same group inherit same permission policy which
    can be assigned using policy documents.
- **Secret Access Key** is shown only once; if lost, delete the key pair at user
    page, then recreate one; remember to run 'aws configure' or change
    credentials in ~/.aws directory.
- **Do not use single access key** as it is a security risk. If a person leaves
    the company, then key needs to be changed and redistributed to all again.

- Roles will allow you to avoid using access key IDs and secret access keys.
- Roles are always preferred.
- Roles are controlled by policies.
- Changing the policy takes immediate effect on Roles.
- Can attach/detach roles to running EC2 instance without stop/terminating it.

---

## Encryption on device volume

- Can encrypt the root device volume (where OS is) using OS level encryption.
- Can encrypt the root device volume by first taking a snapshot, and creating a
    copy of that snapshot with encryption. Then, make an AMI of this snapshot
    and deploy the encrypted root device volume.
- Can encrypt additional attached volumes using the console, CLI, or API.

---

## AWS Databases Type

- RDS - OLTP
    - SQL; MySQL; postgreSQL; Oracle; MaricaDB; Aurora.

- DynamoDB - NoSQL
- RedShift - OLAP
- Elasticache - In Memory Caching
    - Memcahced; Redis

## Multi-AZ RDS?

- Disaster Recovery only; not used for performance boost - use Read Replicas
    instead.

## Read Replicas?

- Used for scaling.
- Must have Automatic Backups turned on.
- Can have upto 5 Read Replica copies of any db.
- Can have read replicas of read replicas (latency!).
- Each read replica will have its own DNS endpoints.
- Can have read replicas that have Multi-AZ.
- Can also create read replicas of Multi-AZ source dbs.
- Read replicas can be promoted to own db, breaking replication.

## Elasticache

- Scenario will be a db under stress from a lot of workload; which service to
    alleviate this problem?

- Elasticache is good if your db is heavy on read, and not frequently changing.

- RedShift if your db stress results from OLAP transactions.

## ElastiCache Use-Cases

- Use Memcache if
    - want simple solution?
    - object caching?
    - scale horizontally?

- Use Redis if
    - using lists, sets, or hashes.
    - doing sorting and ranking.
    - persistence.
    - multi-AZ.
    - pub/sub capabilities.
