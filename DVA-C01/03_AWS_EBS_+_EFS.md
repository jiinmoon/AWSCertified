EC2 Storage - EBS & EFS
=======================

What is an EBS Volume?
----------------------

- An EC2 machine loses its root volume when it is manually terminated.

- Unexpected terminations can happen (AWS will notify you).

- Thus, you may want to save your instance data in persistent drive.

- This is **Elastic Block Store (EBS) Volume**.

    - It is a network drive thta you can attach to your instances while running.

Elastic Block Store (EBS)
-------------------------

- It is a **network drive** (not a physical one).

    - It uses the network to communicate the instance, which means there may be
        a latency.
    - It can be detached from an EC2 instance and attached to another one
        quickly.

- **It is locked to an AZ.**

    - i.e. an EBS volume in us-west-1a cannot be attached to us-west-1c.
    - Thus, to move a volume across AZ, you must first snapshot it.

- Have a provisioned capacity (size in GBs and IOPS).

    - You pay for all the provisioned capacity, not per usage.
    - Thus, it makes sense to start small, and increase the capacity as you need
        them.

EBS Volume Types
----------------

| Type | Description |
| --- | --- |
| GP2 (SSD) | General Purpose; balance between $ and performance |
| IOI (SSD) | Best performance; for mission critical, low-latency workloads |
| STI (HDD) | Low $; frequently accessed/high throughput workload |
| SCI (HDD) | Lowest $; infrequently accessed workload |

- Difference is in their size, throughput, and IOPS.
- SSD types can be used as boot volumes.

Creating EBS
------------

- EBS volumes can be chosen when we are creating the EC2 instance during step
  (4) add storage.

- The root volume has to be either GP2 or IOI type.
- Additional volumes can be created and attached (i.e. /dev/sdb, /dev/sdc).
- The created EBS volumes can be individually monitored and viewed.

- After attaching the EBS volume, we need to format the block device
    with a file system, and then mount it.
- To do so, SSH into the running EC2 instance, make a filesystem, and mount
    the EBS volume.

        ssh -i KEYPAIR.pem ec2-user@EC2-PUBLIC-IP
        lsblk

- `lsblk` will return attached devices detected to the machine.
- Let's suppose that new EBS volume name is `xvdb`.

- New volumes are raw block devices - let's create a file system.
- To confirm that new device does not have a file system formatted, we run
    following command.

        sudo file -s /dev/xvdb

- If `file` command returns *data*, then there is no file system. Otherwise, it
    will display filesystem information.
- If we need a new filesystem, then we will create an ext4 file system on
    volume. Depending on the OS supplied to create the EC2 instance, you may use
    different file system types such as ext3 or XFS.

- **If there was existing data (for example, mounting an snapshot of another
    volume), then we shouldn't format the volume!**

        sudo mkfs -t ext4 /dev/xvdb

- Then, we need to mount to a directiory - let's say `/data`.

        sudo mount /dev/xvdb /data

- If you want to mount this volume on every system reboot, the `/etc/fstab` file
    should also be fixed.

- Good idea to create backup for the file first.

        sudo cp /etc/fstab /etc/fstab.orig

- Open up the fstab file and append following new line.

        /dev/xvdb /data ext4 defaults,nofail 0 2

- To check whether new entry is working, quickly unmount and mount again.

        sudo umount /data
        sudo mount -a
        lsblk

---

EBS Volume Types and Use Cases
-----------------------------

GP2 (SSD)
---------

- Recommended for most cases.
- Can be system boot volumes.
- Virtual desktops.
- Low-latency interacitve apps.
- Suitable for development and test environments.

- Size 1 GB - 16 TB.
- Small GP2 volumes can burst IOPS to 3000; max at 16,000.
- i.e. 3 IOPS per 1 GB. 16,000 IOPS for volumes greater than 5333 GB.

IOI (SSD)
---------

- For critical business apps that require sustained IOPS performance, or more
    than 16,000 IOPS per volume.
- Large database workloads, such as:
    - MongoDB, Cassandra, Microsoft SQL Server, MySQL, postgreSQL, Oracle.

- Size 4 GB - 16 TB.
- IOPS is provisoned (PIOPS):
    - MIN 100 ~ MAX 64,000 (Nitro instances), else MAX 32,000 (other instances).
