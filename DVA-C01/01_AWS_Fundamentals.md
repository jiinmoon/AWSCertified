# AWS Fundamentals - Part I : Regions, IAM & EC2

---

## AWS Regions

- AWS is a global cloud provider, hence it has many data centers across the
    globe, which is called **Regions**.

- The naming conventions can be: us-east1, eu-west-2, ap-east-1, etc...

- Again, **Region** is a cluster of data centres.

- Most AWS services will be region-based.

## AWS Availability Zones

- Each region has many AZs (between 2 to 6).

- For example, for a Sydney region (ap-southeast-2), it may have:
    - ap-southeast-2a; ap-southeast-2b; ap-southeast-3c.

- Each **AZ** is one or more discrete data centers with redundant power,
    networking, and connectivity.

- But, these AZs are separate. Hence, they are isolated from disasters.

- While they may be separated, still connected with high bandwidth, ultra-low
    latency networking.

- On the management console, you will see your current region setting on the top
    right corner.
    - And when you switch between the AWS services, you will see the scope
        changing.
    - For example, EC2 is a region-based service. However, IAM is a global
        service.

- Visit Amazon Global Infrastructure page for more in-depth understanding.
    - It will list current state such as how many regions are operating, how
        many more regions are coming online, and what services are available in
        which region.

---

## IAM Introduction

- **Identity and Access Management** or IAM.

- Your AWS Security lies here:
    - Users
    - Groups
    - Roles

- Root account should never be used.
    - Root is the account that you first created with.
    - Users should be created with minimal required permissions.

- IAM is the center of AWS, managing the access control for services.

- Policies are written in JSON.

- Within IAM, we have following components:
    - Users : usually a physical person such as a developer, a sys admin, or a
        HR employee.
    - Groups : grouping of users that shares the same access policies such as a
        DevOps, HR, Admins, and so on.
    - Roles : used for internal usage within AWS resources (machines).
    - To each of these componenets, we may apply Policies (JSON doc) which
        defines the permissions.

- IAM is applied **global**.

- Again, permissions are governed by Policies (in JSON format).

- MFA can be set-up (and strongly recommended).

- IAM already has a list of predefined 'managed policies' so that you may not
    have to write the JSON from scratch and ground up.

- Practice good security : Principle of Least Privilege.

## IAM Federation?

- Big enterprises usually integrate their own repository of users with IAM.

- The users from the company can log into AWS using their company credentials.

- Identity Federation uses the SAML standard (Active Directory).

## IAM 101

- One IAM User per Physical Person.

- One IAM Role per Application.

- IAM credential is not to be shared.

- Don't be dunce; never hard code credentials in code.

- Never use root credentials beyond initial set-up.

---

## EC2 Introduction

- **Elastic Compute Cloud** or EC2.

- It is capable of:
    - renting virtual machines (EC2)
    - storing data on virtual drives (EBS - Elastic Block Storage)
    - distributing load across machines (ELB - Elastic Load Balance)
    - scaling the services using an auto-scaling group (ASG - Auto-Scaling Group)

- EC2 is region-based.

- When creating your EC2 instance...
    1. Choose your AMI (Amazon Machine Image)
        - the software environment such as which Nix distro is it running on.
    2. Choose your Instance type
        - how powerful do you need your virtual machine to perform?
        - select different numbers of vCPUs, memory, and etc.
    3. Configure instance details
        - how many instances to run?
        - network
        - subnet (which AZ do you want your instance to be based?)
        - assign IAM roles
        - etc...
    4. Add extra storage
        - this is the EBS volume.
        - need at least one to house our operating system.
    5. Add tags
        - used as quick identifier.
    6. Configure Security Group
        - Firewall.
        - By default, SSH port 22 will be available.
        - We can choose to whitelist specific IPs.
        - If this EC2 instance is used as a web-server, then we would create a
            new security group to add more groups. In particular, we would
            enable port 80 and 443 for HTTP/HTTPS protocols.

- When launching EC2 instance, it will give you key-pair.
    - This is a credential needed to SSH into the EC2 instance.

- Created EC2 instances can be controlled via SSH into their public IP.

    > ssh ec2-user@EC2_IPv4

