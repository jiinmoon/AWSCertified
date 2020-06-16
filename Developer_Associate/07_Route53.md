# Route53

- It is a highly available and scalable cloud Domain Name System (DNS).
- Register and manage domains, create DNS routing rules (i.e. failovers).

- Route53 is like Godaddy or NameCheap - DNS providers; but has 
    built-in synergies with AWS services.

- It can:

    - register and manage domains.
    - create various records sets on a domain.
    - implement complex traffic flows (blue/green deploy, failovers).
    - continuously monitor records via health checks.
    - resolve VPC's outside of AWS.

## Use Case

- Use Route53 to get your custom domains to point to your AWS Resources.

1. Incoming Internect Traffic goes to Route53.

2. Route traffic to our web-app backed by ALB (Load Balancer).

3. Route traffic to an instance we use to tweak our AMI (EC2 instance).

4. Route traffic to API gateway which powers our API.

5. Route traffic to CloudFront which serves our S3 static hosted website.

6. Route traffic to an Elasitc IP (EIP) which is a static IP that hosts our
   company Minecraft server.

## Record Sets

- Previously, we saw many subdomains which were directed to different AWS
    Resources.

- This is done by creating record sets which allows us to point our naked domain
    (i.e. mydomain.co) and subdomains via Domain records.

- For example, we can send our 'www' subdomain 'www.mydomain.co' using an A
    record to point a specific IP address.

- AWS has their own special Alias Record which extends DNS functionality; it
    will route traffic to specific AWS resources.

    - so, when Alias is selected, we can direct select AWS resources such as
        CloudFront, EB, ELB, S3, Resource reocrd set, VPC endpoint, API Gateway.

    - Alias records are smart where they can detect the change of an IP addr and
        continuously keep that endpoint pointed to the correct resource.

- In general, using Alias to route traffic to AWS resources is recommended.

## Routing Policies


