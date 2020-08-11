# 1 AWS Fundamentals: IAM and EC2



## Fundamentals

### AWS Regions

Most AWS Services are "region" based. This implies that it may require extra
work to get services working across regions.

Each region is a cluster of data centres.

Name convention is "us-east-1, eu-west-2, ...".

### AWS Availability Zones

Each region is divided into 2 to 6 availability zones.

Each AZ is its own independent data centre and AZs are connected to each other
through high bandwidth network.

Name convention is "us-east-1a, eu-west-2b, ...".


## IAM

### Introduction

IAM is Identity and Acess Management.

It is a centre point of your AWS security and manages:

- Users : usually a physical person.
- Groups : a group of users (i.e. Admins, DevOps, HR, ...).
- Roles : for AWS resources.

Root account should never be used and shared.

Accesses and permissions are granted through _Policies_ which are written in
JSON.

**IAM is global** - it is not bounded to a specific region.

MFA can be set-up.

Remember the least privilege principles.

### IAM Federation

A way for enterprises to integrate their own Active Directory with IAM. This
way, users can use AWS with their company crendentials.

### Root set-up

When an AWS account is first created, you can see your security status on IAM
console. It prompts you to do following:

- **Delete your root access keys**.
    - Root access keys give programmatic access to all the AWS resources.
    - Best is to never use it.
- Activate MFA for your root account.
- Create individual IAM users.
- User groups to assign permissions.
- Apply an IAM password policy.




## EC2

### Introduction

EC2 offers following:

- renting virtual machines (EC2).
- storing data on virtual drives (EBS).
- distributing load across machines (ELB).
- scaling services using an auto-scaling group (ASG).

### SSH

By default, EC2 security group allows for SSH port 22 access. We can SSH into
it using the newly generated key-pair or existing one. The `.pem` key is used
to SSH as follows:

    $ ssh -i key.pem ec2-user@your-ec2-public-ip

If the `.pem` key has a permission that is too permissible, it will throw an
error and warns you to change the file permission.

    $ chmod 0400 key.pem
    $ ssh -i key.pem ec2-user@your-ec2-public-ip

### EC2 Instance Connect

It is a way to connect to running instance on a browser from the EC2 console.

This is only possible with the EC2 instances that are provisioned with Amazon
AMIs.

### Security Groups

Security Group control how traffic flows into/from the EC2 instances. These are
aking to firewall, but it is not running within the EC2 instances. If the
traffic is blocked, then the EC2 instance would not realize it.

Common question would be why we get time-out on trying to access EC2
instances.

If the EC2 instance response back at all, it is not a security group issue. But
if we simply timesout, then it is likely that our traffic is blocked and
security group needs to be configured to allow our traffics.

Security Group can be attached to multiple instances at a time.

It is **region** and **VPC** scoped.

It can reference specific IP addr, CIDR block, or other security group; but not
DNS name.

### Elastic IP

An EC2 instance's public IP may change on every stop/run; if we need to, we can
request for upto 5 EIPs and associate to the instances (or network interface).

### EC2 User Data

It is used for _bootstrap_ instance when it **first** starts up. It is a script
that runs with `sudo` permissions when the machine boots up. You can use it to
install necessary packages, download files, create directories, and etc.

### EC2 Instance Launch Types

Learn to analyze which type is suited for which situation and allow for maximum
cost savings. There are

- On-demand
- Reserved (minimum 1 year contract)
    - Reserved Instances
    - Convertible Reserved Instances
    - Scheduled Reserved Instances : runs in specified schedule in day/week/month.
- Spot : set max price to pay; best cost savings but can lose instances.
- Dedicated Instances : hardware is not shared with any other customers.
- Dedicated Hosts : book an entire physical server.


#### On-Demand

- Pay per second after first min.
- No long term commitment.
- Great for short or unpredictable workloads.

#### Reserved

- Good savings, but pay upfront for long term commitment of 1 or 3 years.
- Recommended for steady state usage applications (i.e. databases).

- Convertible reserved instances
    - Can change the EC2 instance types.
- Scheduled reserved instances
    - Launch within the time window that you reserve.
    - i.e. on every Thursday 5-10 PM.

#### EC2 Spot Instances

- Best cost saving as you set the max price that you are willing to pay.
- But will "lose" instance if the current price goes above the set max price.
- Recommended for workloads that are resilient to disruptions.

- If the instance is reclaimed by AWS, you can set the behavior to stop or
  terminate. **No hibernate** interruption behavior supported.

#### EC2 Dedicated Instances

- Instance runs on a dedicated hardware.
- No control over the instance placement.

#### EC2 Dedicated Hosts

- Physically dedicated entire EC2 server.
- Have fine grain control over EC2 instance placements.
- Visibility into the underlying sockets and physical cores of the hardware.
- 3 year period reservation.

- Recommended for compliance reasons, or license 

### Elastic Network Interfaces

- ENIs are logical component in a VPC that is a virtual network card.
- Useful to move the ENIs over to different instances for failovers.
- **ENIs are bound to AZs**. They live in a subnet of the VPC.

- When EC2 is created, ENI will also be created and attached to the EC2
  instance as eth0.

- ENI consists of:

    - Private IP
    - Elastic IP
    - MAC addr
    - Security Group(s)

### EC2 Summary

- EC2 Pricing
    - Price (per hour) varies on different parameters such as Region, Instance
      Type, or OS.
    - Billed by second with a minimum 60 seconds.
    - Pay extra for data transfer, storage, EIPs, load balacing...

- AMIs are base image of the running instances.
    - You can provide custom AMI.
    - **AMIs are region locked**.

- Each instances have different characteristics based on RAM, CPU, I/O,
  Network, and GPU.

- There are "burstable" instances (T2) which can use "burst credits" to improve
  performance of CPU when under stress.
