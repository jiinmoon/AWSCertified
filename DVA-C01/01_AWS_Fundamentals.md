01 AWS Fundamentals --- Regions, AZs, IAM and EC2
=================================================

AWS Regions
-----------

AWS infrastructure spans across the globe scale. 

AWS has **Regions** which is a cluster of data centeres which are spread across
the world.

Most AWS services will be **region-basis** - it will require extra work to get
them working if the services are trying to work with each other cross region.

Region name designations are i.e. `us-east-1`, `ap-south-1`, etc.

More info at
[here](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/).

AWS Availability Zones (AZs)
----------------------------

Within a region, there are **Availability Zones** (from 2 to 6).

For example, within the `us-east-1` region, it may have 3 AZs which are

- `us-east-1a`
- `us-east-1b`
- `us-east-1c`

Each AZ has its own independent data centres with redundant power, networking,
and connectivity. This is to provide a _disaster recovery_ ( _high
availability_).

While physically separated, the AZs are connected to each other via high
bandwidth internal network connections.

---

AWS Identity and Access Management (IAM)
----------------------------------------

IAM is a **globally scoped**, central control point of AWS security; here you can
manage:

- Users (usually a physical person)
- Groups (logical grouping containing users. i.e. admins, hr, operations...)
- Roles (for internal AWS resources)

The root account of AWS is the one that was first used to create the AWS
account with. _It is not meant to be shared nor used in anyway_. Best practice
is to create the users, add them into the groups and manage their permissions
through the **IAM Policies** which are JSON documents.

**IAM Federation** is used to integrate the existing users pool from
enterprises which enables the company employees to login to AWS using the
company credentials. It uses the SAML standard (Active Directory AD).

Best practices for security management applies here:

- One user == One physical person.
- One role == One Applicatin (or AWS resource).
- Root account is not to be used outside of its initial setup.
- Do not leave IAM crendentials within the plaintext code.

---

AWS Elastic Compute Cloud (EC2)
-------------------------------

It is a **region-based service** that provides following:

- renting virtual machines.
- storing data on virtual drives (Elastic Block Storage EBS).
- distributing loads across the machines (Elastic Load Balancer ELB).
- scaling the services using an Auto-Sacling Group (ASG).

**Steps to creating an EC2 instance**:

1. Choose your Amazon Machine Image (AMI)
    - this is a software environment (i.e. Linux distro).
    - typically Amazon provides its own version of AMIs.

2. Choose your instance type
    - how powerful of virtual machine do you need?
    - select size of vCPUs, memory and etc.

3. Configure various instance details
    - how many instances to run?
    - network details such as subnets.
    - assign IAM roles.

4. Add extra storage
    - attach extra EBS volumes if needed.
    - by default, at least one is required for OS.

5. Add tags
    - **tags** are used as a quick identifiers.
    - key-value pair.

6. Configure **Security Group**
    - security group is like a **fire wall**; it controls the network traffics
      in and out of the virtual machines.
    - by default, SSH port 22 will be available in and out.
    - we can modify to allow which source is allowed on which port.
    - i.e. if this EC2 instance is used as a web server, we would create a new
      security group to enable port 80 and 443 for HTTP/HTTPs traffics.

When EC2 is created and being launched it will give an access key. This is
a credential required to SSH into the running EC2 instance. Store the `.pem`
file in a secure place.

Created EC2 instance can be SSH into by their public IPs. We will need to
supply the credential (previous `.pem` file).

        $ ssh ec2-user@EC2-PUBLIC-IP -i KEYPAIR.pem

Notice that this will result in an error saying that **`.pem` file has 0644
permission; this is a reminder that your private key should be unavailable to
others**. Modify the permission and we should be able to SSH in successfully.

        $ chmod 0400 KEYPAIR.pem

There is a web-browser based SSH service called **EC2 Instance Connect**; this
is only available with EC2 instances that are provisioned with Amazon Linux
2 AMI.

Security Groups
---------------

Security Groups are the main method of controlling network security within the
AWS. Think of them as **Firewalls** which blocks out in/outbound traffics. By
default, all inbound is blocked whereas outbound is allowed. They regulate
following:

- access to ports.
- authorised IP ranges (IPv4 or IPv6).
- control of inbound network (from OTHERS to THIS).
- control of outbound network (from THIS to OTHERS).
- different types of network protocols such as TCP, HTTP/S, SSH, ...

**Security Group is locked to a specific region and _VPC_**. If we create
a service or an application outside of a region, then we will also need to
create a new security group to assign.

Security Group is not some software running inside the EC2 instance; in fact,
the EC2 instance will not even realize that its traffic are being controlled.

**If an application is not accessible and _times out_, it is most likely
a security group issue**.

**If an application returns connection refused error, then it is an application
error or the instance has not been launched yet**.

Security groups work by referenceing each other. For example, an EC2 instance
has a security group A which allows access from security group B. Then, another
EC2 instance belonging to security group B is able to access the first EC2
instance.

Private vs Public IP
--------------------

As their name suggests, _Public IPs_ are exposed to the "public" or "web"
whereas _Private IPs_ are hidden inside the closed network behind the gateway.
For example, a company will run its own protected private network behind the
gateway, and the gateway has a public IP that is exposed to manage all the
incoming and outgoing traffics.

