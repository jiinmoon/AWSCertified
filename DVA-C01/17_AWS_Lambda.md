AWS Serverless: Lambda
======================


**Serverless** refers to a new paradigm shift where the developers do not have
to worry about not just the infrastructure, but also the servers as well. All
developers need to do is deploy codes (or functions).

Initially, serverless meants FaaS (Function as a Service).

It was pioneered by AWS Lambda, but it includes anything that's managed as well
(such as databases, messaging, storage, etc...).

Serverless does not mean that there isn't a server - just mean that customers
does not have to manage, or deal with them at all.

Serverless in AWS
-----------------

Suppose users will access static content from S3 bucket, and log-in using AWS
Cognito where identities are stored. API Gateway is used to invoke REST API
- which in turn directed to Lambda for how to process the request, and Lambda
  can use DynamoDB as a database while running.

So, serverless in AWS involves:

- Lambda
- DynamoDB
- Cognito
- API Gateway
- S3
- SNS & SQS
- Kinesis Firehose
- Aurora Serverless
- Step Functions
- Fargate

Why use Lambda
--------------

We can use EC2, virtual servers in the cloud; but it is limited by RAM and CPU
and it is always continously running even when not needed. Scaling these
instances mean that some degree of management and work is required to add or
remove servers.

Lambda does away with the servers; only virtual functions. It is limited by
time - so short executions and run on-demand; it only runs when it is invoked.
Scaling is automated.

Benefits of Lambda
------------------

**Easy Pricing**

- only pay per request and compute time.
- free tier of 1 million AWS Lambda requests and 400,000 GBs of compute time.

**Easy Integration**

- works with many of the existing AWS services.
- works in many different programming languages.
- easily monitored through CloudWatch.
- easy to add more resources to Lambda functions (upto 3GB RAM).

Lambda Language support
-----------------------

- Node.js (JS)
- Python
- Java (Java 8)
- C# (.NET Core)
- Golang
- C# / Powershell
- Ruby
- Custom Runtimes (supported by community i.e. Rust)

- __Docker cannot run on Lambda; should use ECS or Fargate__.

Lambda Integrations -- examples
-------------------------------

API Gateway - used to create REST APIs which invokes Lambda functions.

Kinesis - uses Lambda to do data transformation on fly.

DynamoDB - will be used to create triggers; when db changes, Lambda function is
called.

S3 - Lambda can trigger whenever a bucket event happens.

CloudFront

CloudWatch Events / EventBridge - response to events.

CloudWatch Logs - process logs.

SNS

SQS

Cognito - react to user action (i.e. log-in)

Example: Serverless Thumbnail creation
--------------------------------------

Suppose new image has PUT in S3. This will trigger a Lambda function to create
the thumbnail for uploaded image and push it into the bucket or create metadata
and store it in the DynamoDB.

Example: Serverless cron Jobs
-----------------------------

We could provision EC2 instance and set up cron to do scheduled task. But
instead, we can set up CloudWatch Events/EventBridge rules that will trigger in
set amount of interval to Lambda function o perform schedueld task.

Lambda Pricing: example
-----------------------

