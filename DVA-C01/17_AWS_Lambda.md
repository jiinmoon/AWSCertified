AWS Serverless: Lambda
======================

**Serverless** refers to a new paradigm shift where the developers do not have
to wrooy about the infrastructure and the servers; all developers are required
to do is deploy the codes.

It was pioneered by AWS Lambda, but it now includes anything that is managed as
well such as database, messaging, storage and etc.

Serverless does not mean that there is not a server - just that customers does
not have to manage or deal with it at all.

Serverless in AWS
-----------------

Suppose users will access static content from a S3 bucket and lo-in using AWS
Cognito where identities are stored. Within the static content (HTML) served,
there are REST API calls made to AWS API Gateway, which is directed to Lambda
functions to process the requests. As its database, DynamoDB is used.

AWS Lambda
----------

We can use EC2 instances or virtual servers in the cloud, but it is limited by
RAM and CPU. And it will be continuously running even when it is not used.
Scaling theses instances mean that there are certina degree of manual work
required to manage the configurations.

Lambda hides away the need to manage these servers; there are only functions.
It is limited by time so short executions and runs on-demand. It is
auto-scaled.

Lambda supports following languages:

- Node.js
- Python
- Java (8)
- C#
- Go
- Custom runtimes (supported by the community i.e. Rust)

**Docker cannot run on Lambda; ECS or Fargate should be used instead**.

Lambda -- Integration Examples
------------------------------

API Gateway : used to create REST APIs which invokes Lambda functions.

Kinesis : uses Lambda to do data transformation on fly.

DynamoDB : used to reate triggers - when db changes, invoke Lambda function.

S3 : integrates with S3 bucket event triggers.

CloudFront

CloudWatch Event/EventBridge : reponses to events.

CloudWatch Logs

SNS

SQS

Cognito : react to user action (i.e. login).

Lambda -- Example 1
-------------------

Suppose new image has been uploaded to S3. This triggers a Lambda function to
create the thumbnails for uploaded image and push it into the bucket or create
metadata and store it in the DynamoDB.

Lambda -- Example 2
-------------------

We could provision EC2 instance and set up cron job to do a schduled task. But
instead, we can set up CloudWatch Events rule that will trigger in set amount
of interval to Lambda function to perform a scheduled task.

Lambda -- SYNC Invocations
--------------------------

Synchronous calls : CLI, SDK, API Gateway, Application Load Balancer.

SYNC invocations to the Lambda function is returned right away and error
handling must be done at the client-side (retries, exponential back-offs, etc).

Via CLI:

- To list the available functions within a region,

        $ aws lambda list-funcions -region us-west-1

- To invoke the function with payload to deliver:

        $ aws lambda invoke --function-name hello-world --cli-binary-format raw-in-based64-output -playload '{"key1": "value1", "key2": "value2"}' --region us-west-1 response.json

Lambda -- with ALB
------------------

To expose a Lambda function as an HTTP(S) endpoint, we can either use ALB or
API Gateway. To integrate with ALB, the function must be registered as its
**target group**.

ALB converts HTTP(S) request to format understood by the Lambda functionvia
HTTP to JSON. The request payload is converted to JSON that contained ELB
information.

```json
{
    "requestContext": {
        "elb": {
            "targetGroupArn":
            "arn:aws:elasticloadbalancing:use-west-1:1234567890"
        }
    },
    "httpMethod": "GET",
    "path": "/lambda",
    "queryStringParameers": {
        "query" : "1234"
    },
    "headers": {
        "connection" : "keep-alive",
        "user-agent": ...
        ...
    }
}
```

This way, within the Lambda function, we can unpack JSON to see what the
request was about. Similarly, JSON is converted to HTTP format such that ALB
will understand. Following is a simple response from the Lambda function to
ALB:

```json
{
    "statusCode" : 200,
    "statusDescription": "200 OK",
    "headers": {
        "Content-Type": "text/html; charset=utf-8"
    },
    "body": "<h1>hello world</h1>
}
```

