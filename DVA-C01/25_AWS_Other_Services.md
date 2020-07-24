AWS Other Services
==================

Quick overveiw on services that maybe on the exam.

AWS SES -- Simple Email Service
-------------------------------

- Send emails to people using:
    - SMTP interface
    - AWS SDK

- Ability to receive email; integrates with.
    - S3
    - SNS
    - Lambda

- Integrates with IAM for allowing to send emails.

Summary of AWS Databases
------------------------

**RDS** : relational databases, OLTP

- PostgreSQL, MySQL, Oracle
- Aurora and Aurora Serverless
- Provisioned database
- Can only scale vertically

**DynamoDB** : NoSQL db

- managed, key-value, document
- serverless
- scales horizontally

**ElastiCache** : in-memory db

- Redis or Memcached
- Cache capability
- Provisioned

**Redshift** : OLAP - Analytic Processing

- Data Warehousing and Data Lake
- Analytics queries

**Neptune** : Graph Database

**DMS** : Database Migration Service

**DocumentDB** : managed MongoDB for AWS

Amazon Certificate Maanger (ACM)
--------------------------------

- To host public SSL certificates in AWS, you can:
    - Buy own and upload using the CLI.
    - Or, have ACM provision and renew public SSL at free of charge.

- ACM loads SSL certificates on the following integrations:
    - Load Balancers
    - CloudFront
    - APIs on API Gateways

- SSL certificates is overall difficult to manually manage; leverage ACM when
  possible.