Comprehensive pricing is at [here](https://aws.amazon.com/lambda/pricing/).

Pay per __calls__:

- first 1000000 requests are free.
- $0.20 per 1 million requests thereafter.

Pay per __duration__ (in increment of 100ms):

- first 400,000 GB-seconds of compute time per month if FREE
- it equals to 400,000 seconds if function is 1GB RAM
- or equals to 3,200,000 seconds if function is 128 MB RAM
- after FREE period, $ 1.00 per 600,000 GB-seconds.

Lambda -- Synchronous Invocations
---------------------------------

Synchronous: CLI, SDK, API Gateway, Application Load Balancer

- result of the function is returned __right away__.
- error handling must be done at the client side (retries, exponential
  backoffs, etc).

i.e. SDK/CLI would invoke the lambda function to do something, and its response
will be made as soon as possible.

i.e. Client invokes API Gateway; Gateway will proxy the request to the lambda
function and function will return the response. In turn, Gateway will relay
that response back to the client.

It is available with following services:

- User Invoked:

    - Elastic Load Balancing (ALB)
    - API Gateway
    - CloudFront (Lambda@Edge)
    - S3 Batch

- Service Invoked:

    - Amazon Cognito
    - Step Functions

- Other Services:

    - Lex
    - Alexa
    - Kinesis Data Firehose


Lambda -- Synchronous Invocations -- Hands On
---------------------------------------------

Via CLI:

- To list the avaialble functions within a region:

        $ aws lambda list-functions --region eu-west-1

- To invoke the function with payload to deliver:

        $ aws lambda invoke --function-name hello-world --cli-binary-format raw-in-base64-out --payload '{"key1": "value1", "key2: "value2", "key3": "value3" }' --region eu-west-1 response.json

Lambda + ALB
------------

To expose a Lambda function as an HTTP(S) endpoint, we can either use ALB or API
Gateway.

In order to integrate ALB with Lambda, the function must be registered as
a **target group**.

ALB converts HTTP/S request to format understood by Lambda function via HTTP to
JSON. The request payload is converted to JSON format that contains ELB
information.

```json
{
    "requestContext": {
        "elb": {
            "targetGroupArn":
            "arn:aws:elasticloadbalancing:us-west-1:38583341432"
        }
    },
    "httpMethod": "GET",
    "path": "/lambda",
    "queryStringParameters": {
        "query" : "1234"
    },
    "headers": {
        "connection": "keep-alive",
        "user-agent": ...
    ...
    }
}
```

This way, within the lambda function, we can unpack JSON to see what the
request was about.

Similarly, JSON is converted to HTTP format such that ALB will understand.
Following is a sample reponse from Lambda function:

```json
{
    "statusCode" : 200,
    "statusDescription": "200 OK",
    "headers": {
        "Content-Type: "text/html; charsett=utf-8"
    },
    "body": "<h1>Hello World!</h1>"
}
```

**ALB Multi-Header Values** can be enalbed so that the client can send the
query string parameters that have same name, but multiple values.

i.e. `http://xyz.com/path?name=foo&name=bar`

When enabled, HTTP headers and query string parameters that are sent with
multiple values are shown as arrays within the AWS Lambda event and response
objects.

i.e. Lambda function will receive `"queryStringParameters":{"name": ["foo","bar"]}`

AWS Lambda@Edge
---------------

It is a way to enable global AWS Lambda with the CDN using CloudFront. This
will enable for example, request filtering before applications receive them.

**Lambda@edge** deploys alongside with CloudFront CDN so that

- you can build more responsive applications
- do not have to manage servers; lambda is deployed globally
- customize the CDN content
- pay per usage

You can use Lambda to chagne CloudFront requests and responses.

Normally, user will request to CloudFront and CloudFront will look for
requested object in the Origin to return.

Here, you can modifiy following:

- After CloudFront receives a request from a viewer (viewer request).
- Before CloudFront forwards the request to the origin (origin request).
- After CloudFront receives the response from the origin (origin response).
- Before CloudFront forwards the response fto the viewer (viewer response).

This implies that you can generate responses to viewers without ever sending
the reqeust to the origin.

Lambda@Edge -- Global Application
----------------------------------

Suppose we have statically hosted HTML website on S3 bucket. Users visit the
website and makes Dynamic API requests to CloudFront, which forwards the
request to the Lambda@Edge function. Lambda function will then maybe query data
with DynamoDB or etc.

Lambda@Edge -- Use Cases
------------------------

- Website Security and Privacy
- Dynamic Web Application at the Edge
- Search Engine Optimization (SEO)
- Intelligent Route Across Origins and Data Centres
- Bot mitigation at the Edge
- Realtime Image Transformation
- A/B Testing
- User Authentication and Authorization
- User Prioritzation
- User Tracking and Analytics

---

Lambda -- Asynchronous Invocations
----------------------------------

Async used with S3, SNS, CloudWatch Events.

i.e. S3 bucket registers new event (a new file uploaded). This event is queue'd
at **Event Queue** which the Lambda function will read from and process.

Lambda will retry on errors upto 3 times (1 min, 3 min).

Make sure that the processing is **idempotent** (in case of retries, the result
should be same).

If retries occur, **duplicate log entries will be registered in CloudWatch
logs**.

We can define DLQ (dead-leter queue) - via SNS or SQS - for failed processing.
To do so will require correct IAM permissions attached.

Async invocations allow you to speed up the processing if you do not need to
wait for the result (i.e. 1000 files to process).

Lambda -- Async -- Services
---------------------------

- S3
- SNS
- CloudWatch Events / EventBridge
- CodeCommit (Trigger on new branch, new tag, new push, ...)
- CodePipeline (invoke during pipeline; lambda must call back)
- so on...

---

Lambda with CloudWatch Events / EventBridge
-------------------------------------------

CRON or RATE EventBridge rule will trigger a job every 1 hour which sends
request to Lambda function to perform a task.

CodePipeline EventBridge rulle can also trigger a job on state changes to send
request to Lambda function.

Lambda with S3 Event notifications
----------------------------------

Events from S3 can trigger on `S3:ObjectCreated`, `S3:ObjectRemoved`, and so
on.

You may also filter by the object name (i.e. `*.jpg`).

Different types of architectural pattern is available here:

1. Have S3 send notification to SNS; and multiple SQS to handle SNS (fan-out).

2. Have S3 send notification to SQS; and Lambda function reads from queue.

3. Or S3 notification will go directly to Lambda for async invocation. If
   request is bad or process failed, the message can go to DLQ in SQS.

S3 notifications will be delivered in seconds ~ mins.

If two writes are made to a single non-versioned object at the same time, it is
possible that only a single event notification will be sent (so enable
versioning).

---

Lambda - Event Source Mapping
-----------------------------

- Kinesis Data Streams
- SQS & SQS FIFO Queue
- DynamoDB Streams

The records need to be polled from the source; Lambda is triggered sync.

i.e. If the lambda is configured to read from the Kinesis, within Lambda, there
will be **Event Source Mapping** created; and this will be responsible for
polling from the Kinesis and receive the returned batches. Then, it will invoke
the lambda function synchronously with the event batch.

Two categories:

**1) Streams & Lambda (Kinesis & DynamoDB)**

