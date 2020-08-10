AWS Messaging Services: SQS, SNS and Kinesis
============================================

As we start to deploy multiple applications, we need middlware to facilitate
communication between each other. There are two patterns for application
communication:

1) SYNC (APP to APP)

- direct connection.
- can be problematic in case of sudden traffic spikes.

2) ASYNC (event driven)

- requests are first placed in the queue.
- decouples the applications to their roles.
- SQS : queue model.
- SNS : pub/sub model.
- Kinesis : realtime streaming model.

Simple Queue Service (SQS)
--------------------------

SQS is a queue service that holds the messages sent by one or more producers;
and on the other side, there will be consumers who poll from the queue to
retrieve the message.

SQS -- Standard Queue
---------------------

- SQS is the one of oldest AWS services (~10 years old).
- It is a fully managed service.
- It can scale from 1 to 10,000 messages per second.
- Its default message retention period is 4 days (max 14 days).
- Unlimited number of messages maybe placed within the queue.
- Low latency service (~10 ms on publish and receive).
- Horizontal scaling in terms of the number of consumers.
- Can result in a duplicated message occasionally.
- It is best effort ordering (can be out of order).
- **Limits 256 KB per message**.

SQS -- Delay Queue
------------------

- Delays a message such that consumers would not be able to see the message
  right away.
- Default is 0 second, and can be set at the queue level.
- Override the default setting with `DelaySeconds` parameter.

SQS -- Producing Messages
-------------------------

Message is composed of

- Message Body : content less than 256 KB.
- Optional Metadata : message attributes in `Name:Type:Value`.
- Optional Delay Delivery.

When the message is received from the queue, it contains

- Message identifier
- MD5 hash of the body

SQS -- Consuming Messages
-------------------------

Consumers poll SQS queue for messages (can receive upto 10 at a time) and they
are required to process the message within the **visibility timeout**.

Once the message is processed, consumer can send request to delete the message
in the queue using message Id and recipt handle.

SQS -- Visibility Timeout
-------------------------

A mechanism that will block other consumers from realizing that the message is
there while it is being serviced by other consumer. The message will stay
_invisible_ to other consumers for a set period of time. The timeout ranges
from 0 second to 12 hours (default is set to 30 seconds).

If it is set too high and the consumer fails to process the message, the other
consumers will have to wait until the timeout finishes. 

Likewise, if it is set too low, then consumers may not get to finish processing
the message and it can be processed more than once by other consumers.

`ChangeMessageVisibility` API is available to change the visibility period
while processing a message; thus, if consumer requires more time, it can
request to increase the timeout (or shorten it if done quicker).

SQS -- Dead-Letter Queue (DLQ)
------------------------------

If a consumer fails to process a message within the visibility timeout, then
the message will go back to the queue. To avoid having a message sent back
repeatedly, we can set a threshold of how many times that the message can go
back - it is called **redrive policy**.

After the threshold is exceeded, the message goes to the Dead Letter Queue if
set. To enable this, create a SQS first and designate as a DLQ.

SQS -- Long Polling
-------------------

A consumer can wait on an empty queue until a message arrives instead of
constantly having to call poll API to see if there is a message or not - this
is called Long Polling.

This decreases the number of API calls and increases the efficiency in general.
The long-polling wait time can be set between 1 to 20 seconds. It is enabled at
the queue level or at the API level using `WaitTimeSeconds`.

SQS -- Using CLI
----------------

Basic format:

        $ aws sqs <command>

List all queue within a region:

        $ aws sqs list-queues --region us-west-1

Send message to the queue:

        $ aws sqs send-message --queue-url https://queue.amazonaws.com/... --region us-west-1 --message-body "hi!"

SQS -- FIFO Qeueus
------------------

This feature is not available in all regions.

The name of the queue must end in `.fifo`.

Lower throughput (upto 3,000 per second with batching; 300 without).

**Messages are processedin the order by the consumer**.

Deduplication (do not send same message twice):

- provide a `MessageDeduplicatedId` with your message.
- deduplication interval is 5 mins; if duplicated message is detected within
  the interval, it will be discarded automaticall.y
- content based duplication : `MessageDeduplicationId` is generated as
  a SHA-256 of the message body.

Sequencing:

- to ensure strict ordering between messages, specify a `MessageGroupId`.
- messages with differnt Group ID may be received _out of order_.
- i.e. to order messages for a user, use user's ID as a Group ID.
- messages with the same Group ID are delievered to one consumer at a time.

SQS -- Extended Client
----------------------

To send a larger message above 256 KB, we can use the SQS Extended Client (Java
lib) where this uses a S3 bucket as a medium. The message is first placed in
the S3 bucket, and sends the metadta message to the SQS Queue. When a consumer
processes the message, it now knows that there is a message in the S3 bucket.

SQS -- Security
---------------

Encryption during flight can be done using HTTPS.

Can enable SSE-KMS.

- Option to use CMK (Customer Master Key).
- Option to set the data key reuse period (1 min to 24 hours).
    - Lower reuse period == increase API calls to KMS.
    - SSE only encrypted the body; not metadata.
- IAM Policy must allow usage of SQS.
- SQS queue access policy offers fine grain control over range of IPs.

SQS -- APIs
-----------

- `CreateQueue`
- `DeleteQueue`
- `PurgeQueue`
- `SendMessage`
- `ReceiveMessage`
- `DeleteMessage`

- Batch APIs are available for Send, Delete, ChangeMessageAvailabiliy to save
  costs.

---

AWS Simple Notification Service (SNS)
-------------------------------------

SNS is used for publishing a message to multiple subscribers to a "topic". Each
subscriber to the tpoic will get all the messages (able to filter the
messages).

Upto 10,000,000 subscriptions per topic and 1,000,000 topic limit.

Subscribers can be any of following:

- SQS
- HTTP/HTTPS
- Lambda functions
- E-mails
- SNS messages
- Mobile notifications

SNS -- Integrations
-------------------

Some services can send directly to SNS for notification:

- CloudWatch Alarms
- ASG notifications
- S3 bucket events
- CloudFormation
- etc

SNS -- Publishing
-----------------

Topic Publishing (within AWS server using SDK):

- Create a topic
- Create subscription(s)
- Publish to the topic

Direct Publishing (for mobile apps SDK)

- Create a platform application
- Create a platform endpoint
- Publish to the platform endpoint
- Works with Google GCM, Apple APNS, Amazon ADM, etc

SNS -- Fan-Out Pattern
----------------------

One common pattern with SNS and SQS is that publish once to a SNS topic, then
have it received by many SQS queues. This decouples the services and messages
will not be lost. Also, we can scale and attach more queues to the topic if the
need arises. SQS offers delayed processing and retries.

---

AWS Kinesis
-----------

**Kinesis** is a managed laternative to Apache Kafka which is used for

- application logs, metrics, IoT, and clickstreams.
- real-time big data.
- streaming processing frameworks (Spark, NiFi, etc).

The data is automatically replicated to three AZs.

**Kinesis Stream** is a low latency streaming ingest at scale.

**Kinesis Analytics** performs real-time analytics on streams using SQL.

**Kinesis Firehose** loads streams into S3, Redshift, and ElasticSearch.

Kinesis -- Example
------------------

Multiple services will put their data onto Kinesis Streams. Then, Kinesis
Analytics will perform analysis on the stream. Once finished, the data is
stored elsewhere using Kinesis Firehose (to S3 bucket, Redshift, etc).

Kinesis Streams
---------------

- Streams are divided in order **Shards and Partitions**.
- To scale up, we would increase the number of shards.
- Data retention period is by default 1 day (up to 7 days).
- Think of Kinesis Streams as a highway, we do not want to have stale data.
- It can reprocess and replay data.
- Multiple applications can consume from the same stream.
- Once the data is inserted into Kinesis, it cannot be deleted.

Kinesis Streams -- Shards
-------------------------

- Single stream is made of many shards
- 1 MB/s or 1,000 messages/s at WRITE per shard.
- 2 MB/s READ per shard.
- **Billing is per shard provisioned** (unlimited shards).
- Batching is available.
- The number of shards are flexible (can merge or reshard).
- **Records are ordered per shard basis**.

Kinesis API -- Put Records
--------------------------

`PutRecord` API + Partition key gets hashed.

Every data is attached with Message Key which is hashed to determine the shard
ID. Thus, same key always goes to the same partition.

