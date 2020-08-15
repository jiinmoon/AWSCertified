# 12 Elastic Beanstalk

- It is rather difficult as a developer to manage all the infrastructure and
  different AWS services.

- Elastic Beanstalk is an abstract layer on top of EC2, ASG, ELB, RDS, and so
  on. It manages the provisioning and deployments.

- All we need to do is write code and deploy our apps.

- It is a managed service:
    - Beanstalk handles all the instance configuration and OS.
    - Deployment strategy is configurable byt performed by Elastic Beanstalk.

- Three models:
    - Single Instance Deployments (for dev environment).
    - LB and ASG (for production web apps).
    - ASG only (for worker environments).

- It is composed of:
    - Application
    - Application version (each deployment is its own version)
    - Envrionment name (dev, test, prod, ... )

- You deploy application versions TO environments and can promote application
  versions to the next environments.

- **If rollbacks occur, it goes back to previous application ersion**.
- Environments' life cycles are controlled.

- It supports many language platforms as well as Docker containers (single,
  multi, preconfigured Docker).

- If it is not supported, you can also write your own custom platform.

## EB Deployment Modes

- Single Instance - deploys to a single AZ with EC2 instance attached to an
  EIP; it has DNS name mapped to the EIP. Great for development.

- High Availability with Load Balancer - deploys to multiple EC2 instances
  which are spread across multi-AZ and is load balanced; its DNS name is the
  ELB's DNS name. Used for production.

### EB Deployment Options for Updaing App

- To update the application on the environment, there are four deployment
  options available:

- All at once : is fastest but has downtime.
- Rolling : updates few batch at a time; slight decrease in availability.
- Rolling with additional batches : crates new instances to move the batch;
  maintains same availability.
- Immutable : creates new instances in a new ASG; deploys version to these
  instances then move over to the new ASG if ok.

#### All at once

- Fastest to deploy since every instance is taken down for update.
- Meaning there is a downtime while the instances are down.
- Good for development; no additional cost.

#### Rolling

- Set a batch size; and number of instances of that size will be taken down at
  a time for update.
- The environment will be serving new and old applications at the same time.
- No additional cost; but long time to deploy.

#### Rolling with additional batches

- Application will always be available at its capacity (or more at times).
- Additional batches with new app version are created and replaces the old
  batch at a time.
- Small cost to create the new instances.
- Longer to deploy.

#### Immutable

- Create a new temporary ASG with new instances with updated apps.
- Then, new instances from the ASG will be added to the current ASG and old
  instances will be taken down.
- Highest cost since it will temporary double the capacity while updating.

#### Blue/Green

- Not a direct feature but it is available.
- Create a new environment where new apps are deployed. Use Route53 to shift
  the traffic gradually over to the new envrionment.

## EB CLI

- There is an additional CLI for the Elastic Beanstalk that works with the EB
  on command line.

- Basic commands include:

    eb create
    eb status
    eb health
    eb events
    eb logs
    eb open
    eb deploy
    eb config
    eb terminate

- Used for creating automated deployment pipelines and scripts.

## Deployment Process

- Describe the dependencies (for example, include `requirements.txt` for Python).
- Package all the codes as a zip file (with dependencies description).
- We can either deploy using:
    - Console : upload the zip file to create a new app version and deploy.
    - CLI : create new app version with CLI command (uploads zip) and deploy.

- EB will automatically deploy zip on each EC2 instances, resolve dependencies
  and start the application.

## Beanstalk Lifecycle Policy

- EB can at most store 1000 application versions; if old versions are not
  removed, we cannot deploy anymore.

- We use lifecycle policy to phase out the old application versions:
    - based on time.
    - based on space.

- Current versions in use are not deleted.
- Option to backup the source buindle in S3.

## EB Extensiuons

- To deploy a new app, a zip file containing the code must be deployed to EB.
- We can also configure the parameters using the code in the file that we
  deploy with:
    - place the `.config` files in the `.ebextensions/` at the root of source
      code.
    - the config is in either YAML or JSON.
    - it can modify default settings under `option_settings`.
    - it can add additional resources such as RDS, ElastiCache, DynamoDB, ...
- Note that resources created with `.ebextenions` will be deleted if the
  envrionemnt is removed; so if using external resource that needs to persist
  such as RDS or DynamoDB, you should create it separately.

## EB and CloudFormation

- EB uses CloudFormation to provision its resources.
- So, we can define CloudFormation resources in `.ebextensions` to provision
  any other AWS resources as we need.

## EB Clone

- You can clone existing environment with its exact same configuration.
- It is useful feature to deploy test environ,ent.
- All resources and configurations are preserved:
    - load balancer type and configuration
    - RDS database type (but data is not preserved)
    - environment variables

## EB Migrations : Load Balancer

- Note that you cannot change the ELB type once the environment is created.
- To change this, we first need to migrate the existing environment:

    1. create a new environment with the same configuration except Load
       Balancer.
    2. Deploy your application to the new environment.
    3. Perform CNAME swap or Route53 redirect update.

## EB and RDS

- RDS can be provisioned along with the EB.
- But this can cause problem since the lifecycle of the RDS is tied to the
  lifecycle of the environment.
- Best practice is to create RDS instance separately; then provide application
  the connection string (or IAM role to access).

- To migrate an existing RDS:

    1. create a snapshot of existing RDS.
    2. change RDS setting to protect it from deletion.
    3. create a new EB environment without RDS; change application to use the
       existing RDS.
    4. perform CNAME swap or Route 53 update.
    5. terminate the old environment.
    6. delete the CLoudFormation stack (in `DELETE_FAILED` state).

## EB and Docker
### EB - Single Docker Container

- Running app inside a single Docker container.
- We need to provide either:
    - Dockerfile : EB will build image accordingly and run the container.
    - Dockerrun.aws.json : describes where existing Docker image is.
- Note that in Single Docker Container type, EB does not use ECS; just a single
  EC2 instance.

### EB - Multi-Docker Container

- multiple docker containers per EC2 instance in EB.
- it will create:
    - ECS Cluster
    - EC2 instances, configured for ECS CLuster
    - Load Balancer (in High Availability)
    - Task definitions and execution
- requires a `Dockerrun.aws.json` config at the root of source code; it is used
  to generate the ECS task definition.

## EB and HTTPS

- To enable SSL/TSL with Elastic Beanstalk:
    - Load the SSL certificate onto the Load Balancer
    - This is done with either at the console or
    - Provie the config `.ebextensions/securelistern-alb.config`.
    - Certificates can be provisioned with ACM or CLI
    - Must configure security group to allow inbloud on port 443.

- You can also redirect HTTP to HTTPS:
    - Configure instance to redirect or
    - Configure ALB (and only ALB) with a rule.
    - Make sure that health checks are not being redirected.

## Web Server vs Worker Environment

- If app performs tasks that will take long to complete, decouple and offload
  them to a dedicated worker environments.
- Create a Web Environment that will serve the requests with ELB + EC2 managed
  with ASG.
- Then, the workloads are placed onto the SQS queue and worker environment will
  poll and process these works.
- Periodic tasks can be set up using `cron.yaml`.

## Custom Platform

- You can create own platform from OS, additional softwares and scripts.
- Useful for when your application langauge is not supported and does not use
  Docker.
- To create one:
    - define an AMI using `platform.yaml`
    - build with packer software.
- This is not custom image (AMIs); custom AMI is to tweak the existing
  platform.
- Custom platform builds an new platform.


