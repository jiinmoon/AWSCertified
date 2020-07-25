AWS Elastic Block Store and Elastic File System
===============================================

AWS Elastic Block Store (EBS)
-----------------------------

An EC2 instance loses its root volume when it is manually terminated, and
unexpected terminations can happen. As such, we need a persistent block storage
device - this is **EBS volume**. It is a _network drive_ that you can attach to
the running instances.

- It uses the network to commnicate the instance (expect latency).
- It can be detached and attached to another instance easily.

EBS volumes are **locked to AZs**. Thus, an EBS volume located in us-west-1a
may not be attached to running EC2 instance on us-west-1b. To move a volume
across, you must first snapshot and create the volume from it.

It has a provisioned capacity (in GB and IOPS). Thus, you pay for all of the
provisioned capacity, not per usage. Hence, always start small and increase the
capacity.

EBS -- Volume Types
-------------------

| TYPE | Description |
| --- | --- |
| GP2 (SSD) | General purpose; balance between cost and performance |
| IOI (SSD) | Best performance; for mission critical workloads |
| STI (HDD) | Low cost; frequently accessed/high throughput |
| SCI (HDD) | Lowest cost; infrequently accessed |

EBS -- Creation
---------------

- EBS volumes can be chosen when we are creating the EC2 instances.
- The root volume has to either GP2 or IOI.
- The created EBS volumes can be individually monitored.

- After attaching and creating the EBS volume, we need to format the block
  device with a file system and mount it.
- To do so, SSH into the running EC2 instance, format and mount. Suppose that
  the new EBS volume name is `xvdb`. But first, let's check whether the device
  is formatted.

        $ ssh -i KEYPAIR.pem ec2-user@EC2-PUBLIC-IP
        $ lsblk
        $ sudo file -s /dev/xvdb

- If `file` returns _data_, then there is no filesystem; otherwise, it will
  display filesystem information on that device.
- You may choose different filesystems such as `ext3`, `ext4` and `XFS`.
- Here we will use `ext4` and mount the volume to `/data`.

        $ sudo mkfs -t ext4 /dev/xvdb
        $ sudo mount /dev/xvdb /data

- To automatically mount the volume on every reboot, we need to modify the
  `/etc/fstab` file. Open up the fstab file and append following line.

        /dev/xvdb /data ext4 defaults,nofail 0 2

EBS -- Volume Types and Use Cases
---------------------------------

**GP2 (SSD)**

- recommended for most use cases.
- can be system boot volume.
- for low-latency interactive apps, suitable for DEV and TEST environment.
- Size  1 GB - 16 TB.
- Small GP2 volumes can burst IOPS to 3000 to max 16,000.
- i.e. 3 IOPS per 1GB; 16,000 IOPS for volumes greater than 5333 GB.

**IOI (SSD)**

- for ciritical business apps that require sustained IOPS performance (or more
  than 16,000 IOPS per volume).
- large database workloads.
- i.e. MongoDB, Cassandra, Miscrosot SQL Server, MySQL, postgreSQL, Oracle.
- Size 4 GB - 1 TB.
- IOPS are provisioned (PIOPS):
    - min 100 ~ max 64,000; else max 32,000.
    - about 50 IOPS per 1 GB.

**ST1 (HDD)**

- streaming workloads that require consistent, fast throughput at a low price.
- big data, data warehousing, log processing.
- i.e. Apache Kafka.
- _cannot be boot volume_.
- Size 500 GB - 16 TB.
- Max 500 IOPS; Max throughput of 500 MB/s (can burst).

**SC1 (HDD)**

- throughput-oriented storage for large volumes of data that is _infrequently
  accessed_.
- lowest cost.
- _cannot be boot volume_.
- Size 500 Gb - 16 TB.
- Max 250 IOPS; Max throughput of 250 MB/s (can burst).

EBS vs Instance Store
---------------------

- Some instances may not come with Root EBS volumes.
- Instead, they come with **Instance Store**, or ephemeral storage.
- **Instance Store** is _physically_ attached to the machine whereas EBS is
  a network drive.

- Pros:
    - better I/O performance.
    - good for buffer, cache, temporary content.
    - data survives reboots.

- Cons:
    - **on stop or termination, the instance is lost**.
    - cannot resize the instance store.
    - backups are manually operated.

Local EC2 Instance Store
------------------------

- Physical disk is attached to the physical server where your EC2 instance is.
- **Becuase it is physical, it provides high IOPS (upto millions)**.
- Size upto 7.5 TB to 30 TB.
- It is a block storage (a file system).
- Data can be lost when hardware fails.

---

AWS Elastic File System (EFS)
-----------------------------

- EFS is a manage Network File System (NFS) that can be mounsted on many EC2
instances and it works **multi-AZ**.
- It is highly available, scalable, and expensive (3 times the $ of GP2).
- It works behind the security group - leverages security group to control
  access to the EFS.
- Use cases would be content management, web servicing, data sharing and etc.
- Uses NFSv4.1 protocol.

- EFS only works with Linux based AMIs.
- Encryption at rest with AWS Key Management Service (KMS).

- EFS is a POSIX file system that has a standard file API.
- It scales automatically, pay-per-use and no capacity planning required.

EFS -- Peformance and Storage Classes
-------------------------------------

- Once EFS is created, it can be attached to EC2 instance at creation, or you
  may SSH into the running instance and configure yourself.

- You need to install `efs-utils` to mount the EFS.

- Also, need to configure security group attach to the EC2 instance to accept
  the connection from the NFS (TCP port 2049).

- Multiple EC2 instances can access the NFS at the same time.



**EFS Scale**

- supports upto 1000s of concurrent NFS clients with 10+ GB/s throughput.
- grows to petabyte-scale automatically.

**Performance mode** (set at creation time)

- General Purpose (default) : latency-sensitive use cases (web server, CMS).
- Max IO : higher latencyu, throughput, highly parallel (big data, media
  processing).

**Storage Tiers**

- Standard : for frequently accessed files.
- Infrequent Access (EFS-IA) : cost to retrieve files; lower price to store.


---

EBS -- Summary
--------------

- EBS can only be attached to a single instance at a time.
- It is locked to AZ.
- GP2 : IO increases as disk size increases.
- IO1: can increase its IO independently.

- To migrate EBS volume across AZ, first snapshot it and restore it on another
  AZ.

- By default root EBS volume is terminated with the EC2 instance.

EFS -- Summary
--------------

- Mounting 100s of instance across AZs.
- EFS can be used to share website files.
- Only works with POSIX, Linux based insances.

- Pay-per-usage (3 times more expensive than EBS).
- EBS-IA is available to save on cost.

Instance Store -- Summary
-------------------------

- Physical (ephemeral) stoage.
- Used when HIGH, HIGH IOPS is required (in millions).
- Is lost when gone.


