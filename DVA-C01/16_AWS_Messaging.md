AWS Integration & Messaging: SQS, SNS & Kinesis
===============================================

As we start deploying multiple applications, they will require middleware to
facilitate communication between each other.

There are two patterns of application communication:

1) Synchronous communications (app to app) 

- direct connection
- This can be problematic in case of sudden spikes of traffic
- i.e. need to encode 1000 vids instead of usual 10!

2) Asynchronous communications (event based) 

- requests are 'queue'd`
- decouping the applications
- SQS : queue model
- SNS : pub/sub model
- Kinesis : real-time sreaming model


AWS SQS Queue
-------------

SQS is a queue services that holds the messages sent by one or more
_producers_. On the other side, there will be _consumers_ that will continously
poll from the queue to retrieve the messages.

AWS SQS -- Standard Queue
-------------------------

- SQS is the one of oldest AWS service (~ 10 years old). 

- It is fully managed service.

- It can scale from 1 message per second to 10000s per second. 

- It has a default retention of messages for 4 days upto maximum 14 days.

- Unlimited # of messages in the queue.

- Low latency service ( < 10 ms on publish and receive ).

- Horizontal scaling in terms of number of consumers.

- Can have duplicate messages (at least once delivery, occasionally).

- Can have out of order messages (best effort ordering).

- Limits 256 KB per message sent.

AWS SQS -- Delay Queue
----------------------

- Delay a message such that _consumers_ won't see it upto 15 mins when it was
  placed.

- Default is 0 second (message is available right away).

- Can set a default at queue level.

- Can override the default using the `DelaySeconds` parameter.

AWS SQS -- Producing Mesagess
-----------------------------

Message is composed of

- Message Body (~256 kb String); content to deliver.
- Optional Metadata; message attributes in `Name:Type:Value`.
- Optional Dely Delivery.

When message is received from queue it contains

- Message identifier
- MD5 Hash of the body

AWS SQS -- Consuming Messages
-----------------------------

Consumers poll SQS for mesages (receives upto 10 messages at a time).

Consumers have to process the message within the visibility timeout.

Once message has been processed, consumer can send request to delete the
message in the queue using message ID and recipt handle.

AWS SQS -- Visibility Timeout
-----------------------------

A mechanism that will block other consumers from realizing that the message is
there while it was polled and being used by a consumer. The message will be
_invisible_ to other consumers for a defined period, **Visibility Timeout**.

Visibility Timeout range can be set from 0 second to 12 hours (default 30
seconds).

If it is set too high (15 mins) and consumer fails to process the message, you
must wait a long time before the message will be visible to the consumers to
process the message again.

If it is set too low (30 secs) and consumer needs more time to process the
message (2 mins), another consumer will receive the message. Thus, the message
will be processed more than once.

`ChangeMessageVisibility` API is available to change the visibility peroid
while processing a mesage; thus, when consumer requires more time, it can
increase it, or when done faster, shorten it.

AWS SQS -- Dead Letter Queue
----------------------------

If a consumer fails to process a message within the Visibility Timeout, the
message will go back to the queue. To avoid having a message sent back
indenfinitely, we can set a threshold of how many times that the message can go
back; it is called _redrive policy.

After the trheshold is exceeded, the message goes into a _dead letter queue
(DLQ)_. To enable this, create a DLQ first, then designate it as such.

Make sure to process the mesages in the DLQ before they expire.

AWS SQS -- Long Polling
-----------------------

When a consumer requests a message from the queue, it can wait for messages to
arrive if the queue is empty - this is called **Long Polling**.

Long Polling is a generally a good strategy since it can decrease the number of
API calls made to SQS while increasing the efficiency and latency of your
application.

The wait time can be between 1 to 20 secs (with 20 seconds being preferable).

Long polling can be enabled at the queue level or at the API level using
`WaitTimeSeconds`.

AWS SQS -- Using CLI
--------------------

`aws sqs <commands>`

list all queues in the region: 

    aws sqs list-queues --region us-west-1

send message to the queue:

    aws sqs send-message -queue-url htps://queue.amazonaws.com/... --region us-west-1 --message-body "Hello"

AWS SQS -- FIFO Queues
----------------------

First-In-First-Out; not available in all regions.

The name of the queue must end in `.fifo`.

Lower throughput (upto 3000 per second with batching, 300 per second withtout).

Messages are processed in the order by the consumer.

Messages are sent exactly once; no per message dealy (ony per queue delay).

Features:

- Deduplication (don't send same message twice)
    - provide a `MessageDeduplicationId` with your meesage.
    - Depulication interval is 5 mins; if duplicate message is detected in
      interval, it will be discarded.
    - Content based duplication: the `MessageDeduplicationId` is generated as
      the SHA-256 of the message body.

- Sequencing
    - To ensure strict ordering between messages, specify a `MessageGroupId`.
    - Messages with different Group ID may be received out of order.
    - i.e. to order messages for a user, use user_id as Group ID.
    - Messages with same group ID are delivered to one consumer at a time.

---

AWS SQS Advanced -- Extended Client
-----------------------------------

To send a larger message above 256KB, we can use the SQS Extended Client (Java
library). This uses S3 bucket - the message is first put into the S3 bucket,
and then sends the metadata message to the SQS Queue. When consumer receives
the metadata message, it knows that there is a larger message waiting on the S3
bucket, and retrieves it.

AWS SQS Security
----------------

Encryption in flight using HTTPS.

Can enable SSE (Server Side Encryption) using KMS.

- option to use CMK (Customer Master Key)
- option to set the data key reuse period (1 min to 24 hours)
    - Lower and KMS API will be called more often.
    - SSE only encrypts the body; not metadata
- IAM policy must allow usage of SQS
- SQS queue access poicy
    - finer grained control over IP
    - Control over the time the requests come in

AWS SQS API
-----------

- `CreateQueue`
- `DeleteQueue`
- `PurgeQueue` : delete all messages in the queue
- `SendMessage`
- `ReceiveMessage`
- `DeleteMessage`

- Batch APIs available for SendMessage, DeleteMessage, ChangeMessageVisibility
  to save on costs.

---

AWS SNS
-------

SNS is used for sending one message to multiple receivers. It uses
_Publish/Subscribe_ model where one service publishes to the SNS Topic, and
multiple services (subscribers) to that topic will be notified.

_event producer_ only sends message to one SNS topic.

As many _event receivers_ can be subscribe to and listen to the SNS topic
notifications.

Each subscriber to the topic will get all the messages; new feature allows them
to filter the messages.

Up to 10000000 subscriptions per topic; 100000 topics limit.

Subscribers can be:

- SQS
- HTTP / HTTPS
- Lambda
- Emails
- SMS messages
- Mobile Notifications

AWS SNS Integration
-------------------

Some services can send directly to SNS for notifications.

- CloudWatch (Alarms)
- Auto Scaling Groups notifications
- S3 (bucket events)
- CloudFormation (state changes...)
- etc...

AWS SNS -- Publishing
---------------------

Topic Publish (within AWS Sever using SDK)

- Create a topic
- Create a subscription (or many)
- Publish to the topic

Direct Publish (for mobile apps SDK)

- Create a platform application
- Create a platform endpoint
- Publish to the platform endpoint
- Works with Google GCM, Apple APNS, Amazon ADM, ...

Common Pattern -- SNS + SQS: Fan Out
------------------------------------

The idea is to push once in SNS, and receive it on many SQS.

This decouples the services; and no messages will be lost this way. Also, we
can attach more receivers later if we needed.

SQS allows for delayed processing as well as retries.

AWS Kinesis Overview
--------------------

**Kinesis** is a managed alternative to Apache Kafka.

It is great for 

- application logs, metrics, IoT, clickstreams.
- _real time_ big data.
- streaming processing frameworks (Spark, NiFi, etc...)
- Data is automatically replicated to 3 AZs

**Kinesis Streams** : low latency streaming ingest at scale

**Kinesis Analytics** : perform real-time analytics on streams using SQL

**Kinesis Firehose** : load streams into S3, Redshift, ElasticSearch

Kinesis in action
-----------------

Multiple services (Click Steams, IoT devices, Metrics & Logs) will put their
data onto Kinesis Streams. Then, Kinesis Analytics will perform analysis. Once
done, the finished data is stored else where using Kinesis Firehose to S3
bucket, Redshift, and so on.

Kinesis Streams Overview
------------------------

- Streams are divided in ordered _Shards / Partitions_.
- To scale up, we would increase shards.
- Date retention is one day by default - upto 7 days.
- Kinesis Streams is a road - you do not want to have a data sit around.
- Ability to reprocess / replay data (as compared to SQS).
- Multiple applications can consume the same stream.

- Once data is inserted in Kinesis, it cannot be deleted (immutability).

Kinesis Streams Shards
----------------------

- Single stream is made of many shards.
- 1 MB/s or 1000 messages/s at write per shard.
- 2 MB/s a read per shard.
- Billing is per shard provisioned, can have as many shards as you want.

- Batching is available or per message calls.

- The number of shards can change over time (reshard or merge).

- **Records are orderd per shard**.

Kinesis API - Put Records
-------------------------

`PutRecord` API + Partition key that gets hashed.

Every data is attached with Message Key which is hashed to determined the shard
id.

Same key goes to th same partition - helps with ordering for a specific key.

Messages ent get a _sequence number_.

**Choose a partition key that is highly distributed**; this helps to prevent
hot partition - if the key was not well distributed, a single shard can get
overwhelmed.

- user_id if many users.
- counry_id can be problematic depending on where the users access; i.e. 90% of
  users can be from one region.

Use batching with `PutRecord` to reduce the cost and increase throughput.

`ProvisionedThroughputExceeded` will occur when go over limits.

AWS CLI, AWS SDK, or producer libraries from various frameworks can be used.

Kinesis API - Exceptions
------------------------

`ProvisionedThroughputExceeded` Exceptions:

- happenes when sending more data (exceeding MB/s or TPS for any shard).
- most likely due to hot sharding cause by bad partition key.

Solutions:

- retries with backoff strategy
- scale the number of shards
- ensure that the partition key is good one (well distributed)

Kinesis API - Consumers
-----------------------

Can use a normal consumer (CLI, SDK, etc).

Can use Kinesis Client Library (in Java, Node, Python, Ruby, .Net):

- KCL uses DynamoDB to checkpoint offsets and track other workers and share the
  work amonst shards.

Kinesis Client Library
----------------------

KCL is a Java library that helps read record from a Kinesis Streams with
distributed appliations sharing the read workload.

**Rule: each shard is to be read by only one KCL instance at a time**.

i.e. 4 shards == max 4 KCL instances

Progress (how much you have read) is checkpointed into DynamoDB (it will
require IAM permissions to access DynamoDB).

KCL can run on EC2, Elastic Beanstalk, On-Premise application.

**Records are read in the order at the shard level**.

Example of KCL with 4 shards:

Suppose that we have two EC2 instances, each running a KCL app and there are
four shards available. First two shards are being read by first instance, and
rest are used by second instance. KCL apps will checkpoint their progress to
the DynamoDB.

To scale KCL apps, we can have two more EC2 instances for maximum of four EC2
instances each assigned their own shards to read from.

Now, to increase the throughput, you must reshard - increase the number of
shards. **Number of shards determine maximum KCL app scale**.

Kinesis Security
----------------

- Control access / authorization using IAM policies.
- Encryption in flight using HTTPS endpoints.
- Encryption at rest using KMS.
- Possibility to encrypt / decrypt data client side.
- VPC Endpoints available for Kinesis to access within the private VPC.

Kinesis Data Analytics
----------------------

- Performs real-time analytics on Kinesis Streams using SQL.
- Kinesis Data Analytics:
    - auto scales.
    - managed: no servers to provision ourselves.
    - continuous: real-time.
- Pay for actual consumption rate.
- Can create streams out of the real-time queries.

Kinesis Firehose
----------------

- Fully managed service - no administration.
- Near real time (~60 seconds latency).
- Load data into Redshift, S3, ElasticSearch, Splunk...
- Auto scaling.
- Support many data format (but pay for conversion).
- Pay for the amount of data going through Firehose.

---

SQS vs SNS vs Kinesis
---------------------

Which one to use?

**SQS**

- Consumer _pull_ data
- Data is deleted after being consumed
- can have as many workers (consumers) as possible
- No need to provision throughput
- No ordering gurantee (except FIFO queues)
- Individual message delay capability

**SNS**

- Push data to many subscribers
- Upto 100000000 subscribers / Upto 100000 topics
- Data is not persisted (lost if not delivered)
- Publish/Subscription model
- No need to provision throughput
- Integrates with SQS for _fan out_ architecture pattern

**Kinesis**

- Consumers _pull_ data
- As many consumers as possible
- possible to replay data
- meant to be used for real-time big data, analytics, and ETL
- ordering at the shard level
- Data expries after X days
- Must provision throughput (specify number of shards)

Ordering data into Kinesis
--------------------------

Suppose 100 trucks that each have ids (truck01, truck02, ...) on the road and
each of them are sending their GPS location regularly into AWS. We want to
consume the data in order for each truck, so that you can track their movements
accurately. How should we send that data to Kinesis?

- send using a _partition key_ value of the truck id; remember that same
  partition key means that it will always be placed in the same shard.

- i.e. 3 shards available; partition key (truck id in this case) will be hashed
  and mapped to one of the shards. any subsequent data with matching partition
  key will be placed in the same shard.

Ordering data into SQS
----------------------

Remember that SQS Standard Queue does not have ordering.

In SQS FIFO Queue, if you do not provide GroupID, messages are consumed in the
order they are sent, with only one consumer.

In order to scale the number of consumers, but you want the messages to be
grouped when they are related to each other - use the GroupID. This is akin to
Partition Key in Kinesis. Single GroupID can be mapped to single consumer.

Kinesis vs SQS Ordering
-----------------------

Assume 100 trucks, 5 Kinesis shards, 1 SQS FIFO Queue.

**Kinesis Data Streams**:

- on average, you will have 20 trucks per shard
- trucks will have their data ordered within each shard
- max amount of consumers in parallel we can have is 5 (1 shard = 1 consumer)
- can receive upto 5 MB/s of data

**SQS FIFO**:

- Only a single SQS FIFO Queue; but we can have 100 GroupIDs.
- This means that you can have 100 Consumers (due to the 100 GroupIDs).
- You have up to 300 messages per second (or 3000 if batching).


