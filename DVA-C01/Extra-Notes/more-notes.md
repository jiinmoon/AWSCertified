Extra Notes
===========

More study materials for the questions that I have gotten wrong in the practice
exams.




EC2
---

Basic monitoring on CLoudWatch every 5 min period.

Detailed monitoring for every 1 min period.

For more granular monitoring, use custom metrics.

---

Instance metadata is data about EC2 instance that you can use to configure or
manage the running instance; so, you do not have to use CLI or console. This is
useful when writing scripts to run within the running instance.

Make query to following URL: `http://169.254.169.254/latest/meta-data/`.

---

Using roles to grant permissions to apps that run on EC2 instances requires
a bit of extra configuration since it is abstracted from AWS by the virtualized
OS. This is where we create an **instance profile** that is attached to the
instance. This contains the role and can provide the role's temp credentials to
an app that runs on the instance.

i.e. admin can create role that grant bucket access to instance profile. The
instance with the role is launched and app retrieves role credentials from the
instance and uses the bucket.

---







SAM
---

SAM is an open-source framework for building serverless apps. It provides
shorthand syntax to express functions, APIs, databases, and event source
mappings using YAML.

**SAM** uses cloudformation underneath; even the CLI commands will do exactly same
things:

`sam package` == `aws cloudformation package`

`sam deploy` == `aws cloudformation deploy`

CloudFormation
--------------

CloudFormation provides Python helper scripts to install software and start
services on an EC2 instance that created as part of your stack:

- `cfn-init` : use to retrieve and interpret resource metadata, install
  packages, create files, and start services.

- `cfn-signal` : use to signal with a CreationPolicy or WaitCondition, so you
  can sync other resources in the stack when the prerequisite resource or app
  is ready.

- `cfn-get-metadata`

- `cfn-hup` : use to check for updates to metadata and execute custom hooks
  when changes are detected.

These can be called directly from your template.


Lambda
------

**Lambda** function can proces records in DynamoDB Streams stream whenever
a changes has occured on the table.

1. Define an event source mapping to tell Lambda to send records from your
   stream to a function.

2. Attach a managed policy `AWSLambdaDynamoDBExecutionRole`.

---

Lambda natively supports many langauges, but also provides a **Runtime API**
which allows you to use any additional programming languages. You can include
a runtime in your function's deployment package in the form of an executable
file named `bootstrap`.

---

Lambda execution context provides 512 MB of additional disk space in `/tmp`.

---

Best practices:

- Separate the Lambda handler from your core logic.

- Take advantage of Execution Context reuse to improve performance.

- Use AWS Lambda Environment Variables to pass operational parameters to your
  function.

- Control the dependencies in your function's deployment package.

- Minimize your deployment package size to its runtime necessities.

- Reduce the time it takes Lambda to unpack deployment packages.

- _Avoid recursive code_.

---

The unit of scale for Lambda is a **concurrent execution**. And you can set
limits on both account level and the function level.

**Concurrent executions** refers to the number of executions of your function
code that are happeneing at any given time.

When a function process events from event sources that aren't poll-based, each
published event is a unit of work, in parallel, up to your account limits;
hence, # of invocations decide the concurrency.

If you set the concurrent execution limit for a function, the value is deducted
from the unreserved concurrency pool. i.e. if account's concurrent execution
limit is 1000, and have 10 functions, you can specify a limit on one function
at 200 and another at 100. The remaining 700 will be shared among the other 8.

Lambda will kepp the unreserved concurrency pool at a minimum of 100 concurrent
executions, so that function that do not have specific limits et can still
process requests. In practice, if your total account limit is 1000, you are
limited to allocating 900 to individual functions.

---

Lambda authorizer is an API Gateway feature that uses a Lambda function to
control access to your API. When API request is made, API Gateway calls Lambda
authorizer which takes the caller's identity as input and returns an IAM policy
as output. There are two types:

- **Token-based** using bearer's token (JSON Web Token or OAuth).

- **Request parameter-based** using combination of headers, query string
  parameters, stageVariables, and $context variables.

---

API Gateway's API integration type with Lambda: proxy or custom integration.

For an AWS service action, there is only non-proxy type.

---

To allow function to access the resources inside private VPC, you must provide
additional VPC-specific configuration information that includes VPC subnet IDs
and security group IDs. Lambda will use this infromation to set up an ENIs that
enables your function to connect securely to other resources in VPC.

---



S3
--

**S3 Select** use SQL statements to filter the contents of S3 objects and
retrieve just the subset of data required. This is great for reducing the
amount of data that S3 trnasfers, costs and latency.