By using Public IP, the machine (or resource) is identifiable over the
internet; thus it should be unique across the entire web. For this reason, IPv6
is created to provision more IPs since IPv4 was reaching its limit.

Private IPs are used within the private subnet; thus, it only has to be unique
within that private subnet. The gateway (or proxy) is used to conect to the
web, and it handles the _port forwarding_ - mapping between private IPs to its
public IP and ports. This allows to mask the private IP over the router's IP
and port. This is the reason why a machine behind the gateway appears to send
the request from the IP of the router.

Elastic IP
----------

**EC2 instance's Public IP can change when it restarts**. To fix this, you may
request a **fixed** public IP called **Elastic IP (EIP)**. It is a fixed public
IPv4 that you own and you may request upto 5 of them (can request more).

However, using EIP usually reflects a poor architectural decision as it
directly exposes the AWS services. Better approach is to use DNS - register
a hostname and use the random public IP, or use Load Balancer to avoid using
a public IP at all.

EC2 User data
-------------

It is used to _bootstrap_ the EC2 instances with a command-line scripts. When
EC2 starts up, it will automatically execute the script in order. This is
useful for automating boot tasks such as updating, installing and downloading
necesary files. This script will run as a root.

Example of a User Data Script:

```bash
#!/bin/bash
sudo su -
yum update -y && yum install -y httpd
service httpd start
echo "New web-server is created!" > /var/www/html/index.html
```

EC2 Instance Types
------------------

**On-Demand**

- pay-per-usage (per second after first miniute).
- has the highest cost, but no contract and no long-term commitment.
- recommended for short-term and uninterrupted workloads that you cannot
  predict.

**Reserved Instances**

- 75% discount compared to On-Demand.
- 1 year or 3 years contract (pay upfront).
- specify the type of instance to use.
- recommended for steady state usage (i.e. databases).

- **Convertible**
    - allows changing of its instance type.
    - costs more (54% vs 75% discount).

- **Scheduled**
    - only launch within the specified time-frame.
    - for the tasks you need to run only during a fraction of day, week, or
      month.

**Spot**

- ~90% discount compared to On-Demand.
- You choose the maximum price to pay.
- **Instance can be lost at any point if the price you specified is less than
  the current spot price**.
- recommended for tasks that are resilient to failures such as:
    - batch jobs.
    - data analysis.
    - image processing.

**Dedicated Hosts**

- A dedicated, physical EC2 server.
- provides full control of EC2 instance placement.
- **able to go in-depth into the underlying hardware (to the socket and
  physical cores)**. This is useful for license purposes.
- pay per host with 3 year period reservation.
- recommended for software that has complicated licensing model, or for company
  that has regulatory requirements.

**Dedicated Instances**

- Instances running on hardware dedicated just to you.
- May share hardware with other instances in same account.
- Pay per instance.

EC2 Elastic Network Interface (ENI)
-----------------------------------

It is a logical network interface in a VPC, representing a _virtual network
card_. On a running EC2, its `eth0` will have ENI attached, providing a network
connectivity.

ENI has following features:

- it has a primary private IPv4 and one or more secondary.
- i.e. a second ENI can attach itself to EC2 instance at eth1.
- one EIP per one private IPv4.
- one or more security groups.
- MAC address.

ENI can be created independently of EC2 instances, and attach them however you
would like on the fly - useful for failovers.

**ENIs are bounded to AZs**.

---

EC2 Summary
===========

EC2 Pricing Models
------------------

- EC2 instance prices (per hour) varies based on following:
    - region
    - instance type
    - On-Demand vs Spot vs Reserved vs Dedicated Host
    - running OS

- Billed every second after first minute.

- Also charged for other add-ons such as storage, data transfer service, EIPs,
  load balancing and etc.

AMI
---

- EC2 instances can be breated with base images (Ubuntu, Fedora, Windows...)
  and can be further customized with User Data scripts.
- However, you may provide your own image called AMIs.

- Custom AMIs are used for following reasons:
    - prepackage the images with necessary softwares.
    - faster boot time (no User Data scripts to run).
    - manage and maintain own images for security reasons.
    - Active Directory (AD) integrations.
    - leverage on someone else's AMIs.

- **AMIs are region specific**.

EC2 Instances Overview
----------------------

- EC2 instances are categorized depending on its specialized hardwares
  associated with it.
- There are five characteristics to identify them by:
    - RAM
    - CPU
    - I/O
    - Network
    - GPU

- Detailed information at <https://aws.amazon.com/ec2/instance-types>.

- R, C, P, G, H, X, I, F, Z, CR designation is used to denote specialization of
  their hardware resources.

- `M` instances are most balanced.
- `T2`, `T3` instance types are **burstable**.

Burstable EC2 Instances
-----------------------

- `T2` machines have balaned performance, but when required, it may _burst_ to
  boost its CPU performance.
- While in _burst_, it drains **burst credits**; it is replenished overtime
  when not in burst.
- If running on `T2` instance and it is constantly low on burst credits, it is
  a sign that the application requires a better performing machine type.

- `T2 Unlimited` has unlimited amount of burst credits; but costly.


