# AWS ELB and ASG

---

## Availability and Scalablity

- Scalability means that an app / system can adapt to changing workloads.
- Two Types:
    - Vertical.
    - Horizontal.

- **Scalaility != High Availability**; they are linked, but different.

## Vertical Scalability

- increasing the size of the instance.

- i.e. The call centre example. We have a junior operator and a senior operator.
    A junior can take 5 calls per hour while a senior can take 10. This change
    from replacing junior to senior is vertical scaling.

- In AWS example, vertically scaling an app running in t2.micro is to run the
    app within t2.large instance.

- It is common for non-distributed systems such as databases.

- RDS, ElastiCache are services that can scale vertically.

## Horizontal Scalability

- increasing the number of instances / systems for your app.

- i.e. When calls are overloaded, we higher more operators to increase the
    capacity.

- Horizontal scaling implies distributed systems.

- common for web apps/modern apps.

## High Availability

- It usually is linked with horizontal scaling.

- It means running your system in at least 2 data centers (AZs).

- Goal is to survive the data centre loss.

- Can be passive (enabled by default). i.e. RDS Multi-AZ.

- Can be active (by horizontal scaling).

## In terms of EC2,

- Vertical Scaling: increase the size of instance.
    - from t2.nano - 0.5G of RAM, 1 vCPU
    - to u-l2tvl.metal - 12.3TB of RAM, 448 vCPUs

- Horizontal Scaling: increase the # of instances.
    - ASG, Load Balancer...

- High Availability: run instances for same app across multi AZs.
    - Multi ASG or Multi Load Balancer.

---

# Elastic Load Balancers (ELB)

## What is Load Balacing?

- Load balancers are serverse that forward internet traffic to multiple servers
    (say EC2 instances) downstream.

## Why use it?
    - Spread the load across multiple downstream instances.
    - Expose a single point of access (DNS) to your application.
        - No one has to know all the public IP of EC2 instances to access your
            web app for example.
    - Seamlessly handle failures of downstream instances.
    - Do regular health checks on your instances.
    - Provide SLL termination (HTTPS) for your websites.
    - Enforce stickiness with cookies.
    - High availability across zones.
    - Allows for clean separation between public/private traffic since all
        inbound public traffic comes from ELB.

- ELB is a **manged load balaner**.
    - AWS gurantees that it will be working.
    - AWS taks care of upgrades, maintenance, high avilability.
    - AWS provides only a few configuration knobs.

- It will cost less to set-up your own load balancer; but it will be a lot of
    effort to maintain.

- It will intergrate with many AWS services easily.

## Health Checks

- A crucial information for load balancers.
- This is what LB uses to decide to which instance that it can forward a
    traffic.
- Health Check is done on a port and a route (/health).
- If the response is anything but 200, instance is unhealthy.
- i.e. ELB will perform health check via sending traffic to route /health.

## Type of AWS Load Balancers

- AWS provides three managed ELBs.

- Classic Load Balancer (very frist one; older generation; 2009).
    - HTTP, HTTPS, TCP

- Appplication Load Balancer (version 2; newer generation 2016).
    - HTTP, HTTPS, WebSocket

- Network Load Balancer (version 2; 2017).
    - TCP, TLS, UDP

- Recommened to use newer generation.

- You can set up **internal** (private) or **external** (public) ELBs.

## ELB Security Groups

- We may attach security groups (remember - Firewall) to ELB to how it should
    handle inbound/outbound traffics.

- For example, ELB may want to be able to receive HTTP/HTTPS traffic from
    anywhere. To do this, we would add two rules enabling TCP port 80/443 from
    source anywhere to the security group; then attach the security group to the
    Load Balancer.

- Also, you may restrict the EC2 instance to recieve HTTP traffic from ELB only.
    This is also done with security group, where we will change the source to
    ELB.

## ELB+

- ELBs can scale but not instantaneously - need to contact AWS for a warm-up.
- Error Codes:
    - 4xx client induced errors.
    - 5xx app induced errors.
    - ELB 503 error means at capacity or no registered target.
    - If ELB can not connect to your app, check security group settings first.

- Monitoring:
    - ELB access logs will log all access requests (allows debugging per
        request).
    - CloudWatch Metrics will give you aggregate statistics (connections count).

---

## Classic Load Balancer

- Supports TCP (Layer 4), HTTP, & HTTPS (Layer 7).
- Health Checks are either TCP or HTTP based.
- Fixed hostname: 'xxx.region.elb.amazonaws.com'.

## Application Load Balancer

- Only Layer 7 based (HTTP).
- Load balancing to multiple HTTP applications across machines (target groups).
- Load balancing to multiple applications on the same machine (ex: containers).

- Support HTTP/2 and WebScoket.
- Support redirects (from HTTP to HTTPS).

- Routing tables to different target groups:
    - Routing based on path in URL.
        - For example, example/users, and example/posts are two different paths.
            Hence, ALB can be set up to route them to different instances.
    - Routing based on hostname in URL.
        - i.e. xyz.example.com and abc.example.com.
    - Routing based on query string, or headers
        - i.e. example.com/users?id=123&order=false.

- Thus, ALB is perfect for micro services and container-based application such
    as Docker or Amazon ECS.

- Has a port mapping feature to redirect to a dynamic port in ECS.
- In comparison, we would require multiple Classic Load Balancer per app.

## ALB Example: HTTP Based Traffic

- We would have a external LAB set up that would receieve inbound traffic from
    users at various routes such as /user or /search.
- Then, behind the ALB, we would have 'target groups' which has a group of
    instances which are dedicated to serve the specific routes.

## Target Groups

- EC2 instances (can be managed by an Auto-Scaling Group) - HTTP.
- ECS tasks (managed by ECS itself) - HTTP.
- Lambda functions - HTTP request is translated into a JSON event.
- IP addresses - must be private.

- ALB can route to multiple target groups.
- Health checks are at the target group level.

## ALB+

- Has a fixed hostname (xxx.region.elb.amazonaws.com).
- App servers do not see the IP of the client directly.
    - **True** IP of the client is inserted in the header X-Forwarded-For.
        - Makes sense, instance where app is hosted is behind the load balancer;
            load balancer gets the traffic first, then resends it to the app.
        - Hence, its request source would not be same.
        - To resolve this, extra header is used to attach original source IP.
    - Likewise, you will get Port (X-Forwarded-Port) and proto
        (X-Forwarded-Proto).

- Very easy to set up various rules on how to handle the incoming traffic.
- i.e. HTTP 80 Rules can be generated such as:
    - **IF** path is /test, **THEN Forward to** specific target group.

## Network Load Balancer

- Operates at layer 4: TCP/UDP.
- Handles millions of requests per seconds.
- Less latency ~100 ms compared to ALB ~400 ms.
- NLB has one static IP per AZ, and supports assigning Elastic IP.
    - CLB and ALB has static hostname!

- Used for extreme performance; TCP, UDP traffics.
- There isn't a free tier.

## Load Balancer Stickiness

- You can implement stickness: it allows the same client to be directed to the
    same instance behind the load balancer.
- This is possible for CLB and ALB since it works at Layer 7.
- The cookie used for stickiness has an expiration date you control.
- i.e. you do not want the user to lose session data.
- Enabling stickiness may result in 'imbalances'.

- Enabled under Target Groups; set amount of seconds for stickness to last (1
    second to 7 days).


