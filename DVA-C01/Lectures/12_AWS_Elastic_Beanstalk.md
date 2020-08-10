AWS Elastic Beanstalk (EB)
==========================

A way to deploy apps in AWS easily, safely and predictably.

Developer Problem
-----------------

Previously we have seen a typical 3 Tier Architecuture:

- Route53 to handle DNS resolution (map hostname to ELB).
- ELB, placed on the Public subnet, receives user traffic and routes them to
  the EC2 instances managed by an ASG within the Private subnet.
- The EC2 instances are spread across multi-AZ for availability.
- Each instance can access the RDS or the ElastiCache as its storage solution.

However, as a developer, there is too much manual work to micro manage:

- managing infrastructures.
- deploying codes.
- configuring databases, load balancers, etc.
- scaling policies.

Since many web applications share the same architecture, we want to leverage on
automated process - where we can simply upload the code.

EB -- Overview
--------------

Elastic Beanstalk (EB) is a Platform as a Service (PaaS) model that solves this
developer's problem. It is a layer on top of multiple AWS services such as EC2,
ASG, ELB, RDS and so on.

It still allows for a full-control of individual componenets if necessary.

**EB is a free-service; you only pay for underlying provisioned services**.

It is a managed service:

- Instance configuration and OS is handled.
- Deployment strategy is configurable, but performed by EB.

Three main models:

- Single Instance deployment : mostly for DEV purposes.
- LB + ASG : PROD or TEST web applications.
- ASG Only : good for non-web applications in PROD (workers).

EB -- Componenets
-----------------

**Three Componenets of EB**

- Application.
- Application version : each deployment is assigned a verion.
- Environment name (such as DEV, TEST, PROD).

You deploy application version to envrionment and can promote application
version to the next environment. This versioning allows for a rollback to
previous application version and fully control over the lifecycle of
envrionments.

**Supported Platforms**

- It supports many popular platforms such as
- Go
- JavaSE
- Java w/ Tomcat
- .NET w/ Windows Server
- Node.js
- ...
- **If unsupported, you may write a custom platform**.

EB -- First Environment
-----------------------

Here, we are going to creat a simple EB - a web application. First, we will
need to choose a platform (i.e. Node.js version 10).

You can see the full event logs of the EB envrionment, and can use it to see
the automated process of how EB is setting up:

1. Amazon S3 Bucket is provisioned to store the environment data.
2. Securit group is created.
3. EIP was created.
4. Initializes and launches EC2 instance and added to the environment.
5. Application is now available.

EB -- Second Environment
------------------------

When creating a second environment, we can select either **web server** or
**worker**.

You can configure EB environment in different ways:

- Presets:
    - Single Instance
    - Single Instance with Spot instances
    - High Availability
    - High Availability with Spot and On-Demand instances
    - etc

- Software:
    - X-Ray
    - S3 Log Storage
    - Instance Log streaming to CloudWatch

- Instances:
    - Root volume for boot
    - EC2 security groups

- Capacity:
    - ASG
        - min/max number of instances.
        - composition (On-Demand, Spot, ...)
        - instance type (t2.micro, etc)
        - AMI id
        - number of AZs and placement
    - Scaling Triggers
        - based on Metric (CPU Utilization)
        - based on Statistic

- Load Balancer
    - CLB, ALB or NLB
    - **Once choosen, cannot change its type later**.

- Rolling Updates and Deployments
- Security
- Monitoring
- Managed Updates
- Notifications
- Networking
- Database (RDS)
    - **RDS database will disappear as environment is terminated**.

EB -- Depoyment Modes
---------------------

**Single Instance**

- Great for DEV.
- Single EC2 instance running web application server with EIP attached.
- It is managed under an ASG and can be provisioned with a database.
- Placed in a single AZ.
- DNS name maps to its EIP.

**High Availability with Load Balancer**

- Great for PROD.
- Multiple EC2 instances managed under an ASG and can be provisioned with
  more than one database.
