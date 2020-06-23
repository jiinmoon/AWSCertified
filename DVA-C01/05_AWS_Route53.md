#  AWS Route53

- Route53 is a managed DNS (Domain Name System).
- DNS is a collection of rules and records which helps clients understand how to
    reach a server through its domain name.

- In AWS, common records:
    - A: hostname to IPv4
    - AAAA: hostname to IPv6
    - CNAME: hostname to hostname
    - Alias: hostname to AWS resource

## Route 53 -- A Record Example

- Suppose there is a web browser and an application server at IP
    123.123.123.123.

- Web brower knows the app server's hostname, say myapp.mydomain.com. But it
    does not know its IP.
- Hence, it will send out DNS Request to Route 53, asking where the hostname is
    located. Route 53 will in response send back the IP address (A record:
    hostname to IP).
- Now, the browser makes the HTTP request to the recieved IP; accessing the
    Application Server.

## Route 53 -- Overview

- Route53 can use:
    - Public domain names you own (or buy).
        - app1.mypublicdomain.com
    - Private domain names that can be resolved by your instances in your VPCs.
        - app1.company.internal

- Route53 has advanced features such as:
    - Load balacning (through DNS - also called client load balancing).
    - Health checks (although limited...).
    - Routing policy: simple, failover, geolocation, latency, weighted,
        multi-value.

- Pay $0.50 per month per hosted zone.

## Creating Domain Name

- You can register a domain (.com, .net, .org, ...).
- Then, you may add records under Hosted Zones;
    - i.e. add type A record to myapp.mydomain.com : 11.22.33.44.

- The regsitered domain and its record can be checked by running `host` or
    `dig`.

## Route53 - EC2 Set-up

- We first create few EC2 instances at various regions.
    - Let's have Apache web server running, serving a html.
    - Make sure that security group is set to allow inbound port 80.

- Make note of all the public IPs of the instances running at different regions.

- We will create a Load Balancer;
    - and link one of the EC2 instance.

## DNS Records TTL

- Web browser makes a DNS request to Route53, and receives A record back (URL to
    IP) with TTL 300 s.
- Then, web browser will cache the DNS request/response for the TTL specified.
- There is a chance that record has been updated - i.e. web server went down and
    needed to be replaced.

- So, when record changes, we have to know that client's TTL has to expire
    before they can see the changes.

- Also, need to think about load on the DNS itslef as well.
    - High TTL ~24 hours; less traffic on DNS.
    - Low TTL ~60 seconds; more traffic on DNS.

## CNAME vs Alias

- AWS Resources (Load Balancer, CloudFront...) exposes an AWS hostname. i.e.
    lb1-1234.us-west-1.elb.amazonaws.com. However, we want to use our domain
    name instead.

- CNAME:
    - Points a hostname to any other hostname (app.mydomain.com -> abc.xyz.com).
    - It only works for non-root domain (i.e. abc.mydomain.com).

- Alias:
    - Similar to CNAME but for AWS Resources (abc.mydomain.com ->
        xyz.amazonaws.com).
    - It works for root and subdomain as well! (i.e. mydomain.com).
    - Free of Charge.
    - Native Health Check.

## Routing Policy

### Simple Routing Policy

- Web browser does DNS request and Route 53 gives back a record.
- Use when you need to redirect to a single resource.
- Cannot attach health checks.

- **If multiple values are returned. a random one is chosen by client.**

- i.e. create A record to abc.mydomain.com to a EC2 instance ip.
- You may enter multiple Values to the record set; adding another EC2 instance
    ip. Client will choose at random; you can confirm this with `dig`.

### Weighted Routing Policy

- Control the % of the requests that go to specific endpoint.

- We will have several IPs and assign them weights (i.e. 70, 20, 10). Now,
    Route53 will retuurn a record that is appropriate to these weights. i.e. 70%
    of the requests will get A IP, and so on.

- An example of the use case maybe to conduct the testing - direct few % of the
    users to newer version of app and record their experiences.

- Or split traffic between two regions to balance the load.

- Client will be unaware whether weighted routing policy has been used - `dig`
    will return only a single record.

### Latency Routing Policy

- Redirect to the server that has the least latency close to us.
- Helpful when latency ofusers is a priority.
- Latency is evaluated in terms of user to designated AWS Region.
- i.e. Germany may be directed to the US if it is the lowest latency.
- i.e. We have two EC2 instances at Asia and NA. The users can be directed to
    whichever one that is closest to them.

## Route53 and Health Checks

- Have X health checks failed? Unhealthy (default x = 3).
- Passed X health checks? Healthy (default x = 3).
- Default health check interval: 30s (can set to 10s for higher cost).
- About 15 health checkers will check the endpoint health.
    - 15 checkers doing check every 30s == one request per 2 s on average.
- Can set up HTTP, TCP and HTTPS health checks (w/o SSL Verification).
- Can integrate CloudWatch for health checks.
- Health checks can be linked to Route53 DNS queries.

## Failover Routing Policy

- Suppose we have a primary instance and secondary instance (for disaster
    recovery). Route 53 performs health checks to primary; if it fails, then it
    is changed to failover - secondary.

- failover-primary; failover-secondary.
- need to create a health check prior; and attach to the failover.

## Geolocation Routing Policy

- Different from **Lantency**; this is based on actual user location.
- We can specify, traffic from certain region should go to spcific IP.
- Default policy is needed in case there is no match on location.

## MultiValue Routing Policy

- Use when routing traffic to multiple resources.
- Want to associate a Route53 health checks with records.
- Up to 8 healthy records are returned for each Multi-Value query.
- Multi-Value is not a substitue for having an ELB!

- Each value is associated with custom health checks.