**ALB Multi-Header Values** can be enalbed so that the client can send the
query string parameters that have same name with multiple values such as
follows, `https://xyz.com/path?name=cat&name=dog`.

When enabled, HTTP headers and query string parameters sent as multi-values
will be shown as arrays within the AWS Lambda event and response objects such
as `"queryStringParameters": {"name": ["cat", "dog"]}`.

AWS Lambda@Edge
---------------

It is a way to enable global AWS Lambda with the CDN using CloudFront. This
will enable for example, request filtering before applications receive the
requests.

Lambda@Edge deploys alongside with CloudFront CND so that it can:

- allow you to build more responsive applications.
- allow you to not have to manage servers.
- customize the CDN content.

You can use Lambda to change CloudFront requests and responses.

Normally, user wll request to CloudFront and CloudFront will look for requested
object in the Origin to return. Here, we can modify following:

- After CloudFront receives a request from a viewer (viewer request).
- Before CloudFront forwards the reuqest to the origin (origin request).
- After CloudFront receives the response from the origin (origin resposne).
- Before CloudFront forwards the response to the viewer (viewer resposne).

This implies that you can generate reponses to viewers without ever sending the
request tothe origin.

Lambda@Edge -- Global Application
---------------------------------

Suppose we have static HTML website hosted on S3 bucket. Users visit the
website and makes a dynmaic API requests to CloudFront, which forwards the
request to the Lambda@Edge function. The Lambda function will then query data
with DynamoDB and perform tasks.

Lambda@Edge -- Use Cases
------------------------

- Website security and privacy.
- Dynamic Web Application at the Edge.
- Search Engine Optimization (SEO).
- Intelligent Route Accross origins and Data Centres.
- Bot mitigation at th Edge.
- Real-time Image Transformation.
- A/B Testing.
- User Authenticationand Authorization.
- User Prioritzations.
- User Tracking and Analytics.

Lambda -- ASYNC Invocations
---------------------------

ASYNC Invocations ae used with S3, SNS and CLoudWatch Events.

i.e. S3 bucket registers a new event (new file is uploaded). This event is
queue'd at **Event Queue** of the Lambda, which will be processed by the
available function.

Lambda will retry on errors up to 3 times with exponential back-off waiting.

Make sure tha the processing is **idempotent** where the results should be same
regardless or ordering.

**If retries occur, the duplicate log entires will be regsitered in CloudWatch
Logs**.

 We can also define DLQs with SNS or SQS for storing messages that are failed
 to be processed. This will require an additional IAM permissions.

ASYNC invocations allow you to speed up to processing if you do not need to
wait for the result (i.e. 10,000 images to process).

Lambda -- ASYNC Services
------------------------

- S3
- SNS
- CloudWatch Events/EventBridge
- CodeCommit
- CodePipelines
- etc

Lambda -- CloudWatch Events/EventBridge
---------------------------------------

CRON or RATE EventBridge rule will trigger a job every 1 hour to send request
to the Lambda function for a task.

CodePipeline EventBridge rule can also trigger a job on a state change to send
a request to the Lambda function.

Lambda -- S3 Event Notification
-------------------------------

Events from S3 can trigger on `S3:ObjectCreated`, `S3:ObjectRemoved` and etc.

You may also filter by the object name so that it triggers on certain files
(i.e. `*.jpg`).

Different types of architectural patterns are avaiable:

- Have S3 send notification to SNS; and mutiple SQS to handle the message.
- Have S3 send notification to SQS; and Lambda function reads from queue.
- Have S3 notifications go directly to Lambda for ASYNC invocation. If the
  message is bad or process is failed, it can go to DLQ.

If two writes are made to a single non-versioned object at the same time, it is
possible that only a single event notification will be sent.

Lambda -- Event Source Mapping
------------------------------

- Kinesis Data Streams
- SQS and SQS FIFO Queue
- DynamoDB Streams

The records need to be polled from the source and Lamba is triggered SYNC.

i.e. if the Lambda is configured to read from the Kinesis, there will be
a **Event Source Mapping** component created. This will be responsible for
polling from the Kinesis and receive the returned batches. Then it will invoke
the Lambda function SYNC with the event batches.