- DNS name maps to ELB DNS name.

EB -- Deployment Options for Updates
------------------------------------

**All-At-Once**

Fastest method, but has a downtime while updating all the instances.

**Rolling**

Update a "bucket" at a time until all the buckets are updated.

**Rolling with Additional Batches**

Like rolling, but it will spin up new instances to the batch; it will maintain
the same capacity during updates.

**Immutable**

Create a new ASG and instances; deploys new version to theses instances and
replace the old ASG.

EB -- Deployment -- All-At-Once
-------------------------------

Suppose we have app v1 running within multiple instances. EB will take down all
the instances, update the apps to v2, then run again.

- **Fastest, but has downtime**.
- No additional cost.

EB -- Deployment -- Rolling
---------------------------

Suppose we have app v1 running within four instances. If we set the bucket size
to 2, then EB will take down 2 instances and updates them to v2. This process
repeates until no more buckets are left.

- Adjustable bucket size.
- Maintains same capacity.
- Application v1 and v2 is running at the same time during updates.
- No additional cost.

EB -- Deployment -- Rolling with Addtional Batches
--------------------------------------------------

From previous example, instead of taking down the 2 instances for updates, EB
will create two new instances and deploy the v2 there. Then, EB will stop the
2 instances running app v1. This process repeates until finished.

- Additional batches will be terminated in the end.
- Small additional cost (new instances created).

EB -- Deployment -- Immutable
-----------------------------

Suppose we have an ASG managing four instance with app v1. EB will create
a new ASG and run a single instance to test the health of app v2 deployed. If
passes, it populates the ASG to the capacity. Then, new instances will be
transfered to the new ASG, and terminate the old.

- Zero downtime.
- High cost (double the capcacity at times).
- Longest to deploy.
- Quick to rollback in case of failure (old ASG is available until last step).

EB -- Deployment -- Blue/Green
------------------------------

It is not a native feature of EB, but it provides zero downtime and release
facility.

- Create a new stage environment where app v2 is deployed.
- New environment (GREEN) can be validated independently and rollback in case
  of failures.
- Route53 is then configured to use weighted policy; redirect certain % of
  traffic from old EB envrionment to new.
- Once testing is finished and ready, we can swap the URLs.

EB -- Deployment -- Summary
---------------------------

Read more at
[here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-verions.html).

EB -- CLI
---------

- It is an additional CLI that is designed to work with EB specifically.

- It is helpful for automating deployment pipelines.

EB -- Deployment Process
------------------------

- First, describe the dependencies for your application.
    - i.e. `requirements.txt` for Python
    - i.e. `package.json` for Node.js

- Code is packaged within a .zip file.

- With **Console**, simply upload the .zip file and create the new app version
  to deploy.

- With **CLI**, create the new app version by uploading the .zip, and deploy.

- Once finished, EB will deploy the .zip on each instances, resolve the
  dependencies and start the application.

EB -- Lifecycle Policy
----------------------

EB is capableof storing upto 1000 application versions. If you do not remove
old versions, you cannot deploy anymore. Thus, use lifecycle policy to phase
out the old application versions as necessary.

It can be based on following:

- based on time (FIFO).
- based on space.

Versions that are currently used would not be deleted; and there is an option
not todelete the source bundle in S3 to prevent any loss of code.

EB -- Extensions
----------------

- A .zip file containing our code must be deployed to EB.
- All the parameters set in the UI can be configured with code.
- Requirements:
    - It is placed within `.ebextensions/` in the root of source code.
    - It is in either YAML or JSON.
    - Must have `.config` exntensions (i.e. `logging.config`).
    - Ableto modify some default settings using `option_settings`.
    - Ability to add resources such as RDS, ElastiCache, etc.

- Note that resources specified in `.ebextensions` gets deleted if the
  environment disppears.

EB under the hood: CloudFormation
---------------------------------

