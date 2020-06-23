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