---

S3 and CloudFront can be used to deliver static content and decrease the
end-user latency by cahcing the content close to your end-users.

---

All objects are **private by default**. You can share objects with others by
creating pre-signed URL to grant time-limited permission to download.



Cognito
-------

Can add MFA to a user pool to protect the identity of your users.

You can also choose to use SMS text messages, or time-based one-time passwords
as second factors in signing in your users.

---

**Cognito Sync** is an AWS service and client libary that enables cross-device
syncing of application-related user data. It can be used to sync user profile
data across mobile devices and the web without requiring your own bakcend.




DynamoDB
--------

GSI is an index with a partition key and a sort key that can be different from
base table. You can create upto 20 GSIs per table by default. GSI inherit the
read/write capacity mode from the base table..

- Queries or scans on this index consume capacity units from the index (not
  from base).
- Queries on this index support **eventual consistency only**.

---

LSI notes:

- Queries on LSI can choose either eventual or strong consistency.
- Queries or scans consume RCU from base table.
- For each partition key value, the toal size of all indexed items must be 10
  GB or less.

---

_Optimistic locking_ is a strategy to ensure tha tthe client-side item that you
are updating (or deleting) is the same as the item in DynamoDB. Your DB will be
protected from being overwritten by the writes of other. Each item will have an
attribute that acts as a version number. If you retrieve an item from a table,
the app records the version number of that item. Note that:

- DynamoDB global tables use a "last writer wins" between concurrent updates
  (locking strategy will not work).

- DynamoDBMapper transactional operations do not support optimistic locking.

---

Design application such that it will perform its queries uniformly across the
logical partition keys in the Table and its secondary indicies. 

---

`ReturnConsumedCapacity` returns WCU consumed by the write operations:

- `TOTAL` returns the total number of WCU used.

- `INDEXES` returns the total number of WCU used, with subtotals for the table
  and any secondary indicies that were affected by the operation.

- `NONE` nil.

---



Security
========

**AWS Secrets Manager** is an AWS service that manages database credentials,
passwords, third-party API keys, and arbitary text.

It can perform automatic rotation of credentials.

---

When encrypting data, you have to protect not just your data but also the key
that is used to encrypt your data (DEK). _Envelope encryption_ is a scheme that
encrypting plaintext data with a data key, and then encrypting the data key
under another key.

To encrypt:

1. Use `GenerateDataKey` to get a DEK.
2. Use plaintext DEK to encrypt the data. Throw away the plaintext DEK and
   store the encrypted DEK (which received in step 1) with the encrypted data.

To decrypt:

1. Request `Decrypt` on the encrypted DEK.
2. Use received plaintext DEK to decrypt the encrypted data.

---



Monitoring
==========

CloudWatch Events can be used to detect and react to chages in the state of
a pipeline, stage, or action. 

---

You can send trace data to X-Ray in the form of segment document which is
a JSON formatted string that contians information about the work. It can be
upto 64 kB and contain a whole segment with subsegments. Use
`PutTraceSegments`. Alternatively, you can send segments and subsegmentsto an
X-Ray daemon, which will buffer and upload to X-Ray API in batches.

---

API Gateway executions can be monitored with CloudWatch; the stats are recorded
for two weeks. By default, the metrics are sent to CloudWatch in 1 min periods.

It can monitor **CacheHitCount** and **CacheMissCount** iff API caching is
enabled.

---

The compute resources running your app logic send data about their work as
**segments** to X-Ray which contains resource's name, details about request and
work done.

A subset of segment fields are indexed by X-Ray for use with **filter
expressions**. You can search for segments associated with specific information
in console or `GetRaceSummaries` API.







CICD
====

CodeBuild cannot trigger a Lambda function directly.

---







Else
====

Mostly not as important but may come up.

---

**AWS Simple Work Flow (SWF)** can use _markers_ to record events in the
workflow execution history for application specific purposes - useful when you
want to record custom infromation to help implement decider logic. (i.e. count
the number of loops).

---

**AWS Organizations** has the consolidated billing feature which enables you to
consolidate payment for multiple AWS accounts or multiple AISPL accounts.

Each organization in AWS Organizations has a master account that pays the
charges for all the member accounts. If you have access to the master account,
you can see a combined view of the AWS charges that are incurrsed by the member
accounts. You can get a cost report for each memeber account.

---

**AWS WAF** is a web application firewall that monitors HTTP and HTTPS requests
that are forwarded to an API Gateway API, CLoudFront or Application Load
Balancer.
