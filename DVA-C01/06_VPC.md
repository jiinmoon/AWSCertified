# VPC

- Should study much more indepth for Solutions Architect & SysOps Admins.
- At Developer level, you should know following:
    - VPC, Subnets, Internet Gateways & NAT Gateways.
    - Security Groups, Network ACL (NACL), VPC Flow Logs.
    - VPC Peering, VPC Endpoints.
    - Site to Site VPN & Direct Connect.

---

## VPC & Subnets

- **VPC**: Private Network to deploy your resources (regional resource).
- **Subnet**: allows you to partition your network inside your VPC.
    - Availability Zone resource.

- i.e. On a Availability Zone 'X', we may have two subnets, private and public.
    - **Public subnet** is accessible from the internet.
    - **Private subnet** is not accessible from the internet.

- To define access to the internet and between subnets, we use **Route Tables**.

- Imagine our resources within the AWS Cloud.
    - Within the Cloud, we have a Region, and inside the region, we have a VPC.
        The VPC has a CIDR Range (Range of IP addrs under manage).
    - Now, within region there are several Availability Zones.
    - Within AZ 1, we would have a Public/Private Subnets. Same is true for all
        other AZs.

## Internet Gateway & NAT Gateways

- Suppose we have a EC2 instance within the Public Subnet - inside the AZ. For
    this instance to access the web (www), **Internet Gateways** is what helps
    our VPC instances connect to the internet.

- Public Subnets have a route to the internet gateway.
    - Thus, EC2 instance running within the Public Subnet will be connected to
        the Internet Gateway.

- Suppose we have an instance runnning in Private Subnet; and we want to access
    the web but does not wish for outside traffic to reach it.
- This is **NAT Gateways** (AWS-managed) and **NAT Instances** (self-managed)
    allow your instances in your **Private Subnets** to access the internet
    while remaining private.
    - It works by deploying NAT Gateways in the public subnet. Then, route the
        instances traffic from private subnet to it. NAT will then have a route
        to the Internet Gateways to the web.

## Network ACL & Security Groups

- Suppose a public subnet within the VPC; it is protected by following:

- NACL (Network ACL)
    - A firewall which controll traffic from and to subnet.
    - Have ALLOW and DENY rules.
    - Are attached at the Subnet level.
    - Rules only include IP addresses.
    - Think of first line of defense for subnet.

- Security Groups
    - A firewall that controlls traffic to and from an ENI / an EC2 instance.
    - Can have only ALLOW rules.
    - Rules include IP addrs as well as other security groups.

## Network ACL vs Security Groups

- They are quite similar but here are few differences:

- SG operates at the instance level; NACL at subnet level.
- SG only ALLOWs; NACL supports both ALLOW and DENY rules.

- SG is stateful - return traffic is allowed.
- NACL is stateless - return traffic must be also allowed by rules.

- SG evaluates all rules before allowing traffic in.
- NACL process rules in number order when deciding traffic to allow in.

- SG applies to an instance only if someone specifies the SG when launching the
    instance, or associates the SG with the intance later.
- NACL automatically applies to all instances in the subnet; thus, users to not
    have to specify the SG.

## VPC Flow Logs

- Capture information about all IP traffic going into your interfaces:
    - VPC Flow Logs
    - Subnet Flow Logs
    - Elastic Network Interface Flow Logs

- Helps to monitor and troubleshoot connectivity issues. Examples:
    - Subnets to internet.
    - Subnets to subnets.
    - Internet to subnets.

- Captures network information from AWS managed interfaces as well: ELB,
    ElastiCache, RDS, ...

- VPC Flow logs data can go to S3 and CloudWAtch Logs.

## VPC Peering

- Connect two VPC, privately using AWS' network.
- Make them behave as if they were in the same netowork.
- Should not have overlapping CIDR (IP addresses ranges)!

- VPC Peering connection is **not transitive**; must be establied for each VPC
    that need to communicate with one another. Suppose subnet A and B is peer
    connected and, A and C is peer connected. This does not mean that B and C
    can talk to each other.
    - Thus, you would need to establish VPC peering on B and C separately.

## VPC Endpoints

- Endpoints allow you to connect to AWS Services **using a private network**
    instead of the public WWW network.
- Way for AWS Services to securely communicate with each other.
- Also improves the latency as well.
- Suppose we have a EC2 instance within the private subnet that wishes to access
    S3 or DynamoDB that is outside of its VPC. To do so, we would create VPC
    Endpoint Gateway which creates a secure connection point for theses services
    to connect to one another without having to go out in public.

- VPC Endpoint Gateways : S3 and DynamoDB.
- VPC Endpoint Interface : Rest of AWS Services.

- **Only used within your VPC.**

## Site-to-Stie VPC & Direct Connect

- Site-to-Site VPN (Virtual Private Network)
    - Connect an on-premises VPN to AWS.
    - The connection is automatically encrypted.
    - The traffic moves over the public internet.
    - This grants encrypted connection to VPC.

- Direct Connect (DX)
    - Establish a **physical** connection between on-premises and AWS.
    - The connection is private, secure and fast.
    - Goes over a private network.
    - Needs few months to establish - construction.

- These cannot access VPC endpoints;

---

## VPC Summary

- VPC: Virtual Pirvate Cloud

- **Subnets** are tied to an AZ; network partition of the VPC.
- **Internet Gateways** at the VPC level; provides internet access.
- **NAT Gateways / Instances** give internet access to subnets.
- **NACL** : stateless; subnet rules for inbound/outbound.
- **Security Groups** : stateful; operate at the EC2 level or ENI.
- **VPC Peering** : non-transitive connecting between two VPCs (no overlapping).
- **VPC Endpoints** : private access to AWS Services within VPC.
- **VPC Flow Logs** : network taffic logs.
- **Site to Site VPN** : VPN over public internet between on-premises DC & AWS.
- **Direct Connect** : direct private connection to AWS.
