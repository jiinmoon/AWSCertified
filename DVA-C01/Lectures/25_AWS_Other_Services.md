AWS Other Services
==================

AWS Simple Email Service (SES)
------------------------------

Send emails to people using SMTP interface and SDK. It can also receive email
which integrates with S3, SNS, or Lambda.

Interates with IAM for allowing to send emails.

Summary of AWS Databases
------------------------

**RDS** : Relational database

- PostgreSQL, MySQL, Oracle.
- Aurora and Aurora Serverless.
- Provisioned database.
- Only scales vertically.

**DynamoDB** : NoSQL database

- Managed, Key-Value, Document.
- Serverless.
- Scales horizontally.

**ElastiCache** : In-memory database

- Redis or Memcached.
- Provisioned.

**Redshift** : OLAP for Analytic Processing

- Data Warehousing and Data Lake.
- Analytics queries.

**Neptune** : Graph database

**DMS** : Database migration service

**DocumentDB** : managed MongoDB for AWS

Amazon Certificate Manager (ACM)
--------------------------------

- To hosts public SSL/TLS Certificates in AWS,
    - Buy own and upload using the CLI.
    - Or, have ACM provision and renew certificates at free of charge.

- ACM loads SSL certificates on following:
    - Load Balancers.
    - CloudFront.
    - APIs on API Gateways.

- SSL Certificates is overall difficult to manually manage; leverage ACM as
  much as possible.


