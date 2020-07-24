AWS Security and Encryption
===========================

3 Types of Encryptions
----------------------

**Encryption in flight (SSL)**

- Data is encrypted before sending and decrypted after receiving.
- SSL certificates are used for encryption in HTTPS protocol.
- Encryption in flight ensures no _Man in the middle attack_ can take place.

**Server side encryption at rest**

- Data is encrypted after being recieved by the server.
- Data is decrypted before being sent back.
- The encryption and decryption keys must be managed somewhere and server
  should have access to it.

**Client side encryption**

- Data is encrypted by the client and never decrypted by the server.
- Data is meant to be decryted after receving it.
- The server wouldn't know what it is; just simply store it (i.e. in S3
  bucket).
- Could leverage Ennvelope Encryption.

KMS Overview
------------

- **Key Management Service (KMS)**
- Whenever encryption is required in AWS, KMS is most likely invovled.
- It provides easy way to control access to your data as AWS manages the keys
  for us.
- Fully integrated with IAM for authorization.
- Seamlessly integrated into:
    - EBS : volume encryption.
    - S3 : server side encryption of objects.
    - Redshift
    - RDS
    - SSM : parameter store.
    - ...

KMS -- Customer Master Key Types
--------------------------------

**Symmetric (AES-256)**

- First offering of KMS; single encryption key that is used to Encrypt and
  Decrypt.
- AWS services that are integrated with KMS use Symmetric CMKs.
- Necessary for envelope encryption.
- Never get access to the Key unencrypted (must call KMS API to use).

**Asymmetric (RSA and ECC Keypairs)**

- Public and Private key-pair.
- Used for encryptions and signatures.
- Use case: encryption outside of AWS by usres who can't call the KMS API.

KMS -- Features
---------------

- Able to fully manage the keys and policies:
    - Create
    - Rotation policies
    - Disable
    - Enable
- Able to audit key usage (via CloudTrail)
- 3 Types of CMK:
    - AWS Managed Service Default CMK : free
    - User Keys created in KMS : $1 per month
    - User Keys imported (AES-256) : $1 per month
- Plus additional charges per API calls to KMS ($0.03 per 10000 calls)

KMS -- 101
----------

- Anytime that you need to share sensitive information, use KMS.
    - Database passwords
    - Credentials to external devices
    - Private Key of SSL certificates

- The value in KMS is that the CMK used to encrypt data can never be retrieved
  by the user and the CMK can be rotated for extra security.

- **Never store your secrets in plaintext (especially in code)**.
- Encrypted secrets can be stored in the code and environment variables.
- KMS can only help in encrypting upto 4KB of data per call.
- If data is greater than 4 KB, use envelope encryption.
- To give access to KMS to someone:
    - The Key Policy should allow the user.
    - The IAM Policy should allow API calls.

KMS -- Copying Snapshots across regions
---------------------------------------

Suppose we have a encrypted EBS volume with KMS key in one region. Since the
keys are tied to the specific region, first, you need to create the snapshot of
the encrypted volume with KMS. Copy over the snapshot but with specific
operation to KMS ReEncrypt with new KMS key in another region.

KMS -- Key Policies
-------------------

- Similar to S Bucket Policies that control access to the KMS Keys.
- Difference is that you cannot control access without them.

- **Default KMS Key Policy**:
    - Created if you do not provide a specific KMS Key Policy.
    - Give complete access to the key to the root user (entire AWS account).
    - Gives access to the IAM polocies to the KMS key.
- **Custom KMS Key Policy**:
    - Define usres, roles that can access the KMS key.
    - Define who can administer the key.

KMS -- Copying Snapshots across accounts
----------------------------------------

1. Create a Snapshot, encrypted with own CMK.
2. Attach a KMS Key Policy to authorize cross-account access.
3. Share the encrypted snapshot.
4. In target account, create a copy of the Snapshot, encrypting it with a KMS
   key in the account.
5. Create a volume from the snapshot.

KMS API -- Encrypt and Decrypt
------------------------------

Suppose we have a secret (password) less than 4 KB and wish to encrypt it.
Using CLI, we can send request via Encrypt API to KMS. KMS  will check the IAM
permissions, and if allowed, then will perform encryption for us using CMK and
return the encrypted data.

For decryption, same process applies.

Problem is that the data being encrypted and decrypted is limited to only less
than 4 KB. For this, we use _envelope encryption_.

KMS API -- Envelope Encryption
------------------------------

