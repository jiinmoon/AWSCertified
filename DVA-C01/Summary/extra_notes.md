EC2 User Data is used to perform automated config tasks when instance starts
(sh scripts). It runs with root privileges and runs only during the boot cycle
when the instance is first launched.

EC2 Spread placement groups is a new instance placement strategy that places
groups in a own rack. Can span multi-AZ in same region; max 7 instances per AZ
per group.

IAM - Access Advisor helps to identify the unused roles.

AWS Trusted Advisor helps you to provison resources following best practices on
cost optimization, security, fault tolerance, service limits, and performance
improvement.

IAM - Access Analyzer helps to identy the resources shared with an external
entity.

CloudTrail can log, monitor and retain account activity related to actions
across your AWS infrastructure. "Who made an API call to modify X resource?".

Think resource performance monitoring, events, and alerts : CloudWatch.

Think account-specific activity and audit : CloudTrail.

Think resource-specific history, audit, and compliance : Config.

Cognito Identity Pools is to allow users obtain temp credentials to access AWS
services. It works with other identity providers such as **Cognito user
pools**, social identity providers, SAML, OpenID, and etc.

AWS Bugets lets customers set custom budgets and receive alerts if their costs
or usage exceed the set amount. It requires 5-weeks of usage data to generate
budget forecasts.

API Gateway usage plans specifies who can access API stages and methods (how
fast and how much they can access them). The plan uses API keys to identify API
clients and meters access to the associated API stages for each key. It also
lets you configure throttling limits and quota limits that are enforced on
individual client API keys.


**API Gateway Mapping Templates** allows you to map the payload from a method
request tot he corresponding integration request and vice versa.


You can give a Lambda function created in one account permissions to assume
a role in another account to access resources such as DynamoDB or S3. To do so,
create an execution role in first account that gives the Lambda function the
necessary permissions. Then, create a role in second account that the Lambda
function in first assumes to gain access to the cross-account resources. Modify
the trust policy of the role in second account to allow the execution role of
Lambda to assume this role. Finally, update he Lambda function code to add the
AssumeRole API call.

SAM supports following resource types within its YAML file:

- `AWS::Serverless::Api`
- `AWS::Serverless::Application`
- `AWS::Serverless::Function`
- `AWS::Serverless::HttpApi`
- `AWS::Serverless::LayerVersion`
- `AWS::Serverless::SimpleTable`
- `AWS::Serverless::StateMachine`


SSM Parameter Store can store data such as passwords, databse strings, and
license codes as parameter values; but it cannot be used to automatically
rotate the database credentials.

AWS Budgets - usage bugets help you plan how much you want to use on one or
more services.

By default, IAM users do not have access to the AWS Billing and Cost Management
console - grant access to the users by activating IAM user access to the
Billing and Cost Management console and attaching an IAM policy to the users.
Then, activate IAM user access for IAM policies.

IAM and ACM can be used as a certificate manager for SSL/TLS. The ACM is
preferred as you can request or deploy. ACM automatically renews the
certificates.

If `--region` parameter is not set with CLI commands, it is executed against
the default AWS region.

CloudFormation template has an optional Outputs section which declares output
values that you can import into the other stacks (for cross-stack references).
Use the Export Output Values to export the name of the resource output - for
each AWS account, export names must be unique within a region.


Use an AWS Organizations Service Control Policy to define the maximum
permissions for account members of an organization or organizational unit.

A Permissions Bounady which is a managed policy used for an IAM entity which
defines the maximum permissions that the identity-based policies can grant to
an entity - but does not grant permissions.

Lambda Authorizer is a Lambda function that you provide to control access to
your API; it uses bearer token authentication strategies (OAuth or SAML).
Before creating API Gateway Lambda Authorizer, must create a LAmbda function
that implements the logic to authorize and to authenticate.

You manage access in AWS by creating policies and attaching them to IAM
identities (users, groups or roles) or AWS resources. A policy is an object in
AWS that associated with an identity or resource and defines their permissions.

**Trust policies** define which principal entities (accounts, users, roles and
federated users) can assume the role. An IAM role is both an identity and
a resource that supports resource-based policies. 


ASG cannot span across multiple Regions; an ASG can contain EC2 instances in
one or more AZs within the same Region.

Elastic Load Balancer provides **acess logs** that cpature detailed information
about requests sent to your load balancer. Each log contains information such
as the time the request was received, the client's IP, latencies, request paths
and server responses. This is disabled by default and stores logs in S3 bucket.
Each log files are encrypted by SSE-S3.

Keypairs are used only for Amazon EC2 and Amazon CloudFront.

