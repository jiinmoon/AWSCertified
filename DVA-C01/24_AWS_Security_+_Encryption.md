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

