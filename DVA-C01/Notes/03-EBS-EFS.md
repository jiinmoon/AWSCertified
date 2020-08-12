# 3 EBS and EFS

## EBS Volume

- EC2 instance will lost its root volume when it is manually terminated by
  default. Also, unexpected terminations may happen or you may need another
  persistent storage.

- This attached volume is EBS which is a **network drive** that you can attach
  to the running EC2 instance.

- Since it is not physical drive, you can detach and attach EBS volumes to one
  instance to another.

- **It is locked to AZ**; to move across AZs, you need to take a snapshot
  first.

- It has a provisioned capacity of size and IOPs.

- If the volume is new and not from another, then you also need to create
  a filesystem.

- When volume is provisioned, it is not mounted.

- Configure fstab if you wish to automatically mount the volume when stop and
  start again.

### Volume Types

- SSD:
    - GP2: general purpose; balance between cost and performance.
    - IO1: highest-performance.

- HDD:
    - ST1: low cost; for frequently accessed, throughput intensive workloads.
    - SC1: lowest cost; for infrequently accessed workloads.

- **Only SSD types can be root volume**.

#### GP2

- Can be system boot volume.
- For low-latency interactive applications and virtual desktops.
- 1 GB ~ 16 TB.
- Small GP2 bursts to 3,000 IOPs to max 16,000.
- Reaches max 16,000 IOPs at 5,334 GB (3 IOPs per 1 GB).

#### IO1 

- Can be system boot volume.
- For critical applications that requires sustained IOPS performance or **more
  than 16,000 IOPs per volume (exceeding GP2 limit)**.
- Good for large database workloads (MongoDB, Cassandra, Microsoft SQL
  Server...).
- 4 GB ~ 16 TB.
- IOPs is provisioned from 100 to 64,000 (for Nitro instance); otherwise,
  maximum IOPs to 32,000 (50 IOPs per 1 GB).

#### ST1

- Cannot be system boot volume.
- Consistent fast throughput for big data, log processing, etc.
- 500 GB ~ 16 TB.
- Max IOPs is 500.
- Max throughput of 500 MB/s (burstable, 40 MB/s per 1 TB).

#### SC1

- Cannot be system boot volume.
- Infrequently accessed, throughput oriented storage.
- Lowest cost.
- 500 GB ~ 16 TB.
- Max IOPs is 250.
- Max throughput of 250 MB/s.

### EBS vs Instance Store

- Instance can come with Instance Store as root volume (ephemeral storage).
- It is "physically" attached to the machine (whereas EBS was a network drive).
- It provides better I/O performance, cache, and persists over reboots.
- **But on stop or termination, the instance store is lost**.
- Instance store is also cannot be resized.
- Backups are done manually.

### Local EC2 Instance Store

- It is a physical disk attached to the physical server where your EC2 instance
  is.
- Extremely high IOPs (100,000 ~ millions).
- Risk losing data if the hardware fails.

## EFS

- Managed NFS that can be mounted on many EC2.
- **EFS works across AZs**. 
- Highly available and scalable; but also expensive (pay per use).

- Used mainly for content management, web serving, data sharing...
- Uses NFSv4.1 protocol.
- Uses security groups to control access to EFS.
- Compatible only with Linux based AMIs.

- Encryption at rest using KMS.

### EFS Performance

- EFS Scales
    - supports upto 1000s of concurrent NFS clients with 10 GB/s throughput.
    - automatically grows to petabyte-scale network file system.

- Performance mode (set at EFS creation)
    - General Purpose: latency sensitive use cases (web server).
    - Max I/O: higher latency, throughput, highly parallel (big data, media
      processing).

- Storage Tiers (move files after x days)
    - Standard: frequently accessed files.
    - Infrequent Access (EFS-IA): costs to retrieve files but lower price to
      store.

## EBS and EFS Summary
### EBS

- EBS volumes:
    - can be attached to only one instance at a time.
    - are AZ locked.
    - GP2 : IOPs increases as size increases.
    - UI1 : IOPs is provisioned.

- To migrate an EBS volume across AZ:
    - Take a snapshot.
    - Restore the snapshot in target AZ.
    - Backups will use IO and should not be doing it while there are a lot of
      traffic.

- **Root EBS Volumes of instances get terminated by default if the EC2 instance
  gets terminated**.

### EFS

- Mounts 100s of instaces across AZs.
- EFS share website files.
- Only for Linux AMIs (POSIX based).
- EFS has a higher price tag than EBS.
- Can leverage EFS-IA and lifecycle management for cost-saving.