- But this requires the key-pair credentials that we were given previously.
    Supply that '.pem' key file to SSH.

    > ssh ec2-user@EC2_IPv4 -i EC2_KEYPAIR.pem

- Notice that you will get an error that reminds you the key file has file
    permission that is open (i.e. 0644). This is your private key that should be
    unavailable to others.

    > chmod 0400 EC2_KEYPAIR.pem

- Now, you should be able to SSH into the running EC2 instance.

- Alternatively, you can connect SSH from your browser at the EC2 dashboard.
    This is called, **EC2 Instance Connect**; only available with EC2 instances
    provisioned with Amazon Linux 2 AMI.

## Security Groups

- One of fundamentals of security in AWS.

- Controls inbound/outbound traffics in/from the EC2 instances.

- Essentially, they are Firewalll on EC2 instances.

- They are used to regulate:
    - Access to ports.
    - Authorised IP ranges - IPv4 or IPv6.
    - Control of Inbound Network (from other to the instance).
    - Control of Outbound Network (from instance to other).

- i.e. your running EC2 instance can be configured to have a security group to
    accept inbound traffic from authorised IP at port. For instance, we can
    configure it such that only my own personal IP can SSH into the EC2.

- Security Groups can be attached to multiple instances and vice versa.

- It is locked down to a region / VPC combination.
    - Outside of region or VPC? Need new security groups.

- Security group is not some software running on EC2 instance; hence, if the
    traffic is blocked by the security group, EC2 instance won't even realize
    it.

- It is good to maintain one separate security group for SSH access.

- If your app is not accessible and times out, most likely a security group
    issue.

- If your app returns an actual connection refused error, then it is an app
    error or the instance has not been launched.

- By default, all inbound is blocked; all outbound is authorized.

- We can reference other security groups.
    - i.e. EC2 instance can have a security group that authorises another
        security group. Thus, another EC2 instance with that security group
        attached can access the first EC2 instance.

---

## Private vs Public IP (IPv4)

- Networking involves two IPs: IPv4 and IPv6.
    - IPv4 is the addr. But due to its contraint on maximum value, we are runing
        out of addr to hand-out (although we have work arounds).
    - IPv6 is the new addressing protocol.

- Public IPs are what is exposed in the 'public'.
- i.e. two web servers can communicate to each other using their public IP.

- Private IPs are hidden from the public since they are used within the closed
    network behind the gateway. For instance, a company will run its own
    protected private network behind the router (or gateway) which has the
    public IP to manage traffics in and out.

## Elastic IP?

- When you stop and start an EC2 instance, it can change its public IP!

- If you require fixed public IP, this is when you need an Elastic IP.

- It is an public IPv4 that you can own so long as you don't delete.

- With the Elastic IP address, you can mask the failure of an instance or
    software by rapidly remapping the address to another instance in your
    account.

- By default, you can only have upto 5 Elastic IP in your account (or increase
    it by asking AWS).

- However, best practice is to avoid using Elastic IP:
    - They are too simple; and often reflect poor architectural decisions.
    - Instead, better approach would be to use a random public IP and register
        DNS hostname (Route53).
    - Or, Load Balancer can be used to avoid using a public IP.

## How does it work with EC2?

- By default, EC2 machine has:
    - A private IP for the internal AWS Network.
    - A public IP for the web.

- Hence, when we SSH into the EC2 machine, we cannot use the private IP since we
    are not 'in' the same network. This is why we use public IP to SSH.

- **When Ec2 machine stops and start again, public IP can change**.

- SSH into the running EC2 instance, and then check its IP.

    > ssh ec2-user@ec2-ip -i keypair.pem
    > ifconfig -a

- Under the ethernet connection (eth0), we will see that its inet IPv4 is the
    private IP; not the same Public IP used to SSH in.

- Try to stop and start the EC2 instance; you will notice that private IP
    remains same, but public IP changed (may or may not).

- You can allocate Elastic IP address in the EC2 console.
- Once allocated, Elastic IP needs to be associated; associate it with EC2
    instance running.

- Now, when you check the running EC2 instance, it's public IP has changed to
    that of Elastic IPs instantly.
    - Even when EC2 instance is stop/start, the Elastic IP remains.

