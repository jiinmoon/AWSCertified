AWS Elastic Beanstalk
=====================

- A way to deply apps in AWS safely and predictably.

Developer Problem
-----------------

- Previously, we have seen a typical Web App 3-tier architecture:

    - Route53 to manage the DNS; resolve hostnames to our ELB.
    - ELB on the Public Subnet receives user traffics and routes them to EC2
      instances managed within Auto Scaling Group.
    - EC2 instances are spreaded across multiple AZs and placed within Private
      Sbnet for security reasons.
    - Each instances can access either ElastiCache or RDS for data storage.

- As a developer, there is a lot to micro manage on AWS.
    - managing infrastrucure
    - deploying code
    - configuring databases, load balancers...
    - scaling policies

- But most of the web applications have the same architecture (ALB + ASG).

- So, as a developer, we just want to upload code to make it happen.

- Possibly, consistently across different applications and environments.

AWS Elastic Beanstalk Overview
------------------------------

- Elastic Beanstalk is a PaaS that is focused on this developer problem.
- It is a layer on top of multiple different AWS componenets such as EC2, ASG,
  ELB, RDS and so on.

- It still allows for full control over the configuration if necessary.

- **Beanstalk is free - you pay for underlying instances and resources.**

**Elastic Beanstalk**

- it is a managed service:
    - instance configuration / OS is handled by Beanstalk.
    - deployment strategy is configurable but performed by Elastic Beanstalk.

- developer is only responsible for application code.

- Three models:
    - Single Instance deployment: for development.
    - LB + ASG: production or pre-production web applications.
    - ASG Only: good for non-web applications in production (workers).

Elastic Beanstalk Components
-----------------------------

**Three components**

- Application.
- Application version: each deployment gets assigned a _version_.
- Evnrionment name (dev, test, prod): free naming.

You deploy application versions to environments and can promote application
versions to the next environment.

It has a rollback feature to previous application version.

Full control over liftcycle of enviornments.

Create Application + Create Environments -> Upload Version -> Relase to
beanstalk environment.

**Supported platforms**:

- Go
- Java SE
- Java w/ Tomcat
- .NET on Windows Server with IIS
- Node.js
- PHP
- Python
- Ruby
- Packer Builder
- Single Container Docker
- Multicontainer Docker
- Preconfigured Docker
- if unsupported, you can write a custom platform.

Beanstalk First Environment
---------------------------

Create a simple beanstalk -- first web application. We need to choose
a platform (i.e. Node.js 10).

You can see the events of your beanstalk environment - which displays logging
information from the environment. When we examine the event logs, we see the
process of how the environment was set up:

1. Amazon S3 storage bucket is provisioned to store environment data.
2. Security Group is created.
3. EIP was created.
4. Initializes and launches EC2 instance; once launched, instance is added to
   the environment.
5. App is made available.

All these provisioned resources can be managed in their own services.

For detailed logs, you can see the output under _Logs_ section.

Health information is also available - information about latency, loads, ###
responses, and so on.

Beanstalk Second Environment
----------------------------

When creating a second environment, you can select either _web server
environment_ or _worker environment_.

You can configure beanstalk environment in many ways:

- presets are available to set recommended values:
    - Single Instance
    - Single Instance (using Spot instance)
    - High availability
    - High availability (using Spot and On-Demand instances)
    - Custom

- Software:
    - X-Ray
    - S3 log storage
    - instance log steaming to CloudWatch

- Instances:
    - Root volume (boot devices)
    - EC2 security groups

- Capacity:
    - Auto Sacling Group
        - min/max # of instances
        - composition (On-Demand? or Spot?)
        - instance type (t2.micro? etc...)
        - AMI id
        - number of AZs and placement (eu-west1a? eu-west1b?)
    - Scaling triggers
        - Metric (CPU Utilization...)
        - Statistic 

- Load Balancer
    - Choose ALB, CLB or NLB
    - _Once chosen at creation time, cannot change Load Balancer type later._

- Rolling updates and deployments

- Security

- Monitoring
- Managed Updates
- Notifications
- Networking
- Database (RDS?)
    - created RDS stroage in beanstalk environment will disappear as
      environment is also terminated.

Elastic Beanstalk Deployment Modes
----------------------------------

**Important for exam: which deployment modes to select in certain situations?**

**Single Instance**

Great for development. Single EC2 instance running web app server with EIP
attached; managed under ASG; and maybe with a database. All within a single AZ. DNS name
maps straight to EIP.

**High Availability with Load Balancer**

Great for production grade web apps. Multiple EC2 instances running web servers
managed under ASG; spans across multiple AZs; and maybe with a database or two.
Load Balancer is used to route traffics to instances. DNS name is routed to ELB
DNS name.

Elastic Beansalk Deployment Options for Updates
-----------------------------------------------

**All at once (deploy all in one go)**

Fastest, but instances would not be available to serve traffic for a bit
(introduces downtime in services).

**Rolling**

Update a few instances at a time (bucket); then move onto the next bucket once
the first bucket is healthy.

**Rolling with additional batches**

