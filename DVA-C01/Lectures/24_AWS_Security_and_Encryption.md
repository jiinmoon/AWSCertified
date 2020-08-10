AWS Security and Encryption
===========================

3 Types of Encryptions
----------------------

**Encryption in-flight (SSL)**

- Data is encryped before sending and decrypted after receiving.
- Prevents the man in the middle attack.
- SSL/TLS Certificates are needed.

**Server-Side Encryption at rest**

- Data is encrypted after being recieved by the server.
- Data is decrypted before sending back to the user.

**Client-Side Encryption**

- Data is encrypted by the client and never decrypted in server.
- Data is meant to be decrypted after receiving it from the server.
- Works with Envelope Encryption.

AWS Key Management Service (KMS)
--------------------------------

- Whenever encryption is required in AWS, KMS is involved.
- It provides easy way to control access to your data as AWS manages the keys
  for us.

KMS -- Customer Master Key Types
--------------------------------

**Symmetric (AES-256)**

- Single encryption key that is used to encrypt and decrypt.
- AWS services integrated with KMS use Symmetric CMKs.
- Necesary for envelope encryption.
- Never get access to the key unencrypted (require KMS API call).

**Asymmetric (RSA and ECC Key-pairs)**

- Pub/Private key-pairs.
- Used for encryptions and signatures.
- Use case: encryption outside of AWS by users who cannot call the KMS API.

KMS -- Features
---------------

Able to fully manage the keys and policies (Create, Rotate, Disable, and
Enable).

Able to audit key usage by CloudTrail.

3 Types of CMK:

- AWS managed : default, free.
- User keys created in KMS : $1 per month.
- User keys imported (AES-25) : $1 per month.

It charges additioanl per API calls to KMS ($0.03 per 10,000 calls).

KMS -- 101
----------

Anytime that you need to share sensitive information, use KMS.

- Database passwords.
- Credentials to external devices.
- Private key of SSL certificates.

**Never store your secrets in plaintext**.

Encrypted secrets can be stored in the code and environment variables.

KMS can help encrypting up to 4 KB of data per call; thus, if more is required,
then use Envelope ENcryption.

KMS -- Copying Snapshots across regions
---------------------------------------

Suppose we have aencrypted EBS volume with KMS key in one region. Since the
keys are tied to a single region, first you need to create the snapshot of the
encrypted volume with KMS. Copy over the snapshot bu with specific operations
to KMS ReEncrypt with new KMS key in that region.

KMS -- Key Policies
-------------------

Similar to S3 Bucket Policies that control access to the KMS Keys - difference
is that you cannot control access without them.

**Default KMS Key Policy**

- Created if you do not provide a specific KMS Key Policy.
- Give complete access to the key to the root user.
- Gives access to the IAM policies to the KMS key.

**Custom KMS Key Policy**

- Define users and roles that can access the KMS key.
- Define who can administer the key.

KMS -- API for Encrypt and Decrypt
----------------------------------

Suppose we have a secret less than 4 KB for encryption. Using the CLI, we can
send request via Encrypt API to KMS. KMS will check the IAM permissions, and
perform encryption if allowed.

KMS -- API for Envelope Encryption
----------------------------------

Now, the data is greater than 4 KB. In this case, `generateDataKey` API is used
instead. it works by requesting the KMS to send the data key to encrypt the
data with on the client side (DEK). Along wiht the encrypted file, we create
a envelope that contains the encrypted version of DEK as well which is received
from KMS (which is encrypted DEK using CMK).

KMS -- API for Envelope Decryption
----------------------------------

The envelope should contain the encrypted file and encrypted DEK. `DecryptAPI`
is called with the encrypted DEK, so that KMS can decrypt the DEK for us. With
decrypted DEK, it is now used to decrypt the entire file.

AWS Encryption SDK
------------------

- AWS Encryption SDK has envelope encryption feature.
- It is available as a CLI tool.
- It features Data Key Caching:
    - It reuses data keys instead of creating new ones for each encryption.
    - Helpful to reduce cost at security trade-off.

KMS -- Symmetric API Summary
----------------------------

- `Encrypt` : up to 4 KB of data through KMS using CMK.
- `GenerateDataKey` : generates unique symmetric data key (DEK).
- `GenerateDataKeyWithoutPlaintext`
    - purpose is tu use the DEK later.
    - note that `GenerateDataKey` returns both plaintext and encrypted DEK.
- `Decrypt` : up to 4 KB of data (including DEK).

