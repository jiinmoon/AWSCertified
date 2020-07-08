# Advanced S3 + Atehna

---

## S3 MFA Delete

- MFA forces user o generate a conde on a device before doing important
    operations on S3.

- To use MFA-Delete, enable Versioning on S3 bucket.
- Need MFA to:
    - perma delete an object version.
    - suspend versioning on the bucket.
- Do not need MFA to:
    - enabling versioning.
    - listing deleted versions.

- Only the bucket owner (root account) can enable/disable MFA-Delete.
- MFA-Delete currently can only be enabled using the CLI.

    > aws s3api put-bucket-versioning --bucket <bucket-name> --versioning-configuration Status=enalbed,MFADelete=Enabled --mfa <arn-of-root-mfa-device> --profile <use-root-credentials>

---

## S3 Default Encrpytion vs Bucket Policies

- The old way to enable default encryption was to use a bucket policy to refuse
    any HTTP command wothout proper headers.

- The new way is to simply use the "default encryption" option in S3.
- Note that bucket policies are evalustaed before default encryption.

---

## S3 Access Logs

- For audit purpose, log all access to S3.
- Any requests to S3, from any account, authorized or unauthorized, will be
  logged into another S3 bucket.
- This data can be analyzed using data analysis tools or *Amazon Athena*.

## S3 Acess Logs: Warning

- Never set your own bucket as a logging bucket!
    - This creates a feedback loop - grows exponentially.

- As simple as turning on the Server access logging settings on the main
  bucket, and choose logging bucket as a target. (remember to turn Versioning
  on the main).

---

## S3 Replication (CRR & SRR)

- We wish to replicate a bucket in one region to another asynchronously.

- To d oso, must enable versioning in source and dest.
- Cross Region Replication
- Same Region Replication
- Buckets can be in different accounts
- Copying is asynchronous
- Must give proper IAM permissions to S3!

- CRR use cases would be for compliance, lower latency access, replication
  across acounts.

- SRR use cases are log aggregation, live replication between prouction and
  test accounts.

## S3 Replication Notes

- After activating, ONLY NEW OBJECTS are replicated (not retroactive).

- For DELETE Operations:
    - if you delete without a version ID, adds a delete marker, not
      replicated.
    - if you delete with a version ID, deletes in the source, not replicated.

- There is no 'chaining' of replication.
    - If bucket 1 has replication into bucket 2 which has a replication into
      bucket3.
    - Then objects created in bucket 1 does not replicated to bucket 3.

---

## S3 Pre-signed URLs

- Can generate presigned URLs using SDK or CLI.
    - for downloads, CLI works fine.
    - for uploads, must use SDK.

- Valid for a default of 3600 seconds, can change timeout with --expires-in
  <time-in-seconds> argument.

- Users given a pre-signed URL inherit the permissions of the person who
  generated the URL for GET / PUT requests.

- i.e.
    - allow only logged-in users to download a premium video on  your S3
      bucket.
    - allow a changing list of users to download files by generating URLs
      dynamically.
    - allow an user to temporary upload a file to a precise location in bucket.

    > aws configure set default.s3.signature_version s3v4

    > aws presign <S3Url> [--expires-in <value>] --region <region>

---

## S3 Storage Classes

- S3 Standard - General Purpose (GP)
- S3 Standard - Infrequent Access (IA)
- S3 One Zone Infrequent Access
- S3 Intelligent Tiering
- Glacier
- Glacier Deep Archive
- S3 Reduced Redundancy Stroage (deprecated).

## S3 Standard General Purpose

- High durability of objects across multiple AZ.
- If 10 million objects are stored in S3, on average you expect to lose
  a single object every 10,000 years.
- 99.99% Availability over a given year.
- Sustain 2 concurrent facility failure.

- Use Cases:
    - big data analysis
    - gaming applications
    - content distribution

## S3 Standard Infrequent Access