- an event source mapping creates an iterator for each shard, processes
  items in order.
- can read starting from new items, from begging or form timestamp.
- processed items aren't removed from the stream (other consumers can read
  them).
- low traffic stream: can use batch window to accumulate records before
  processing.
- you can process multiple batches in parallel.
    - upto 10 batches per shard.
    - inorder processing is still guaranteed for each partition key.
- Error handling:
    - by default, if function returns an error, entire batch is reprocessed
      until the function succeeds, or the items in the batch expire.
    - to ensure inorder processing, processing for the affected shard is
      paused until the error is resolved.
    - configure the event source mapping to:
        - discard old events
        - restrict the number of retries.
        - split the batch on error (workaround Lambda timeout)
    - discarded events can go to a **Destination**.

**2) SQS & SQS FIFO**

- Event Source Mapping will poll SQS via __Long Polling__. 
- Specify the batch size between 1 to 10 messages.
- Recommended to set the queue visibility timeout to 6x the timeout of your
  Lambda function.
- If DLQ is nededed, we set up on the SQS queue (not on the Lambda since DLQ
  for Lambda is for async invocations only).
- or use the Lambda destinations.

- Lambda supports in-order processing for FIFO queues; scaling up to the number
  of active message groups.

- For standard queues, items won't be i norder.
- Lambda scales up to process a standard queue as quickly as possbile.
- When an error occurs, batches are returned to the queue as individual items
  and might be processed in a different grouping than the original batch.

