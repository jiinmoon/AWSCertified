AWS Monitoring
==============

Why monitor?
------------

We have seen how we can deploy applications:

- safely
- automaticcally
- using Infrastructure as code
- leveraing the best AWS components

Our apps are deployed, and our users wouldn't care how we did it; does our app
working and servicing correctly?

To measure this, we need to monitor our apps behavior.

- Application latency; is it increasing over time?
- Application outages?
- Customer reports problems?

Internally, we want to monitor so that we can prevent issues from happening;
also, we want to know our app's performance and its cost. How can we improve
further based on this data?

Monitoring in AWS
-----------------

**AWS CloudWatch**

- Metrics
- Logs: collect, analyze and store log files.
- Events: send notifications when events happen in AWS.
- Alarms: react in realtime to metrics / events.

**AWS X-Ray**

- Troubleshooting applications performance and errors
- Distributed tracing of microservices

**AWS CloudTrail**

- Internal monitoring of API calls being made
- AUdit changes to AWS Resource by your users

---

AWS CloudWatch Metrics
----------------------

- CloudWatch provides metrics for every services in AWS.
- **Metric** is a variable to monitor (i.e. CPUUtilization, NetworkIn...).
- Metrics belong to **namespaces**.
- **Dimension** is an attribute of a metric (instance id, environmnet, etc...).
- Upto 10 dimensions per metric.
- Metrics have **timestamps**.
- Can create visual dashboards of metrics.

AWS CloudWatch EC2 Detailed monitoring
--------------------------------------

- EC2 instance metrics have metrics _every 5 mins_.
- if we want the monitor in more detail every 1 min, you enable detailed
  monitoring for extra cost.

- use detailed monitoring if you want to prompt scale your ASG (react to CPU
  changes much quicker).

- Free Tier allows to have upto 10 detailed monitoring metrics.

- Note: EC2 Memory usage is by default not pushed. To enable tracking of RAM
  usage, you must push it from inside the instance as a custom metric.

AWS CloudWatch Custom Metrics
-----------------------------

- It is possible to define and send your own custom metrics to CloudWatch

- You can use dimensions (attributes) to segment metrics:
    - Instance.id
    - Environment.name

- Metric resolution:
    - Standard: 1 minute
    - High Resolution: upto 1 second (**StorageResolution** API parameter);
      higher cost.

- Use API call **PutMetricData**
- User exponential back off in case of throttle errors


AWS CloudWatch Alarms
---------------------

- Alarms are used to trigger notifications for any metric.
- Alarms can go to ASG, EC2 Actions, SNS notifications
- Various options (sampling, %, max, min, etc...)
- Alarm States:
    - `OK`
    - `INSUFFICIENT_DATA`
    - `ALARM`
- Period:
    - Need to specify length of time in seconds to evaluate the metric.
    - High resolution custom metrics can only choose 10 sec or 30 sec.

- i.e. ASG alarms set to increase/decrease number of EC2 instances as network
  traffics increase/decrease.

AWS CloudWatch Logs
-------------------

- Applications can send logs to CLoudWatch using the SDK

- CloudWatch can collect log from:
    - Elastic Beanstalk: collection of logs from app
    - ECS: collection from containers
    - AWS Lambda: collection from function logs
    - VPC Flow Logs: VPC specific logs
    - API Gateway
    - CloudTrail based on filter
    - CloudWatch log agents: i.e. on EC2 machines
    - Route53: log DNS queries

- CloudWatch Logs can be exported to
    - Batch exporter to S3 for archive
    - Stream to ElasticSearch cluster for further analytics

-  CloudWatch Logs can use filter expressions

- Logs stroage architecture:
    - Logs groups: arbitary name, usually representing an application
    - Log stream: instances within application, log files and containers

- Can define log expiration policies (never expire, 30 days, etc).

- Using AWS CLI we can tail CloudWatch logs

- To send logs to CloudWatch, make sure **IAM permissions are correct**.

- Security: encryption of logs using KMS at the Group level.

AWS CloudWatch Logs for EC2
---------------------------

- By default, no logs from your EC2 instance will go to CloudWatch; to do so,
  you need to run a _CloudWatch agent_ on EC2 to push the log files.

- **Make sure that IAM permissions are correct**.

- CloudWatch log agent can be setup On-premises EC2 instances as well.

CloudWatch Logs Agent & Unified Agent
-------------------------------------

- for virtual servers (EC2 instances, on-premise servers...)

- **CloudWatch Logs Agent**
    - older version of the agent
    - can only send to CloudWatch Logs

- **CloudWatch Unified Agent**
    - collects additional system-level metrics such as RAM, processes, etc.
    - collect logs to send to CloudWatch Logs
    - centralized configuration using SSM Parameter Store

- This is more for DevOps; not much for Devs exam.

CloudWatch Logs Metric Filter
-----------------------------

