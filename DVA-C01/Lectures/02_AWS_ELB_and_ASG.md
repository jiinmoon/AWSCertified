AWS Elastic Load Balancer and Auto-Scaling Groups
=================================================

Introduction to Availability vs Scalability
-------------------------------------------

"Scaling" means that an application or a system can adapt to shifting
workloads; when less work is required, we would like to _scale in_ whereas if
workload increases, we _scale out_. And when we do, we scale either
**vertically** or **horizontally**.

**Vertical Scalability**

It is a concept of increasing the size of the box. If box cannot fit the
increased number of objects, then you replace the orignal box with a bigger
box. For example, in terms of AWS, vertically scaling an application would be
changing its instance type from t2.micro to t2.large. This is common for
database types.

**Horizontal Scalability**

It refers to increasing the number of boxes. The box cannot accomodate the
increased number of objects; so, we create more boxes of same size to fit them
all. This implies that the distributed system is used underneath, and it is
a common to see this in web applications.

**High Availability**

The concept of high availability is linked to horizontal scaling. *It refers to
running a system in at least two or more data centres (i.e. AZs) in order to
provide redundancy in case of disasters or data cetre loss*. Some AWS services
provides this by default (i.e. AWS RDS).

Scalability, Availability and EC2
---------------------------------

- _Vertical Scaling_ means to increase the size of instance.
    - i.e. from t2.nano (0.5 G RAM, 1 vCPU) to u-2v1.metal (12.3 TB RAM, 448
      vCPUs)
- _Horzontal Scaling_ means to increase the number of instances.
    - i.e. use Auto-Scaling Group (ASG) or Load Balancer to distribute loads
      among multiple instances.

- _High Availability_ means running the instances for same application across
  multiple AZs.

---

AWS Elastic Load Balancers (ELB)
--------------------------------

**Load Balancers** are servers that forward incoming traffics to multiple
servers downstream (i.e. to EC2 instances). Hence, this _balances_ and
distributes the workload on the applications - no idling applications.

ELB exposes a single point of access (DNS) to your application, meaning that
you do not have to worry about how to make all the public IPs of each EC2
instances available so that it is available to users.

It provides many benefits such as:

- gracefully handles failures of downstream instances via performing regular
  _health checks_.
- provide SSL termination (HTTPS).
- enforces stickiness with cookies.
- enables high availability across AZs.
- cleanly separates between public and private traffic since all inbound
  traffics must come from ELB.
- it is managed by AWS, meaing that:
    - AWS gurantees it will be working.
    - AWS takes care of upgrades, maintenance and high availability.
- it integrates seamlessly with many AWS services.

**Health Checks** are a way for the load balancer to determine which instance
is "alive" and "ready" to receive the incoming traffic. It is done on
a specific port + route (i.e. `/health`) - if response is not 200, it is
unhealthy.

AWS ELB -- Types
----------------

| Type | Description |
| --- | --- |
| Classic Load Balancer | Outdated; supports HTTP, HTTPS, TCP |
| Application Load Balancer | suports HTTP, HTTPS, WebSocket (HTTP/2) |
| Network Load Balancer | supports TCP, TLS, UDP |

AWS ELB -- Security Groups
--------------------------

Security groups can be attached to the ELBs to handle in/outbound traffics.

i.e. ELB may want to be able to receive HTTP and HTTPS traffics from anywhere.
To allow this, create a security group rules enabling TCP port 80 and 443, and
attach it to the ELB.

It is also possible so that the EC2 instances may only receive HTTP traffic
from ELB only for securit reasons. This is done with security group by
specifying the ELB as a trusted source.

AWS ELB -- Summary
------------------

- ELBs can scale but not instantaneously.
- Important Error Codes to know:
    - `4xx` are client related errors.
    - `5xx` are app related errors.
    - `503` referes to capacity reached or no registered target to forward the
      traffic to.
    - General rule is that if the ELB cannot connect to the application, it is
      most likely a security group issue.

- Monitoring:
    - ELB access logs will log all access request.
    - AWS CloudWatch Metrics is where you can view aggregated statistics (such
      as total connection counts).

AWS ELB -- Classic Load Balancer
--------------------------------

- Supports TCP (layer 4) and HTTP/HTTPS (layer 7).
- Health Checks are either TCP or HTTP based.
- Has a fixed hostname: `*.region.elb.amazonaws.com`.

AWS ELB -- Application Load Balancer
------------------------------------