- about 50 IOPS per 1 GB.

ST1 (HDD)
---------

- Streaming workloads that require consistent, fast throughput at a low price.
- Big data, Data warehourses, Log processing.
- Apache Kafra.
- Cannot be a boot volume.

- Size 500 GB - 16 TB.
- Max IOPS is 500;
- Max throughput of 500 MB/s - can burst.

SC1 (HDD)
---------

- Throughput-oriented storage for large volumes of data that is infrequently
    accessed.
- Lowest cost.
- Cannot be a boot volume.

- Size 500 GB - 16 TB.
- Max IOPS is 250;
- Max throughput of 250 MB/s - can burst.

---

EBS vs Instance Store
---------------------

- Some instances may not come with Root EBS volumes.
- Instead, they come with **Instance Store**, or ephemeral storage.

- Instance store is *physically* attached to the machine whereas EBS is a network
    drive.

- Instance Store Pros:

    - Better I/O performance.
    - Good for buffer / cache / scratch data /temporary content.
    - Data survives reboots.

- Instance Store Cons:

    - On stop or termination, the instance store is lost.
    - Cannot resize the instance store.
    - Backups are manually operated.

- Main question is, are you OK with losing your data? If so, instance store.

Local EC2 Instance Store
------------------------

- **Physical disk attached to the physical server where your EC2 is**.
- Becuase it is physical, very High IOPS - reaching upto million.
    - Exam note here: require high IOPS? Consider instance store.
- Disks up to 7.5 TB; can grow to reach 30 TB.
- Block Strage (just like EBS) - so is a file-system.
- Cannot be increased in size.
- Risk of data loss if hardware fails.

---

Elastic File System - EFS
-------------------------

- **Managed NFS (Network File System) that can be mounted on many EC2**.
- EFS works with EC2 instances in multi-AZ.

- Highly available, scalable, expensive (3 times GP2); but pay per use.
- EFS works with multiple EC2 instances behind the security group; and these EC2
    instances can be from different AZs.

- Use cases: content management, web serving, data sharing...

- Uses NFSv4.1 protocol.
- Uses security group to control access to EFS.
- EFS only works with **Linux** based AMIs.
- Encryption at rest using KMS.

- POSIX file system (Linux) that has a standard file API.
- File System scales automatically, pay-per-use, no capacity planning.

EFS - Performance & Storage Classes
-----------------------------------

- **EFS Scale**
    - 1000s of concurrent NFS clients, 10 GB+/s throughput.
    - Grow to Petabyte-scale network file system automatically.

- **Performance mode (set at EFS creation)**
    - General purpose (default) : latency-sensitive use cases (web server, CMS).
    - Max IO : higher latency, throughput, highly parallel (big data, media
        processing).

- **Remember This: Storage Tiers**
    - (lifecycle management feature - move file to different tiers after N days).
    - Standard: for frequently accessed files.
    - Infrequent access (EFS-IA): cost to retrieve files; lower price to store.

- Once EFS is created, it can be attached to EC2 instance at creation, or you
    may SSH into the running instance and configure.

- There will be a mounting instructions available; in short, you need to install
    efs-utils, use it to mount the EFS.

- You will also need to configure security group attached to EC2 instance to
    accept the connection from the NFS (TCP 2049, from appropriate source).

- Multiple EC2 instances can access the NFS at the same time.

---

EBS vs EFS
==========

EBS Summary
-----------

- EBS
    - attached to only one instance at a time.
    - locked to AZ.
    - GP2: IO increases as disk size increases.
    - IO1: can increase IO independently.

- To migrate an EBS volume across AZ:
    - need to take a snapshot;
    - then, restore that snapshot on anoter AZ.

- Root EBS volume by default is terminated along with EC2 instance; but can
    disable.

EFS Summary
-----------

- Mounting 100s of instances across AZs.
- EFS can be used to share website files.
- Only works with Linux based instances (POSIX).

- Pay per usage.
- About 3 times more expensive than EBS.
- Can use EFS-IA for cost-saving.

Instance Store
--------------

- Ephemeral storage.
- Physical drive; high IOPS.
- Lost if gone!