- CloudWatch Logs can use filter expressions
    - i.e. find a specific IP inside of a log or count occurrences of "ERROR"
    - Metric filters can be used to trigger alarms

- **Filters do not retroactively filter data**. Filters only public the metirc
  data points for events that happen after the filter was created.

CloudWatch Events
-----------------

- Sechedule: Cron jobs
- Event Pattern: Event rules to react to a service triggers
    - i.e. CodePipeline state changing
- Triggers to Lambda functions, SQS/SNS/Kinesis Messages
- CloudWatch Event creates a small JSON document to give information about the
  change.

---

AmazonEventBridge
-----------------

- It is a next evolution of CloudWatch Events.

- **Default event bus**: generated by AWS services (CloudWatch Events).

- **Partner event bus**: receive events from Saas service or applications such
  as Zendesk, DataDog, Segment, Auh0, ...

- **Custom event bus**: your application can publish its own events.

- Event buses can be accessed by other AWS accounts.

- **Rules**: how to process events.

Amazon EventBridge Schema Registry
----------------------------------

- EventBridge can analyze the events in your bus and infer the **schema**.

- **Schema Registry** allows you to generate code for your application that
  will know in advance how data is structured in the event bus.

- Schema can be versioned.

Amazon EventBridge vs CloudWatch Events
---------------------------------------

- EventBridge builds upon and extends CloudWatch Events.
- It uses same service API and endpoint, and the same underlying service
  infrastructure.
- EventBridge allows extension to add event buses for your custom applications
  and your third-party SaaS apps.
- On top, we also have Schema Registry capability.

- EventBridge has a different name to mark the new capabilities.
- Overtime CloudWatch Event name will be replaced with EventBridge.

---

AWS X-Ray
---------

- Typically, debugging in Production involves:
    - test locally
    - add log statements everywhere
    - redeploy in production

- Log formats may differ across applications using CloudWach and analytics is
  hard.

- Debugging a monolith maybe is easy, but distributed services will be
  difficult.

- X-Ray provides the common view into your applications.

X-Ray: Visual Analysis of Applications
--------------------------------------

X-Ray provides graphical representations of our services; clients making
a request and how it propagates along other services. Allowing for Tracing.

Its advantages are:

- troubleshooting performance issues (where is it bottlenecked?)
- understand dependencies in a microservice architecture
- pinpoint service issues
- review request behavior
- find errors and exceptions
- identify users that are impacted

X-Ray compatibility
-------------------

- AWS Lambda
- Elastic Beanstalk
- ECS
- ELB
- API Gateway
- EC2 instances or any application service (+ On-Premise)

X-Ray Tracing
-------------

- Tracing is an end to end way to following a "request".
- Each componenet dealing with the request adds its own "trace".
- Tracing is made of segments (+ sub segments).
- Annotations can be added to traces to provide extra-information.
- Ability to trace:
    - every request
    - sample request (percentage or a rate per min)
- X-Ray Security:
    - IAM for authorization
    - KMS for encryption at rest

How to Enable X-Ray?
--------------------

- The code must import the AWS X-Ray SDK
    - very little modification is needed to code base.
    - SDK will then capture:
        - calls to AWS services
        - HTTP / HTTPS requests
        - Database Calls (MySQL, PostgreSQL, DynamoDB)
        - Queue calls (SQS)

- Then, install X-Ray daemon or enable X-Ray AWS Integration
    - X-Ray daemon works as a low level UDP packet interceptor
    - AWS Lambda / other AWS services already run the X-Ray daemon

- **Each app must have IAM rights to write data to X-Ray**.

i.e. Application code will use AWS X-Ray SDK, and it will send traces to X-Ray
daemon running on the same instance. daemon will send the batch every second to
AWS X-Ray.

X-Ray Troubleshooting
---------------------

- If X-Ray is not working on EC2
    - ensure that EC2 IAM Role has proper permissions
    - ensure that EC2 instance is running the X-Ray daemons

- To enable on AWS Lambda
    - ensure it has an IAM execution role with proper policy
      `AWSX-RayWriteOnlyAccess`.

X-Ray Instrumentation in code
-----------------------------

**Instrumentation** refers to measure of product's performance, diagnose
errors, and to write trace information.

To instrument our app, we use **X-Ray SDK**.

This is an example of how to use X-Ray SDK with Node.js and Express:

```js
var app = express();

var AWSRay = require('aws-xray-sdk');
app.use(AWSXRay.express.openSegment('MyApp'));

app.get('/', function (req, res) {
    res.render('index');
});

app.use(AWSXRay.express.closeSegment());
```

You can modify app code to customize and annotate the data that the SDK sends
to X-Ray, using _interceptors, filters, handlers, middleware_, ...

X-Ray Concepts
--------------

**Segment** : each application / service will send them

**Subsegment** : more details in the segment

**Trace** : segments collected together to form an end-to-end trace

