AWS Step Functions
==================

Step Functions build serverless visual workflow to orchestrate the Lambda
functions which is represented as a **JSON state machine**. It can integrate
with EC2, ECS, On-premises servers, and API Gateay.

Maximum execution time of 1 year.

Step Functions -- Visual Workflow
---------------------------------

First define JSON workflow, then it will generate a visual representation of
how the work is going to execute. As the work is executing, the representation
also updates to show which section has succeeded, failed, cancelled or in
progress.

Step Functions -- Error Handling
--------------------------------

Any State can encounter runtime errors for various reasons:

- State machine definition issues (i.e. no matching rule in "Choices" section).
- Task failures.
- Transient issues.

By default, a state error causes the Step Function to entirely fail the
execution.

**Retrying failures** with `IntervalSeconds`, `MaxAttempts`, and `BackoffRate`.

**Moving on** with catch `ErrorEquals`, and `Next`.

Step Functions -- Standard vs Express
-------------------------------------

- Standard has max duration of 1 year; Express has 5 minutes.
- They also have different execution start and state transition rate where
  Express is much faster.

---

AWS AppSync
-----------

AppSync is a managed service that uses **GraphQL**, which makes it easy for
applications to get the data. This includes combining data from one or more
sources:

- NoSQL data stores, Relational databases, HTTP APIs.
- Integrates with DynamoDB, Aurora, ElastiSearch.
- Custom sources with AWS Lambda.
- Retrieves data in **real-time with WebSocket or MQTT on WebSocket**.
- Starts with uploading a GraphQL Schema.

AppSync security uses:

- `API_KEY`.
- `AWS_IAM` : IAM users, roles and cross-account access.
- `OPENID_CONNECT` : OpenID Connect provider and JSON Web Token.
- `AMAZON_COGNITO_USER_POOLS`