**Choose a partition key that is highly distributed**; this helps to prevent
hot partition. If the key is not well evenly distributed, a single shard can
get overwhelmed. Here are few examples:

- user-id is great for many users.
- county-id can be problematic if many users access the application from
  a single region.

Use batching with `PutRecord` to reduce the cost and increase throughput.

`ProvisionedThroughputExceeded` error will be raised when we go over the
limits.

Kinesis API -- Exceptions
-------------------------

`ProvisionedThroughputExceeded` Exception:

- Happens when sending more data (exceeding MB/s or TPS for any shard).
- Most likely due to hot sharding cause by a bad partition key used.

Solutions:

- Retry with exponential back-off strategy.
- Scale the number of shards.
- Ensure that the partition key is a good one.

Kinesis API -- Consumers
------------------------

Can use a normal consumer (CLI, SDK, etc).

Can use Kinesis Client Library which uses DynamoDB to checkpoint offsets and
track otherworks. It also shares the work amongst the shards.

Kinesis API -- Client Library
-----------------------------

KCL is a Java library that helps read record from a Kinesis Streams with
distributed applications sharing the read workload.

**Rule** : each shard has to be read by only one KCL instance at a time.

i.e. 4 Shards == 4 Maximum KCL instances.

Progress is checkpointed into DynamoDB (and it will require IAM permissions to
access DynamoDB).

KCL can run on EC2, EB, On-Premises applications.

**Records are read in order at the shard level**.

Example of KCL with 4 shards:

Suppose that we have two EC2 instances, each running a KCL app and there are
four shards available. First tow hsards are being read by first instance and
rest are used by another. KCL apps will checkpoint their progress to DynamoDB.

To scale KCL apps, we can have two more EC2 instances for max four EC2
instances each assigned a shard of its own. But past this point, we need to
reshard and increase the number of hsards.

Kinesis -- Security
-------------------

- Control access and authorization using IAM policies.
- Encryption in-flight using HTTPS endpoints.
- Encryption at rest using KMS.
- Possibility to encrypt and decrypt data client-side.
- VPC Endpoints available for Kinesis to access within the private VPC.

Kinesis Data Analytics
----------------------

- Performs real-time analytics on Kinesis Streams using SQL.
- It features:
    - Auto-scaling.
    - Managed, thus no servers to provision.
    - Continuous, real-time.
- Pay-per-usage.

Kinesis FireHose
----------------

- Fully maanged service.
- Near real-time (~ 60 seconds latency).
- Loads data into Redshift, S3, ElasticSearch, Splunk, ...
- Auto-scaling.
- Support many data format but pay for conversion.
- Pay-per amount of data.

---

SQS vs SNS vs Kinesis
---------------------

**SQS**

- Consumers pull the data from the queue.
- Data is deleted after being consumed.
- Can have as many workers as possible.
- No ned to provision throughput.
- No ordering is guranteed (best-effort) except FIFO queues.
- Individual mesage delay capability.

**SNS**

- Publish data to many subscribers to a topic.
- Data is not persisited and lost if not delivered..
- No need to provision throughput.
- Integrates with SQS for common fan-out pattern.

**Kinesis**

- Consumers pull the data from the stream.
- Can have as many consumers as possbile.
- Meant to be used for real-time big data, analytics and ETL.
- Ordering is done at shard-level.
- **Mustprovision throughput (specifiy the number of shards)**.

Ordering Data -- Kinesis
------------------------

Imagine there are 100 trucks which are identified by their truck-id. And these
trucks send their GPS location regularly to AWS. How do we send that data to
Kinesis?

- Send using their truck-id as the partition key; same partition key means that
  they will always be placed in the same shard.
- i.e. 3 shards are available. Then, partition key will be hashed and mapped to
  one of these shards. Any subsequent data with matching partition key will be
  placed in the same shard.

Ordering Data -- SQS
--------------------

SQS Standard queue does not gurantee any ordering, it is only possible with
FIFO queue. And within the FIFO queue, we should use GroupID. In order to scale
the number of consumers, but you want the messages to be grouped when they are
related to each other - we use GroupID. This is like partition key for Kinesis.
A single GroupID can be mapped to a single consumer.

