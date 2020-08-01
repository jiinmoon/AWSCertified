# AWS Elastic Beanstalk FAQs

## General

**What is AWS Elastic Beanstalk?**

It makes easier for developers to quickly deploy and manage apps in the AWS
Cloud. Developers simply upload their apps, and EB automatically handles the
deployment details of capacity provisioning, load balancing, auto-scaling, and
app health monitoring.

**Will AWS Elastic Beanstalk upport other languages?**

They will, but at the moment, cannot use unsupported languages.

**What elements of my app can I control when using AWS Elastic Beanstalk?**

- Select the OS that matches app requirements.
- Choose EC2 instance types (On-demand, Reserved instances, and Spot
  instances).
- Choose from available database and storage options.
- Enable login access to EC2 instances for direct troubleshooting.
- Multiple AZ deployment.
- HTTPS protocol on load balancer.
- CloudWatch monitoring and getting notification on app health and etc.

## Databases and Storage

**Does AWS Elastic Beanstalk store anything in Amazon S3?**

EB stores app files and optionally server logs in S3.

**What database solutions can I use with AWS EB?**

EB does not restrict you to any specific persisitence techs. You can choose
from Amazon RDS, Amazon DynamoDB, Microsoft SQL Server, Oracle, or other
relational databases running on EC2.

## Security

**How do I make my application private?**

By default, your app is available publicly at `myapp.elasticbeanstalk.com`.

You can provision Amazon VPC to provision a private, isolated section of your
app in a virtual network which can be made private through specific security
group rules, network ACLs, and custom route tables.

You capp can also run in Virtual Private Cloud.

## Billing

**How much does AWS Elastic Beanstalk cost?**

No additional charges for EB; only pay for resources underneath.


