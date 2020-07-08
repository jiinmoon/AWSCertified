# AWS Fundamentals: Regions, IAM & EC2

---

## AWS Regions

- **Region** is a cluster of data centres.
- As AWS is a global cloud provider, it has many data centres (Regions) around
  the world. 
- The naming convetions can be "us-east1, eu-west-2, ap-east-1 ...". 
- Most AWS Services will be per region basis.

## AWS Availability Zones (AZ)

- Each region has many AZs (between 2 to 6).

- For example, a Sydney Region (i.e. ap-southeast-2) may have: ap-southeast-2a;
  ap-southeast-2b; and ap-southeast-3c.

- Each **AZ** is one or more discrete data centers with redundant power,
  networking, and connectivity.

- AZs are located physically separated locations. Thus, they are isolated from
  disasters since another AZ in the same region will be operating even if an AZ
  goes down.

- AZs are connected to each other via high bandwidth, ultra-low latency
  networking.

- On the management console, you will see your current region setting on the
  top right corner.
    - And when you switch between the AWS services, you will see the scope
      changing.
    - For example, AWS EC2 is a region-based service. However, IAM is a global
      service.

---

## IAM Introduction

- **Identity and Access Management** (IAM)

- It is where you can manage sercurity for *users, groups, and roles*.
- Note: **Root account should never be used**.
    - Root account is the very first AWS account you have created.
    - Remember the cardinal rule of security: always assign users with minimum
      required permissions.

- IAM is the control center of AWS. It manages the access control for services
  via security policies and roles.

- **Policy** is a permission control that is written in JSON format - we attach
  policies to components to enfore permissions.

- Within IAM, we have following components:
    - **Users** : usually a physical person such as a developer, a sys admin,
      or a HR employee.
    - **Groups** : grouping of users that shares the same access policies such
      as a DevOps, HR, Admins, and so on.
    - **Roles** : used for internal usage within AWS resources (machines).

- Note: changes made with IAM is applied **globally**.

- Multifactor-Authentication (MFA) can be set-up, and is strongly recommended.

- There is already a list of predefined *managed policies* so that you may not
  have to write the JSON from scratch and ground up. And there is also a policy
  generator.

## IAM Federation

- It is a way for enterprise companies to integrate their own existing
  repository of users with IAM.

- This allows users of these companies to log into AWS using their company
  credentials.

- Identity Federation uses the SAML standard (Active Directory).

## IAM 101 - UCS: Use Common Sense

- 1 User = 1 Physical Person. **Never share your account**.

- 1 Role = 1 Application. **App has its own lifecycle - need its own role**.

- Again, **Never share your account**.

- **Never hard code credentials in code**. Or leave it as a env variable inside
  running virtual machine for convenience.

- **Never use root credentials beyond initial set-up**.

---

## EC2 Introduction

- **Elastic Compute Cloud** (EC2)

- It is capable of:
    - renting virtual machines (EC2)
    - storing data on virtual drives (EBS - Elastic Block Storage)
    - distributing load across machines (ELB - Elastic Load Balance)
    - scaling the services using an auto-scaling group (ASG - Auto-Scaling
      Group)

- EC2 is region-based service.

- When creating your EC2 instance:
    1. Choose your AMI (Amazon Machine Image)
        - the software environment such as which Linux distro is it running on.
    2. Choose your Instance type
        - how powerful do you need your virtual machine to perform?
        - select different numbers of vCPUs, memory, and etc.
    3. Configure various instance details
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
    6. Configure *Security Group*
        - Security Group acts as a Firewall - controling traffics.
        - By default, SSH port 22 will be available.
        - We can choose to whitelist specific IPs.
        - If this EC2 instance is used as a web-server, then we would create
          a new security group to add more groups. In particular, we would
          enable port 80 and 443 for HTTP/HTTPS protocols.

- When launching EC2 instance, it will give you key-pair.
    - This credential is needed to SSH into the running EC2 instance.
    - Securely store the `.pem` credential file.

- Created EC2 instances can be controlled via SSH into their public IP.

    >$ ssh ec2-user@<EC2-IP4>