- Used to encrypt data that is more than 4 KB.
- `GenerateDataKey` API is used.
- It works by requesting the KMS to send the data key to encrypt the data with
  on the client side using the received key (DEK).
- Along with the encrypted file, we create a envelope that contains the
  encrypted version of DEK which is received from KMS by encrypting the EDK
  with CMK. 

KMS API -- Envelop Decryption
-----------------------------

- We have a envelopfile that contains the encrypted file and encrypted DEK.
- `DecryptAPI` is called to KMS and we pass along the encrypted DEK.
- KMS will check the IAM permissions, and if allowed, it will decrypted the DEK
  using the CMK and send back to us in plaintext form.
- Now, with DEK in plaintest form, it can be used on client side to decrypt the
  message.

Encryption SDK
--------------

- AWS Encryption SDK has envelope encryption feature.
- SDK is also available as CLI tool.
- Implmentations exist for Java, Python, C, JavaScript.
- Feature: **Data Key Caching**
    - re-use data keys instead of creating new ones for each encryption.
    - helpful to reduce the number of calls to KMS.
    - but it is a security trade-off.
    - use `LocalCryptoMaterialsCache` (define max age, max bytes, max # of
      messages).

KMS Symmetric -- API Summary
----------------------------

- `Encrypt` : encrypt upto 4 KB of data through KMS.
- `GenerateDataKey` : generates a unique symmetric data key (DEK).
    - returns both plantext and encrypted copy of DEK using CMK.
- `GenerateDataKeyWithoutPlaintext`
    - purpose is to use the DEK later (not immediately).
    - DEK is encrypted under the CMK that you specify (must decrypt DEK to use
      it).
- `Decrypt` : decrypt upto 4 KB of data (including DEK).

KMS Request Quotas
------------------

- When exceed a request quota, `ThrottlingException`.
- To remedy, try **exponential backoff** strategy.
- For cyptographic operations, they share a quota.
- This includes requests made by AWS on your behalf as well (i.e. SSE-KMS).
- For `GenerateDataKey`, consider using DEK caching from the Encrption SDK.
- Otherwise, request for Quotas increase through API or AWS support.

KMS and Lambda function example
-------------------------------

Suppose that we wish to create a Lambda function to access the database. We do
not want to store the database credentials in the code itself. Using
envrionment variable would be better, but what would be better is using the
encryption settings on the envrionment variables.

You can set encryption in transit as well as use KMS key to encrypt at rest.

Then in the code, grab the envrionment variable of encrypted password and
decrypt it using the AWS SDK (i.e. `boto3` for Python) into plaintext.

But you need to make sure that IAM policy has been properly set such that the
Lambda function can use `decrypt` API. i.e. attach an inline policy to Lambda
function that allows decrypt action on KMS.

S3 Ecnryption for Objects
-------------------------

There are 4 methods of encrypting objects in S3:

- SSE-S3 : encrypts using keys handled and managed by AWS.
- SSE-KMS : uses KMS to manage encryption keys.
- SSE-C : uses custom keys maanged by you.
- Client Side Encryption

_Important: know which one is used in which situations_.

SSE-KMS
-------

- SSE-KMS : encryption using keys handled and managed by KMS.
- KMS Advantages : user control and audit trail.
- Object is encrypted server side.
- Must set header : `"x-amz-server-side-encryption":"aws:kms"`.

SSE-KMS leverages the `GenerateDataKey` and `Decrypt` KMS API calls since the
objects being encrypted in S3 are likely greater than 4 KB. These API calls
will be logged in CloudTrail.

To perform SSE-KMS, you need:

- A KMS Key Policy that authorizs the user and role.
- An IAM Policy that authorizes access to KMS.
- Otherwise, access will be denied (error).

These S3 calls to KMS for SSE-KMS will count towards KMS limits.

- In case of throttling, try exponential backoff.
- If it does not improve, request to increase in KMS limits.
- Thus, the service throttling issue is not within S3 but in KMS.

S3 Bucket Policy -- Force SSL
-------------------------------

- To force SSL, create a S3 bucket policy with a **DENY** on the condition
  `aws:SecureTranspot = false`.

- Note: using an allow on `aws:SecureTransport = true` would allow anonymous
  `GetObject` if using SSL.

S3 Bucket Policy -- Force Encryption of SSE-KMS
-----------------------------------------------

- Deny incorect encryption header; make sure that it cludes `aws:kms`.

- Deny no encryption header to ensure objects are not uploaded un-encrypted.

---

SSM Parameter Store
-------------------