- Suitable for data that is less frequently accessed, but still require rapid
  access when needed.
- Same durability across multiple AZs.
- 99.9% Availiability.
- Lower than GP.

- Use cases:
    - as a data store for disaster recovery.
    - backups.

## S3 One Zone - Infrequent Access

- Same as IA but in a single AZ.
- Same durability, but data is lost when AZ is lost.
- 99.5% Availability.
- Low latency and high throughput performance.
- Supports SSL for data at transit and encryption at rest.
- Low cost compared to IA by 20%.

- Use cases:
    - storing secondary backup copies of on-premise data.
    - storing data you can recreate easily.
        - i.e. store the images on general purpose, but thumnails can be
          generated easily so we store it in One Zone IA.

## S3 Intelligent Tiering

- Same low latency and throughput performance of S3 standard.
- Small monthly monitoring and auto-tiering fee.
- Automatically moves objects between two access tiers based on changing access
  patterns.
- Designed for durability acorss multiple AZs.
- Resilient against events that impact an entier AZ.
- Designed for 99.9% availiability over a given year.

## Glacier

- Low cost object storage meant for archiving / backup.
- Data is retained for the long term (~10s of years).
- Alternateive to on-premise magnetic tape storage.
- Very low cost to store per month + retrieval cost.
- Each item in Glacier is called "Archive" (upto 40TB).
- Archives are stored in Vaults.

## Glacier && Glacier Deep Archive

- Glacier has 3 retrieval options:
    - Expedited (1 to 5 mins)
    - Standard (3 to 5 hours)
    - Bulk (5 to 12 hours)
    - Min storage duration of 90 days.

- Glacier Deep Archive - for longer term stroage; cheaper.
    - Standard (12 Hours)
    - Bulk (48 Hours)
    - Min storage duration of 180 days.

---

## S3 - Moving between stroage classes

- It is possible to transition objects between storage classes.
- i.e. Standard can change to all 5 types (IA, One Zone IA, IT, Glacier, Deep
  Archive). But as we move down, we cannot transition up. IA cannot be move up
  to Standard, IT cannot move to IA, One Zone IA cannot move to IT or IA,
  Glacier can only move down to Deep Archive...etc.

- Moving the objects can be automated via **lifecycle configuration**.

## S3 Lifecycle Rules

- Transition actions: it defines when objects are transitioned to another
  storage class.
  - move objects to Standard IA 60 days after creation.
  - move to Glacier for archiving after 6 months.

- Expiration actions: configure objects to expire (delete) after some time.
    - Access log files can be set to delete after an year.
    - Can be used to delete old versions of files (if versioning is enabled).
    - Can be used to delete incomplete multi-part uploads.

- Rules can be created for a certain prefix.
    - i.e. apply rules only to s3://bucketname/mp3/...
- Rules can be created for a certain tags as well.

## S3 Lifecyle Rules - Scenario I

- Your app on EC2 creates images thumbnails after profile photos are uploaded
  to S3. These thumbnails can be easily recreated, and only need to be kept for
  45 days. The source images should be able to be immediately retrieved for
  these 45 days, and afterwards, the user can wait up to 6 hours. How would you
  design this?

- S3 source images can be on Standard with a life cycle configuration to
  transition them to Glacier after 45 days.
- S3 thumnails can be on One Zone IA - as we can recreate them easily; then set
  life cycle to delete after 45 days.

## S3 Lifecycle Rules - Scenario II

- A rule in your company states that you should be able to recover your deleted
  S3 objects immediately for 15 days, although this may happen rarely. After
  this time, and for upto an year, deleted objects should be recoverable within
  48 hours.

- First, we enable versioning to have object versions such that 'deleted
  objects' are in fact just hidden by a 'delete marker' and can be recovered
  easily.
- We can transition these noncurrent versions of the object to S3 IA. Since we
  rarely access them; but when we do, we still need to access them asap.
