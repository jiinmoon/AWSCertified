Virtual Private Cloud -- VPC
============================

Note that this section should be stuided much more for Solutions and SysOPs
Admins. At the Dev level, you should know the following:

- VPC, Subnets, Internet Gateways and NAT Gateways.
- Security Groups, Network ACL (NACL) and VPC Flow Logs.
- VPC Peering and VPC Endpoints.
- Site to SIte VPN and DIrect Connect.

VPC and Subnets
---------------

**VPC** is a private network where AWS resources are deployed (regional
resources). And **Subnets** allow you to partition your network inside your VPC
such as AZ resources.

i.e. within AZ "X", there may be two subnets: private and public. **Public
subnet** is accessible from the internet whereas **Private subnets** is not.

To define access to the internet and between subnets, we define **Route
Tables**.

Within the AWS cloud, we have a region; within the region, we have a VPC that
has a CIDR range (which is a range of IP addresses under management).

Within a region there are several AZs; within an AZ, there would be a Public
and Private Subnets.

Internet Gateway and NAT Gateways
---------------------------------

Suppose we have a EC2 insance within the Public Subnet inside an AZ. For this
instance to access the www, **Internet Gateways** is required. **Public Subnets
have a route to the Internet Gateways**.

This is not the case of EC2 instances running within the Private Subnet. In
order to access the www, **NAT Gateways** and **NAT INstances** are required.
It works by deploying NAT Gateways in the public subnet first; then, route the
instances traffic from the private subnet to it.

Network ACL and Security Groups
-------------------------------

Suppose a public subnet within the VPC; it is protected by following:

- Network ACL (NACL)

    - A firewall that controls in/outbound traffics from/to the subnet.
    - Have ALLOW and DENY rules.
    - It is attached at the subnet level.
    - Rules only includes IP addresses.

- Security Groups

    - A firewall that controls traffic from/to an ENI and an EC2 instance.
    - Can have only ALLOW rules.
    - Rules include IP addreses and other security groups.

Network ACL vs Security Groups
------------------------------

They are very similar, but here are notable differences:

- SG operates at the instance level; NACL at Subnet level.
- SG only ALLOWs; NACL ALLOWs and DENYs.

- SG is stateful; return traffic is allowed.
- NACL is stateless; return traffic must be also allowed by rules.

- SG evaluates all rules before allowing traffic in.
- NACL process rules in number order when deciding traffic to let in.

- SG applies to an instance only if someone specifies the SG when launching the
  instance, or associates the SG with the instance.
- NACL automatically applies to all instances since it operates at Subnet
  level.

VPC -- Flow Logs
----------------

It captures information about all IP traffics going into interfaces:

- VPC Flow Logs
- Subnet Flow Logs
- ENI Flow Logs

It helps to monitor and troubleshoot connectivity issues such as:

- Subnets to internet
- Subnets to subnets
- Internet to subnets

It captures network information from AWS managed interfaces such as ELB,
ElastiCache, RDS and etc.

VPC Flow Logs data can go to S3 and AWS CloudWatch Logs.

VPC -- Peering
--------------

- Connects two VPCs privately using AWS network.
- It allows to behave two VPCs as if they were in the same network.
- Two VPCs should not have overlapping CIDR (IP addresses ranges).
- This is not a **transitive connection**, meaning A -> B -> C does not man
  A -> C.

VPC -- Endpoints
----------------

- Endpoints allow youto connect to AWS resources **using a private network**
  instead of the exposed public www network.

- Since AWS network is used, it is secure and low in latency.
- Suppose we have an EC2 instance within the private subnet thatwishes to
  access a S3 which is outside of its VPC. To do so, we would create **VPC
  Endpoint Gateway** which creates a secure connection point for these AWS
  resources to connect to one another.

- **VPC Endpoint Gateways** are used by S3 and DynamoDB.
- **VPC Endpoint Interface** are used by rest of AWS services.
- **This is meant to be used only within VPC**.

VPC -- Site-to-Site and Direct Connect
--------------------------------------

- Site-to-Stie Virtual Private Network (VPN)
    - Connect an on-premises VPN to AWS.
    - The connection is automatically encrypted.
    - The traffic moves over the public internet.
    - This grants encrypted connection to VPC.

- Direct Connect (DX)
    - Establish a **physical connection** between on-premises and AWS.
    - The connection is private, secure and fast.
    - Needs few months to establish since...construction.

- These cannot access VPC endpoints.

VPC -- Summary
--------------

- **Subnets** are tied to an AZ; network partition of the VPC.
- **Internet Gateway** at VPC level; provides www access.
- **NAT Gateways / Interfaces** gives www access to subnets.
- **NACL** is stateless firewall; subnet rules for in/outbound.
- **Security Groups** is stateful firewall; operates at instance level or ENI.
- **VPC Peering** is non-transitive connection between VPCs.
- **VPC Endpoint** is private access point to AWS resources within VPC.
- **VPC Flow Logs** logs the network traffics.
- **Site-to-Site VPN** provides secure VPN over public internet between
  on-premise and AWS.
- **Direct Connect** provides direct, physical private connection to AWS.

---

A Typcial 3 Tier Architecture
-----------------------------

- User wants to access the web application.
- We would first setup an ELB on multi-AZ; since it needs to be accessed by the
  customers, it is placed on the public subnet.
- To access the ELB, users will perform DNS query to Route53, then send their
  traffic to ELB which it will distribute to our instances on backend.

- The EC2 instances will be managed under ASG, and it will be placed under
  private subnet separated from the public subnet where the ELB is placed.
- The EC2 instances will be spreaded across multi-AZs and receive the traffic
  from ELB via routing table.

- The database will reside within _data subnet_; setup RDS as a main database,
  and use ElastiCache to store the session data.

