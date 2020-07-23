Amazon Cognito
==============

- Cognito is used to give users their identity so that they can interact with
  our applications.
- This is not the IAM users but the users outside of cloud and AWS.

**Cognito User Pools**:

- Sign-in functionality for app users.
- Integrate with API Gateway and Application Load Balancer.

**Cognito Identity Pools (Federated Identity)**:

- Provide AWS credentials to users so they can access AWS resources directly.
- Integrate with Cognito User Pools as an identity provider.

**Cognito Sync**

- Sychronize data from device to Cognito.
- Deprecated and replaced by AppSync.

**Cognito vs IAM**

- The keywords are: "hundreds of users", "mobile users", or "authenticate with
  SAML".

Cognito User Pools -- User Features
-----------------------------------

- _creates a serverless database of user for your web and mobile applications_.
- Simple login: username (or emial) and password combination.
- Password reset.
- Email and Phone Number verification.
- MFA.
- Federated Identities: users from Facebook, Google, SAML, ...
- Feature: block users if their crednetials are compromised elsewhere.
- Login sends back a JSON Web Token (JWT).

Cognito User Pools -- Integrations
----------------------------------

Cognito User Pools integrates with **API Gateway** and **Application Load
Balancer**.

i.e. User authenticate with Cognito User Pools to retrieve the token. Then,
makes REST API call to API Gateway with the token. API Gateway will verify the
user against the Cognito User Pools to allow for access.

i.e. User can send traffic to ALB that has Linsterns and Rules set-up such that
it will automatically try to authenticate the user on Cognito User Pools, then
route the traffic to target group if allowed.

Cognito User Pools -- Lambda Triggers
-------------------------------------

- It can invoke a Lambda function synchronously on various triggers.
    - Authentication Events
    - Sign-Up
    - Messages
    - Token Creation
- More details found
  [here](https://doc.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools-working-with-aws-lambda-triggers.html).

Cognito User Pools -- Hosted Authentication UI
----------------------------------------------

- Cognito features a _hosted authentication UI_ that you can add to your app to
  handle the signups and signin workflows.

- Using the hosted UI, you have a foundation for integration with social
  logins, OIDC or SAML.

- Can customize with a custom logo and custom CSS.

Cognito Identity Pools (Federated Identities)
---------------------------------------------

- Note: this is NOT same as Cognito User Pools.
- Get identities for "users" so they obtain temporary AWS credentials.
- The identity pool (i.e. identity source) can include:
    - Public Providers (i.e. Amazon, Facebook, Google, Apple).
    - Users in an Amazon Cognito user pool.
    - OpenID Connect Providers and SAML Identity Providers
    - Developer Authenticated Identities (custom login server).
    - Cognito Identity Pools allow for _unauthenticated (guest) access_.

- Once authenticated, users can access AWS services directly or through API
  Gateway.
    - IAM Policies applied to the credentials are defined in Cognito.
    - They can be customized based on the `user_id` for fine grained control.

i.e. Suppose web and mobile applications wishes to access the private S bucket
or DynamoDB table. To do so, they need AWS Credentials, but we do not want to
create an IAM user for the apps. First, we will have the apps login and get
token from either Social Identity Providers (such as Google or Facebook) or
Cognito User Pools. Then, apps will exchange their token for temporary AWS
credentials at Cognito Identity Pools.

Cognito Identity Pools -- IAM Roles
-----------------------------------

- Default IAM roles are applied to the authenticated and guest users.
- Define rules to choose the role for each user based on the user's ID.
- You can partition your users' access using **policy variables**.
- IAM credentials are obtained by Cognito Identity Pools through STS.
- The roles must have a "trust" polcy of Cognito Identity Pools.

i.e. Cognito Identity Pools - Guest User example:

```json
{
    "Version": "2020-10-10",
    "Statement": [
        {
            "Action": [
                "s3:GetObject"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s:::mybucket/assets/my_picture.jpg"
            ]
        }
    ]
}
```

i.e. Congito Identity Pools - Policy variable on S3:

```json
{
    "Version": "2020-10-10",
    "Statement": [
        {
            "Action": [
                "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s:::mybucket"
            ],
            "Condition": {"StringLike": {"s3:prefix": ["${cognito-identity.amazonaws.com:sub}/*"]}}
        },
        {
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s:::mybucket/${cognito-identity.amazonaws.com:sub}/*"]
        }
    ]
}
```

Cognito User Pools vs Identity Pools
------------------------------------

**Cognito User Pools**

- It is a database of users for your web and mobile application.
- Allows federation of log-ins through Public Social, OIDC, SAML, ...
- Can customize the hosted UI for authentication (including logo).
- Has triggers with AWS Lambda during the authentication flow.

**Cognito Identity Pools**

- Obtain AWS credentials for your users.
- Users can login through Public Social, OIDC, SAML and Cognito User Pools.
- Users can be unauthenticated (guest).
- Users are mapped to IAM roles and policies; and leverage policy variables.

In short, Cognito User Pools is for managing the users and passwords; Cognito
Identity Pools is for granting access to AWS services.

Cognito Sync
------------

- Deprecated now; use AWS AppSync.
- Store preferences, configuration, state of app.
- Cross device synchronization (any platform - iOS, Android, etc).
- Offline capability (synchronization when back online).
- Store data in datasets (upto 1 MB); upto 20 datasets to synchronize.
- Push Sync: silently notify across all devices when identity data changes.
- Cognito Stream: stream data from Congito into Kinesis.
- Cognito Events: execute Lambda functions in response to events.