- But this requires the key-pair credentials that we were given previously.
  Supply that '.pem' key file to SSH.

    >$ ssh ec2-user@<EC2-IP4> -i KEYPAIR.pem

- Note: you will get an error that key file has 0644 permission.
- **This is your private key that should be unavailable to others**.

    > chmod 0400 KEYPAIR.pem

- Now, you should be able to SSH into the running EC2 instance.

- **EC2 Instance Connect** allows you to SSH to EC2 instance from your browser;
  this currently is only available with EC2 instances provisioned with *Amazon
  Linux 2 AMI*.

- Note: Windows 10 powershell provides ssh tool - no need for CYGWIN, putty, or
  etc...

---

## Security Groups Introduction

- Main method for network security in AWS;

- It allows control over inbound/outbound traffics in/from the EC2 instances.

- In short, they are **firewalls** on EC2.

- They are used to regulate:
    - Access to ports.
    - Authorised IP ranges - IPv4 or IPv6.
    - Control of Inbound Network (from other to the instance).
    - Control of Outbound Network (from instance to other).
    - Different types such as HTTP, SSH, TCP, ...

- For example, we may have a security group attached to running EC2 instance to
  allow only specific IP source at port 22; thus, it uses this allowlist to
  check incoming traffics.

- **Security Group is locked to a Region & VPC**.
    - If the service or app is outside of a region, we need to assign new
      security groups.

- EC2 is not an software running on EC2; it lies outside, and EC2 won't even
  realize that the traffics are getting controlled by it.

- It is good to maintain one separate security group for SSH access.

- **If your app is not accessible and _times out_, most likely a security group
  issue**.

- **If your app returns connection refused error, then it is an app
  error or the instance has not been launched**.

- By default, all inbound is blocked; all outbound is authorized.

- Security groups can reference each other.
    - i.e. An EC2 instance has a security group 1 that authorises access from
      another security group 2. Then, another EC2 instance belonging to
      security group 2 will be able to access the EC2 instance with security
      group 1.

---

## Private vs Public IP (IPv4) Review

- Networking involves two IPs: IPv4 and IPv6.
    - IPv4 is the addr. But due to its contraint on maximum value, we are
      runing out of addr to hand-out (although we have work arounds).
    - IPv6 is the new addressing protocol.

- Public IPs are what is exposed in the 'public'.
- i.e. two web servers can communicate to each other using their public IP.

- Private IPs are hidden from the public since they are used within the closed
  network behind the gateway.
- For instance, a company will run its own protected private network behind the
  router (or gateway) which has the public IP to manage traffics in and out.
 
## Private vs Public IP Differences

- Public IP
    - The machine is identifiable over the internet (www).
    - It should be unique across entire web.

- Private IP
    - It is used within the private subnet.
    - It only has to be unique within its own network.
    - **Gateway** (or proxy) is used to connect to the web.
    - A router is usually the gateway, and it handles the **port
      forwarding**. It does so by mapping its own IP:port to private IP. Thus,
      traffic from/to that private IP is masked over router's IP and port.
    - This is reason behind why a machine behind the gateway appears to send
      the request from the IP of the router (and not the private IP assigned to
      it by the router).

## Elastic IP

- **EC2 public IP can change when it restarts.**

- If you require fixed public IP for the EC2, you need **Elastic IP**.

- **Elastic IP** is a fixed public IPv4 that you can *own* so long as you don't delete.

- With the Elastic IP address, you can mask the failure of an instance or
  software by rapidly remapping the address to another instance in your
  account.

- By default, you can only have upto 5 Elastic IP in your account.
    - If you need more, ask AWS.

- **Best practice is to avoid using Elastic IP**.
    - They are too simple; and often reflect poor architectural decisions.
    - Instead, better approach would be to use a random public IP and register
      DNS hostname (Route53).
    - Or, Load Balancer can be used to avoid using a public IP.

## How does it work with EC2?

- By default, EC2 machine has:
    - A private IP for the internal AWS Network (VPC).
    - A public IP for the web.

- We SSH into the EC2 instance using the public IP since we are not on the same
  internal network as it is.

- Again, **when Ec2 machine stops and start again, public IP can change**.