Like rolling; but spins up new instancesto move the batch (old application is
still available). Thus, this maintains the capacity during updates roll out.

**Immutable**

Spins up new instances in a new ASG; deploys version to these instances and
then swaps all the instances when everything is healthy.

Deployment Options: All at once
-------------------------------

Suppose we have version 1 applications in multiple instances. Elastic Beanstalk
will take down all the instances, update the applications to version 2, and run
again.

- Fastest deployment.
- Has downtime.
- Great for quick iterations in development environment.
- No additional cost.

Deployment Options: Rolling
---------------------------

Application will run below capacity while update is happening. Suppose we have
version 1 applications running on 4 instances. If we set bucket size to 2, then
Elastic Beanstalk will taken down 2 instances, update to version 2. This
process repeats until no more bucket.

- Bucket size is adjustable.
- Application is running both versions simultaneously.
- No additional cost.
- Can be long to deploy all.

Deployment Options: Rolling with Additional Batches
---------------------------------------------------

Apps will run at capacity. From previous example, we will create two additional
instances where version 2 apps are running on top of 4 instances running
version 1. Now, Elastic Beanstalk stops 2 instances running the version
1 applications, and repeat the process.

- Additional batch will be terminated in the end.
- Longer deployment.
- Small additional cost.
- Good for production environment.

Deployment Options: Immutable
-----------------------------

Suppose we have an ASG running four instances with version 1 apps. Elastic
Beanstalk will create a new, temporary ASG and runs a single instance with
version 2 app to check for its health. Once passes, it populates the instances
to the capacity. Then, new instances will be transfered to over current ASG
running the version 1 apps, and terminates the old instances.

- Zero downtime.
- New code is deployed to new instances on a temporary ASG.
- High cost; double the capacity.
- Longest to deploy. 
- Quick to rollback in case of failure (terminate new ASG).
- Great for production.

Deployment Options: Blue / Green
--------------------------------

- Not a native _feature_ of Elastic Beanstalk.
- Zero downtime and release facility.
- Create a new _stage_ environment where verions 2 apps are deployed.
- New environment (green) can be validated independetly and roll back if
  issues.
- Route53 can be setup using weighted policies to redirect certain portion of
  traffic to the stage envrionment.
- Once ready, we can swap URLs using Beanstalk.

Deployment Summary from AWS Doc
-------------------------------

- doc found at
  [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html).

| Method | Impact of Failed Deployment | Deploy Time | Zero Downtime | No DNS Change | Rollback Process | Code Deployed To |
| --- | --- | --- | --- | --- | --- | --- |
| All-at-once | Downtime | 1 | X | Y | Manual Redeploy | Existing instances |
| Rolling | Single batch out of service | 2 | Y | Y | Manual Redeploy | 
| Rolling w/ additional Batch | Minimal if first fails | 3 | Y | Y | Manual Redeploy | New & Exsiting instances |
| Immutable | Minimal | 4 | Y | Y | Terminate New Instances | New instances |
| Blue/Green | Minimal | 4 | Y | N | Swap URL | New instances | 

---

Beanstalk CLI
-------------

- It is an additional CLI called `EB cli` which designed to work with EB
  specifically.

- Basic commands include:
    - `eb create`
    - `eb status`
    - `eb health`
    - `eb events`
    - `eb logs`
    - `eb open`
    - `eb deploy`
    - `eb config`
    - `eb terminate`

- Helpful for automating deployment pipelines.
- Note: This is more for AWS DevOps exam.

Elastic Beanstalk Deployment Process
------------------------------------

- Describe dependencies of your application.
    - i.e. `requreiements.txt` for Python or `package.json` for Node.js

- Code is packaged as a `zip` file, and describe dependencies.

- **Console**: upload zip file; creates new app version, and then deploy.

- **CLI**: create new app version with CLI (uploads zip), and then deploy.

- Once finished, EB will deploy the zip on each EC2 instance, resolve
  dependencies and start the application.

- Note: CLI is out of scope for Dev exam.

Beanstalk Lifecycle Policy
--------------------------

Elastic Beanstalk is capable of storing upto 1000 application verions. If you
do not remove old versions, **cannot deploy anymore**.

Thus, use **Lifecycle Policy** to phase out old application versions.

Phase out policy can be based on following:

- Based on time (old versions are removed).
- Based on space (when you have too may versions).

Versions that are currently used won't be deleted. Also, there is an option not
to delete the source bundle in S3 to prevent data loss.

Lifecycle Policy is available under _Application Versions_ > _Settings_.


Beanstalk Extensions
--------------------

- A zip file containing our code must be deployed to EB.
- All the parameters set in the UI can be configured with code using files.
- Requirements:
    - placed within `.ebextensions/` in the root of source code.
    - it is in either `YAML` or `JSON` format.
    - must have `.config` extensions (i.e. logging.config).
    - able to modify some default settings using: `option_settings`
    - ability to add resources such as RDS, ElastiCache, DynamoDB, etc...
- Note: resource managed by `.ebextensions` get deleted if the environment goes
  away.


---

Beanstalk under the hood -- CloudFormation
------------------------------------------

