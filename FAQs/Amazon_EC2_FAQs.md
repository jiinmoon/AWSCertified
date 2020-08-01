# Amazon EC2 FAQs

## General

**What is Amazon Elastic Compute Cloud (Amazon EC2)?**

A web service that provides resizable compute capacity in the cloud.

**What is the difference between using the local instance store and Amazon
Elastic Block Store (Amazon EBS) for the root device?**

With EBS, data on the root device will persist independently from the lifetime
of the instance (i.e. stop and restart the instance).

With the local instance store, it only persists during the life of the
instance.

## Storage

### Amazon Elastic Block Storage

**What happens to my data when a system terminates?**

The data persists as long as the instance is alive with a local instance store;
with Amazon EBS volume, it will persist independently from the instance. For
persistance, use EBS or backup data to the S3.

If using EBS volume as a root partition, you will need to set the Delete On
Terminate flag to "N" if you want EBS volume to persist outside of life of the
instance.

**What kind of performance can I expect from Amazon EBS volumes?**
**Which volume type should I choose?**

- SSD-backed storage
    - transactional workloads (performance depends on IOPS).
    - IOPS-intensive database workloads.
    - boot volumes.
    - high IOPS.
- HDD-backed storage
    - throughput intensive workloads (measured in MB/s).
    - big-data workloads.
    - large I/O sizes and sequential I/O patterns.

**Do you offer encryptions on Amazon EBS volumes and snapshots?**

Yes.

### Amazon Elastic File System (EFS)

**How do I access a file system from an Amazon EC2 instance?**

Mount the file system on an Amazon EC2 Linux-based instance using the standard
Linux mount command and the file system's DNS name.

Amazon EFS uses NFSv4.1 protocol.

**How do I access my file system from outside of my VPC?**

Amazon EC2 instances within your VPC can access your file system directly;
Amazon EC2 Classic instances outside of VPC can mount via _ClassicLink_;
On-premises servers can mount via an _AWS Direct Connect_ connection to your
VPC.

**How many Amazon EC2 instances can connect to a file system?**

Supports between 1 to 1000 instances concurrently.

## Networking and security

### Elastic IP

**Why am I limited to 5 EIP addresses per region?**

IPv4 is scarce resource; you can apply for more EIP addresses.

**Do I need 1 EIP per each instance running?**

No. By default, every instance comes with a private IP and an internet routable
public IP. The private IP will remain associated with the network interface
when the instance is stopped and restarted, and only released when the instance
is terminated.

The public address is associated exclusively with the instance until it is
stopped, terminated or replaced with an EIP.

### Elastic Load Balancing

**What load balancing options does the Elastic Load Balancing service offer?**

- Classic Load Balancer : routes based on either app or network level.

- Apaplication Load Balancer : routes based on advanced app level information
  within the content of the request.

### Security

**Can I get a history of all EC2 API calls made on my account for security
analysis and operational roubleshooting purpose?**

To receive a history of all EC2 API calls (including VPC and EBS) within an
account, turn on CloudTrail.

## Management

### Amazon CloudWatch

**What is the min tume interval granularity for data that Amazon CloudWatch
receives and aggregates?**

Metrics are received and aggregated at 1 min intervals.

**Can I access the metric data for a terminated Amazon EC2 instance or
a deleted Elastic Load Balancer?**

Amazon CloudWatch stores metrics for terminated EC2 instances or deleted ELBs
for 2 weeks.

### Amazon EC2 Auto Scaling

**Can I automatically scale Amazon EC2 Auto Scaling Groups?**

Amazon EC2 Auto Scaling is a fully managed service; it launches and terminates
Amazon EC2 instances automatically to help ensure the correct number of Amazon
EC2 instances available to handle the load for your app.

## Billing and purchase options

### Billing

**How will I be charged and billed for my use of Amazon EC2?**

Pay only for what you use. Pricing is displayed hourly, but you pay by the hour
or second (min 60 sec).

Data transferred between AWS services in different regions will be charged as
Internet Data Transfer on both sides.