- Secure storage for configuration and secrets.
- Optional Seamless Encryption using KMS.
- Serverless, scalable, durable, easy SDK.
- Version tracking of configurations and secrets.
- Configuration management using path and IAM.
- Notifications with CloudWatch Events.
- Integration with CloudFormation.

SSM Parmeter Store Hierarchy
----------------------------

- A tree structure (i.e. just like `pass`).
- It is like a file system.

    /my-company/
        /my-app/
            /dev/
                /db-url
                /db-password
            /prod/
                /db-url
                /db-password
        /other-app/
            ...

- Reference as `/aws/reference/secretsmanager/secret_ID_in_Secrets_Manager`.

SSM Parameter -- Standard and Advanced Parameter tiers
------------------------------------------------------

- Standard stores 10,000 parameters; 100,000 for advanced.
- Max size is 4 KB for standard and 8 KB for advanced.
- Only advanced tier allows for Parameter policies.
- Free to use and store in standard; Advanced has fees.
- API interactions for standard only pays on higher throughput; advanced tier
  pays on normal API interactions as well $0.05 per 10,000 interactions.

SSM Parameter -- Parameter Policies (advanced parameter tiers)
--------------------------------------------------------------

- Allows for assignment of a TTL to a parameter (expiration date) to force
  updating or deleting sensitive data such as passwords.
- Can assign multiple policies at a time.

i.e. example of Expiration (delete a parameter) :

```json
{
    "Type":"Expiration",
    "Version":"1.0",
    "Attributes":{
        "Timestamp":"2020-12-02T21:24:22.000Z"
    }
}
```

i.e. example of ExpirationNotification (CW Events) : 

```json
{
    "Type":"ExpirationNotificatoin",
    "Version":"1.0",
    "Attributes":{
        "Before":"15",
        "Unit":"Days"
    }
}
```

i.e. example of NoChangeNofitication (CW Events) :

```json
{
    "Type":"NoChangeNotification",
    "Version":"1.0",
    "Attributes":{
        "After":"10",
        "Unit":"Days"
    }
}
```

SSM Parameter Store with Lambda
-------------------------------

```python
import boto3
import os

ssm = boto3.client('ssm', region_name='us-west-1')
devOrProd = os.environ['DEV_OR_PROD']

def lambda_handler(event, context):
    db_url = ssm.get_parameters(Names=["/my-app/" + devOrProd + "/db-url"])
    db_password = ssm.get_parameters(Names=["/my-app/" + devOrProd + "/db-password"], WithDecryption=True)
    # code
```

- Make sure that the Lambda function has the correct permissions to access the
  SSM parameters. Attach additional IAM policy that allows `GetParameters`.
- Also, when decrypting a secret that is encrpyt with KMS key, the Lambda
  function also requires an additional IAM policy to allow `decrpyt` API calls.
- Because of the tree like structure of parameters, you can easily change the
  parameters on the fly depending on the environment variable set within the
  funcion or stage.

---

AWS Secrets Manager
-------------------

- Newer service, meant for stroing secrets.
- Capability to force **rotation of secrets** every X days.
- Automate generation of secrets on rotation (uses Lambda).
- Integration with **Amazon RDS** (MySQL, PostgreSQL, Aurora).
- Secrets are encrypted using KMS.
- **Mostly meant to be used with RDS**.

SSM Parameter Store vs Secrets manager
--------------------------------------

**Secrets Manager** :

- Is costlier ($$$).
- Automatic rotation of secrets with AWS Lambda.
- Integration with RDS, Redshift, DocumentDB.
- KMS encryption is mandatory.
- Can integrate with CloudFormation.

**SSM Parameter Store** :

- Free tier avaialble and cheaper ($).
- Simplier API.
- No secret rotation.
- KMS encryption is optinal; allows for plaintext storage.
- Can integrate with CloudFormation.
- Can pull a Secrets Manager secret using the SSM Parameter Store API.

---

CloudWatch Logs -- Encryption
-----------------------------

- You may encrypt CloudWatch logs with KMS keys.
- Encryption is enabled at the log group level, by associating a CMK with a log
  group, either when you create the log group or after it exists.
- You cannot associate a CMK with a log group using the CloudWatch console.
- You must use the CloudWatch Logs API:
    - `associate-kms-key` : if the log group already exists.
    - `create-log-group` : if the log group does not exist yet.

Code Build Security
-------------------

- To access resources in your VPC, needs to specify a VPC configurations for
  your CodeBuild.

- Secrets in CodeBuild:
    - **Do not store them as plaintext in env variables**.
    - Instead, env variabls can reference parameter store or secrets manager.



