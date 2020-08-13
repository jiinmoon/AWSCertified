# 5 Route53

- It is a managed DNS.
- DNS is a collection of rules and records which helps clients understand how
  to reach a server through a domain name.
- Common record types:
    - A: hostname to IPv4
    - AAAA: hostname to IPv6
    - CNAME: hostname to hostname
    - Alias: hostname to AWS resource

- It can use
    - Public domain names you own or buy.
    - Private domain names that can be resolved by your instances in your VPCs.

- It can perform
    - Load balancing through DNS (client load balancing)
    - Limited health checks
    - Routing policy

- Pays $0.50 per month per hosted zone.

## Simple Example

- A browser wishes to reach a hostname "xyz.example.com", but it does not know
  its IP (123.123.123.123).
- So, it will first make a DNS request to Route 53; and Route 53 will return
  the matching record, "123.123.123.123", back to the browser.
- Browser is now able to reach the application with the returned IP.

## TTL

- It is a way to control the lifetime of DNS record within the browser.
- It is used to reduce the number of requests to the DNS server.
- The result of DNS request from Route 53 will be cached in the browser for the
  set amount of TTL.
- Because of TTL, browser may still visit the old application when new
  application has been put up and DNS records have been updated until the TTL
  expires and sends another request to discover the changed IP.

## CNAME vs Alias

- AWS Resources such as Load Balancer, and CloudFront exposes an AWS hostname
  such as "lb1-1111.us-west-1.elb.amazonaws.com".
- But you want simple and attach it to something like "myapp.mydomain.com".

- CNAME:
    - Points a hostname to any other hostname
    - **only for non-root domain** (i.e. "myapp.mydomain.com")

- Alias:
    - Points a hostname to an AWS Resource
    - **works for root domain and non-root domain** (i.e. "mydomain.com")

## Routing Policy
### Simple Routing Policy

- Use when you need to redirect to a single resource.
- i.e. request "abc.example.com"; response "1.2.3.4".
- Cannot attach health checks.

### Weighted Routing Policy

- Controls the % of the requests that go to specific endpoint.
- If there are 3 records each with 50, 30, and 20 weights, then Route53 will
  response back the request to this record appropriately to the weights of
  these record.
- So, 50% of the response is to A, 30% of the response is to B, and 20% to the
  C.

### Latency Routing Policy

- Redirect to the server that has the least lantecy close to the requested
  user.
- Latency is evaluated in terms of user to designated AWS Region.

### Geolocation Routing Policy

- This is based on the user location.
- Used for copy-right policies and etc.
- Location is determined with 3rd party app.

### Multi-Value Routing Policy

- Routing to multiple resources.
- This is not a ELB.
- It will perform health checks to the endpoints and route appropriately.

## Health Checks

- Route53 performs regular health checks to its end points; if it is failed to
  response back 3 times (default), it is marked unhealthy.
- If 3 checks are passed, then the endpoint is marked healthy again.
- By default, health checks are performed every 30s (can set at 10s but $$$).
- There will be around 15 health checkers that sends one request every
  2 seconds on average.

- Can perform HTTP, HTTPs, and TCP health checks (but no SSL verification).
- Possible to integrate with CloudWatch.

## Failover Routing Policy

- There will be primary and secondary endpoints which is used for disaster
  recovery.
- When health checks on primary fails, then routing is shifted over to the
  secondary.


