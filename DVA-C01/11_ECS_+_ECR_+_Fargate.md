ECS, ECR & Fargate - Docker in AWS
==================================


AWS ECS - Essentials
--------------------

- ECS is a new section in exam; can be tricky!
- Know docker tech.
- ECS: Cluster? Services? Tasks? Tasks Definition?
- ECR
- Fargate



What is Docker?
---------------

- **Docker** is a dev platform to deply applications.
- Apps are packed within _containers_, which makes sure that the app is run
  exactly the same way regardless of underlying architectures or systems.
    - it will run on any machine;
    - no compatibility issues;
    - predictable behavior.

- i.e. on any server (such as EC2 instance) we may run different containers
  that can run different environments (Java app + Node app + MySQL...) and make
  them able to communicate one another.

- thus, it is indefinitely scalable as we only need to run more containers if
  the workload increases.

**Docker Images**

- they are stored in **Docker Repositories**.
- main Public repo is [Docker Hub](https://hub.docker.com/).
    - you can find base images for many techs and OS.

- Private: **Amazon ECR (Elastic Container Registry)**

Container vs VM
---------------

- With containers, resources are shared with the host.

- VMs runs on top of Hypervisor which sits on top of Host OS; each VM is
  a guest OS.

- Containers are managed by Docker Daemon which is a background app. No
  hypervisors or guest OS.

Docker Containers Management
----------------------------

- We need a container management platform. For AWS we have following:
    - ECS
    - Fargate : serverless platform
    - EKS : Amazon's managed Kubernetes

---

ECS Clusters Overview
---------------------

- ECS Clusters are logical grouping of EC2 instances.
- EC2 instances run the ECS agent (Docker container).
- ECS agents registers the instance to the ECS cluster;
- EC2 instances run a spcial AMI made specifically for ECS.

- When ECS Cluster is created, it is an Auto Scaling Group of EC2 instance.
    - security groups, roles...

ECS Tasks Definitions
---------------------

- **Task Definitions** are metadata in JSON form to instruct ECS how to run the
  Docker Container.

- It contains information such as:
    - image name.
    - port binding for container and host.
    - memory and CPU required.
    - environment variables.
    - networking information.

- i.e. on a EC2 instance, we will have a docker container running ECS Agent.
  And we will run another container running apache web server, port mapped to
  our host EC2 instance's open port to serve web traffics.

- **Task** gets an optional IAM roles - common exam question to troubleshoot
  why the container cannot access or do anything in AWS.

ECS Service
-----------

- ECS Service help to define how many tasks should be run and how they should
  be run.
- They ensure that the number of tasks desired is running across our fleet of
  EC2 instances.
- They can be linked to ELB / NLB / ALB if needed.

- Service is created at the level of Cluster.

ECS Service w/ Load Balancer
----------------------------

- Suppose we run several EC2 instances in the cluster, and we want to run
  several web servers.

- Problem, there is only one host port per EC2 instance that container port can
  occupy - meaning that if one container port 80 is mapped to host port 8080,
  another container cannot use the same host port 8080.

- Instead of mapping a static host port, we will assign a random host port to
  the containers. But, now how do we route our traffic to these different port
  numbers?

- This is where we use load balancer to route the traffics to different host
  ports that mapped to different containers running the web servers.

- This is enabled by **Dynamic port forwarding**.

- Update task definition to configure container definition: this time, leave
  the port mapping empty; this will make it random.

- To attach a load balancer, it can only be done during service creation; we
  cannot attach to already existing services without it.

- Does not work with Classic Load Balancer.

- We need to configure security group to allow load balancer to access range of
  ports.

---

ECR
---

- We may want to manage and maintain our own docker images.
- **ECR** is a private docker image repository in AWS.
- Access to ECR is controlled via IAM (permission errors? policy).

- How to authenticate into ECR using CLI?
    - **AWS CLI v1 login command**: `$(aws ecr get-login --no-include-email --region eu-west-1)`
    - `$` is required since we need the output from ecr get-login command!
    - **AWS CLI v2 login command (pipes)**: `aws ecr get-login-password
      --region eu-west-1 | docker-login --username AWS --password-stdin
      1234567890.dkr.ecr.eu-west-1.amazonaws.com`
    - first command greps password and piped into `docker login`.

- Docker push & pull:
    - `docker push 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`
    - `docker pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest`

Fargate
-------

- Thus far, when we launch an ECS Cluster, we have to create our EC2 instances.
- If we need to scale, we need to add more EC2 instances.

- Fargate is serverless; we do not provision EC2 instances anymore.
- Create task definitions and AWS will manage for us.
- To scale, just increase the number of tasks.

- otherwise, pretty identical.

---

ECS IAM Roles Deep Dive
-----------------------

- How IAM Roles work with ECS?
- **EC2 Instance Profile** is attached to the running EC2 instance that is
  running docker container with ECS Agent.
    - This is used by ECS Agent to make API calls to the ECS service.
    - Used to send container logs to CloudWatch Logs
    - Pull Docker image from ECR.

- For tasks inside the instance that is under EC2 Agent's management, it
  requires its own role to interact with other AWS.
- This is **ECS Task Role**.
    - allows each task to have a specific role.
    - i.e. for task A, attach a role A that only grants this task to interact
      with S3 bucket.
    - So, use different roles for the different ECS services.
    - Task role is defined in **Task Definitions**.

---

ECS Task Placement and Constraints
----------------------------------

- when a task of type EC2 is launched, ECS must determine where to place it,
  with the contraints of CPU, memory and available port.

- i.e. suppose we already have a cluster of 3 EC2 instances which are running
  various different tasks. When ECS Service wants to add the new container,
  where would it place it?

- similarly, when a service scales in, ECS needs to determine which task to
  terminate.

- This is where **Task placement strategy** and **Task placement contraints**
  comes into play.

- Note: **This is not for Fargate which doesn't have EC2 to manage**.

ECS Task Placement Process
--------------------------

- Task placement strategies are a _best effort_.

- When ECS places tasks, it uses the following process to select container
  instances:
    1. identify instances that satisfy CPU, memory, and port requirements in
       the task definition.
    2. identify the instances that satisfy the task placement contraints.
    3. identify the instances that satisfy the task placement strategies.
    4. select the instance for task placment.

ECS Task Placement Strategies
-----------------------------

**Binpack**

- place tasks based on the least available amount of CPU or memory.
- helps to minimizes the number of instance in use (cost-saving).

```json
"placementStrategy": [
    {
        "field": "memory",
        "type": "binpack"
    }
]
```

- in other words, it will try to fill up EC2 instance with containers. then, if
  it cannot fit anymore, it would spin up another EC2 instance to continue.

**Random**

- No logic; just random placement.

```json
"placementStrategy": [
    {
        "type": "random"
    }
]
```

**Spread**

- suppose we have 3 instances spread across three AZs.
- we will try to place task evenly based on the specified value.
- i.e. `attribute:ecs.availability-zone` will spread tasks across instances on
  multiple zones.

```json
"placementStrategy": [
    {
        "field": "atrribute:ecs.availability-zone",
        "type": "spread"
    }
]
```

- maximizes availability.

- Note that these strategies can mix.

ECS Task Placement Contraints
-----------------------------

- **distinctInstance**: place each task on a different container instance.
- **memberOf**: places task on instances that satisfy an expression.
    - uses the Cluster Query Language.

```json
"placementConstraints": [
    {
        "expression": "attributes.instance-type =~ t2.*",
        "type": "memberOf"
    }
]
```
- above will place tasks only on instances of type t2.\*.

---

ECS -- Service Auto Scaling
---------------------------

- CPU and RAM is tracked in CloudWatch at the ECS service level.
- **Target Tracking** : target a specific average CloudWatch metric.
    - i.e. CPU utilization should be 60% across.
- **Step Scaling** : scale based on CloudWatch alarms.
- **Schedueld Scaling** : based on predictable changes.

- much similar to ASG; in fact, it is same.

- **ECS Service Scaling (task level) != EC2 Auto Scaling (instance level)**.
- in other words, scaling up ECS Service does not mean EC2 instances are
  scaling up as well.

- Fargate Auto Scaling is much easier to setup due to serverless architecture.

ECS -- Cluster Capacity Provider
--------------------------------

- used in association with a cluster to determine the infrastructure that
  a task runs on.