- SSH into the running EC2 instance, and then check its IP.

    >$ ssh ec2-user@<EC2-PUBLIC-IP> -i KEYPAIR.pem
    
    >$ ifconfig -a

- Under the ethernet interface (eth0), note that its inet IPv4 is the
  private IP; not the same public IP used to SSH in.

- Probably there is a public web facing gateway that relays traffic between the
  EC2 instance and you. More on this when we talk about **VPCs**.

- You can allocate *Elastic IP* address in the EC2 console.
- Once allocated, Elastic IP needs to be associated to EC2 instance.

---

## Side Activity: Creating Apache Web Server on EC2

- First, ssh into the running EC2 instance. Elevate your privilege to update
  the software packages, and then install apache web server.

    >$ ssh ec2-user@<EC2-PUBLIC-IP> -i KEYPAIR.pem 
    
    >$ sudo su yum update -y yum install
    
    >$ httpd -y

- Then, we will start the Apache service and check that it is running; and
  serving the web page.

    >$ service httpd start 
    
    >$ service httpd status 
    
    >$ curl http://localhost

- However, when you visit the EC2 instance on your browser, it will not work.
  This is because of **security group** as we discussed earlier.

- **By default, inbound traffic on port 80 (HTTP) is not enabled**.

- We can attach the new security group. To do so, create a new security group;
  adding in Inbound rules for HTTP/HTTPS from anywhere. 

- Now, when you visit the EC2 instance through browser should serve the webpage
  correctly now.

---

## EC2 User Data

- It is possible to bootstrap our instances using an EC2 User data script;

- **Bootstrapping** means automatically executing set of commands when the
  machine starts up.

- EC2 user data is used to automate boot tasks such as:
    - installing updates.
    - installing necessary softwares.
    - downloading files.
    - ...

- Script will run with root privileges.

## EC2 User Data Script

- Create a new EC2 instance.

