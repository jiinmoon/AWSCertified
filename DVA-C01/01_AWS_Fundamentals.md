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