**(1) Streams and Lambda (Kinesis or DynamoDB)**

- An event source mapping creates an iterator for each shard; processes items
  in order.
- Can read starting from new items from beginning or from a timestamp.
- Processed items are not removed from the stream (other consumers can also
  read them).
- Low traffic stream: can use batch window to accumulate records before
  processing.
- You can process multiple batches in parallel:
    - Up to 10 batches per shard.
    - In-order processing is still guranteed for each partition key.
- Error Handling:
    - By default, if function returns an error, entire batch is reprocessed
      until the process suceeds or the items the in batch expire.
    - To ensure in-order processing, processing for the affected shard is
      paused until the error is resolved.
    - Configure the event source mapping to:
        - discard old events.
        - restrict the number of retries.
        - split the batch on error (workaround Lambda timeout).
    - Discarded events can go to a **destination**.

**(2) SQS, SQS FIFO and Lambda**

- Event source mapping will poll with _Long Polling_.
- Specify the batch size between 1 to 10 messages.
- Recommended to set the qeueu visibility timeout to six times the timeout of
  your Lambda function.
- If DLQ is needed, we need to set up on the SQS queue (DLQ for Lambda is for
  ASYNC only), or alternatively, use the **destination**.
- Lambda supports in-order processing for FIFO queues; scaling up to the number
  of active message groups.
- For standard queue, remember that the items would not be in order.
- Lambda scales up to process a standard queue as quickly as possible.
- When an error occurs, the batches are returned to the queue as individual
  items and might be processed in a different grouping than the original batch.
- Lambda deletes items from the queue after successful processing.
- Can configure the source queue to send items to DLQ if they cannot be
  processed.

**Lambda Event Mapper Scaling**

- Kinesis Data Streams and DynamoDB Streams:
    - One lambda invocation per stream shard.
    - If parallelized, up to 10 batches processed per shard.
- SQS Standard:
    - Lambda adds 60 more instances per min to scale.
    - Up to 1,000 batches of messages processed simulataneously.
- SQS FIFO:
    - Messages with the same **GroupID** will be processed in order.
    - Lambda function scales to the number of active message groups.

Lamdba -- Destinations
----------------------

ASYNC Invocations can define destinations for successful and failed events:

- SQS
- SNS
- Lambda
- EventBrdige bus

Recommended to use destinations instead of DLQ as it has more features, and
targets.

Event Source Mapping discards event batches in either SQS or SNS.

Lambda -- Execution Role (IAM)
------------------------------

IAM Roles must be attachd to the functions so that it can access other AWS
servcies and resources. A sample managed policies for Lambda may include:

- `AWSLambdaBasicExecutionRole` : upload logs to CloudWatch.
- `AWSLambdaKinesisExecutionRole` : read from Kinesis streams.
- `AWSLambdaDynamoDBExcutionRole` : read from DynamoDB streams.
- `AWSLambdaSQSQueueExecutionRole` : read from SQS queues.
- `AWSLambdaVPCAccessExecutionRole` : deploy Lambda within VPC.
- `AWSXrayDaemonWriteAccess` : upload trace data to X-Ray.

When using an event source mapping to invoke the functions, the Lambda uses the
execution role to read the event data.

Lambda -- Resource-based Policies
---------------------------------

Use resource-based policies to give other accounts and AWS services permission
to use your Lambda resources. This is similar to S3 bucket policies.

An IAM principal can access Lambda:
    - if the IAM policy attached to the principal authorizes it.
    - or, if the resource-based policy authorizes it.

When an AWS service like S3 or ALB calls your Lambda function, the
resource-based policy gives it access.

Lambda -- Environment Variables
-------------------------------

Lambda functions can have its own environment variables which changes its
behavior. Once they are set, it is available inside the code (there are also
system envrionment variables as well).

Note that you can store secrets safely since it can be encrypted with KMS.

For example, in Python, you may access it as follows:

```python
import os

def lambda_handler(event, context);
    os.getenv("ENV_KEY")
    # code
```

