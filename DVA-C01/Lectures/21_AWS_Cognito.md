AWS Cognito
===========

Cognito is used to give users their identity so that they can interact with our
applications. This is NOT the IAM users, but users outside of cloud and AWS.

**Cognito User Pools**

- Sign-in functionalit for application users.
- Intgrate with API Gateway and Application Load Balancer.

**Cognito Identity Pools (Federated Identity)**

- Provide AWS credentials to users to they can access AWS resources directly.
- Integrate with Cognito user Pools as an identity provider.

**Cognito Sync**

- Synchronize data from device to Cognito.
- Deprecated and replaced by AppSync.

**Cognito vs IAM**

- Keywords are "hundreds of users", "mobile usres", or "authenticate with
  SAML".

Cognito User Pools -- User Features
-----------------------------------

- Creates a serverless database of users for your web and applications.
- Simple login using username (or email) and password.
- MFA is available.
- Federated Identities from Facebook, Google, SAML, ...
- Can block users if they are blocked elsewhere.
- Login sends back a JSON Web Token (JWT).

Cognito User Pools -- Integrations
----------------------------------

Cognito User Pools integrates with API Gateway and ALB.

i.e. User authenticate with Cognito User Pools to retrieve the token, then
makes REST API call to API Gateway with the token. API Gateway will validate
the token against the Cognito User Pools to allow for access.

i.e. User can send traffic to ALB that has listeners and rules set-up such that
it will automatically try to authenticate the user on Cognito User Pools, then
route the traffic to the target group.

Cognito User Pools -- Lambda Triggers
-------------------------------------

It can invoke a Lambda function synchronously on various triggers:

- Authentication Events.
- Sign-up.
- Messages.
- Token Creation.

Cognito User Pools -- Hosted Authentication UI
----------------------------------------------

- Cognito features a hosted authentication UI that you can add to your app to
  handle the sign-ups and signin workflows.

- Using the hosted UI, you havea foundation for integration with social logins,
  OIDC or SAML.

- Can customize with a custom logo and custom CSS.

Cognito Identity Pools (Federated Identities)
---------------------------------------------

- This is not same as User Pools.
- Get identities for "usres" so they obtain temp AWS credentials.
- The identity poll can include:
    - Public Providers (Google, Facebook, ...).
    - Users in an Amazon Cognito User Pools.
    - OpenID Connect Providers and SAML Identity Providers.
    - Developer Authenticated Identities.
    - Cognito Identity Pools allow for unauthenticated guest access.

- Once authenticated, users can access AWS services directly or through API
  Gateway.
    - IAM policies applied to the credentials are defined in Cognito.
    - They can be customized based on the `user_id` for fine control.

Cognito Identity Pools -- IAM Roles
-----------------------------------

- Default IAM roles are applied to the authenticated and guest users.
- Define rules to choose the role for each user based on the user's ID.
- You can partition your users' access using **policy variables**.
- IAM credentials are obtained by Cognito Identity Pools through STS.
- The role must have a "trust" policy of Cognito Identity Pools.

i.e. Cognito Identity Pools -- Guest User example:

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
                "arn:aws:s3:::mybucket/assets/my_picture.jpg"
            ]
        }
    ]
}
```

Cognito user Pools vs Identity Pools
------------------------------------

**Cognito User Pools**

- It is a database of users for your web and mobile app.
- Allows federation of logins through public social mediums.
- Can customize the hosted UI for authentication.
- Has triggers with AWS Lambda during the authentication flow.

**Cognito Identity Pools**

- Obtain AWS crendentials for your users.
- Users can login through Public Social, OIDC< SAML and Cognito User Pools.
- Users can be unauthenticated (guest).
- Users are mapped to IAM roles and policies and leverage policy variables.

In short, Cognito User Pools is for managing the users and passwords and
Cognito Identity Pools isfor granting access to AWS services.

Cognito Sync
------------

Deprected; use AppSync now. It is used to store preferences, configuration, and
state of application. It offers corss device synchronization.