---

## Side Activity: Creating Apache webserver on EC2

- We will install Apache web server on the EC2 instance.

- First, ssh into the running EC2 instance. Elevate your privilege to update the
    software packages, and then install apache web server.

    > ssh ec2-user@ec2-ip -i keypair.pem
    > sudo su
    > yum update -y
    > yum install httpd -y

- Then, we will start the Apache service and check that it is running; and
    serving the web page.

    > service httpd start
    > service httpd status
    > curl http://localhost

- However, when you visit the EC2 instance on your browser, it will not work.
    This is because of **security group** as we discussed earlier.

- By default, inbound traffic on port 80 (HTTP) is not enabled.

- We can attach the new security group. To do so, create a new security group;
    adding in Inbound rules for HTTP/HTTPS from anywhere. 

- Now, when you visit the EC2 instance through browser should serve the webpage correctly now.

---

## EC2 User Data

- It is possible to bootstrap our instances using an EC2 User data script;

- When the machine first starts up, it will run the script once.

- EC2 user data is used to automate boot tasks such as:
    - installing updates.
    - installing necessary softwares.
    - downloading files.
    - ...

- Script will run with root privileges.

## EC2 user-data script

- Let's try writing a sample script that will install necessary componenets to
    display a simple webpage.

- Create a new EC2 instance; while configuring the details, we can find the User
    data section under advanced details.

    ```sh
    #!/bin/bash
    
    # upgrade to root.
    sudo su

    # update/install/start httpd service.
    yum update -y
    yum install -y httpd
    service httpd start
    echo "Hello Everyone : )" > /var/www/html/index.html
    ```

- Make sure to attach the appropriate Security Groups; and try visiting the new
    instance's web page when ready.

---

## EC2 Instance Launch Types

- Which type of EC2 instance will maximize the performance for your need while
    balance out the cost?

- **On-Demand** : short workload, predictable pricing.
- **Reserved** : minimum 1 year contract.
    - Reserved Instances : long workloads.
    - Convertible Reserved Instances: long workloads with flexible instances.
        - may change the running instance types.
    - Scheduled Reserved Instances: example - every Thursday between 1 - 5 pm.
- **Spot** : short workoads, cheap, but may lose instances.
- **Dedicated Hardware** : no one will share the hardware.
- **Dedicated Hosts** : book an entire physical server; control instance
    placement.

## EC2 On Demand

- pay per usage (billing per second, after first min).
- has the highest cost but no upfront payment.
- no long term commitment.

- Recommended for short-term and un-interrrupted workloads; where you cannot
    predict how your apps will behave.

## Reserved

- Upto %75 discount vs On-Demand.
- Pay upfront for long term commitment; 1 or 3 years.
- Specify what type of instance to use during reservation.
- Recommended for steady state usage apps such as databases.

- Convertible Reserved Instance
    - allows you to change the instance type but more $.
    - %54 discount.

- Scheduled Reserved Instances
    - only launch within time window reserved.
    - For when you need to run only a fraction of day / week/ month.

## Spot

- Can be upto %90 discount vs On-Demand.
- Instance can be 'lost' at any point in time if the price you have specified to
    pay is less tha nthe current spot price.
- But it is most cost-efficient instances in AWS.
- Useful for workloads that are resilient to failure such as
    - Batch jobs.
    - Data analysis.
    - Image processing.
    - etc...

- Should not be used for critical jobs.

- One great way to use it would be to use Reserved Instances as a base-line, and
    use On-Demand and Spot instances when the need arises.
    - i.e. running web server on EC2 instance. You may want to spin up extra web
        servers during a peak times such as weekends where traffic increases.

## Dedicated Hosts

- Physical dedicated EC2 server.
- Full control of EC2 Instance Placement.
- Visibility into the underlying socket / physical cores of the hardware.
- 3 year period reservation.
- Useful for software that have complicated licensing model (BYOL), or for
    company that have a regulatory requirements to fulfill.

## Dedicated Instances

- Instances running on hardware that is dedicated to you.
- May share hardware with other instances in same account.
- No control over instance placement.
- Know the difference vs Hosts.
- Biggest is that Dedicated Instances are charged per instances.

