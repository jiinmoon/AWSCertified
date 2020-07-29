Monitoring
==========

CloudWatch
----------

CloudWatch monitors AWS resources such as

- EC2 instances
    - ASG
    - ELB
    - Route53 Health Checks
- Storage and Content Delivery
    - EBS
    - Sotrage Gateways
    - CloudFront
- Databases and Anlytics
    - DynamoDB
    - ElastiCache
    - RDS instances
    - Redshift
- Else
    - SNS Topics
    - SQS Queues
    - CloudWatch Logs

CloudWatch -- EC2
-----------------

Measures metrics such as CPU, Network, Disk and Status Checks.

**RAM Utilization is a custom matric**.

**EC2 monitoring is done every 5 mins by default; detailed monitoring decreases
it to 1 min**.

Cloud Metrics are stored indefintely; can change the retention period for each
log group.

You can retrieve data from any terminated EC2 or ELB after its termination.

CloudWatch -- Alarms
--------------------

Alarms can be created to monitor any CloudWatch Metrics.

CloudWatch vs CloudTrail vs Config
----------------------------------

CloudWatch monitors performance.

CloudTrail monitors API calls.

AWS Config records the state of AWS environment and notifes.