- sometimes, the event source mapping might receive the same item from the
  queue twice, even without function error.

- Lambda deletes items from queue after successful processing.

- Can configure the source queue to send items to DLQ if they cannot be
  processed.


**Lambda Event Mapper Scaling**

- Kinesis Data Streams & DynamoDB Streams:
    - one lambda invocation per stream shard
    - if paralleized, upto 10 batches processed per shard ismultaneousy

- SQS Standard:
    - lambda adds 60 more instances per minute to scale up
    - upto 1000 batches of messages processed simultaneously

- SQS FIFO:
    - messages with the same **GroupID** will be processed in order
    - lambda function scales to the number of active message groups

Lambda Destinations
-------------------

- Async invocations: can define destinations for successful and failed event
    - SQS
    - SNS
    - Lambda
    - EventBridge bus

- Recommends to use destinations instead of DLQ (both can be used at the same
  time, but destination has more features and targets).

- Event Source Mapping: for discarded event batches
    - SQS
    - SNS

- You may send events to a DLQ directly from SQS.

---

Lambda Execution Role (IAM Role)
--------------------------------

IAM Roles must be attached so that Lambda function has appropriate permissions
to AWS services and resources.

Sample managed policies for Lambda includes:

- `AWSLambdaBasicExecutionRole` : upload logs to CloudWatch
- `AWSLambdaKinesisExecutionRole` : read from Kinesis
- `AWSLambdaDynamoDBExecutionRole` : read from DynamoDB Streams
- `AWSLambdaSQSQueueExecutionRole` : read from SQS
- `AWSLambdaVPCAccessExecutionRole` : deploy Lambda function in VPC
- `AWSXrayDaemonWriteAccess` : upload trace data to X-Ray

When you use an __event source mapping__ to invoke your function, Lambda uses
the execution role to read event data!

Best practice is to __one exection role per one function__.

Lambda Resource Based Policies
------------------------------

Use resource-based policies to give other accounts and AWS services permission
to use your Lambda resources.

This is similar to S3 bucket policies for S3 bucket.

An IAM principal can access Lambda:
    - If the IAM policy attached to the principal authorizes it (i.e. user
      access)
    - Or if the resource-based policy authorizes (i.e. service access)

When an AWS service like Amazon S3 or ALB calls your Lambda function, the
resource-based policy gives it access.

Lambda Environment Variables
----------------------------

Remember that the environment variables are KEY:VALUE pair in String format.

This allows you to adjust how the function behaves without changing the code.

Once env variables are set, it is available to the code; Lambda Services also
adds its own system environment variables as well.

Note that you can actually store secrets since it can be encrypted by KMS (with
the Lambda service key or your own CMK).

i.e. in Python, you may access the env varaibles as follows:

```python
import os

def lambda_handler(event, context): 
    os.getenv("ENVIRONMENT_KEYNAME")
    # code
```

Lambda Logging & Monitoring
---------------------------

**CloudWatch Logs**

- Lambda execution logs are by default always stored in AWS CloudWatch Logs.
- Make sure that the Lambda function has an execution role with an IAM policy
  that authorizes writes to CloudWatch Logs.

**CloudWatch Metrics**

- Lambda metrics involve 
    - invocations, durations, concurrent executions...
    - error count, success rates, throttles
    - async delivery failures
    - iterator age (Kinesis & DynamoDB Streams)

**Lambda Tracing with X-Ray**

- Enable in Lambda conficguration for **Active Tracing**.
- Runs the X-Ray daemon automatically.
- Use AWS X-Ray SDK in Code.
- Ensure that Lambda has the correct IAM Execution Role
    - managed policy is called `AWSXRayDaemonWriteAccess`
- Enviornment variables to communicate with X-Ray
    - `_X_AMZN_TRACE_ID`: contains the tracing header
    - `AWS_XRAY_CONTET_MISSING`: by default, `LOG_ERROR`
    - `AWS_XRAY_DAEMON_ADDRESS`: the X-Ray daemon `IP_ADDRESS:PORT`

