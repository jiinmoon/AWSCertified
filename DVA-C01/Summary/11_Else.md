Other Exam-Tips
===============

SQS Delay Queues
----------------

You can change a queue to postpoon delivery of the new messages. By default,
0 seconds and max up to 900 seconds.

This is useful for distributed applications that require more time in
processing.

Sending Large Files over SQS
----------------------------

SQS has a size limit of 256 KB.

SQS Extended Client Library for Java allows you to use S3 as a medium to upload
large files.

You first upload the files to S3 via Extended Client, then the metadata will be
inserted to the queue. Consumer will receive the message, and retrieve the
original file within the S3.

AWS CLI Pagination
------------------

Pagination controls the number of items included in the output from CLI command
which returns a page of 1,000 items by default. Sometimes, this default is too
high and may get timeout serrors. We would need to decrease the max size to
fetch in smaller sizes at a time (but will make more calls).

**This still returns all the data; it is just how many API calls are being made
in the back**.

Use `--page-size` in CLI to request a smaller number of items with each API
calls.

IAM Policy Simulator
--------------------

It tests the IAM policies before applying them in production.

<https://policysim.aws.amazon.com>

Kinesis Shards and Consumers
----------------------------

**Capacity of Kinesis stream is the total capaciy of shards**. Meaning that you
can have as many number of consumers as the number of shards provisioned.

5 Read transactions per second (up to 2 MB per second).

1,000 Wrtie records per second (up to 1 mB per second).

If the data throughput increases, resharding is needed to maintain the flow
(increase the number of shards).

Consumers use Kinesis Client Library (KCL) and it tracks the number of shards
- automatically detecting new shards.

i.e. One consumer can read from 4 shards, but within the consumer instance,
there will be 4 record processors managed under KCL. If we increase the number
of consumers to two, then KCL adjusts the number of record processors such that
there will be 2 record processors on each instance to read from 2 shards each.

This implies that you do not need multiple instances to match the number of
shards to process if it can handle the load. Should use CPU utilization metric
to decide the number of consumer instances.

Thus, it is a good idea to set an ASG to scale the number of consumers based on
the CPU loads.

Lambda Concurrent Executions Limit
----------------------------------

**Tip: there would not be many questions on limits.** But you should be aware
of Lambda concurrent limit.

This is a safety that limits the total number of Lambda concurrent executions
**across all functions in a given region per account**.

Default is 1,000 per region; going over the limit returns
`TooManyRequestsException` and HTTP 429.

You can request a limit increase to AWS Support Centre.

Lambda Versions
---------------

The current Lambda function that is being worked on is at $LATEST.

Can create multiple versions of the function code and use aliases to reference
the version. Think of alias as a pointer to a specific version of function.

Lambda and VPC
--------------

Lambda requires permissions to access AWS resources that are residing within
a private VPC. To do so, we need to allow the function to connect to the
private subnet; and it requires following VPC Configuration informations:

- Private Subnet ID
- Security group ID

Lambda will set up ENIs using these informations using an available IPs from
the private subnet.

From CLI, configure VPC information with `--vpc-config`.

X-Ray Configuration
-------------------

Instances run X-Ray SDK to send the data to X-Ray Daemon running in the
background; then, daemon forwars the informationto the X-Ray.

On premises and EC2 insances will need to install the X-Ray daemon.

Elastic Beanstalk will need X-Ray daemon installed inside the EC2 instances.

ECS will require X-Ray daemon on its own Docker container in ECS cluster.

**Annotations** are for additional informations. These are **Key-value pairs**
that you can index easily later in the console.

Docker and EB
-------------

EB does support Docker containers.

It runs in either Single Docker Container or Multiple Docker Containers.

In Single Docker Container, the EB will provision an EC2 instance to run the
Docker container. In Multiple Docker Containers, ECS cluster is created and
deply multiple Docker containers on each instance.

To Deploy, we upload .zip file of code directly or a public S3 bucket.

---

Additional Resources
--------------------

- Official AWS FAQs
    - <https://aws.amazon.com/faqs/>
    - Serverless : Lambda, API Gateway, DynamoDB, ElastiCache, S3
    - Dev Tools : CodeCommit, CodePipeline, CodeDeploy, CodeBuild
    - Secuirty : IAM, Cognito, KMS
    - Automation & Monitoring : EB, CloudFormation, CloudWatch, X-Ray
    - Containers : ECS, ECR
    - Messaging & Streaming : SQS, SNS, Kinesis


- Whitepapers
    - Practicing CICD on AWS.
    - Introduction to DevOps on AWS.
    - Blue/Green Deployments on AWS.
    - Serverless Architectures with Lambda.
    - Docker on AWS: Running Containres in the Cloud.
    - Running Containerized Microservices on AWS.
    - Optimizing Enterprise Economics with Serverless Architectures.
    - AWS Security Best Practices.

