AWS Advanced S3 and Athena
==========================

S3 -- MFA Delete
----------------

MFA enforces users to generate a code from their device before doing any
important operations on S3. **To use MFA-Delete, must enable versioning**.

It will require MFA to:

- permanently delete an object version.
- suspend versioning on the bucket.

However, you won't need MFA to:

- enabling versioning.
- listing deleted versions.

This can only be set by root account (bucket owner) using the CLI command:

        $ aws s3api put-bucket-versioning --bucket <bucket-name> --versioning-configuration Status=enabled,MFADelete=Enabled --mfa <arn-of-root-mfa-device> --profile <root-profile>

S3 -- Default Encryption vs Bucket Policies
-------------------------------------------

- The old way to enable default encryption was to use a bucket policy to DENY
  any HTTP command without proper headers.

- The new way is to simply use the "default encryption" option in S3.

S3 -- Access Logs
-----------------

- For the purposes of audit, all S3 access is logged.
- Any requests can be logged into another S3 bucket.
- This logs can be analyzed using data analysis tools or Amazon Athena.
- **Setting your own bucket as logging bucket will create a feedback loop**.

S3 -- Replication
-----------------

To replicate a bucket across one region to another ASYNC, we must first enable
versioning in source and destination. You can do this Cross-Region or
Same-Region and buckets can be from different accounts.

Copying process is ASYNC process.

It requires proper IAM permissions.

If activated, **only new objects are replicated**.

For DELETE:

- If you delete without a version ID, adds a delete marker - not replicated.
- If you delete with a version ID, deletes in the source - not replicated.

You cannot chain replication: if bucket 1 has replication to bucket 2 which has
replication into bucket3, then objects uploaded to bucket 1 does not get
replicated to bucket 3.

S3 -- Pre-signed URLs
---------------------

You may generate pre-signed URLs using SDK or CLI, but for uploads, you must
use SDK. The pre-signed URLs are by default valid for 3600 seconds, and its
timeout can be changed with `--expires-in <time-in-seconds>` option.

        $ aws configure set default.s3.signature_version s3v4
        $ aws presign <S3-bucket-url> [--expires-in <time>] --region <region>

S3 -- Storage Classes
---------------------

**S3 Standard - General Purpose (GP)**

- high durability of objects across multiple-AZs.
- can sustain 2 concurrent facilit failure.
- use cases:
    - big data anaylsis
    - gaming applications
    - content distribution

**S3 Standard - Infrequent Access (IA)**

- suitable for less frequently accessed data; but still reuqire rapid access
  when needed.
- use cases:
    - as a data store for disaster recovery
    - backups

**S3 One Zone - Infrequent Access**

- same as S3-IA, but on a single AZ.
- data is lost when AZ is lost.
- supports SSL for data at transit and encryption at rest.
- use cases:
    - storing secondary backup copies of on-premise data.
    - storing easily created data (i.e. thumbnails of images).

**S3 Intelligent Tiering**

- same low latency and throughput performance of S3 Standard.
- small monthyl monitoring and auto-tiering fee.
- automatically moves objects between two access tiers based on changing access
  atterns.

**Glacier and Glacier Deep Archive**

- low cost object storage for archiving and backup.
- meant for long term storage (~10s of years).
- each item in Glacier is called "Archive" (upto 40 TB) and stored in Vaults.

- Glacier has 3 retrieval options:
    - Expedited (1 to 5 mins)
    - Standard (3 to 5 hours)
    - Bulk (5 to 12 hours)
    - Min storage duration 90 days.

- Glacier Deep Archive:
    - Standard (12 hours)
    - Bulk (48 horurs)
    - Min storage duration 180 days.

S3 -- Moving between storage classes
------------------------------------

It is only possible to transition objects between storage classes downwards.
For example, from S3 Standard, you can move down to Glacier. But you cannot
move from Glacier to S3 One Zone IA.

Moving the objects to different tiers can be automated with
**Lifecycle Configurations**.

S3 -- Lifecycle Rules
---------------------

**Transition actions** define when objects are transitioned to another storage
class. For exmpale,

- move objects to Standard IA after 60 days of creation.
- move to Glacier for archiving after 6 months.

**Expiration actions** configures objects to be deleted after a set period.

- Access logs can be set to delete after an year.
- Can be used to delete old versions of files.
- Can be used to delete incomplete multi-part uploads.

These rules can apply using prefixes. For example, apply expiration actions
only on files with prefix `s3://myBucket/myFolder/`.

S3 -- Lifecycle Rules -- Scenario
---------------------------------

Suppose your app on an EC2 instance creates image thumbnails after profile
photos are uploaded to S3 bucket. These are easily recreated and only need to
be kept for 45 days. The source images should be immediately retrievable within
these 45 days. And afterwards, users can wait upto 6 hours. How do we design
this?

S3 source images can be on S3 Standard with a life cycle configuration attached
which transitions to Glacier after 45 days.

S3 thumbnails can be placed in S3 One Zone IA and set a life cycle to delete
automatically after 45 days.

S3 -- Baseline Performance
--------------------------

- S3 automatically scales to high request rates (latency 100 ~ 200 ms).
- An application can achieve at least 3,500 PUT, COPY, POST, DELETE and 5,500
  GET, HEAD requests per second per prefix in a bucket.
- No limit on the number of prefixes within the bucket.

S3 -- KMS Limitation
--------------------

- When using SSE-KMS, you should be aware of KMS limits.
- With each upload, it calls `GenerateDatkey` KMS API, and with each download,
  it calls `Decrypt` KMS API.
- **These API calls counts toward the KMS quota per second**.

S3 -- Performance
-----------------

**Multi-part Upload**

- Recommended for file size > 100 MB and is must for > 5 GB.
- Can help parallelize uploads to speed up the transfers.

**S3 Transfer Acceleration (Upload Only)**

- Increases transfer spped by transfering file first to an AWS Edge Location,
  where it will forward the data to target S3 Bucket.
- Compatible with multi-part upload.
- AWS Edge Location uses private AWS network which is more fast.

S3 -- Select and Glacier Select
-------------------------------

We may retrieve less data by performing an SQL with **server side filtering**.
It filters by rows and columns, which reduces the network transfer and saves
cost. Before you would retrieve all the data from S3 and filter in the client,
we change our request queries to filter - so that we will only retrieve the
necessary rows and columns.

S3 -- Event Notifications
-------------------------

We can set up rules to trigger an action which responds to a S3 event such as
`S3:ObjectCreated` or `S3:ObjectRemoved`. For example, we may want to
automatically create thumbnails whenever `.jpg` file is uploaded to a bucket.

For this, we can set up AWS Simple Notification Service (SNS), AWS Simple Queue
Service (SQS) and AWS Lambda function.

**Enable versioning!**

---

AWS Athena
----------

Athena is a **serverless** service that performs analytics directly on the S3
using SQL (has JDBC/ODBC driver).

- Charged per query and amount of data scanned.
- Supports CSV, JSON, ORC, Avro, Presto, ....

