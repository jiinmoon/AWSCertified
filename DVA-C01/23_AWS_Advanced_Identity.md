AWS Advanced Identity
=====================

AWS STS
-------

- **Security Token Service (STS)**
- It allows to grant limited and temporary access to AWS resources (upto
  1 hour).
- `AssumeRole` : assume roles within your account or cross account.
- `AssumeRoleWithSAML` : return credentials for users logged with SAML.
- `AssumeRoleWithWebIdentity`
    - return credentials for users logged with an Identity Providers (Facebook,
      Google, OIDC, ...).
    - AWS recommends against using this; instead, use **Cognito Identity Pools**.
- `GetSessionToken` : for MFA, from a user or AWS account root user.
- `GetFederationToken` : obtain temp credentials for a federated user.
- `GetCallerIdentity` : return details about the IAM user or role used in the
  API call.
- `DecodeAuthorizatioMessage` : decode error mesage when an AWS API is denied.

Using STS to Assume a Role
--------------------------

- Define an IAM Role within your account or cross-account.
- Define which principals can access this IAM Role.
- Use AWS STS to retrieve credentials and impersonate the IAM Role that you
  have access to by calling `AssumeRole` API.
- Temporary credentials can be valid between 15 mins to 1 hour.

Using STS with MFA
------------------

- Use `GetSessionToken` from STS.
- Appropriate IAM policy using IAM Conditions.
- `aws:MultiFactorAuthPresent:true`
- `GetSessionToken` returns:
    - Access ID
    - Secret Key
    - Session Token
    - Expiration date

---

Advanced IAM -- Authorization Model
-----------------------------------

1. If there is an explict DENY, end decision and DENY
2. If there is an ALLOW, end decision with ALLOW
3. Else DENY

i.e. By default, all decisions will start at DENY. Then, all Policies will be
evaluated. If there is a explict DENY, then final decision is to DENY. If not,
then it will look for explict ALLOW. If there is not ALLOW, then final decision
still will be DENY.

IAM Policy and S3 Bucket Policy
-------------------------------

- IAM Policies are attached to users, roles and groups.
- S3 Bucket Policies are attached to buckets.
- When evaluating if an IAM Principal can perform an operation on a bucket, the
  **union** of its assigned IAM Policies and S3 Bucket Policies will be
  evaluated.

i.e. IAM Role attachd to EC2 instance grants RW to S3 bucket but no S3 Bucket
Policy is attached. In this case, EC2 instance can still perform RW to the S3
bucket.

i.e. IAM Role attached to EC2 instance, authorizes RW to bucket. Now, S3 bucket
policy is present with explict deny to the IAM Role. Here, DENY will overrule
and EC2 instance cannot read and write to bucket.

i.e. IAM Role attached to EC2 instance but does not have S3 bucket permissions.
S3 Bucket Policy is attached, explicitly RW ALLOW to the IAM Role. Result is
that EC2 instance is able to perform RW to the bucket.

Dynamic Policies with IAM
-------------------------

- How to assign each user a `/home/<user>` folder in an S3 bucket?
- Option 1:
    - Create an IAM policy allowing user, xyz, to have access to `/home/xyz`.
    - Repeat this process for each of the users.
- Option 2:
    - Create one dynamic policy with IAM.
    - Leaverage the special policy variable `${aws:username}`.

i.e. Dynamic Policy example:

```json
{
    "Sid": "AllowAllSActionsInUserFolder",
    "Action":["s3:*"],
    "Effect":"Allow",
    "resource": [
        "arn:aws:s3:::my-company/home/${aws:username}/*"
    ]
}
```

Inline vs Managed Policies
--------------------------

**AWS Managed Policy**

- Maintained by AWS.
- Good for power users and admins.
- Updated in case of new services and new APIs.
- i.e. `AWSAlexaFullControl`...

**Customer Managed Policy**

- Best practice; re-usable, can be applied to many principals.
- Version Controlled and rollback; central change management.
- i.e. ones that we create as we use AWS.

**Inline**

- Strict one-to-one relationship beween policy and principal.
- Policy is deleted if you delete the IAM principal.
- i.e. simple policy attached to a single user.

Granting a User Permissions to Pass a Role to an AWS Service
------------------------------------------------------------

- To configure many AWS services, you must pass an IAM role to the service
  (happens once during setup).
- The service will later assume the role and perform actions.
- Example of passing a role:
    - To an EC2 instance
    - To a Lambda function
    - To an ECS task
- To allow this, IAM permission `iam:PassRole` is required.
- It often comes with `iam:GetRole` to view the role being passed.

i.e. IAM PassRole example:

```json
{
    "Version": "2020-11-11",
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
            "Resource": "arn:aws:iam::123143245245:role/S3Access"
        }
    ]
}
```

- Roles may only be passed to what their **trust** allows.
- _A trust policy_ for the role that allows the service to assume the role.

i.e. When viewing the list of Roles, we can see that trust policies and trust
relationships with the roles.

Microsoft Active Directory
--------------------------

- Found on any Windows Server with AD Domain Services.
- Database of **objects**: User Accounts, Computers, Printers, File Shares,
  Security Groups.
- Centralized security management, create account, assign permissions.
- Objects are organized in trees; a group of trees is a forest.
- Multiple terminals connect in via single Domain Controller.

AWS Directory Services
----------------------

**AWS Managed Microsoft AD**

- Create your own AD in AWS; manage uers locally and supports MFA.
- Establish trust connections with your on-premise AD.
- On-premise AD and AWS Managed AD works together to serve users.

**AD Connector**

- Directory Gateway (proxy) to redirect to on-premise AD.
- Users are managed on the on-premise AD.
- Only on-premise AD has the database of users; auth requests are simply
  proxied from AD Connector.

**Simple AD**

- AD-compatible managed directory on AWS.
- Cannot be joined with on-premise AD.