Lambda -- Logging and Monitoring
--------------------------------

**CloudWatch Logs**

- Lambda execution logs are by defalt always stored in AWS CloudWatch Logs.
- Make sure that the lambda has the execution role with am IAM policy.

**CloudWatch Metrics**

- invocations count, durations, concurrent number of executions...
- error count, success rates, throttles.
- ASYNC delivery failures.
- Iterator age (Kinesis and DynamoDB streams).

**Lambda Tracing with X-Ray**

- Enable in Lambda configuration for **Active Tracing**.
- Runs the X-Ray daemon automatically and use X-Ray SDK in the code.
- Ensure that Lambda has the `AWSXayDaemonWriteAccess` in IAM execution role.
- Environment varaibles:
    - `_X_AMZN_TRACE_ID` : contains the tracing header.
    - `AWS_XRAY_CONTENT_MISSING` : by default, `LOG_ERROR`.
    - `AWS_XRAY_DAEMON_ADDRESS` : X-Ray daemon `IP:PORT`.

Lambda -- Networking
--------------------

By default, the Lambda function is launched outside of your private VPC.

This implies that the Lambda function would not be able to acces the rsources
within your VPC.

**Lambda with VPC**

To enable Lambda to communicate with private VPC, you must define the VPC ID,
the subnets and the security groups. Then, Lambda will create an ENI in your
subnets. Make sure that `AWSLambdaVPCAccessExecutionRole` is enabled.

**Lambda in VPC with Internet Access**

By default, a Lambda function in private VPC would not have an internet access.
This is also true for public subnet as well. So, **deploying a Lambda function
does not give it a internet access or a public IP**.

To give the Lambda function deployed within the private subnet a internet
access, we create **NAT Gateway or NAT Instance**.

You can use VPC endpoint to privately access AWS services without a NAT.

Note that CloudWatch Logs without VPC endpoints or NAT Gateway.

Lambda -- Function Configuration
--------------------------------

**RAM**

- Scales from 128 MB to 3008 MB in 64 MB increments.
- More RAM you add, more vCPU credits accumulate.
- At 1792 MB, you get more than one CPU and will need to use multi-threading in
  your code to fully utilize it.

**Timeout**

- Default is set to 3 seconds (max 900 seconds).

Lambda -- Execution Context
---------------------------

The execution context is a temp runtime environment that initializes any
external dependencies of your Lambda code which is great for db connections,
clients and etc.

It is maintained in anticipation of another Lambda function invocation - thus,
next Lambda function can leverage on the same environment. It also includes the
`/tmp` directory.

Below is the bad example of not utilizing the execution context:

```python
import os

def get_user_handler(evnet, context):
    DB_URL = os.getenv("DB_URL")
    db_client - db.connect(DB_URL)
    user = db_client.get(user_id = event["user_id"])
    return user
```

Notice that with every function call, the database connections are
re-established. To fix this, we move the db connection code out of the function
scope.

```python
import os

DB_URL = os.getenv("DB_URL")
db_client = db.connect(DB_URL)

def get_user_handler(event, context):
    user = db_client.get(user_id = event["user_id"])
    return user
```

Lambda -- `/tmp` space
----------------------

If the function requires more storage while executing, it has access to the
`/tmp` directory which has a max size of 512 MB.

This directory will remain when the execution context is frozen, allowing code
to use it as a transient cache for multiple invocations and checkpoint the
progress.

For persistent objects, S3 should be used instead.

Lambda -- Concurrency and Throttling
------------------------------------

Lambda functions support up to 1,000 concurrent executions.

We can set a **reserved concurrency** at the function level (limit) so that
when invocation count goes over the limit, it will trigger a throttle.

- if SYNC invocation, returns a `ThrottleError 429`.
- if ASYNC invocation, retry and place it in DLQ.

If require more than 1,000 concurrent executions, open a ticket to AWS.

Lambda -- Concurrency Troubleshooting
-------------------------------------

If you do not reserve (place a limit) on the concurrency, it is possible that
one Lambda function may take up all the concurrency and leave others throttled.

