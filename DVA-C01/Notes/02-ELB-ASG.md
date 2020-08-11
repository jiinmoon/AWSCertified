# 2 ELB and ASG

## High Availability

- High Availability usually means horizontal scaling or scaling out.
- It means that the system is fail-proof as it is running on at least two
  different data centres (or AZs).
- **Disaster Recovery**.

- In terms of EC2,
    - Scaling in/out means to increase the number of instances.
    - Scaling up/down means to change the type of instance size.
    - High availability means running same app across multi AZ.
        - ASG multi-AZ.
        - Load Balancer multi-AZ.

## Elastic Load Balancer

### Overview

- It is a server where it has a single exposed entry into users where it will
  distribute requests downstream into target groups such as EC2 instances.

- A single DNS point of access is exposed.
- Performs regular health checks.
- Provides SSL/TLS (HTTPS).
- Enforce cookie stickness.
- High availability across AZs.

- You can set up internal or external ELBs.

### Health Checks

- ELB performs regular ping to a certain route/port to check for status
  response.

- If the reponse is anything but 200, it is unhealthy.

### 3 Types of ELBs

- Classic Load Balancer (HTTP, HTTPS, TCP)
- Application Load Balancer (HTTP, HTTPS, WebSocket)
- Network Load Balancer (TCP, TLS (secure TCP) & UDP)

### Security Groups

- Must set up the security group attached to target application or target group
  to **allow traffic from load balancer**.

### Important

- LBs can scale but not instantaneously (contact AWS for warm-up).
- Error Codes:
    - 4XX are client side.
    - 5XX are application side.
    - LB 503 error means at capacity or no registered target.
    - If LB cannot connect to application, check security group.

- ELB maintains a log of all access requests.
- CloudWatch Metrics will give aggregate statistics such as connections count.

### Classic Load Balancers

- Supports TCp, HTTP, HTTPS.

- Health checks are TCP or HTTP based.

- Has a fixed hostname "elb-name.region.elb.amazonaws.com".

### Application Load Balancers

- HTTP, HTTPS, and WebSocket.
- Load balances to multiple HTTP applications to target groups.
- It can also load balances to multiple applications on same instance
  (containers).

- Supports HTTP to HTTPS redirects.

- Since it is layer 7 based, it can use routing tables to direct traffics to
  different groups.
    - Routing based on path in URL `example.com/users` or `example.com/posts`.
    - Routing based on subdomain name in URL `this.example.com` or
      `that.example.com`.
    - Routing based on query string and headers `example.com/users?user_id=1`.

- Supports port mapping feature to load balance traffics to a dynmaic port in
  ECS containers.

- Has a fixed hostname "elb-name.region.elb.amazonaws.com".

- True IP of the client is available in the header `X-Forwarded-For`.

### Network Load Balancers

- Works at Layer 4 (TCP and UDP).
- Handles millsion of request per seconds and ~100 ms latency.

- **NLB has static IP per AZ** and supports assigning Elastic IP.
    - Contrast from CLB and ALB which has hostname.

- It is used for extreme performance for TCP and UDP traffics.

### Stickiness

- Possible to enable stickiness such that a client's request is always
  redirected to a specific target group.

- Cookie has a expiration date.

- Great for saving the user session data and states.

- **Works with CLB and ALB** since it is layer 7 based.

### Cross-Zone Load Balancing

- When enabled, load balances distributes evenly to all registered instances
  across all AZs.

- If not, then load balancers can only direct their traffic to its own AZ.

- CLB : Disabled by default.
- ALB : Enabled by default.
- NLB : Disabled by default and you pay extra for inter AZ data crossed. 

### SSL/TLS

- Load Balancer uses an X.509 certificate (SSL/TLS server certificate).
- Can manage certificates using ACM.
- Or, upload your own.

- Load Balancer will implement HTTPS linster:
    - specify the default certificate.
    - clients use SNI to specify the hostname to reach.

- Server Name Indication SNI works only with ALB and NLB (also CloudFront).

- CLB only supports one SSL; thus, need multiple CLB for multiple hostnames.
- ALB and NLB support multiple listeners with SNI.

### Connection Draining

- It is a time to complete in-flight request while the instance is
  de-registering or unhealthy.

- When instance is "draining" or unhealthy or unregistering, connection will
  stay open for already existing ones but no more after.

- 1 ~ 3600 seconds (300 by default).
- Can be disabled (set it to 0).




## ASG

### Introduction

- Allows for scaling in/out operations of the EC2 instances based on the
  metrics such as CPU-Utilization.

- It will automatically register new instances with the load balancer.

- We set Min/Max/Desired amount of EC2 instances to be run.

### ASG Alarms

- Can scale based on CloudWatch alarms.
- Alarm will monitor a metric (i.e. average CPU usage) which is determined as
  an average of all ASG instances.
- Then, it will trigger scaling in/out in ASG.

### ASG Custom Metrics

- We can also scale based on custom metrics.

1. Send custom metric from application on EC2 to CloudWatch using `PutMetric`.

2. Create CloudWatch alarm to react to low/high values.

### Summary

- To update an ASG, you must provide new launch configuration or template.
- IAM roles assigned to ASG will be assigned to EC2 instances.
- ASG is free; only pay for instances provisioned.