**Sampling** : decreases the amount of requests sent to X-Ray; reduces cost

**Annotations** : Key-Value pairs used to index traces and use with _filters_;
enables you to search through the traces using your defined indicies

**Metadata** : Key-Value pairs, but **not indexed**; it is not for searching

X-Ray agent / daemon is able to send traces cross acount through its config,
but make sure that IAM permissions are correct; the agent will assume the role.
This allows use to have a central account for all your application tracing.

X-Ray Sampling Rules
--------------------

With sampling rules, we control the amount of data recorded.

Modifying the sampling rules does not require you to change code.

By default, X-Ray SDK records the first request each second, and five percent
of any additional requests.

That one request per each second is the **reservoir**; this ensures that at
least one trace is recorded each second as long the service is serving
requests.

Later five percent is the **rate**; additional requests beyond the reservoir
size are sampled.

X-Ray Custom Sampling Rules
---------------------------

You can create own rules with the **reservoir** and **rate**.

i.e. we may set POST to have reservior of 10 requests per second; and rate of
10 percent to be sampled beyond the reservoir.

When the rules have changed, daemon automatically updates itself.

X-Ray Write APIs used by X-Ray daemon
-------------------------------------

Example: `arn:aws:iam::aws:policy/AWSXrayWriteOnlyAccess`

```
"Effect": "Allow",
"Action": [
    "xray:PutTraceSegments",
    "xray:PutTelemetryRecords",
    "xray:GetSamplingRules",
    "xray:GetSmplingTargets",
    "xray:GetSamplingStatisticSummaries"
],
"Resource": [
    "*"
]
```

`PutTraceSegments` uploads segment docs to AWS X-Ray

`PutTelemetryRecords` used by X-Ray daemon to upload telemetry

- SegmentsReceivedCount
- SegmentsRejectedCount
- BackendConnectionErrors

`GetSamplingRules`  retrives sampling rules automatically

`GetSamplingTargets` & `GetSamplingStatisticSummaries` -- advanced

X-Ray daemon needs to have an IAM policy authorizing the correct API calls to
function correctly.

X-Ray Read APIs
---------------

`GetServiceGraph` is the main graph.

`BatchGetTraces` retrieves a list of traces specified by ID; each trace is
a collection of segment documents that originates from a single request.

`GetTraceSummaries` retrieves IDs and annotations for traces available for
a specified time frame using an optional filter; to get full traces, pass the
trace IDs to `BatchGetTraces`.

`GetTraceGraph` retrives specific graph for a specific trace ID.

---

X-Ray with Elastic Beanstalk
----------------------------

AWS Elastic Beanstalk platforms include the X-Ray daemon; and we can run the
daemon by setting an option in the Beanstalk console or with a configuration
file (in `.ebextensions/xray-daemon.config`).

```yaml
option_settings:
    aws:elasticbeanstalk:xray:
        XRayEnabled: true
```

Make sure that the instance profile has the correct IAM permissions such that
X-Ray daemon can function correctly.

App code should be instrumented with X-Ray SDK.

Note that X-Ray daemon is not provided for Multicontainer Docker (managed via
ECS).

X-Ray with ECS
--------------

**ECS Cluster**

We can run the X-Ray Container as a Daemon. Each instances will have one X-Ray
Daemon Container.

Or, we can run X-Ray Container as a "side-car"; within each application
conainer, we will also run the X-Ray daemon as well. So, every app container
spawned each has its own X-Ray daemon.

**Fargate Cluster**

Here, we do not have a control; only option is to use the "side-car" pattern
since we do not have a control over individual instances.

```JSON
{
    "name" : "xray-daemon",
    "image" : "...",
    ...
    "portMappings" : [
        {
        "hostPort": 0,
        "containerPort": 2000,
        "protocol": "udp",
        }
    ],
},
...
```

Notice the `portMappings`: container port must be 2000 and protocol uses udp.

Also, environment variables must be set:
`AWS_XRAY_DAEMON_ADDRESS="xray-daemon:2000"`

And finally link to `"xray-daemon"`.

---

AWS CloudTrail
--------------

Provides governance, compliance and audit for your AWS Account; it is enabled
by default. This logs all history of events / API calls made within your AWS
Account by Console, SDK, CLI, AWS Services.

It is possible to put logs from CloudTrail onto CloudWatch Logs.

_If resources is deleted in AWS, check the CloudTrail first_.

CloudTrail vs CloudWatch vs X-Ray
---------------------------------

**CloudTrail**

- Audit API calls made by users / services / AWS console
- useful for track down unauthorized calls or root cause of changes

**CloudWatch**

- CloudWatch Merics over time for monitoring
- CloudWatch Logs for storing application logs
- CloudWatch Alarms to send notifications in case of unexpected metrics

**X-Ray**

- Automated Trace Analysis & Central Service Map Visualization
- Latency, Errors, and Fault analysis
- Request tracking across distributed systems