Underneath the EB, it is powred by another AWS service called,
**CloudFormation** which is used to orchestrate and to provision other AWS
services.

i.e. you may define CloudFromation resources within `.ebextensions` to
provision ElastiCache, S3 bucket or anything else.

EB -- Cloning
-------------

You may clone an existing environment with its exact same configuration
- useful for deploying a TEST envrionment.

All the resources are preserved:

- Load balancer type and its configuration.
- RDS database type (but data is NOT preserved).
- Envrionment variables.

EB -- Migrations: Load Balancer
-------------------------------

After creating an EB envrionment, Load Balancer type cannot be changed (only
its configurations). To change the type, perform **migrations**:

- Create a new envrionment with the same configuration except load balancer.
- Deploy your application onto the new envrioment.
- Perform a CNAME swap or Route53 update so that DNS is directed to the new
  type of load balancer that is running with new environment.

EB -- RDS
---------

- RDS can be provisioned with Beanstalk.
- This is not ideal since its lifecycle will be tied to the lifecycle of the
  environment (when environment is gone, the RDS is as well).
- Thus, it is best to provision RDS spearately, then connect to it.

EB -- Migrations: Decouple RDS
------------------------------

1. Create a snapshot of RDS DB (failsafe).
2. Goto the RDS console and protect the RDS database from deletion.
3. Create a new EB envrionment without RDS, then point your appliction to
   existing RDS.
4. Perform a CNAME swap (BLUE/GREEN) or Route53 update so that DNS is directed
   to the new EB environment .
5. Terminate the old environment and delete the CLoudFormation stack.

EB -- Docker: Single Docker
---------------------------

You can run application as a single Docker container; to do this, you provide:

- `Dockefile` : EB will build and run the Docker container.
- `Dockerrun.aws.json` : describes where already built Docker image is.

**EB in Single Docker Container does not use ECS**.

EB -- Docker: Multi-Docker Container
------------------------------------

It helps to run multiple containers per E2 instance in EB.

This creates:

- ECS Cluster.
- EC2 instances (configured for ECS).
- Load Balancer (in high availability mode).
- Task Definitions and execution.

It requires a config file, `Dockerrun.aws.json` at the root of the source code
which is used to generate the **ECS Task Definitions**.

The Docker images must be pre-built and stored in Docker repositories such as
Docker Hub or ECR for example. Within EB envrionment, there is a load balancer
and ECS Cluster + ASG manaing multiple EC2 instances. Within the instances,
there would be multiple Docker containers - and load balancer will know how to
connect to each.

Sepcify Docker Platform when creating an EB environment.

EB -- Advanced -- HTTPS
-----------------------

**EB with HTTPS**

To do in-flight encryption with EB, we need to load the SSL certificates onto
the load balancer, which is done in two ways:

- from the concole (EB Console -> Load Balancer Configuration).
- from code: `.ebextensions/securelinstner-alb.config`.

SSL Certificates can also be provisioned through AWS ACM or CLI.

_Remember to set a security group rule to allow incoming traffic on port
443_.

**EB with HTTP to HTTPS redirect**

Configure your instances as specified in
[here](https://github.com/awsdocs/elastic-beanstalksamples/tree/master/configuration-files/aws-provided/security-configuration/https-redirect).

Alternatively, you can configure ALB with a rule (make sure that health checks
are not redirected).

EB -- Advanced -- Web Server vs Worker Environment
--------------------------------------------------

If a task is taking too long to complete, off-load tasks to a dedicated
**worker environment** - by decoupling the application into two tiers.

EB -- Advanced -- Custom Platform
---------------------------------

It allows you to define EB from scratch. Use if your application language is
currently unsupported and does not uses Docker.

To create own platform:

- Define an AMI using `Platform.yaml`.
- Build that platfrom using the `Packer` tool.

**Custom Platform vs Custom Image**:

- Custom image is to tweak an existing EB platform .
- Custom platform is to create an entirely new EB platform.