- While configuring the details, we can find the `User Data` section under advanced details

    ```sh 
    #!/bin/bash
    
    # upgrade to root.
    sudo su

    # update/install/start httpd service.  
    yum update -y && yum install -y httpd
    service httpd start 
    echo "Hello from $(hostname -f)" > /var/www/html/index.html
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
    - Scheduled Reserved Instances: i.e. run every Thursday between 1 - 5 pm.
- **Spot** : short workoads, cheap, but may lose the instances.
- **Dedicated Hardware** : no one will share the hardware.
- **Dedicated Hosts** : book an entire physical server; control instance
  placement.

## EC2 On-Demand

- Pay per usage (billing per second, after first min).
- Has the highest cost but no upfront payment.
- no long term commitment.

- **Recommended for short-term and un-interrrupted workloads**.
    - i.e. you cannot predict how the application will behave.

## EC2 Reserved Instances

- 75% discount vs On-Demand.
- Pay up-front for long term commitment - 1 or 3 years.
- Specify what type of instance to use during reservation.
- **Recommended for steady state usage apps such as databases**.

- **Convertible Reserved Instance**
    - allows you to change the instance type but costs more.
    - 54% discount vs 75% discount.

- **Scheduled Reserved Instances**
    - only launch within time window reserved.
    - for when you need to run only a fraction of day / week / month.

## EC2 Spot Instances

- Upto 90% discount vs On-Demand.
- **Instance can be _lost_ at any point in time if the price you have specified
  to pay is less than the current spot price**.
- It is most cost-efficient instances in AWS.
- **Useful for workloads that are resilient to failure**.
    - Batch jobs.
    - Data analysis.
    - Image processing.

- *Should not be used for critical jobs*.

- One great way to use it would be to use Reserved Instances as a base-line,
  and privison more On-Demand and Spot instances when the need arises.
    - i.e. running web server on EC2 instance. You may want to spin up extra
      web servers during a peak times such as weekends when traffic increases.

## EC2 Dedicated Hosts

- A dedicated, physical EC2 server.
- Full control of EC2 Instance Placement.
- **Visibility into the underlying socket / physical cores of the hardware**.
    - useful for licensing reasons.
- **per host billing**; 3 year period reservation.
- Useful for software that have complicated licensing model (BYOL), or for
  company that have a regulatory requirements to fulfill.

## EC2 Dedicated Instances

- Instances running on hardware that is dedicated only to you.
- May share hardware with other instances in same account.
- No control over instance placement.
- Know the difference vs Hosts.
- **per instance billing**.

## Choosing the right instance type

- Think in terms of renting a room at Hotel.

- On-Demand is coming in whenever you would like, paying the full price.
- Reserved is planning ahead, and staying for extended time, getting
  a discount.
- Spot is when hotel allows people to bid on the room for how much they would
  like to pay for; can get kicked out if price goes up.
- Dedicated Hosts is just renting out the entire building.

---

## EC2 ENI

- **Elastic Network Interface** (ENI)

- Logical component in a VPC that represents a **virtual network card**.

- Let's say within an AZ, an EC2 instance is running. To this EC2 instance's
  eth0, ENI will be attached to provide your EC2 instance with network connectivity.

- ENI has following attributes:
    - Primary private IPv4, one or more secondary IPv4.
        - i.e. a second ENI can attach itself to EC2 instance at eth1.
    - One elastic IP per one private IPv4.
    - One Public IPv4.
    - One or more security groups.
    - A MAC address.

- Can create ENI independently and attach them on the fly on EC2 instances for
  failover. i.e. We can move over one instance's ENI to another.

- **ENI is bound to the AZ**.

- Network interfaces is one of displayed information of running instances.
    - Network interface eth0 is the primary one.
    - You will see interface ID attached which begins with 'eni-...'.

- Under Network & Security, there is also a Network Interfaces tab; here, you
  can create a new ENI.
    - You can attach a new ENI to running instances.
    - When EC2 instance has two ENIs attached, the secondary ENI will be
      attached to network interface eth1.

---

# EC2 Summarized

## EC2 Pricing

- EC2 instance prices (per hour) varies based on:
    - region
    - instance type
    - On-Demand vs Spot vs Reserved vs Dedicated Host
    - running OS (Linux v Window v Private OS)

- Billed every second with a min 60 seconds.

- Pay for other factors such as storage, data transfer, fixed IP public
  addresses, load balancing.

- **You do not pay for stopped instances**.

## AMI

- EC2 instance comes with base images (Ubuntu, Fedora, RedHat, Windows ...).
- which can be customized further at boot via using EC2 User data.
- **But we may provide our own image, we use AMI**.
    - It is an image to use to create our instances.

- Custom AMIs are advantageous.
    - Pre-install packages,
    - faster boot time (no need to write a long user data script).
    - security concerns.
    - maintain your own AMIs.
    - Active Directory Integration out of the box.
    - install apps ahead of time for faster deploying.
    - using someone's AMI...

- **AMIs are bulit for specific AWS region**.

## EC2 Instances Overview

- Instances have 5 distincitve characteristics:
    - RAM (amount, type, generation)
    - CPU (cores, frequency)
    - I/O
    - Network
    - GPU

- https://aws.amazon.com/ec2/instance-types/

- R/C/P/G/H/X/I/F/Z/CR is used to denote specialization of their RAM, CPU, I/O,
    Network, and GPU capabilities.

- M instances are balanced.
- T2/T3 instances types are **burstable**.

- For exam, do not worry about memorizing these!

## Burstable Instances (T2)

- T2 machines are those that have a balanced performance; but when needed, it
    can **burst** and increase CPU performance.
- While bursting, it uses up the **burst credits**.
- When burst credit depletes, CPU performance goes down.
- When machine is in non-burst, credits accumulate over time.

- If your running T2 instance has constantly low credits, consider moving to
    diferent non-burstable instance (get one with increased CPU).

## T2 Unlimited

- Unlimited burst credit for cost.
- Can get expensive if left unmonitored!

---

# EC2 Checklist

- Know how to ssh into EC2 (and changing .pem file permissions).
- Know howto properly use security groups.
- Know the difference between private v public v elastic IP.
- Know how to use User Data to customize your machine at boot time.
- Know that you can supply own AMI.
- EC2 instances are billed by the second!

---

