# Virtual Private Cloud

**Required much more in-depth for Architect Associate or SysOps Admins**.

For Dev Associate, should know about:

- VPC, Subnets, Internet Gateways and NAT Gateways
- Security Groups, Network ACL, VPC Flow Logs
- VPC Peering, VPC Endpoints
- Site to Site VPN, Direct Connect

## VPC and Subnets

- VPC is a private network to deploy your AWS resources (**Regional**).
- Subnets partition the network within the VPC (**AZs**).

- Public subnet has an access from/to internet.
- Private subnet is not exposed to internet.

- To define access to the internet and between subnets, we use **Route
  Tables**.

## VPC High-level View

- Within a Region, we can have a VPC set up across multiple-AZs with the range
  of CIDR (i.e. 123.0.0.0/16).

- Within an AZ, there can be public subnet and private subnets.

## Internate Gateway and NAT Gateway

- **Internet Gateway** allows the instance within the VPC to connect to
  internet.
- Public Subnets have a route to the internet gateway.

- **NAT Gateway** (AWS managed) and **NAT Instance** (self-managed) allow your
  instances in your **Private Subnets** to access the internet while remaining
  private.

- This is done by first setting up a NAT Gateway in the Public Subnet that has
  route to the Internet Gateway. Then, from private subnet, it will connect
  to the NAT Gateway.

## Network ACL and Security Groups

- Network ACL
    - is a firewall that controls traffic from and to subnet.
    - has ALLOW and DENY rules.
    - attached at Subnet level.
    - rules may only deal with IP addresses.

- Security Groups
    - is a firewall that controls traffic to and from an ENI and an EC2
      instance.
    - has only ALLOW rules.
    - Rules include IP addresses as well as other security groups.

## VPC Flow Logs

- It captures information about IP traffic going into the interfaces:
    - VPC Flow Logs
    - Subnet Flow LOgs
    - ENI Flow Logs

- Helps to monitor and troubleshoot connectivity issues.
    - Subnets to internet
    - Subnets to subnets
    - Internet to subnets

- This information includes other AWS resources as well.
- Logs can be stored to S3 or CloudWatch Logs.

## VPC Peering

- Connecting two VPC privately which are in different account using AWS
  network.

- This will make them behave as a single network.

- The networks should not have an overlapping IP address ranges (CIDR).

- It is not transitive. A -> B and B -> C does not mean that A -> C.

## VPC Endpoints

- Endpoints allow you to connect to AWS Services using a private network
  instead of the public network.

- VPC Endpoint Gateway : S3 and DynamoDB
- VPC Endpoint Interface : else

## Sit to Site VPN and Direct Connect

- A way to connect on-premise data centre with AWS.

- Site to Site VPN
    - Connect an on-premises VPN to AWS.
    - Connection is auto-encryoted.
    - Goes over the public internet.

- DIrect Connect
    - Establishes a PHYSICAL connection between on-premises and AWS.
    - Connecton will be private, secure and fast.

## VPC Summary

- VPC is a virtual private cloud in a region.
- Subnets are tied to an AZ (a network partition of VPC).

- Internet Gateway works at VPC level that gives it a public internet access.
- NAT Gateway/Instances gives private subnets an access to internet.
- NACL controls the subnet rules for inbound/outbound traffics; stateless (IP
  only).
- Security Groups operate at EC2 or ENI level; stateful.

- VPC Endpoints : provide private access to AWS Services within VPC.

- VPC FLow Logs : logs of all traffics in/out of VPC.
- SIt to Site VPN : VPN over public network.
- Direct Connect : physical connection made betwen on-premises and AWS.


