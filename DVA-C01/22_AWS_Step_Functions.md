AWS Step Functions
==================

- Step Functions build serverless visual workflow to orchestrate your Lambda
  functions.
- It is represented as a **JSON state machine**.
- It features: sequence, parallel, conditions, timeouts, error handling...
- Can integates with EC2, ECS, On-premise servers, API Gateway.
- Maximum execution time of 1 year.
- Possibility to implement human approval feature.
- Use cases:
    - Order fulfulment
    - Data processing
    - Web applications
    - ...

Visual Workflow
---------------

- First define JSON workflow; then it will generate a visual representation of
  how the work is going to execute.
- Then, as the work is executing, the representation also updates to show which
  section has succeeded, failed, cancelled or in progress.

Step Functions -- Error Handling
--------------------------------

- Any state can encounter runtime errors for various reasons:
    - State machine definition issues (i.e. no matching rule in a Choice
      state).
    - Task failures (i.e. an exception in Lambda function).
    - Transient issues (i.e. network partition events).
- By default, a state error causes Step Function to entirely fail the
  execution.
- **Retrying failures**
    - **Retry**: `IntervalSeconds`, `MaxAttempts`, `BackoffRate`.
- **Moving on**
    - **Catch**: `ErrorEquals`, `Next`

- Best practice is to include data in the error messages.

Step Functions -- Standard vs Express
-------------------------------------

- Standard has max duration of 1 year; Express has 5 minutes.
- They also have diferent execution start and state transition rate where
  Express is much faster.
- Their use cases are different.

AWS AppSync -- Overview
-----------------------

- **AppSync** is a managed service that use **GraphQL**.
- **GraphQL** makes it easy for applications to get the data.
- This includes combining data from one or more sources
    - NoSQL data stores, Relational databases, HTTP APIs...
    - Integrates with DynamoDB, Aurora, Elasticsearch and others.
    - Custom sources with AWS Lambda.
- Retrieves data in **real-time with WebSocket or MQTT on WebSocket**.
- For mobile apps: local data access and data sync.
- Starts with uploading a GraphQL schema.

GraphQL Example
---------------

Within AppSync, we will have a GraphQL Schema uploaded that may look like
following:

```
type Query {
    human(id: ID!): Human
}

type Human {
    name: String
    appearsIn: [Episode]
    starships: [Starship]
}

enum Episode {
    NEWHOPE
    EMPIRE
    JEDI
}

type Starship {
    name: String
}
```

Then, client may perform GraphQL query that is formated like so:

```
{
    human(id: 1002) {
        name
        appearsIn
        starships {
            name
        }
    }
}
```

AppySync will then run the Resolver - in this case, DynamoDB and returns the
query result in the JSON format to the client.

AppSync -- Security
-------------------

- `API_KEY`
- `AWS_IAM`: IAM users, roles and cross-account access
- `OPENID_CONNECT`: OpenID Connect provider and JSON Web Token
- `AMAZON_COGNITO_USER_POOLS`

- For custom domain and HTTPS, use CloudFront infront of AppSync.


