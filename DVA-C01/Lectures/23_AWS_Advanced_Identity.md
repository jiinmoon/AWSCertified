AWS Advanced Identity
=====================

AWS Security Token Service (STS)
--------------------------------

It allows to grant limited and temporary access to AWS resources (up to an
hour).

`AssumeRole` : assume roles withint your account or cross acount.

`AssumeRoleWithSAML` : return credentials for users logged with SAML.

`AssumeRoleWithWebIdentity`

- return credentials for users logged with an Identity Providers.
- recommeneded to use Cognito Identity Pools instead.

`GetSessionToken` : for MFA

`GetFederationToken` : obtain temp credentials for a federated user.

`GetCallerIdentity` : return details about the IAM user or role used in the API
call.

`DecodeAuthorizationMessage` : decode error message when an AWS API is denied.

Using STS to Assume a Role
--------------------------

- Define an IAM Role within your account or cross-account.
- Define which principals can access this IAM Role.
- Use AWS STS to retrieve credentials and impersonate the IAM Role that you
  have access to by calling `AssumeRole` API.
- Temp credentials can be valid between 15 mins to 1 hour.

Using STS with MFA
------------------

- Use `GetSessionToken` from STS.
- Appropriate IAM policy using IAM COnditions.
- `GetSessionToken` returns:
    - Access ID
    - Secret Key
    - Session Token
    - Expiration Date

---

Advanced IAM -- AUthorization Model
-----------------------------------

1. If there is an explict DENY, end decision and DENY

2. If there is an ALLOW, end decision with ALLOW

3. Else DENY

i.e. By default, all decisions wil lstart at DENY. Then, all policies will be
evaluated - if there is a explic DENY, then final decision is to DENY. If not,
then it will look for explict ALLOW. If there is not ALLOW, then final decision
still will be DENY.

IAM Policy and S3 Bucket Policy
-------------------------------

- IAM Policies are attached to users, roles and groups.
- S3 Bucket Policies are attached to buckets.
- When evaluating if an IAM Principal can perform an operation on a bucket, the
  **union** of its assigned IMA policies and S3 bucket policies will be
  evaluated.

Dynamic Policies with IAM
-------------------------

i.e. How to assign each user a `/home/<user-name>` folder in S3 bucket?

Create one dynamic policy with IAM and leverage the special policy variable
`${aws:username}`.

```json
{
    "Sid": "AllowAllS3ActionsInUserFolder",
    "Action": [ "s3:*" ].
    "Effect": "Allow",
    "Resource": [
        "arn:aws:s3:::myCompany/home/${aws:username}/*"
    ]
}
```

Inline vs Managed Policies
--------------------------

**AWS Managed Policy**

- Maintained by AWS.
- Good for power users and admins.
- Updated in case of new services and new APIs.
- This are preset and provided by AWS.

**Customer Managed Policy**

- Best practice; re-usable, can be applied to many principals.
- Version controlled and rollback.
- This is what we create as we use AWS.

**Inline**

- Strict one-to-one relationship between policy and principal.
- Policy is eleted if you delete the IAM principal.

Granting a User Permissions to Pass a Role to an AWS Service
------------------------------------------------------------

- To configure many AWS services, you must pass an IAM role to the service.
- The service will later assume th erole and perform actions.
- To allow this, IAM permission `iam:PassRole` is required.
- It often comes iwth `iam:GetRole` to view the role being passed.

```json
{
    "Version": "2020-10-10",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::1234567890:role/S3Access"
        }
    ]
}
```

- Role may only be passed to what their trust allows.
- A turst policy for the role allows the service to assume that role.

Microsoft Active Directory
--------------------------

- Found on any Windows Server with AD Domain Services.
- Database of **Objects** : User Accounts, Computers, Printers, File Shares,
  Security Groups.
- Centralized secuirty management, create account, assign permissions.
- Objects are organized in trees; a group of tree is called forest.

AWS Directory Services
----------------------

**AWS Managed Microsoft AD**

- Create your own AD in AWS; manage users locally and supports MFA.
- Establish trust connections with your On-Premises AD.
- On-Premises AD and AWS Managed AD works together to serve users.

**AD Connector**

- Directory Gateway (proxy) to redirect to On-Premises AD.
- Users are managed on the On-Premises AD.
- Only On-Premises AD has the database of users and auth requests are simply
  proxied from AD Connector.

**Simple AD**

- Ad-compatible managed directory on AWS.




