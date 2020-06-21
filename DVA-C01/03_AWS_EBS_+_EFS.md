# EC2 Storage - EBS + EFS

---

## What is EBS Volume?

- An EC2 machine loses its root volume when it is manually terminated.
- Unexpected terminations might happen from time to time (AWS would notify).
- Thus, you may want to save your instance data elsewhere.
- This is EBS (Elastic Block Store) Volume;
    - It is a network drive thta you can attach to your instances while running.
    - It is a persistent data storage.

## EBS

- It is a network drive (not a physical one).
    - It uses the network to communicate the instance, which means there may be
        a latency.
    - It can be detached from an EC2 instance and attached to another one
        quickly.

- **It is locked to an AZ.**
    - i.e. EBS volume in us-west-1a cannot be attached to us-west-1c.
    - Thus, to move a volume across AZ, you must first snapshot it.

- Have a provisioned capacity (size and IOPS).
    - You pay for all the provisioned capacity, not per usage.
    - Thus, it makes sense to start small, and increase the capacity as you need
        them.

## EBS Volume Types

- 4 Types
    - GP2 (SSD) - General Purpose SSD; balance between price & performance.
    - IOI (SSD) - Highest-performance SSD; for critical, low-latency workloads.
    - STI (HDD) - Low cost HDD; frequently accessed/high throughput workloads.
    - SCI (HDD) - Lowest cost HDD; infrequently accessed workloads.

- Difference is in their size, throughput, and IOPS.
- GP2, and IOI can be used as boot volumes.

##
