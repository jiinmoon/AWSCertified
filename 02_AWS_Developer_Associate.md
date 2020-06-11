# Introduction To Elastic Beanstalk

- Quickly deploy and manage web-apps on AWS without worrying about the
    underlying infrastructure (PaaS).

## What is PaaS?

- Platform as a Service is a business model that provides platform to the
    customers to develop, run, and manage apps without the complexity of
    building and maintaing the infrastructure typically associated with
    developing and launching an app.

- similar to Heroku.

## Intro to EB

- In short, choose your platform, upload your code and it runs with little
    knowledge of the infrastructure.

- Not recommended for Production apps (enterprise level apps).

- EB is powered by a CloudFormation template setups for you:

    - Elastic Load Balancer
    - Autoscaling Groups
    - RDS Database
    - EC2 Instance preconfigured (or custom) platforms
    - Monitoring (CloudWatch, SNS)
    - In-place and Blue/Green deployment methodologies
    - Security (Rotates passwords)
    - Can run Dockerized environments

## Supported Languages

- Ruby (Rails)
- Python (Django)
- PHP (Laravel)
- Tomcat (Spring)
- NodeJS (ExpressJS)
- Comes with preconfigured platform...

## Web vs Worker Environment

- When creating EB for first time, we need to choose an environment.
- In web enviroment, there are EC2 instances managed by the ASG (Auto Scaling Groups),
    interacts with Load Balancer (optional) and goes out to the internet.
- In worker enviroment, this is for background jobs.
    - Creates an SQS Queue.
    - CloudWatch Alarm to dynamically scale instances based on health (load
        capacity).

## Web Envrionment Types

- Load Balanced Environment:

    - uses ASG and set to scale
    - uses an ELB (Elastic Load Balancer)
    - designed to scale
        - traffic increases? creates more instances
    - EC2 instance(s) in ASG <-> ELB <-> Internet

- Single-Instance Environment

    - running a single server!
    - still uses ASG but desired capacity set to 1 to ensure server is always
        running.
    - no ELB (to save $)
    - public IP addr has to be used to route traffic to server

## Deployment Policies

- Available Deployment Policies:

    - Load Balanced or Single-Instance Env
    - Load Balanced has all deployment policies
    - Single-Instance misses Rolling and Rolling with additional batch
        - Rolling requires ELB as it attaches and detaches instances in batches
            from the ELB

## Deployment Policies - All At Once

- deploys the new app version to all instances at the same time.
- takes all instances out of service while the deployment processes.

- fastest, but most dangerous deployment method.
- in case of failure, we need to roll back the changes by redeploying the
    original version again to all instances.

## Deployment Policies - Rolling

- 