## Choosing the right instance type

- Think in terms of renting a room at Hotel.

- On-Demand is coming in whenever you would like, paying the full price.
- Reserved is planning ahead, and staying for extended time, getting a discount.
- Spot is when hotel allows people to bid on the room for how much they would
    like to pay for; can get kicked out if price goes up.
- Dedicated Hosts is just renting out the entire building.

## Basic Pricing

- Hourly costs.
- On-Demand $0.10
- Spot Instance $0.032 - 0.450
- Spot Block (1 to 6 hours)
- Reserved (12 months) $0.062
- so on...

---

## EC2 Elastic Network Interfaces (ENI)

- Logical component in a VPC that represents a **virtual network card**.

- Let's say within an availability zone, an EC2 instance is running. And to this
    EC2 instance's eth0, ENI will be attached to provide your EC2 instance with
    network connectivity.

- ENI has following attributes:
    - Primary private IPv4, one or more secondary IPv4.
        - i.e. a second ENI can attach itself to EC2 instance at eth1.
    - One elastic IP per private IPv4.
    - One Public IPv4.
    - One or more security groups.
    - A MAC Addr.

- Can create ENI independently and attach them on the fly on EC2 instances for
    failover.

- ENI is bound to the AZ.

- For example, we may have a two running EC2 instances. From one instance, we
    may move over its primary ENI to another.

- Network interfaces is one of displayed information of running instances.
    - Network interface eth0 is the primary one.
    - You will see interface ID attached which begins with 'eni-...'.

- Under Network & Security, there is also a Network Interfaces tab; here, you
    can create a new ENI.
    - You can attach a new ENI to running instances.
    - When EC2 instance has two ENIs attached, the secondary ENI will be
        attached to network interface eth1.

- You can freely detach and attach your ENIs so long as in same AZ.

---

# EC2 Summary+

## EC2 Pricing

- EC2 instance prices (per hour) varies based on:
    - region
    - instance type
    - On-Demand vs Spot vs Reserved vs Dedicated Host
    - running OS (Linux v Window v Private OS)

- Billed every second with a min 60 seconds.

- Pay for other factors such as storage, data transfer, fixed IP public
    addresses, load balancing...

- Do not pay for stopped instances.

## AMI

- EC2 instance comes with base images (Ubuntu, Fedora, RedHat, Windows ...).
- And customized further at boot via using EC2 User data.
- But we may provide our own image - AMI.

- Custom AMIs are advantageous since...
    - we can pre-install packages as needed.
    - faster boot time (no need to write a long user data script).
    - security concerns.
    - you want to maintain your own AMIs.
    - Active Directory Integration out of the box.
    - install apps ahead of time for faster deploying.
    - using someone's AMI...

- **AMIs are AWS region specific!**

## EC2 instances overview

- Instances have 5 distincitve characteristics:
    - RAM (amount, type, generation)
    - CPU (cores, frequency)
    - I/O
    - Network
    - GPU

- There are over 50 different type for various combinations.
- https://aws.amazon.com/ec2/instance-types/

- R/C/P/G/H/X/I/F/Z/CR is used to denote specialization of their RAM, CPU, I/O,
    Network, and GPU capabilities.
- M instances are balanced.
- T2/T3 instances types are burstable

- For exam, do not worry about knowing these!

## Burstable Instances (T2)

- T2 machines are those that have a balanced performance; but when needed, it
    can 'burst' and increase CPU performance.
- While bursting, it uses up the burst credits.
- When burst credit depletes, CPU performance goes down.
- When machine is in non-burst, credits accumulate over time.

- If your running T2 instance has constantly low credits, consider moving to
    diferent non-burstable instance (get one with increased CPU).

## T2 Unlimited

- unlimited credit for cost; can get expensive if left unmonitored!

---

# EC2 Checklist

- Know howto ssh into EC2 (and changing .pem file permissions).
- Know howto properly use security groups.
- Know the difference between private v public v elastic IP.
- Know how to use User Data to customize your machine at boot time.
- Know that you can supply own AMI.
- EC2 instances are billed by the second!

