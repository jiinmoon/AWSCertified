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













