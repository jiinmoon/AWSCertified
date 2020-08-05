Amazon Cognito is an user identity verficiation service that provides solutions
to following:

- Authentication - provides user sign-up/sign-in. Eables support for
  federations with Enterprise Identities or Social Identities.

- Authorization - sets the permissions for an user.

- User Management - allow management of user lifecycles.

---

**Amazon Cognito User Pools**

It is used for authentication. To verify an user's identity, you want to have
them login using username/password or federated login using Identity Providers
such as Amazon, Facebook, Google, SAML (Microsft AD).

1. Users send authentication requests to Cognito User Pools.
2. Cognito user pool verifies the identity of the user OR sends the request to
   Identity Providers for verification.
3. Cognito user pool returns a token to the user.
4. User can now use the token to access other services such as API Gateway.

You can also create an quick login-page to integrate with the application.

---

**Amazon Cognito Identity Pools**

It is used for user "authorization". You can create unqiue identities for your
users and federate them with your identity providers. With it, you can grant
temporary AWS credentials for users.

Think of it as an actual mechanism authorizing access to AWS resources - when
identity pool is created, it is defining who is allowed to get AWS credentials
and use those credentials to access.

1. Web or mobile app sends its authentication token to Identity Pools; the
   token can come from Cognito User Pools, Amazon, Facebook, etc.
2. Identity Pool exchanges the auth token for temp AWS credentials for the app
   to access the resources such as S3 or DynamoDB.

---

