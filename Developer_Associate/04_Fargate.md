# Chapter 04: ECS Fargate

- Fargate is serverless containers; do not worry about maintaining servers, just
    run containers and pay per usage.

- You can create an empty ECS cluster (no EC2 provisioned) and then launch Tasks
    as Fargate.
- Do not have to provision, configure and scale clusters of EC2 instances to run
    containers.
- Charged for at least one min - then per second; pay based on duration and
    consumption.

- Difference from ECS is that under ASG, ECS cluster maintaines EC2 Containers
    that runs tasks and services inside; but Fargate does not have EC2
    Containers within ECS Cluster, but simply tasks and services.

## Configuring Fargate Tasks

- In Fargate Task Definition you define the memory and vCPU.
- Then add your containers and allocate the memory and vCPU required for each.
- When you run the Task you can choose what VPC and subnet it will run in.
- Apply a Security Group to a Task (***) - always at task level; not server.
- Apply an IAM role to the Task.

- **You can apply SG and IAM role for BOTH ECS and Fargate Tasks and Services!**

## Serverless? THen what is the difference between Fargate vs Lambda?

- While they are similar, here are differences.

- Fargate:
    - unlimited duration;
    - memory upto 30 GB;
    - can supply own containers;
    - integeration requires more manual labour;
    - pay at least 1 min then every second after.

- Lambda
    - 15 mins of duration:
    - upto 3 GB of memory;
    - limited to standardize containers;
    - seamlessly integrates with other serverless services;
    - pay per 100 ms.



