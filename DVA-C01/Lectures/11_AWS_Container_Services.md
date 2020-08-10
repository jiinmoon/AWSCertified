AWS Container-based Services: ECS, ECR and Fargate
==================================================

Docker Technology
-----------------

**Docker** is a development platform to deplo the applications; the
applications are packaged within discrete units called, containers, which makes
sure that **applications runs always the same way regardless of underlying
architectures or systems**. Thus, it will run on any machine without
compatibility issues and will always be predictable.

Also, containerizing the applications makes it extremely easy to scale as well
since we only need to add more containers as the workload increases.

**Docker Images**

Docker images are stored in **Docker repositories** (main public repository is
the [Docker Hub](https://hub.docker.com) where you can find many base images
prepackaged with different softwares and OS). For private Docker repository,
Amazon offers **Elastic Container Registry (ECR)**.

**Container vs Virtual Machine**

The containers are simply process running on top of host - sharing the same
resources and managed under a single daemon.

Virtual machines require a running hypervisor on top of the host OS, and each
virtual machines run their own guest OS.

**Docker Container Management**

There are several container management platform; and Amazon offers following:

- AWS Elastic Container Service (ECS)
- AWS Fargate - serverless platform.
- AWS Elastic Kubernetes Service (EKS)

---

ECS -- Clusters
---------------

**ECS Clusters** are logical grouping of EC2 instances where each of the
instances run the **ECS Agent** within the Docker container. ECS Agent
registers the instances to the ECS cluster; for this, the instances needs to
run on a special AMI made to be used with ECS.

In short, it is an ASG of the EC2 instances with typical security groups and
roles.

ECS -- Task Definitions
-----------------------

**Task Definitions** are metadata in JSON that instruct ECS on how to run the
Docker containers. It contains informations such as:

- name of the image
- port binding between container and host
- memory and CPU required
- envrionment variables
- networking information

**Task** can get an optional IAM roles - this is required for the application
inside the containers to access the other AWS services.

ECS - Service with Load Balancer
--------------------------------

Suppose that we have several EC2 instances in a ECS Cluster, and want to run
several web servers. _The problem is that only one host port per EC2 instance
that container port can be mapped to_. For example, if one container port 80 is
mapped to host port 8080, another container cannot use the same host port 8080.

Thus, instead of mapping to static host port, **we will assign random host port
to the containers**. However, how do we route our traffic to thses randomized
port numbers? Answer: we use **load balancer** which is enabled by **Dynmaic
port forwarding**.

To enable this, when creating a task definition, we configure the port mapping
to be empty - this assigns a random port. Also, the load balancer may only be
attached during creation - already running cluster cannot use it.

_Classic Load Balancer_ does not work. Also, we need to configure security
group to allow the load balancer to access range of ports.

---

ECR Overview
------------

We will want to manage and to maintain our own Docker images; for this, AWS ECR
provides a private Docker image repository where access is controlled with IAM.

You can access ECR via CLI using following commands:

1. **AWS CLIv1 Login Command**

        $ $(aws ecr get-login --no-include-email --region us-west-1)

    - the output of `aws ecr get-login` generates the `docker login` command.

2. **AWS CLIv2 Login Command**

        $ aws ecr get-login-password --region us-west1 | docker login --username USERNAME --password-stdin 1234567890.dkr.ecr.us-west1.amazonaws.com

    - first command grabs password which is piped to `docker login`.

For push and pulling Docker images:

        $ docker push 1234567890.dkr.ecr.us-west1.amazonaws.com/demo:latest
        $ docker pull 1234567890.dkr.ecr.us-west1.amazonaws.com/demo:latest


Fargate
-------

Fargate is serverless; we do not need to provision EC2 instances. Simply create
the task definitions and AWS will manage. To scale, we would increase the
number of tasks.

---

ECS -- IAM Roles
----------------

**EC2 Instance Profile** is attached to the running EC2 instances that are
running Docker container with ECS Agent.

- It is used by ECS Agent to make API calls to the ECS service.
- It is used to send container logs to CloudWatch Logs.
- It is used to pull Docker images from ECR. 

The tasks running inside the instances that are managed under ECS Agent, it
will require its own role to interact with other AWS services; it is called,
**ECS Task Role**.

- It allows each task to have a specific role.
- i.e. for Task A, attach a role A such that it only grants the task to read
  and write from a specific S3 bucket.
- Task role is defined in **Task Definitions**.

ECS -- Task Placement and Contraints
------------------------------------

When a task of type EC2 is launched, ECS must determine where to place it with
the contraints of CPU, memory and available port. Simiarly, when a service
needs to scale in, ECS needs to determine which task to terminate.

This is where **Task placement strategy** and **Task placement contraints** are
used. Note that this is unnecessary in Fargate as we do not manage the EC2
instances.

ECS -- Task Placement Process
-----------------------------

Task placement strategies are _best effort_ based; when ECS places tasks, it
uses following process to select container instances:

1. Identify instances that satisfy CPU, memory and port requirements in the
   Task Definitions.
2. Identify instances that satisfy the task placement contraints.
3. Identify instances that satisfy the task placement strategies.

ECS -- Task Placement Strategies
--------------------------------

**Binpack Strategy**

- places tasks based on the least available amount of CPU or memory.
- helps to minimize the number of instance in use for cost-saving.

```json
"placementStrategy": [
    {
        "field" : "memory",
        "type" : "binpack"
    }
]
```

- in other words, it will try to fill up the EC2 instance with containers as
  much as possible. And if it cannot, then spins up another EC2 instance.

**Random Strategy**

```json
"placementStrategy": [
    {
        "type" : "random"
    }
]
```

**Spread Strategy**

- suppose that we have 3 instances spread across 3 AZs.
- we will try to place tasks evenly based on the specified value.
- i.e. `attribute:ecs:availabilty-zone` will spread tasks across instances on
  multiple AZs.

```json
"placementStrategy": [
    {
        "field" : "attribute:ecs:availability-zone",
        "type" : "spread"
    }
]
```

- this maximizes the availability.

**Note that theses strategies can mix**.

ECS -- Task Placement Contraints
--------------------------------

- `distinctInstance` places each task on a different container instance.
- `memberOf` places task on instances that satisfy an expression (using the
  Cluster Query Lanauge).

```json
"placementConstraints": [
    {
        "expression" : "attributes.instance-type =~ t2.*",
        "type" : "memberOf"
    }
]
```

- Above will place tasks only on the instance of type `t2.*`.

ECS -- Service Auto-Scaling
---------------------------

- CPU and RAM is tracked by CloudWatch at the ECS service level.
- **Target Tracking** shows a specific average CloudWatch metric of a target
  such as CPU utilization.
- **Step Scaling** scales based on CloudWatch alarms.
- **Scheduled Scaling** is based on predictable changes.

- In short, this is just like ASG (in fact, they are same).

- **ECS Service Scaling != EC2 Auto-Scaling**; meaning that scaling up the ECS
  service does not necessarily mean that the number of EC2 instances are going
  to scale as well.

ECS -- Cluster Capacity Provider
--------------------------------

- It is used in conjunction with a cluster to determine the infrastucture that
  a task runs on.

- For ECS and Fargate users, `FARGATE` and `FARGATE_SPOT` capacity providers
  are added automatically.
- FOR ECS on EC2 isntances, you need to associate the capacity provider with an
  ASG.
- When running a task or a service, you define a capacity provider strategy to
  prioritize in which provider to run on.
- This allows the capacity provider to automatically provision infrasturcture.

Suppose that we have an EC2 instances filled with containers even though its
CPU utilization is below 30%; in thiscase, they have no more capacity to add
more containers. Here, new task will be launched on a new EC2 instance by the
capacity provider.

ECS -- Summary
--------------

ECS is used to run Docker containers and comes in 3 flavours:

- ECS Classic : provision EC2 instances to run containers.
- Fargate : serverless; no EC2 instances to manage.
- EKS : managed Kubernetes by AWS.

ECS -- Clasic
-------------

- EC2 isntances are created to host the containers.
- Configure `/etc/ecs/ecs.config` with the cluster name.
- EC2 instance must run an ECS Agent within a Docker container.
- EC2 instance can run multiple containers of same type:
    - To do so, **must not specify the host port to randomize**.
    - And also, **use ALB with dynamic port mapping**.

- ECS Tasks can have IAM roles to access other AWS services.
- Security groups operate at the instance level (not at individual task).

---

Quick Recap:

ECR -- Docker Hub for AWS
-------------------------

- ECR is integrated with IAM.
- **Remember the two different Docker login command for CLIv1 and CLIv2!**
- **If an EC2 instance cannot pull an image, it is likely an IAM issue**.

Fargate
-------

- Serverless.
- AWS provisions containers and assigns them ENIs.
- Fargate containers are provisioned by the container spec (CPU and RAM).
- Fargate tasks ca have IAM Roles to execute actions on other AWS services.

ECS
---

- ECS integrates with CloudWatch Logs.
    - Setup logging at the Task Definitions.
    - Each container will have a different log-stream.
    - **EC2 Instance Profile needs to have the correct IAM permissions**.

- Use IAM Task Role for the tasks.
- Different **Task Placement Strategies** : binpack, random, and spread.
- Service Auto-Scaling : target-tracking, step-scaling, or scheduled.
- Cluster Auto-Scaling by Capacity Providers.



