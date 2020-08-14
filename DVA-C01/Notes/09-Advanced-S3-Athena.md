# Advanced S3 Topics

## S3 MFA Delete

- We can enable MFA on S3 so that the users will ahve to generate a code on
  a device before doing any operations on S3.

- To use MFA-delete, **enable S3 versioning**.

- Once enabled, MFA will be required to
    - permanently delete an object version
    - suspend versioning on the bucket

- You do not ned MFA for
    - enabling versioning
    - listing deleted versions

- Note that **only root can enable/disable MFA-deletes**.

```
aws s3api put-bucket-versioning 
            --bucket <bucket-name>
            --versioning-configuration Status=Enabled,MFADelete=Enabled 
            --mfa <arn-of-MFA-device> --profile <aws-profile>
```

## S3 Default Encrption vs Bucket Policies

- Old way to enable default encrytion was to use a bucket policy and refuse any
  HTTP command without the matching headers for encryption.

```
"Effect": "Deny",
"Principal": "*",
"Action": "s3:PutObject",
"Resource": <arn-of-bucket>,
"Condition": }
    "StringNotEquals": {
        "s3:x-amz-serverside-encryption": "AES256"
    }
}
...
```

- Newer way is to use the default encryption option in the S3.
- Note that bucket policies will take precedence over the default encryption.

## S3 Access Logs

- For audit purpose you may want to log all access to S3 buckets.

- Any requests made to S3, from any account, authorized or denied; all can be
  logged into another S3 bucket that is designated as a logging bucket.

- These logs can be further analyzed using Athena.

- **Note that you should not set your bucket itself as a logging bucket**.
  Doing so will create a feedback loop that will grow exponentially in size and
  cost.

## S3 Replication (Cross Region or Same Region)

- Must enable versioning in order to replicate in both source and destination
  buckets.

- You can either replicate within a region or cross regions.
- Buckets can even be in different accounts as well.

- Copying process is ASYNC.
- S3 requires proper permissions to perform this action.

- After activated, only the new objects are replicated.
- For DELETE operations:
    - if delete without version ID, it only adds delete marker (not
      replicated).
    - if delete with version ID, it deletes in the source (not replicated).

- Replication is not "chained"; If A -> B and B -> C; then putting object in
  bucket A does not make it replicate all the way down to C.

## Presigned URLs

- Pre-signed URLs are created with either SDK or CLI for both
  downloads/uploads.
    - Uploads must use SDK.

- This URL will be valid for 300 seconds by default; but can change by using
  `--expires-in <time-in-seconds>` option.

- Users that have this URL will inherit the permissions of the user who created
  the URL.

```
aws s3 presign s3://<bucket-name>/<object-key> --expires-in 300 --region <region>
```

## S3 Storage Classes
### S3 Standard - General Purpose

- High durability across multi-AZ.
- 99.99% availability over a given year.
- Can sustain 2 concurrent facility failures.

### S3 Standard - Infrequent Access

- Suitable for infrequently accessed data but still require fast access when
  eneeded.

### S3 One Zone - Infrequent Access

- Same as Standard IA but hosted only on a single AZ.
- 99.5% availability.
- Supports HTTPS for data in transit and encryption at rest.

### S3 Intelligent Tiering

- Automatically moves objects between two access tiers based on changing access
  patterns.
- Small monthly monitoring and auto-tiering fee.
- Multi-AZ.

### Amazon Glacier and Glacier Deep Archive

- Low cost storage for archiving and backups.
- Data is retained for the longer term (in decades).
- Alternative to magnetic tape on-premise.
- There is a retrieval cost.
- Each item is "Archive" ~40 TB which are stored in "vaults".

- Glacier retrieves in:
    - Expedited (1 to 5 min)
    - Standard (3 to 5 hours)
    - Bulk (5 to 12 hours)
    - Min storage duration of 90 days

- Deep Archive retrieves in:
    - Standard (12 hours)
    - Bulk (48 hours)
    - Min storage duration of 180 days

## S3 LifeCycle - Moving between classes

- You can transition objects between storage classes automatically.

- For example, you can decide to arhive objects in Standard class to One Zone
  or Glacier after certain number of days.

- Transition actions define when objects are transitioned to another storage
  class.

- Expiration actions define when objects are deleted.

- These rules can be created for a certain glob (i.e.
  `s3://<bucket-name>/*.jpg`).

- These rules can also be applied to objects with certain tags.

## S3 Performance
### Baseline Performance

- S3 automatically scales to high request rates (~200 ms).
- Application can achieve at least 3,500 PUT/COPY/POST/DELETE and 5,500
  GET/HEAD request per second per prefix within a bucket.

- No limits on amount of prefixes in the bucket.

### SSE-KMS Limits

- If using SSE-KMS, there is a KMS limit to be aware of.
- Each encryption and decryption is an API call to the KMS (`GenerateDataKey`
  and `Decrypt`).
- This will count towards the KMS quota per second.
- There is no quota increase request for KMS.

### Multi-Part Upload and S3 Transfer Acceleration

- Multi-Part Upload
    - It is a must-use for any objects greater than 5 GB.
    - But it is recommended for objects greater than 100 MB.
    - It can be parallelized.

- S3 Transfer Acceleration
    - It is used to speed up the upload process.
    - Does so by leveraging on the edge locations and internal low bandwitdh
      AWS network.

## S3 Select and Glacier Select

- It is a service offered to "qeury" using SQL to perform server-side
  filtering.
- So, you will fetch less data to save cost.

## S3 Event Notifications

- We can create event notification whenever S3 action is triggered.
- For example, we can detect whenever an image is uploaded to the S3 and 
  a lambda function to create a thumbnail.

- It can target SNS, SQS, or Lambda function.

- It is possible that when two writes made to a single object in
  a non-versioned bucket, only one event will be sent.

- So, **enable versioning** to avoid this.

## Athena

- It is a serverless service to perform analytics directly against S3.
- Uses SQL query.
- Has a JDBC/ODBC driver.
- Charge per query and amount of data scanned.

## S3 Object Lock and Glaicer Valut Lock

- S3 Object Lock
    - Blocks an object version deletion for a specified amount of time

- Glacier Valut Lock
    - Locks the policy for future edits (can no longer be changed)