- Supports only layer 7 (HTTP, HTTPS, WebSockets).
- Supports HTTP to HTTPS redirects.
- It can route traffic to:
    - multiple HTTP applications across instances (target groups).
    - multiple applications on the same machine (containers).

- Routing tables to different target groups:
    - Routing based on path in URL.
        - i.e. `www.example.com/users` and `www.example.com/posts` are two
          different resource paths; thus, ALB can set up to route them to
          different target groups (separate instances).
    - Routing based on hostname in URL.
        - i.e. `xyz.example.com` vs `abc.example.com`.
    - Routing based on query string, or headers.
        - i.e. `example.com/users?id=111&order=false`.

- ALBs work with microservices and container-based applications such as Docker
  or Amazon Elastic Continaer Service (ECS).

- Has a port mapping feature to redirect to a dynmaic port in AWS ECS.

i.e Suppose we would like to set up a ALB that will receive inbound traffic
from users at different routes such as `/user` or `/search`. We can define
_target groups_ to route to a specific grouping of instances which are
dedicated to resolving specific routes.

AWS ALB -- Target Groups
------------------------

- ALB routes its traffics to various target groups as follows:
    - EC2 instances (can be under management of an ASG) - HTTP.
    - ECS tasks (managed by ECS) - HTTP.
    - AWS Lambda functions - HTTP requeest is translated into JSON.
    - IP addresses - private IPs.

- ALB can route to multiple target groups.
- Health Checks are done at the target group level.

AWS ALB -- Summary
------------------

- Has a fixed hostname (`*.region.elb.amazonaws.com`).
- Application servers do not see the IP of the client directly.
    - "True" IP of the client is inserted in the header `X-Forwarded-For`.
    - Since instance is behind the ALB, it only receives the traffic from ALB;
      hence, it would not be able to determine where the original request is
      from. This is why extra header is provided by ALB to attach the original
      source IP.
    - Likewise, you will also get `X-Forwarded-Port` and `X-Forwarded-Proto`.

AWS Network Load Balancer
-------------------------

- Operates at layer 4: TCP/UDP.
- Handles millions of request per seconds.
- ~100 ms latency compared to ~400ms of ALBs.
- NLB has one static IP per AZ; and supports assigning EIP.
- **Notice that CLB and ALB has a static hostnames instead**.
- It is used in cases where performance is critical.
- There isn't a free tier available on NLB.

Load Balancers -- Stickiness
----------------------------

"Stickness" allows the same client to be always directed to the same instance
behind the load balancer which is possible since CLB and ALB operates at Layer
7. This is used for the cases where you want to store the session data for the
   users.

The stickness _cookie_ has a expiration date that you can control. It is
enabled under target groups and it can last between 1 second to 7 days.

Load Balancers -- Cross-Zone Load Balancing
-------------------------------------------

- **It enables load balancers to distribute evenly across all registered
  instances in all of AZs**.

- i.e. A load balancer in AZ X can distribute loads to not only to AZ X, but to
  other instances in AZ Y, AZ Z, and etc.

- **By default, load balancers only work within its registered AZ**.

| Type | Pricing Model |
| --- | --- |
| CLB | off by default; no charges for inter-AZ if enabled |
| ALB | on by default; no charges for inter-AZ |
| NLB | off by default; charges for inter-AZ |

---

Basics on SSL/TLS
-----------------

SSL (TLS) Certificates allow the traffic between your clients and your load
balancer to be encrypted during flight. SSL refers to _Secure Sockets Layer_
and TLS is _Transport Layer Security_ which is newer; and both terms are used
interchangeably most of the time.

These certificates are issued by Certificate Authorities (CA) such as Comodo,
Symantec, GoDaddy, GlobalSign and etc. With the certificate, we can attach it
to the load balancer to enable secure connection over HTTPS. It has expiration
date which must be constantly renewed.

Load Balancers -- SSL Certificates
----------------------------------

Users will send the encrypted HTTPS request using public SSL Certificates, and
load balancer will need to use private SSL Certificate to be able to decrypt
the content. Load Balancers use X.509 certificate and it can be managed by AWS
Certificate Manager (ACM) where it can be renewed automatically or you may
use your own.

Enabling HTTPS LInster:

- Specify a default certificate.
- Add an optional list of certificates to support multiple domains.
- Clients can use Server Name Indication (SNI) to specify the hostname that
  they are attempting to reach.
- Ability to specify a security policy to support older versions of SSL/TLS.

