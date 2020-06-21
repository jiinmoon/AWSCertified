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

## Cross-Zone Load Balancing

- Each load balancer instance distributes evenly across all registered instances
    in all AZ.

- i.e. With Cross-Zone load balancing enabled, a load balancer at one AZ is not
    only be able to distribute to instances in its own AZ, but also other AZs.

- So, by default, load balancer only works within its registered AZ.

- Pricing:
    - CLB
        - Disabled by default.
        - No charges for inter-AZ data if enabled.
    - ALB
        - On by default (cannot be disabled).
        - No charges for inter-AZ data.
    - NLB
        - Disabled by default.
        - Charge for innter-AZ data.

- Can find the Cross-Zone balancing option by selecting each running load
    balancers (without exception of ALB since it is always-on).

---

## SSL/TLS Basics

- A SSL Certificate allows the traffic between your clients and your load
    balancer to be encrypted in transit (in-flight encryption).

- SSL: Secure Sockets Layer, used to encrypt connections.
- TLS: Transport Layer Security, newer version of SSL.

- TLS Certificates are mainly used but people would still refer to it as SSL.

- The Public SSL Certificates are issued by Certificate Authorities (CA).
    - i.e. Comodo, Symantec, GoDaddy, GlobalSign, etc...

- Certificates have an expiration data and must be renewed.

## Load Balancers & SSL Certificates

- Users will send the encrypted HTTPS request using public SSL Certificate.
- Load Balancer will then have to use private SSL Certificate to be able to read
    its content.
- Then, Load Balancer will forward a new request to appropriate EC2 instance
    over HTTP since now the traffic is within secure private network.

- The load balancer uses an X.509 certificate (SSL/TLS server certificate).
- Certificates can be managed by ACM (AWS Certificate Manager).
- You can upload your own certificates.
- HTTPS Listener:
    - Specify a default certificate.
    - Add an optional list of certificates to support multiple domains.
    - Clients can use SNI (Server Name Indication) to specify the hostname they
        reach.
    - Ability to specify a security policy to support older versions of SSL /
        TLS (legacy client).

## SSL - SNI

- Server Name Indication solves the problem of loading multiple SSL certificates
    onto one web server (to serve multiple websites).
- This is a newer protocol; and requires the client to indicate the hostname of
    the target server in the intial SSL handshake.
- The server will then find the correct certificate, or return the default one.

- Only Works for ALB & NLB, CloudFront.

### Elastic Load Balancers - SSL Cert

- CLB
    - Supports only one SSL Certificate.
    - Multi use multiple CLB for multiple hostname with multiple SSL Certifcate.

- ALB, NLB
    - Supports mulitiple listeners with multiple SSL Certificates.
    - Uses SNI to make it work.

---

## ELB - Connection Draining

- Feature naming:
    - CLB: Connection Draining
    - Target Group: Deregistration Delay (for ALB & NLB)

- Time to complete 'in-flight requests' while the instance is de-registering or
    unhealthy.

- Stops sending new requests to the instance which is de-registering.

- i.e. User sends a request and traffic is routed by ELB to an EC2 instance.
    However, the instance is now unhealthy and goes under 'draining' mode. EC2
    waits for existing connections will be completed. And any subsequent 
    requests from the user is rerouted to another EC2 instance.

- By default, 300 seconds; but 1 - 3600 seconds customizable.

- Can be disabled (set value to 0).

- Short request? set it short; vice versa.

---

## ASG - Auto Scaling Group

- The load on your websites and application can change in real life.
- In the cloud, you can create and get rid of servers very quickly.

- ASG allows for:
    - Scaling Out (add more EC2 instances) to increase capacity.
    - Scaling In (remove EC2 instances) to save costs.
    - In short, it ensures we have max/min number of machines running to match
        the needed capacity.
    - Automatically Register new instances to a load balancer.

## ASG in AWS

- Minimum Size: number of machines to have on.
- Atual Size / Desired Capacity: number of machines running currently.
- Maximum Size: maximum capacity of machines that can be added to group.

## ASG + Load Balancer

- Load Balancer will route the traffic to the existing machines managed within
    the ASG.
- Also, when new instances are created and added to the group, the load balancer
    will also be alerted, and will route the traffic to the new instances as
    well.

## ASG Attributes