Lambda Networking -- Default
----------------------------

By default, the Lambda function is launched outside of your VPC (in
a separated, AWS-owned VPC).

This implies that the Lambda function would not be able to access the resources
within your VPC (RDS, ElastiCache, internal ELB, ...).

Lambda Networking -- VPC
------------------------

To enable Lambda to communicate with private VPC, you must define the VPC ID,
the Subnets and the Security Groups.

Lambda will create an ENI (Elastic Network Interface) in your subnets. 

`AWSLambdaVPCAcessExecutionRole` is required!

i.e. Within a private subnet, ENI will be created with Lambda Security Group
to enable access from Lambda function, and access services within the private
subnet.

**Lambda in VPC with Internet Access**

By default, a lambda function in your VPC does not have internet access. This
is not only true for private subnets, but also public subnets as well. In other
words, **deploying a Lambda function in a public subnet does not give it
a internect access or a public IP**.

To give a Lambda function deployed within the private subnet a internet access,
we need to set up a **NAT Gateway or NAT Instance**.

i.e. Lambda function is working within the private subnet, and in order for it
to access the external API on the web, it must go through the NAT Gateway on
the separate public subnet, which directs the traffic to IGW and to the web.

You can use **VPC endpoints** to privately access AWS services without a NAT.

i.e. Create a VPC Endpoint for DynamoDB such that the Lambda function on
private VPC can securely access it.

Note: CloudWatch logs will work regardless of VPC endpoints or NAT Gateway.

---

Lambda Function Configuration
-----------------------------

**RAM**

- from 128 to 3008MB in 64MB increments
- more RAM you add, more vCPU credits accumulate
- at 1792MB, you get more than one CPU, and need to use multi-threading in your
  code to truly benefit from it.
- _If the application is CPU-bound (very computation heavy), then increase the
  RAM_.

**Timeout**

- default 3 seconds, maximum 900 seconds (15 mins)

Lambda Execution Context
------------------------

The execution context is a temp runtime environment that initializes any
external dependencies of your lambda code.

Great for database connections, HTTP clients, SDK clients and so on.

The execution context is maintained in anticipation of another Lambda functino
invocation - so next function invocation can _reuse_ previous context to save
on execution and setup time.

The execution contexxt includes the `/tmp` directory.

i.e. bad example of not leveraging execution context:

```python
import os

def get_user_handler(event, context):
    DB_URL = os.getenv("DB_URL")
    db_client = db.connect(DB_URL)
    user = db_client.get(usr_id = event["user_id"])
    return user
```

Above code never utilizes how the execution context can be saved; thus it will
try to reconnect to DB everytime the Lambda function is invoked.

i.e. good example of leveraging execution context:

```python
import os

DB_URL = os.getenv("DB_URL")
db_client = db.connect(DB_URL)

def get_user_handler(event, context):
    user = db_client.get(usre_id = event["user_id"])
    return user
```

Notice how the `db_client` is outside of the function scope; it will be
maintained across the invocations.

Lambda functions and `/tmp` space
---------------------------------

If the function while executing require to store temporary space (i.e. download
a big file to process with), then it may need a disk space to perform further
operations. For this reason, the function has access to `/tmp` directory.

`/tmp` has a max size of 512MB.

The directory content remains when the execution context is frozen, providing
transient cache that can be used for multiple invocations (helpful to
_checkpoint_ your work).

For persistent objecs, you must use S3 instead.

Lambda functions and Concurrency + Throttling
---------------------------------------------

Lambda function can have upto 1000 concurrent executions.

We can set a **reserved concurrency** at the function level (= limit). After
invocation over the concurrency limit wil trigger a _Throttle_.

- If synchronous invocation, returns ThrottleError 429
- If async invocation, retry and then go to DLQ