AWS CodeDeploy is a deployment service that automates application deployments
to Amazon EC2 instances, on-premises instances, or serverless Lambda functions.
AWS COdeBuild is a fully maanged continuous integration service that compiles
source code, runs tests, and produces software packages that are ready to
deploy.

IAM username and password cannot be used to access CodeCommit.

To prevent other consumers from processing the same message, SQS sets
a visibility timeout (default 30 seconds; 0 ~ 12 hours). Consumer who is
currently processing the message can request ChangeMessageVibility API to
extend the timeout.

To enable SSL/TLS with Load Balancer, create a HTTPS linster with SSL
termination; and configure SSL certificate with ACM.


Concurrency is the number of requests that a Lambda function is serving at any
given time; if a function is invoked again while a request is still being
handled, another instance is allocated. To ensure that a function can always
reach a certain level of concurrency, you can configure the function with
reserved concurrency - and no other function can use it. This is a way to set
the max concurrency for the function and applies to the function as a whole
across versions and aliases.

Provisioned concurrency is mostly a way to deal with cold starts.

IAM Access Analyzer helps you identify the resources in your organization and
accounts that are shared with an external entity. It can identfy unintended
access to your resources and data.

Dedicated Instances are Amazon EC2 instances that run in a virtual private
cloud (VPC) on hardware that's dedicated to a single customer.

For Amazon CloudFront, you can use key pairs to create signed URLs for private
content, such as when you want to distribute restricted content that someone
paid for - CloudFront key pairs are created only by the root.

CloudFormation template does not allow you to use Conditions in Parameters
section.

When a host needs to sned many records per second to Amazon Kinesis, repeatedly
calling PutRecord API is not enough; to increase throughput, the app should
batch records and implement parallel HTTP requests.

There are no message limits for storing in SQS.

To reference a parameter in the CloudFormation template, use `!Ref`.

```yaml
AnotherEIP:
    Type: "AWS::EC2::EIP"
    Properties:
        InstanceId: !Ref AnotherEC2Instance
```

`GetAtt` is used to value of an attribute from a resource such as `!GetAtt
thisResource.thisAttributeName`.

General Purpose SSD (GP2) provides burst to 3,IOPS. Between 100 IOPS (at 33.33
GB and below) and maximum of 16,000 IOPS (at 5,334 GB and above).

When Management Console is used to launch ASG, it uses basic monitoring by
default. You will need to specify detailed monitoring if so.

The correct way to reusing SSH keys in your AWS Regions:

- Generate a public SSH key file from the private SSH file.
- Set the AWS Region you wish to import to.
- Import the public SSH into the new Region.

EC2 Auto Scaling cannot add a volume to an existing instance if the existing
volume is approaching capacity.

Security Token Service is a web service that enables you to request temp
credentials for AWS IAM users or for users that you authenticate. This is not
supported by API Gateway. API Gateway can use following mechanisms for
authentication and authorization:

- Resource policies.
- Standard IAM roles and policies.
- IAM tags (used with IAM policies).
- Endpoint policies for interface VPC endpoints.
- Lambda authorizers.
- Amazon Cognito user pools.

Surge in the traffic can still throttle the Lambda functions and its
concurrency limits; you can configure Application Auto Scaling to manage
provisioned concurrency on a schedule or based on utilization.

Provisioned IOPS SSD (io1) volumes has 50:1 ratio of provisioned IOPS to
requested volume size in GB.

CloudFormation supports following parameter types:

```
String
Number
List<Number>
CommaDelimitedList
AWS::EC2::KeyPair::KeyName
AWS::EC2::SecurityGroup::Id
AWS::EC2::Subnet::Id
AWS::EC2::VPC::Id
List<AWS::EC2::VPC::Id>
List<AWS::EC2::SecurityGroup::Id>
List<AWS::EC2::Subnet::Id>
```

RDS MySQL and RDS PostGreSQL allows IAM database authentication.

AWS CodeBuild monitors functions on your behalf and reports metrics through
Amazon CloudWatch. These metrics include the number of total builds, failed
builds, successful builds, and the duration of builds. You can export log data
from your log groups to an S3 bucket and use the data for further analysis.

ALB + Beanstalk can create docker environments with a multi-container Docker
platform; but ECS will give you a finer control.

DynamoDB transcations make coordinated all-or-nothing changes to multiple items
both within and across tables - used to maintain data correctness.

RDS MySQL complete many operations in a single transaction block to maintain
data correctness.

With Spot Instance, you cannot reboot the instance when interrupts occur
(remember, intrruptions are due to price going above). You can either stop,
hibernate, or terminate the Spot instances.

---

When deleting stacks from CloudFormation, you must delete all the imports
before you can delete the exporting stack or modify the output value.

---

