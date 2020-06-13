# Elastic Container Service (ECS)

- ECS is a fully-managed container orchestration service.
- Highly secure, reliable, and scalable way to run containers.

## ECS Componenets

- Under management of ASG (Auto Scaling Group):
    - We have a ECS Cluster that may have several EC2 containers running.

- Cluster:
    - multiple EC2 instances which will house the docker containers.

- Task Definition:
    - A json file that defined the configuration of containers to run (upto 10).

- Task:
    - launches containers defined in Task Definition.
    - tasks do not remaining running once workload is complete.

- Service:
    - ensures tasks remaining running (i.e. web-app).

- Container Agent:
    - Binary on each EC2 instance that monitors, starts, and stops tasks.

## Creating a Cluster

- EC2 Linux + Networking
    - Recourses to be created:
        - Cluster
        - VPC
        - Subnets
        - ASG with Linux AMI (Amazon Machine Image)

- several options:
    - use Spot or On Demand?
    - EC2 instance type
    - number of instances
    - EBS Storage Volume
    - EC2 can be Amazon Linux 2 or Amazon Linux 1
    - Choose a VPC or create a new VPC
    - Assign an IAM Role
    - option to turn on CloudWatch Container Insights
    - Choose Key-pair
        - you can SSH into an EC2 container instance and make changes; however,
            this is not recommended.

- Task Definitions File
    - Task Definition JSON File example
    - [create a new Task Definition] wizard.
    - (docker) images can be provided either via ECR or an official docker repos
        (i.e. DOcker Hub).
    - it must have one essential container - if this container fails or stops
        than all other containers will be stopped.

## Elastic Container Registry (ECR)

- ECR is a fully managed Docker container registry that makes it easy for
    developers to store, manage, and deploy Docker container images.
- Docker image can be pushed to ECR (which is like a repo).
    - then, it can be accessed by various AWS services including ECS.
        - others include Fargate, EKS,On-Premise

# ECS Follow-Along

## Creating an ECS Service

- have an app inside the docker image, pushed onto ECR.
- we need to create an ECS Instance Role.
    - in IAM console, choose Roles: search for 'ecsInstanceRole'.
    - Role -> Create Role -> EC2 -> 'ec2containerServiceorEC2Role'

- goto ECS.
    - Create Cluster -> Select Cluster (EC2 Linux + Networking).
    - this builds the ECS Cluster, apply ECS Instance IAM Policy, and
        CloudFormation Stack.

- once cluster is created, we can create Task/Service.
    - Create new Task Definition.
        - it will require container definitions to add Container image.
    