- A launch configuration:
    - AMI + Instance Type
    - EC2 User Data
    - EBS Volumes
    - Security Groups
    - SSH Key Pair

- Min/Max Size, Initial Capacity.
- Network and Subnet information.
- Load Balancer information.
- Scaling Policies.

## ASG Alarms

- It is possible to scale an ASG based on CLoudWatch alarms.
- Alarm -> triggers scaling to ASG.
- An Alarm monitors a metric (such as average CPU usage).
- **Metrics are computed for the overall ASG isntances**.
- Based on the alarm,
    - we can create scale out/in policies (increase/decrease instances).

## Auto Scaling New Rules

- It is now possible to define 'better' auto scaling rules that are directly
    managed by EC2.
    - Target Average CPU usage.
    - Number of requests on the ELB per instance.
    - Average Network In/Out
- These rules are easier to set up and makes more sense.

## Auto Scaling Custom Metric

- We can auto scale based on a custom metric (i.e. number of connected users).

1. Send custom metric from application on EC2 to CloudWatch (PutMetric API).
2. Create CloudWatch alarm to react to low/high values.
3. Use CloudWatch alarm as the scaling policy for ASG.

## ASG+

- Scaling Policies can be based on any metric, or even custom metric provided by
    your application to CloudWatch.
- Can use Launch configureations or Launch Templates (new).
- To update an ASG, you must provide a new launch configuration or launch
    template.
- IAM roles attached to an ASG will be assigned to EC2 instances.
- ASG are free; only pay for underlying resources being launched.
- Having instances under an ASG means that if they get terminated for whatever
    reason, the ASG will automatically create new ones as a replacement.
- ASG can terminate instances marked as unhealthy by an LB and replace it.

## ASG Creation

- Creating an ASG requires either first creating a launch template, or
    configuration.
    - configuration is the older version; used to create one group.
    - template is newer; used for fleets.

- We can create template with various settings:
     - AMIs (Amazon Linux 2, etc...).
     - Storage.
     - Key-pairs.
     - Instance type (t2.micro, etc...).
     - Security Groups.
     - IAM roles.
     - User Data.
     - etc...

- Then, we can create an ASG using the template.
    - We pick our pricing for launching Instances (On-Demand, Spot).
    - Choose Desired, Min, Max capacity.
    - Scaling Policies.

---

## ASG - Scaling Policies

- **Target Tracking Scaling**
    - Simple way to set-up.
    - i.e. set average ASG CPU at around 40%.

- **Simple / Step Scaling**
    - When CloudWatch alarm is triggered (i.e. CPU greater than 70%), then add more units.
    - When CloudWatch alarm is triggered (i.e. CPU less than 30%), then remove.

- **Scheduled Actions**
    - Anticipate a scaling based on known usage patterns.
    - i.e. increase the minimum capacity at 6 PM on every Friday.

## ASG - Scaling Cooldowns

- cooldown period helps to ensure that your ASG does not launch or terminate
    additional instances before the previous scaling acitivity takes effect.

- In addtion to default cooldown for ASG, can create new cooldowns that apply to
    a specific simple scaling policy.

- The scaling specific cooldown overrides the default cooldown period.

- One common use for scaling-specific cooldowns is with a scale-in policy; a
    policy that terminates instances based on a specific criteria or metric.
    Because of this policy terminates instances, EC2 Auto Scaling needs less
    time to determine whether to terminate additional instances.

- If default 300 seconds is too long, reduce costs by applying a
    scaling-specific cooldown period of 180 seconds to the scale-in policy.
    - this will allow the instances to terminate faster.

- If your app is scaling up and down multiple times each hour, modify the ASG
    cooldown timers and the CloudWatch Alarm Period that triggers the scale in.

- Scaling action occurs -> Default CD in effect?
    - Yes -> Ignore.
    - No -> Launch or terminate instance.

## ASG - Auto Scaling

- We may attach a Scaling Policy to the ASG; if we do not have one, create a new
    Policy.

- When creatin the Scaling Policy, we choose
     - Scaling Type (Target Tracking Scaling, Simple, Step).
     - Metric Type.
     - Value.

- Say, you have creates a Scaling Policy with CPUUtilization.

- Then, CloudWatch will set up alarms to notify ASG whenever CPUUtilization goes
    above the target or below the target.