- Then, we transition them to Deep Archive.

---

## S3 Baseline Performance

- S3 automatically scales to high request rates, latency 100~200 ms.
- Your app can achieve at least 3500 PUT/COPY/POST/DELETE and 5500 GET/HEAD
  request per second per prefix in a bucket.
- No limits to the number of prefixes in a bucket.
- i.e. object path == prefix:
    - bucket/folder1/sub1/file == prefix is /folder1/sub1/
    - bucket/folder1/sub2/file == prefix is /folder1/sub2/
    - bucket/1/file == prefix is /1/
    - bucket/2/file == prefix is /2/
- If you can spread reads across all prefixes evenly, you can achieve 22,000
  request per second for GET and HEAD.

## S3 KMS Limitation

- If you use SSE-KMS, you may be impacted by the KMS limits.
- When you upload, it calls GenerateDataKey KMS API.
- When you dwonload, it calls Decrypt KMS API.
- This counts toward the KMS quota per second.
    - (5500, 100000, 300000 req/s based on region)
- You cannot request for quota increase for KMS!

## S3 Performance

- Multi-Part upload:
    - recommended for files greater than 100 MB; must for over 5 GB.
    - can help parallelize uploads (speeds up transfers).

- S3 Transfer Acceleration (upload only)
    - increases transfer speed by transferring file to an AWS edge location
      which will forward the data to the S3 bucket in the target region.
    - compatible with multi-part upload.
    - edge location leverages existing private, fast AWS network.

## S3 Performance - S3 Byte-Range Fetches

- Parallelize GETs by requesting specific byte ranges.
- Better resilience in case of failures.

- Can be used to speed up downloads.
- Can be used to retrieve only partial data (i.e. head or less of a file).

---

## S3 Select & Glacier Select

- Retrieve less data using SQL by performing **server side filtering**.
- Can filter by rows & columns (simple SQL statements).
- Less network transfer, less CPU cost client-sdie.

- The idea is this, before, you would retrieve all the data from S3 and filter
  client-side. Instead, we change our request that our queries are already
  filtered. Thus, we will only get the rows and columns we need; not entire
  thing.

- i.e. clinet performs S3 Select to ge ta CSV file. S3 on server side performs
  filtering on the file and sends only the parts that we need.

---

## S3 Event Notifications

- Say an event has occurred to our S3 bucket (S3:ObjectCreated,
  S3:ObjectRemoved, ...).
- We can create rules to trigger an action.
- We can filter object names as well (such as all files end with .jpg).
- One use case would be generate thumnails of images uploaded to S3.

- We have AWS SNS - email service, and SQS - simple queue service, and lambda
  function to trigger the events.

- Enable versioning first; then we can create events.

---

## AWS Athena

- It is a **Serverless** service to perform analyics directly against S3 files. 
- Uses SQL language to query the files; and has a JDBC/ODBC driver.
- Charged per query and amount of data scanned.
- Supports CSV, JSON, ORC, Avro, and Parquet (Presto).

- Uses cases are business intelligence / analytics / reporting, analyze & query
  VPC Flow Logs, ELB Logs, CloudTrail trails, ...

- Exam tips: How to analyze data directly on S3? Use Athena.

- First, we create the database -> and create the tables.
- This is better explained on AWS document page: how to access my logs from AWS
  Athena.

- This gives us various statics; you can query to see which request url was
  used or how many requests failed and received 400...and so on.

---

## S3 Object Lock & Glacier Vault Lock

- S3 Object Lock
    - Adopt a WORM model (Write Once Read Many) model.
    - Block an object version deletion for a specified amount of time.

- Glacier Vault Lock
    - Adopt a WORM model.
    - Lock the polucy for future edits (can no longer be changed).
    - For compliance reasons - company wishes to upload data on a bucket that
      gurantees no one can delete it.

- In other words, the object inside the vault cannot be deleted.

