AWS Route53
===========

- **Route53 is a managed Domain Name System (DNS)**.
- DNS is a collection of rules and records which helps clients to translate the
  domain name to IP.
- In AWS, common records defined are:
    - A : hostname to IPv4.
    - AAAA : hostname to IPv6.
    - CNAME: hostname to hostname.
    - Alias: hostname to AWS resource.

Route53 -- A Record Example
---------------------------

Suppose there is a web browser and an application server at IP 111.111.111.111.
The browser knows the app server's hostname, say `myapp.mydomain.com`, but it
does not know its IP. Hence, it will send out DNS request to Route53, asking
where the hostname is located, Route53 will in response send back the IP
address (A record : hostname to IP). Now, the browser is able to make HTTP
request to the IP and reach the application server.

Route53 -- Overview
-------------------

Route53 can use either:

- Public domain names you own such as `app1.my-public-domain.com`.
- Or. Private domain names that can be resolved by your instances in your VPCs
  such as `app1.company.internal`.


Route53 features:

- Load balancing through DNS (also known as client load balancing).
- Health checks.
- Routing Policy: simple, failover, geolocation, latency, weighted,
  multi-value.
- Pay $0.50 per month per hosted zone.

Route53 -- Creating Domain Name
-------------------------------

- You can register a new domain (`.com`, `.net`, `.io`, ...).
- Then, you may add records under Host Zones.
    - i.e. add type A record to `myapp.mydomain.io : 123.123.123.123`.
- The registered domain and its record can be checked with network utils such
  as `host` or `dig`.

Route53 -- DNS Records TTL
--------------------------

- Web browser makes a DNS request to Route53, and receives a record back with
  TTL of 300 seconds. Then, web browser will cache the DNS request and response
  for the TTL specified.
- This is due to the chance that the record has been updated.

Route53 -- CNAME vs Alias
-------------------------

- AWS Resources (ELB, CloudFront...) exposes an AWS hostname such as
  `lb1-1234.us-west-1.elb.amazonaws.com` want to use our domain name
  instead.

- CNAME:
    - Points a hostname to any other hostname (`app.mydomain.com` ->
      `abc.xyz.com`).
    - It only works for non-root domain (i.e. `abc.mydomain.com`).

- Alias:
    - Similar to CNAME but for AWS resources (`abc.mydomain.com` ->
      `xyz.amazonaws.com`).
    - It works for root and subdomain as well (i.e. `mydomain.com`).
    - Provides native helath check.

Routing Policy -- Simple Routing Policy
---------------------------------------

- Web browser does DNS request and Route53 gives back a record.
- Used to redirect to a single resource.
- Cannot attach health checks.
- **If multiple values are returned, a random one is chosen by the client**.
- i.e. create A record to `abc.mydomain.com` to a running EC2 instance's IP.
- You may enter multiple values to the record set. For example, adding another
  EC2 instance IP. The client will choose a random.

Routing Policy -- Weighted Routing Policy
-----------------------------------------

- Control the % of the requests that go to specific endpoint.
- We will have several IPs and assign them weights. Now, Route53 will return
  a record that is appropriate to these weights. i.e. 70% of the DNS requests
  will get first IP, and remaining 30% receive other.
- Use case would be to conduct a testing by redirecting % of users to newer
  version of the application, or balance the traffic across the regions.

Routing Policy -- Latency
-------------------------

- Redirect to the server that has the least latency and close to us.
- Helpful when latency of users are critical.
- Latency is determined in terms of user to designated AWS Region.

Routing Policy -- Failover
--------------------------

- Suppose we have a primary instance and secondary instance for diaster
  recovery. Route53 performs health checks to primary; if it fails, then it is
  changed to secondary automatically.

Routing Policy -- Geolocation
-----------------------------

- Note that this is different from **latency**; users recieve response based on
  their location.

Routing Policy -- Multivalue
----------------------------

- Used when routing traffic to multiple resources.
- This is not a substitue for an actual load balancer.

Route53 -- Health Checks
------------------------

- Tries to contact for X number of times; if failed, it is unhealthy (default
  X = 3).
- Passed X number of contacts? it is healthy (default X = 3).
- Default health check is done every 30 seconds (can set to 10 seconds for
  higher cost).
- Can setup HTTP, TCP and HTTPS health checks (without SSL).
- Can integrate with AWS CloudWatch.