**Server Name Indication (SNI)** is used to solve the problem of loading
multiple SSL certificates onto a single web server (to serve multiple
websites). This requires the client to specify the hostname of the target
server in the initial SSL handshake. This works only with ALB, NLB, and AWS
CloudFront.

ELB -- SSL Certification
------------------------

**CLB** only supports a single SSL certificate. We would need a multiple CLB to
serve multiple hostname with multiple SSL certificates.

**ALB and NLB** supports multiple listeners with multiple SSL certificates and
uses SNI to make it work.

ELB -- Connection Draining
--------------------------

- Two diferent names:
    - CLB : **Connection Draining**
    - Target Group : **Deregistration Delay** (for ALB and NLB)

- This refers to **time to complete "inflight requests" while the instance is
  deregistering or unhealthy**.

- Load balancer will then stop sending new request to the instance that is
  in "de-registering" mode.

- By default, 300 seconds (1 ~ 3600 seconds); can be disabled by setting it to
  0.

---

AWS Auto-Scaling Group (ASG)
----------------------------

The workloads on the applications are changing overtime, and you need to be
flexibly increase and decrease the number of instances - and do it quickly to
adapt.

ASG allows for following:

- Scaling Out (add more EC2 instances) to increase capacity.
- Scaling In (remove EC2 instances) to save on cost.
- In short, it ensures we always have min/max number of machines to match the
  capacity needed.

**Minimum Size** refers to absolute minimum number of machines to have on
always.

**Actual Size** and **Desired Capacity** is the number of machines running
currently.

**Maximum Size** is max amount of machines that can be added to the group.

ASG -- Load Balancers
---------------------

Load Balancer can route the traffic to the existing machines managed within the
ASG. In fact, when new machines are created or terminated, the load balancer is
notified and takes appropriate actions to route the traffics correctly.

ASG -- Attributes
-----------------

- Launch Configuration (machine specifications):
    - AMI and Instance Type
    - EC2 User Data
    - additional EBS Volumes
    - Security Groups
    - SSH Key-pairs

- Min/Max Size and Initial Capacity.
- Network and Subnet information.
- Load Balancer information.
- **Scaling Policies**.

ASG -- Alarms
-------------

- AWS CloudWatch Alarms is one way to trigger scaling event on ASG.
- Once set, alarm will monitor metric on the ASG such as average CPU usage
  (note that metric is computed as an average over all instances in ASG).
- Based on the alarm, we can create **scaling policy** to add or remove
  instance.

ASG -- New Rules
----------------

We can define new preset rules - "better" auto-scaling rules that are directly
managed by EC2.

- Target average CPU usage.
- Number of requests on the ELB per instance.
- Average network trffic flow.

ASG -- Custom Metric
--------------------

It is possible to set up a scaling policy based on custom metric (i.e. number
of connected users).

1. Send custom metric from application on EC2 to CloudWatch (`PutMetric` API).
2. Create CloudWatch Alarm to react to low or high values.
3. Use CloudWatch Alarm as a sacling policy for ASG.

ASG -- Summary
--------------

- **Scaling Policies can be based on any metric (even custom) provided by your
  application to CloudWatch or based on a schedule**.

- To update an ASG, need to provide a new launch configuration or launch
  template.

- IAM roles attached to an ASG will be assigned to its managed EC2 instnace.
- **ASG service itself is free; only pay for underlying resources
  provisioned**.
- ASG can terminate unhealty instances marked by an load balancer and replace
  it.

ASG -- Scaling Policies
-----------------------

- **Target Tracking Scaling**
    - simplest set-up; try to hit the "target".
    - i.e. set average ASG CPU usage at 40%.

- **Simple / Step Scaling**
    - CloudWatch Alarm trigger (i.e. CPU load > 70%) -> add more units.
    - CloudWatch Alarm trigger (i.e. CPI < 30%) -> remove units.

- **Scheduled Actions**
    - based on usage patterns.
    - i.e. increase the min capacity on weekends between 6PM to midnight.

ASG -- Scaling Cooldowns
------------------------

Scaling Cooldown is a period that helps to ensure that ASG does not launch or
terminate additional instance before the previous scaling acitivity takes into
efect. On top of default cooldowns, we can also create new cooldowns specific
to scaling policy.

Common usage for scaling-specific cooldowns would be with a "scale-in" policy;
since this policy terminates instances, ASG needs less time to determine
whether to terminate additional instances.

Default is set to 300 seconds. Setting it lower to help reduce cost since it
will make the instances terminate faster.