If require more than 1000 concurrent executions, you may open a support ticket
to AWS.

Lambda functions and Concurrency Troubleshooting
------------------------------------------------

If you do not reserve (place limit) concurrency, the following may happen:

Suppose we have many users using ALB to lambda function, few users using
API Gateway to lambda function, and SDK / CLI access to invoke lambda function.

When many users access ALB, the lambda function behind it will scale
automatically upto 1000 concurrent executions - this means that other users
using it behind API Gateway or SDK / CLI will be _throttled_!

In other words: the concurrency limit applies to all functions within the
account; so need to watch out for cases where one function will be taking up
all the reserve.

Concurrency and Async Invocations
---------------------------------

Suppose we have S3 bucket event triggering lambda functions async. Many files
have been uploaded, each triggering functions on their own. If the function
does not have enough concurrency available to process all event,s additional
requests are _throttled_.

But since this is async request, throttling errors (429) and system errors
(5xx) will be dealth by Lambda function to return the event back to the queue
and attempts to run the function again for up to 6 hours.

This retry interval will increase exponentially from 1 second after the first
attempt to a maximum of 5 mins.

Cold Starts and Provisioned Concurrency
---------------------------------------

**Cold Start**

- New instance : code is loaded and code outside the handler run (init)
- If the init size is large (heavy code or dependencies), this process will
  take some time.
- In other words, first request served by the new instance will always have
  higher latency than the rest.

**Provisioned Concurrency**

- Concurrency is allocated before the function is invoked (in advance).
- Thus, cold start is never an issue.
- App Auto Scaling can manage concurrency (schedule or target utilization).

Note:

- read about cold starts in VPC being reduced at
  [here](https://aws.amazon.com/blogs/compute/announcing-imporved-vpc-networking-for-aws-lambda-functions/).

---

Lambda Function Dependencies
----------------------------

If your function depends on external libraries (i.e. AWS X-Ray SDK, Database
Clients, etc) then _you must install the packages alongside your code and zip
it together_.

- for Node.js, use npm & `node_modules` directory
- for Python, `pip --target`
- for Java, include the relevant `.jar` files

Once all zipped, upload it to Lambda directly if < 50MB; otherwise, upload to
the S3 first.

For native libraries, they need to be compiled on Amazon Linux environment.

AWS SDK comes by default with every Lambda function.

---

Lambda and CloudFormation -- inline
-----------------------------------

It is possible to define the Lambda function with codes inline within the
CloudFormation yaml file. This is for very simple functions as we cannot
inlcude function dependencies.

Lambda and CloudFormation -- S3
-------------------------------

You can first upload the Lambda function code as zip file on the S3, and
sepcify where CloudFormation can find the code (`S3Bucket`, `S3Key` - full path
to zip file, and `S3ObjectVersion` if S3 is version enabled).

Note that even if the code in your S3 is updated, if you do not update the
`S3Bucket`, `S3Key`, and `S3ObjectVersion`, CloudFormation won't update the
function.

Example of lambda-cloudformation template:

```yaml
Parameters:
    S3BucketParam:
        Type: String
    S3KeyParam:
        Type: String
    S3ObjectVersionParam:
        Type: String

Resources:
    LambdaExecutionRole:
        Type: AWS::IAM::Role
        Properties:
            AssumeRolePolicyDocument:
                Version: '2012-10-17'
                Statement:
                - Effect: Allow
                  Principal:
                    Service:
                    - lambda.amazonaws.com
                  Action:
                  - sts:AssumeRole
            Path: "/"
            Policies:
            - PolicyName: root
                Version: '2012-10-17'
                Statement:
                - Efect: Allow
                  Action:
                  - logs:*
                  Resource: arn:aws:logs:*:*:*
                - Effect: Allow
                  Action:
                  - xray:PutTraceSegments
                  - xray:PutTelemetryRecords
                  - xray:GetSamplingRules
                  - xray:GetSamplingTargets
                  - xray:GetSamplingStatisticSummaries
                  Resource: "*"
                - Effect: Allow
                  Action:
                  - s3:Get*
                  - s3:List*
                  Resource: "*"
    
    LambdaWithXRay:
        Type: "AWS::Lambda::Function"
        Properties:
            Handler: "index.handler"
            Role:
                Fn::GetAtt:
                    - "LambdaExecutionRole"
                    - "Arn"
            Code:
                S3Bucket:
                    Ref: S3BucketParam
                S3Key:
                    Ref: S3KeyParam
                S3ObjectVersion:
                    Ref: S3ObjectVersionParam
            Runtime: "nodejs12.x"
            Timeout: 10
            # Enable XRay
            TracingConfig:
                Mode: "Active"
```

---

Lambda Layers
-------------

Custom Runtimes support:

- [C++](https://github.com/awslabs/aws-lambda-cpp)
- [Rust](https://github.com/awslabs/aws-lambda-rust-runtime)

Externalize dependencies for reuse:

- We have to zip all the dependencies along with our code, which is costly.
- If the dependencies do not change much, then it would be better to
  externalize them into _layers_.
- Layers containing the libraries can be referenced by just the lambda function
  code.
- Since layers are now separate, another function can reference them as well.

Lambda Versions
---------------

When working on the Lambda function, it is at `$LATEST` - mutable version.

When the function is settled, we can publish it and version it - become
immutable. Thus, it is fixed. The version is a combination of the code itself
and the configurations snapshot'd.

Each version will have its own ARN designation, and each version can be
accessed.

Lambda Aliases
--------------

Aliases are _pointers_ to different Lambda function versions. For example, we
can define "dev", "test" and "prod" aliases and have them point at different
Lambda function versions.

Aliases are mutable - they can change their pointers such that it will point at
different version of the Lambda functions. So, an alias can point to version
1 90% of time and change to point at version 2 for 10% of time.

Aliases enable Blue / Green deployment by assigning weights to the Lambda
functions.

Aliases have their own ARNs; but aliases cannot reference other aliases.

Lambda and CodeDeploy
---------------------

CodeDeploy can help automate traffic shift for Lambda aliases. This feature is
integrated within the SAM framework.

i.e. There is a PROD alias that we wish to change its pointer to the latest
version (say from v1 to v2). So, we want to shift the traffic; this can be
managed with CodeDeploy to change the weight of the traffic sent.

- Linear: grow traffic every N minutes until 100%.
    - `Linear10PercentEvery3Minutes`
    - `Linear10PercentEvery10Minutes`
- Canary: try X percent first; then 100%.
    - `Canary10Percent5Minutes`
    - `Canary10Percent30Minutes`
- AllAtOnce: immediate shift over.

We can create Pre & Post Traffic hooks to check the health of the Lambda
function in case new version is bad; and need to rollback.

Lambda Limits - per region
--------------------------

**Execution limits**:

- Memory allocation: 128 - 3008 MB (in 64 MB increments).
- Maximum execution time: 900 seconds (= 15 mins); above? do not use Lambda for
  this task.
- Environment variables: 4 KB
- `/tmp` space: 512 MB
- Concurrency executions: 1000 (open support ticket for increase)

**Deployment lmits**:

- function size (in compressed zip): 50 MB
- uncompressed (code + dependencies): 250 MB

---

Lambda Best Practices
---------------------

- Perform heavy-duty work outside of your function handler!
    - leverage on the execution context.
    - i.e. place db connection outside of function.
    - i.e. initialize SDK outside of function.
    - i.e. pull in other dependencies or datasets outside of function.

- Use environment variables for:
    - db connection strings, S3 bucket names, ...
    - don't put these inside the code
    - passwords and sensitive values can be encrypted using KMS.

- Minimize the package size to its runtime necessities.
    - break down the function if needed
    - be wary of Lambda limits
    - use Layers if necessary

- Avoid using recursive code! never have a Lambda function call itself (will be
  expensive)