Cognito Identity pools enable you to create unique identities for your users
and federate them with identity providers. With an identity pool, you can obtain
temp, limited-privilege AWS crendentials to access AWS services. Identity pool
can support following identity providers:

- Public providers: Amazon, Fackbook, Google, Apple...
- Amazon Cognito user pools.
- OpenID/SAML/Deeloper Authenticated Identities.

---

Cognito user pools cannot be used to obtain temp AWS credentials.

---

With CodeBuild, you can improve build time by caching dependencies - bundle the
dependencies in the source code during the last satege.

---

Again, know the difference between Cognito User v Identity Pools.

**User Pools** are user directories that provide sign-up and sign-in options
for your app users.

**Identity Pools** enable you to grant your users access to other AWS services.

You can use them separetely or together!

Amazon Cognito identity ppols (federated identities) support user
authentication through Amazon Cognito user pools, federated identity proviers,
and unauthenticated identities.

User Pool does not have an option to enable unauthenticated identities.

---

After a Lambda function is executed, AWS Lambda maintains the execution context
for some time in anticipation of another Lambda function invocation - any
declarations in your Lambda function code that is outside of the handler code
will remain initialized, providing extra optimization when the function is
invoked repeatedly.

---

A client can invalidate an exsiting cache entry and reload it from the
integration endpoint for individual request using `Cache-Control: max-age=0`
header.

`Require authorization` checkbox ensures that not every client can invalidate
the API cache. If most or all of the clients invalidate the API cache, then
the cache is pointless and only serves to increase the API calls.

---

To create a Lambda function, you need a deployment package and an execution
role. Deployment package contains your code and execution role grants the
function permission to use AWS servcies such as CloudWatch Logs for log
streaming and AWS X-Ray for request tracing.

`InvalidParameterValueException` will be returned if one of the parameters in
the request is invalid. i.e. if you provided an IAM role in the
`CreateFunction` API that the Lambda function was unable to assume.

`CodeStorageExceededException` is when code size limit is exceeded in the
account.

`ResourceConflictException` is raised when the resource already exists.

`ServiceException` is when the AWS Lambda service encountered an error.

---

When uploading to S3, **multipart** upload allows you to upload a single object
as a set of parts. Should consider using it when the file is >= 100 MB.

Note that in order for Transfer Acceleration to work, the bucket name must be
DNS compliant and does not contain any ".".

---

The S3 object permission is managed by S3 Access Control Lists (ACLs).

`put-bucket-policy --policy` is used to apply policy on bucket level.

---

Redis is a **singlethreaded** server. IT is not designed to benefit from
multiple CPU cores. Redis is a choice when:

- Advanced data structures are concerned.
- Sanpshots
- Replication.
- Transactions.

Memcached is a choice when followings are concerned:

- Keep it simple.
- Large nodes with mutliple cores or threads.
- Ability to scale out and in - adding/removing nodes.
- cache objects (i.e. database)

---

For a Lambda function, its concurrent exections is:

`concurrent executions = invocations per second * average execution duration.`

For example, if a source publishes 10 events per second where the Lambda takes
on average 3 seconds to process, there will be 30 concurrent executions.

By default, 1000 concurrent executions are allowed within a given region in an
account.

---

For the API Gateay to pass the Lambda output as the API response to the client,
the Lambda function must return the result in the JSON format; otherwise, it
will result in 502 errors.

---

For web distributions, you can configure CloudFront to require that vieweres
use HTTPS to request your objects, so connections are encrypted when CloudFront
communicates with viewers. This is also possible for traffics between
CloudFront and the origin as well.

1. A viewer submits an HTTPS request to CloudFront; SSL/TLS negotiation between
   the viewer and CloudFront takes place. The viewer send the request in an
   encrypted format.

2. If the object is in the CloudFront edge cache, CloudFront will encrypt the
   response and returns to the viewer to decrypt.

3. If the object is not in the edge cache, CloudFront performs SSL/TLS
   negotiation with the origin, and then forwards the request to the origin in
   encrypted format.

4. Origin decrypts the request, encrypts the requested object, and returns the
   object to CloudFront.

5. CloudFront decrypts the response, re-encrypts it, then forwards to the
   viewer.

To implement this **Origin Protocol Policy** and **Viewer Protocol Policy** are
required.

---

Origin Access Identity (OAI) is a special CloudFront user which is associated
with the origins. This is typically used to secure items specific items or all
in S3 bucket.

---

Port mappings are specified as part of container definition which is configured
under the task definition.

---

VPC Flow Logs is a feature that enables you to capture information about the
IP traffic going to and from network interfaces in your VPC; the data can be
published to CloudWatch Logs and S3.

It helps you with troubleshooting why specific traffic is not reaching an
instance or use it as a security tool  to monitor the traffic that is reaching
your instance.

---