KMS -- Rquest Quotas
--------------------

- When request quota is exceeded, `ThrottlingException` is raised.
- Retry with exponential back-off strategy.
- For cryptographic operations, they share a quota.
- This includes request made by AWS on your behalf as well (SSE-KMS).
- For `GenerateDataKey` consider data key caching.
- Otherwise, ask for quota increase.

---

S3 Encryption Models for Objects
--------------------------------

- SSE-S3 : encrypts with keys managed by S3.
- SSE-KMS : uses KMS to manange encryption keys.
- SSE-C : uses custom keys managed by the user.
- Client-Side Encryption.

S3 Encryption -- SSE-KMS
------------------------

SSE-KMS provides an advantage of user contorl and audit trail. To enable this,
header must be set to `"x-amz-server-side-encryption":"aws:kms"`.

SSE-KMS leverages on the `GenerateDataKey` and `Decrypt` API calls.

To perform SSE-KMS,

- A KMS Key Policy must authorizes the user and the role.
- An IAM Policy that authorizes access to KMS.

These S3 calls to KMS for SSE-KMS counts toward KMS limits.

S3 Encryption -- Force SSL
--------------------------

To enforce SSL, create a S3 bucket policy with a explicit **DENY** on condition
`aws:SecureTransport = false`.

S3 Encryption -- Force Encryption of SSE-KMS
--------------------------------------------

Explicitly DENY incorrectly set encryption header; making sure that it has
`aws:kms`. Also, DENY no encryption header to make sure that no files are
uploaded unencrypted.

---

AWS SSM Parameter Store
-----------------------

It is a secure Storage for configuration and secrets. It is serverless,
scalable, durable, and easy SDK.

SSM Parameter Store -- Hierarchy
--------------------------------

It has a tree structure just like a file system.

    /myCompany/
        /myApp/
            /Dev/
                /db-url
                /db-password
            /Prod
                /db-url
        /otherApp/
            ...

SSM Parameter Store -- Standard and Advanced Tiers
--------------------------------------------------

- Standard stores 10,000; 100,000 for advanced.
- Max size if 4 KB for standard; 8 KB for advnaced.
- Only advanced tier allos for parameter policies.
- Free to use and store in standard.
- API interactions for standard only pays on higher throughput.
- Advanced tier pays for normal interactions as well ($0.05 per 10,000 calls).

SSM Parameter Store -- Parameter Policies
------------------------------------------

- Allows for assignment of a TTL to a parameter to force updating or deleting
  sensitive data such as passowrds.

SSM Parameter Store -- Lambda
-----------------------------

```python
import boto3
import os

ssm = boto3.client('ssm', region_name='us-west-1')
dev_or_prod = os.environ['DEV_OR_PROD']

def lambda_handler(event, context):
    db_url = ssm.get_parameters(Names=['/myApp/' + dev_or_prod + '/db-url'])
    db_password = ssm.get_parameters(
        Names=['/myApp/' + dev_or_prod + '/db-password'], 
        WithDecryption=True)
    # code
```

- Make sure that the Lambda function has the correct permissions to access the
  SSM parameters. Attach additional IAM policy that allows `GetParameters`.
- Also, when decrypting a seret that is encrypted with KMS key, Lambda will
  require an additional IAM policy to call `decrypt` KMS API call.

---

AWS Secrets Manager
-------------------

- Newer service used for storing secrets.
- Capability to force rotation of secrets every X days.
- Automate gneeration of secrets on rotation (uses Lambda).
- Integrates with Amazon RDS.
- Secrets are encrypted using KMS.
- Mostly meant to be used with RDS.

SSM Parameter Store vs Secrets Manager
--------------------------------------

**Secrets Manager**

- It is costlier.
- Enforces rotation of secrets with the AWS Lambda.
- Integration with RDS, Redshift, and DocumentDB.
- KMS encryption is mandatory.
- Can integrate with CloudFormation.

**SSM Parameter Store**

- Free tier is available and is cheaper.
- Simpler API and no secrets rotation.
- KMS encryption is optional; allows plaintext storage.
- Can integrate with CloudFormation.

---

CloudWatch Logs -- Encryption
-----------------------------

- You may encrypt CloudWatch logs with KMS keys.
- Encryption is enabled at the log group level; by asociating a CMK with a log
  group either when you create the log group or after it exists.
- You cannot associate a CMK with a log group with console; must use CLI:
    - `associate-kms-key` if log group exists.
    - `create-log-group` if not.