Underneath EB, it is powered by **CloudFormation** tech which is used to
provision other AWS services (to be covered later).

Use case: you can define CloudFormation resources in your `.ebextensions` to
provison ElastiCache, an S3 bucket, or anything you would like.

EB Cloning
----------

You can clone an existing environment with its exact same configuration. This
is very useful for deploying a _test_ environment of your applciation.

All the resources are preserved as well:

- Load Balaner type and configuration.
- RDS database type (but data is not preserved).
- Environment variables.

After cloning an environment, you can change settings.

It is available under _Envrionment actions_ in the console.

EB Migrations: Load Balancer
----------------------------

- After creating an EB environment, **Load Balancer type cannot be changed**
  only the configuration can be changed. (i.e. cannot change ALB to NLB).

- To change the type, you must perform **migrations**. To migrate:
    - create a new envrionment with the same configuration except LB (can't
      clone).
    - deploy your application onto the new envrionment.
    - perform a CNAME swap or Route53 update so that DNS is directed to the new
      type of LB that is running with new environment.

RDS with EB
-----------

- RDS can be provisioned with Beanstalk - great for dev / test.
- This is however not great for production envrionment since the database
  lifecycle is tied to the Beanstalk environment lifecycle.
- Thus, best option is to separate the RDS database; provide the EB application
  the connection string.

EB Migrations: Decouple RDS
---------------------------

1. Create a snapshot of RDS DB (as a safeguard).
2. Go to the RDS console and protect the RDS database from deletion.
3. Create a new EB environment, without RDS, point your application to existing
   RDS.
4. Perform a CNAME swap (blue/green) or Route53 update so that DNS is directed
   to the new EB envrionment to confirm it is working.
5. Terminate the old environment (RDS won't be deleted).
6. Delete CloudFormation stack (in `DELETE_FAILED` state).

---

Beanstalk with Docker -- Single Docker
--------------------------------------

You can run your applicaion as a single docker container.

You provide either:

- `Dockerfile`: EB will build and run the Docker container.
- `Dockerrunn.aws.json` (v1): describe where *already built* Docker image is.
    - image, ports, volumes, logging, etc...

**Beanstalk in Single Docker Container does not use ECS.**

Beanstalk with Docker -- Multi Docker Container
-----------------------------------------------

It helps to run multiple containres per EC2 instance in EB.

This will create:
- ECS Cluster
- EC2 instances, configured to use the ECS Cluster
- Load Balancer (in high availability mode)
- Task definitions and execution

Requires a config `Dockerrun.aws.json` (v2) at the root of source code. It is
used to generate the **ECS task definition**.

Your Docker images must be pre-built and stored in Docker repositories such as
Docker Hub or ECR for example.

Within EB envrionment, there is a load balancer and ECS Cluster + ASG managing
multiple EC2 instances. Within the instances, there would be multiple Docker
containers running. Load Balancer will know how to connect to each container.

When creating a EB environment, select _Docker platform_.

---

EB Advanced Concepts -- HTTPS
-----------------------------

**EB with HTTPS**

To do HTTPS with the Beanstalk, we would load the SSL certificate onto the Load
Balancer. This can be done in two ways:

- from the console (EB console, load balancer configuration).
- from code: `.ebextensions/securelistener-alb.config`.

SSL Certificates can be provisioned using ACM (AWS Certificate Manager) or CLI.

Remember to set a security group rule to allow incoming traffic on port 443
(HTTPS).

**EB with HTTP to HTTPS redirecting**

Configure your instances to redirect HTTP to HTTPS
([doc](https://github.com/awsdocs/elastic-beanstalk-samples/tree/master/configuration-files/aws-provided/security-configuration/https-redirect))

Alternatively, you can configure Application Load Balancer (ALB only) with
a rule.

Make sure that health checks are not redirected.

EB Advanced Concepts -- Web Server vs Worker Environment
---------------------------------------------------------

**This is more for DevOps exam.**

If your app performs tasks that are long to complete, offload theses tasks to
a dedicated **worker environment**.

_Decoupling_ your application into two tiers is a common practice.

i.e. processing a video, generating a zip file, and etc...

You can define periodic tasks in a file `cron.yaml`.

Example two tier architecture would be: there is a Web Tier consists of ELB and
EC2 instances. Requests will come through the ALB, directed to the instances
managed under ASG. Then, the instances will request _works_ on different Worker
Tier environment that consists of SQS and EC2 instances. The requests are
queue'd within the SQS Queue, and eventually serviced by instances managed
under ASG.


EB Advanced Concepts -- Custom Platform
---------------------------------------

Very advanced; it allows you to define from scratch:

- OS
- Additional softwares
- Scripts that Beanstalk runs on these platforms

Use case would be if application code language is incompatible with Beanstalk
and does not use Docker.

To create own platform:

- Define an AMI using `Platform.yaml`.
- Build that platform using the `Packer Software` (open source tool to create
  AMIs).

Custom Platform vs Custom Image (AMI):

- Custome Image is to tweak an existing Beanstalk Platform (Python Node.js,
  ...)
- Custom Platform is to create entirely new Beanstalk Platform.


