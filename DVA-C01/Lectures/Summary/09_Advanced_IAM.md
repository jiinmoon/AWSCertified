IAM Extended Topics
===================

Web Identity Federation
-----------------------

_Web Idetntity Federation_ lets your users access AWS resources after
authenticated with web-based identity providers such as Facebook, Google, or
Amazon. Once authenticated, they will receive a JWT which can be trade in for
a temporary AWS security credentials.

**Amazon Cognito** provides such service with following features:

- Signup and Signin for web and mobile applications.
- Guess accesses.
- Synchronizes user data across devices.

Cognito User Pools
------------------

_User Pools_ are user directories (databases) of users that allows for
management of signup and signin for applications.

Users may directly signin with User Pool, or undirectly with identity providers
such as Facebook, Amazon, or Google.

_Identity Pools_ enable you to create unique identities for users, and
authenticate them with identity providers; you may grant temp AWS credentials.

For example, an user may first visit the _User Pool_ and signin with Facebook.
They will receive a JWT Token which is traded to _Identity Pool_ for a AWS
Credentials required to access the AWS services such as a S3 bucket.

Cognito uses **Push Synchronization** to send silent push notification of user
data updates to multiple device types associated with an user ID.

Different IAM Policies
----------------------

IAM is used to define access permissions within AWS with following 3 types of
policies: managed, customer managed and inline policies.

**Managed Policy** is created and administered by AWS; so, it cannot be
changed.

**Customer Managed Policy** is a standalone policy that is created and
administered within an account; can copy an existing managed policy to
customize further.

**Inline Policy** is embedded policy within the user, group or role that gives
strict 1:1 relationship. When the user, group or role is deleted, so is the
policy.

STS `AssumeRoleWithWebIdentity`
-------------------------------

It is a STS (Security Token Service) API that provides temp security
crendentials for usres authenticated by an app or a Web Identity Provider
(Facebook, Google, ...).

For mobile applications, Cognito is recommended (it uses STS underneath).
Regular web applications can use the STS `assumerole-with-web-identity`.

For example, an user authenticates via Facebook, Google or Amazon to receive
a JWT Token. It will then call STS `assumerole-with-web-identity` API to trade
the token for a temporary crendentials. Then, the user can now access the AWS
resources.

It will return a response containing session token, secret access key, access
key id and expiration.