So, **the concurrency limit applies to all functions within the account**.

Lambda -- Concurrency and ASYNC
-------------------------------

Suppose we have a S3 bucket event triggering Lambda function ASYC. Many files
have been uploaded and if it goes over the limit, then additional requests to
the Lambda function can be throttled.

However, since this is ASYC invocations, throttling errors (429) and system
errors (5XX) will be dealt by Lambda function to return the event back to the
queue and attempts to run the function again for up to 6 hours.

This retry interval will increase exponentially from 1 second after the first
attempt up to a max 5 mins.

Lambda -- Cold Starts and Provisioned Concurrency
-------------------------------------------------

**Cold Start**

- New instance : codes are loaded and init is running.
- If the init size is large (large dependencies) then this process can take
  some time.
- Thus, first request to the Lambda function will always have higher latency
  than the rest.

**Provisioned Concurrency**

- Concurrency is allocated before the function is invoked in advance.
- Thus, cold start is never an issue.
- Application auto-scaling can manage concurrency.

Lambda -- Function Dependencies
-------------------------------

If a function depends on external libraries, then you must install the package
alongside your code and zip it together.

- For Node.js, use npm and `node_modules` directory.
- For Python, `pip --target`.
- For Java, include all the relevent `.jar` files.

Once all of them are zipped, upload it to the Lambda directly if < 50 MB.
Otherwise it needs to be uploaded to the S3 first.

Lambda -- CloudFormation -- inline
----------------------------------

It is possible to define the Lambda function with codes inline within the
CloudFormation YAML file; this enables to define very simple Lambda function
that does not include function dependencies.

Lambda -- CloudFormation -- S3
------------------------------

You first upload the Lambda function code as a .zip file on the S3, and specify
where CloudFormation can find the code (`S3Bucket`, `S3Key`,
`S3ObjectVersion`).

Note that even if the code in the S3 is updated, if you do not update the
provided `S3Bucket`, `S3Key` and `S3ObjectVersion`, the CloudFormation won't
update the function.

Lambda -- Layers
----------------

Custom Runtimes Support for other langauges such as C++ or Rust.

Externalize dependencies for reusage:

- Zipping all the dependencies along with the code can get costly.
- If the dependencies do not change much, then it would be better to
  externalize them into the layers.
- Layers containing the libraries can be referenced by the Lambda function
  code.
- Since the layers are separated, another function can reference them as well.

Lambda -- Versions
------------------

Working Lambda function is at its $Latest version.

When the function is settled, we can publish it and version it - becoming
immutable. Each version will have its own arn designation that can be
referenced using aliases.

Lambda -- Aliases
-----------------

Aliases are pointers to different Lambda function versions. For example, we can
define DEV, TEST and PROD aliases and have them point at different Lambda
function versions.

Aliases are mutable - they can change their pointers such that it will point at
different versions (can be weighted). This enables BLUE/GREEN deployment by
assigning weights to the Lambda functions.

Aliases have its own arn but it cannot reference each other.

Lambda -- CodeDeploy
--------------------

CodeDeploy can help automate traffic shift for the Lambda aliases which is
integrated within the SAM framework.

i.e. There is a PROD alias that we wish to change its pointer to the latest
vesion from v1 to v2. So, we want to shift the traffic and this can be managed
with CodeDeploy.

- Linear: grow traffic every N minutes untill 100 %.
    - `Linear10PercentEvery3Minutes`
    - `Linear10PercentEvery10Minutes`
- Canary: try X % first; then shift over.
    - `Canary10Percent5Minutes`
    - `Canary10Percent30Minutes`
- All-At-Once

Lambda -- Limits per region
---------------------------

**Execution Limits**:

- Memory allocation : 128 MB to 3008 MB.
- Max execution time : 900 seconds.
- Environment variables: 4 KB.
- `/tmp` space : 512 MB.
- Concurrency executions: 1,000.

**Deployment Limits**:

- Function size (compressed) 50 MB; 250 MB if uncompressed.


