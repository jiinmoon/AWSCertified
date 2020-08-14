# Docker in AWS

## Docker

- It is a container tech that is used to solve the problem of "it works on my
  machine".

- It provides easily deployable container that can be run in any machine or OS
  without any compatibility issues.

- Each docker container is a separate process that is managed via Docker
  daemon.

- Each docker container based on a Docker image which can be found on either
  public Docker hub or privately on AWS ECR.

- Docker is not VM tech - there isn't a hypervisor that managed multiple guest
  OS. Instead, Docker daemon manages each container as process.

## ECS Clusters

- ECS Clusters are logical grouping of EC2 instances where it run the ECS agent
  in a Docker container.

- ECS agent will register the instance to the ECS cluster; for this, EC2
  instances must be based on a speicific AMI made for ECS.

### ECS Task Definitions

- Task Definitions are metadata in JSON that specifies how ECS should run
  a Docker container.

- It contains crucial information such as which image to use, how to port bind,
  how much memory and cpu to allocate, environment variables and set up network
  details.

### ECS Service

- It is a way to define how many tasks should be run and how they should be
  run.

- It ensures that optimal number tasks are running across our fleet of EC2 instances.

- Can be linked with load balancers.

#### ECS Service with Load Balancers

- If we specify the port mappings, then that specific port becomes unavailable
  for other tasks (containers) to use. This makes it difficult to scale.

- Instead, by not specifiying port mapping, we run the tasks that are mapped to
  hosts port at random.

- Then, we use the Application Load Balancer to use dynamic port forwarding to
  automatically discover them and shift the traffic accordingly.

### ECS IAM Roles

- Because of multiple layers are working here, we need different roles to
  assign proper permissions.

- **EC2 Instance Profile**
    - is used by ECS agent
    - require permissions to make API calls to ECS service
    - require permissions to send logs to CloudWatch Logs
    - require permissions to Docker pull from ECR

- **ECS Task Role**
    - controls each task which role they will assume
    - i.e. Task A may only read from S3 bucket
    - i.e. Task B can interact with another EC2 instance

### ECS Tasks Placement

- When a task of type EC2 is launched, ECS has to determine where to place it
  with the constraints of CPU, memory and available port. And same for
  determining which tasks to terminate when scaling in.

- We use Task Placement Strategy and Task Placement Constraints.

- Note that Fargate does this automatically; this is only for ECS with EC2.

#### ECS Task Placement Process

- It is a best effort:

1. Identify the instances satisfying CPU, memory and port requirements in task
   definition.
2. Identify the instances that satisfy the task placement contraints.
3. Identify the instances that satisfy the task placement strategies.
4. Select the instance for task placement.

#### Binpack

- Places based on the least available amount of CPU or memory.
- Minimizes the number of instance created and saves costs.

#### Spread

- Places the task evenly based on the specified value (i.e.
  `attributes:ecs.availability-zone` will distribute across instances on
  multiple AZs).

#### ECS Task PLacement Constraints

- distinctInstance places tasks on a different container instance.
- memberOf places task on instances that satisfy an expression
    - i.e. only on certain EC2 types such as `t2.*`.

### ECS Service Auto Scaling

- CPU and Memory are tracked in CloudWatch at the ECS service level.

- TargetTracking - target a specific average CloudWatch metric.
- StepScaling - scale based on CloudWatch alarms.
- Scheduled Scaling - based on predictable changes.

- ECS Service Scaling is done at the task level; this is not to be confused
  with the EC2 Auto Scaling which is done at instance level.

- Cluster Capacity Provider determines the infrastructure that a task will run
  on - providing instances as required.

## ECR

- It is a container registry that is private on AWS.
- its access is controlled via IAM.
- Login with either:

    $(aws ecr get-login --no-include-emial --region us-west-1)

    aws ecr get-login-password --region us-west-1 | docker login --username AWS --pasword-stdin 0123456789.dkr.ecr.us-west-1.amazonaws.com

- Docker push and pull remains same:

    docker push 0123456789.dkr.ecr.us-west-1.amazonaws.com/name:tag
    docker pull 0123456789.dkr.ecr.us-west-1.amazonaws.com/name:tag

## Fargate

- It is a serverless version of ECS Cluster.
- We do not have to provision nor manage EC2 instances our selves.
- It will automatically scale.
- We only need to supply the task definitions.

# ECS Summary

- ECS provisions EC2 instances to run conatiners onto.
- Fargate is ECS serverless without managing EC2 instances.
- EKS is a managed Kubernetes by AWS.

- For ECS:
    - EC2 instances must be created.
    - Configure `/etc/ecs/ecs.config` with cluster name.
    - EC2 instances must run ECS agent.
    - EC2 instances can run multiple container on the same type:
        - Do not specify host port.
        - Use Application Load Balancer with dynamic port mapping.
        - Configure EC2 security group to allow ALB traffics.
    - ECS Tasks has its own roles.
    - Security group works at instance level; not per task.

- For ECR:
    - Know the commands to login, push and pull.

- For Fargate:
    - Serverkess; AWS provisions containers and assigns them ENIs.
    - Fargate tasks can have IAM roles.

- ECS integrates with CloudWatch Logs; make sure that EC2 Instance Profile has
  the necessary permissions.

- Use IAM Task Roles to give tasks permissions to perform actions against the
  AWS.


