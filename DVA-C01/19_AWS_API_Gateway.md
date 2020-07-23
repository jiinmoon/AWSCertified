AWS API Gateway
===============

So far, we have created a portion of serverless API that uses Lambda function
to perform CRUD operations on the DynamoDB. Now, we wish to have a client that
can interact with it using the REST API - and this is where API Gateway would
be used.

Here are some features of API Gateway:

- Lambda + API Gateway : no inrastructure to manage at all.
- Supports WebSocket Protocol.
- Handle API versioning (v1, v2, ...).
- Handle different environments (dev, test, prod, ...).
- handle security (Authenication and Authorization).
- Create API Keys and handle request throttling.
- Swagger and OpenAPI import to quickly define APIs.
- Transform and validate requests and responses.
- Generate SDK and API specifications.
- Cache API responses.

API Gateway -- Intergraions at High-Level
-----------------------------------------

**Lambda Function**

- invoke Lambda function.
- easy way to expose REST API backed by AWS Lambda.

**HTTP**

- expose HTTP endpoints in the backend.
- i.e. internal HTTP API on premise, Application Load Balancer, ...
- reason: add  rate limiting, caching, user authentications, API keys, etc.

**AWS Service**

- expose any AWS API through the API Gateway.
- i.e. start an AwS Step Function workflow, pst a message to SQS.

API Gateway -- Endpoint Types
-----------------------------

**Edge-Optimized (default)**

- for global clients.
- requests are routed through the CloudFront Edge locations (improves latency).
- API Gateway is still on a single region.

**Regional**

- for clients within the same region.
- can manually combine with CloudFront (more control over the caching).
  strategies and the distribution).

**Private**

- can only be accessed from your VPC using an interface VPC endpoint (ENI).
- use a resource policy to define access.

API Gateway -- Deployment Stages
--------------------------------

- Making changes in the API Gateway does not mean they are actually "deployed".
- Changes are deployed to "Stages".
- Each stage has its own configuration parameters.
- Stages can be rolled back (stage history).

**API Gateway -- Stages v1 and v2**

- suppose a API v1 stage is deployed and working at
  `https://api.example.com/v1`.
- new v2 stage can be created at new URL `https://api.example.com/v2` that
  perhaps proxy requests to new Lambda function.

API Gateway -- Stage Variables
------------------------------

- Stage variables are like environment variables for API Gateway.
- Use them to change often changing configuration values:
    - Lambda function ARN
    - HTTP Endpoint
    - Parameter mapping templates
- Use cases:
    - Configure HTTP endpoints your stages talk to (dev, test, prod, ...).
    - Pass configuration parameters to Lambda through mapping templates.
- Stage varaibles are passed to the context object in Lambda.

API Gateway -- Stage Varaibles and Lambda Aliases
-------------------------------------------------

- This is an example of a use case of Stage Variables integrating with Lambda
  aliases.

Suppose that we have a Dev stage pointing at Dev Lambda alias that directs 100%
of its traffic to $LATEST function. We can have another Test stage that points
to TEST Lambda alias that directs to some version of function.

Now, we may have Prod stage where it will point to PROD Lambda alias that
directs 95% of its traffic to the stable Lambda function, and 5% to the
unstable test version of the function!

API Gateway -- Canary Deployment
--------------------------------

- It is possible to enable canary deployments for any stage (usually PROD).
- Choose the % of the traffic the canary channel receives.

i.e. Set up a PROD stage for v1 and a PROD canary stage for latest version.
Then, we can redirect 95% of traffic to PROD and 5% to Canary.

- mertirc and logs are separated.
- possible to override stage variables for canary.

- this is equivalent of Blue / Green deployment.

API Gateway -- Integration Types
--------------------------------

**Integration Type MOCK**

- API Gateway returns a response without sending the request to the backend.

**Integration Type HTTP / AWS (Lambda & AWS Services)**

- must configure both the integration request and integration response.
- setup data mapping using _mapping templates_ for the request and reponse.

**Integration Type AWS_PROXY (Lambda Proxy)**

- incoming request from the client is the input to Lambda.
- the function is responsible for the logic of request and response.
- _no mapping template, headers, query string params_; these are all just
  arguments.
- HTTP request is simply passed along to the backend directly.
- HTTP response from the backend if forward by API Gateway.

Mapping Templates (AWS and HTTP Integration)
--------------------------------------------

- Mapping templates can be used to modify request and responses.
- Rename and modify _query string parameters_.
- Modify _body content_.
- Add _headers_.
- Uses Velocity Template Language (VTL) : for-loop, if-else, ...

- Filter response results.

Mapping Example: JSON to XML with SOAP
--------------------------------------

- SOAP API are XML based; REST API are JSON based.
- Client sends JSON payload via RESTful API to API Gateway;
- API Gateway uses Mapping Template to translate the request;
- Message is then forwarded as a XML payload to SOAP API in backend.

- In this case, API Gateway should:
    - Extract data from the reqeust: either path, payload or header.
    - Build SOAP message based on request data (using mapping template).
    - Call SOAP service and receive XML response.
    - Transform XML response to desired format (such as JSON), and relay the
      response back to the user.

Mapping Example: Query String Parameters
----------------------------------------

Suppose that client has sent its HTTP request with string parameters: for
example, it looks like `https://abc.com/path?=name=foo&other=bar`. Then, API
Gateway uses mapping template to translate the parameters into JSON format as
follows:

```json
{
    "myVar" : "foo",
    "otherVar" : "bar"
}
```

Note that you can rename the variables since you can define the mapping however
you would like. This reformatted message will be relay to the backend for
processing.

API Gateway + Swagger and OpenAPI Spec
--------------------------------------

- It is a common way of defining REST APIs, using API definition as code.
- Import existing Swagger and OpenAPI 3.0 epc to API Gateway.
    - Method
    - Method Request
    - Integration Request
    - Method Response
    - and AWS extensions for API Gateway and setup every option

- Can export current API as Swagger and OpenAPI spec.
- Swagger can be written in YAML or JSON.
- Using swagger, we can generate SDK for our applications.

API Gateway Caching -- Responses
--------------------------------

- Caching reduces the number of calls made to the backend;
- API Gateway when receives request from the client, it will first check
  against the Gateway cache.
- Default TTL : 300 seconds (0 ~ 3600 seconds).
- **Caches are defined per stage.**
- Possible to override cache settings _per method_.
- Possible to encrypt cache.
- Cache capacity between 0.5 to 237 GB.
- Cache is expensive; use it for PROD, not in TEST or DEV.

API Gateway Cache Invalidation
------------------------------

- Able to flush the entire cache (invalidate it) immediately.
- Clients can invalidate the cache with `header: Cache-Control: max-age=0` and
  proper IAM authorization.
- If you do not impose an `InvalidateCache` policy (or choose the `Require`
  authorization check box in the console), any client can invalidate the API
  cache.

i.e. example of IAM authorization policy.

```json
{
    "Version": "2020-10-10",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "execute-api:InvalidateCache",
            ],
            "Resource": [
                "arn:...:api-id/stage-name/GET/resource-path-specifier"
            ]
        }
    ]
}
```

API Gateway -- Usage Plans & API Keys
-------------------------------------

- If you want to make an API available as an offering (charge) to your
  customers.
- **Usage Plan**:
    - Who can access one or more deployed API stages and methods.
    - How much and how fast they can access them.
    - Uses API keys to identify API clients and meter access.
    - Configure throttling limits and quota limits that are enforced on
      individual client.

- **API keys**:
    - Alphanumeric string values to distribute to your customers.
    - i.e. `WHhjnehhWHJHSLhlnj8jieofjsldfh...`
    - Can be used with usage plans to control access.
    - Throttling limits are applied to the API keys.
    - Quotas limits is the overall number of maximum requests.

API Gateway -- Correct Order for API Keys
-----------------------------------------

**To configure a usage plan:**

1. Create one or more APIs, configuring the methods to require an API key and
   deploy the APIs to stages.

2. Generate or import API keys to distribute to application developers (your
   customers) who will be using your API.

3. Create the usage plan with the desired throttle and quota limits.

4. _Associate API stages and API keys with the usage plan._

Note that callers of the API must supply the assigned API key in the
`x-api-key` header in requests to the API.

API Gateway -- Logging and Tracing
----------------------------------

**CloudWatch Logs:**

- Enable CloudWatch logging at the Stage level (with Log level).
- Can override settings on a per API basis (i.e. ERROR, DEBUG, INFO).
- Log contains information about request and response body.

**X-Ray:**

- Enable tracing to get more information about requests in API Gateway.
- X-Ray API Gateway and Lambda will give you full picture.

API Gateway -- CloudWatch Metrics
---------------------------------

- Metrics are enabled by stage.
- `CacheHitCount` and `CacheMissCount` show efficiency of the cache.
- `Count` is the toal number of API requests in a given period.
- `IntegrationLatency` is the time between when API Gateway relays a request to
  the backend and when it receives a response from the backend.
- `Latency` is the time beween when API Gateway receives a request from
  a client and when it returns a reponse to the client (so it includes
  integration lantecy and overheads).

- `4XX` Error (client side)
- `5XX` Error (server side)

API Gateway -- Throttling
-------------------------

- **Acount Limit**
    - API Gateway throttles requests at 10000 rps across all API.
    - Soft limit that can be increased upon request.

- In case of throttling, it will return `429 Too Many Requests` (retriable
  error).

- Set **State Limit** and **Method Limit** to improve performance.
- Set **Usage Plans** to throttle per customer.

- _Like Lambda concurrency, one API being overloaded can cause the other APIs
  to be throttled (so use limits wisely)._

API Gateway -- Errors
---------------------

**4XX** Client errors

- 400: bad request
- 403: access denied, WAF filtered
- 429: quota exceeded, throttle

**5XX** Server errors

- 502: bad gateway exception; usually for an incompatible output returned from
  a Lambda proxy integration backend and occasionally for out-of-order
  invocations due to heavy loads.
- 502: service unavilable exception.
- 504: integration failure (i.e. Endpoint Request Timed-out Exception) API
  Gateway requests times out after 29 second max.

API Gateway -- CORS
-------------------

- Cross-Origin Resource Sharing must be enabled when you recieve API calls from
  another domain.
- `OPTIONS` pre-flight request must contain the following headers:
    - `Access-Control-Allow-Methods`
    - `Access-Control-Allow-Headers`
    - `Access-Control-Allow-Origin`
- CORS can be enabled through the console.

i.e. Web Browser has accessed the S3 bucket statically hosted website at Origin
`https://www.example.com`; and it directs browser to access the Cross Origin
`https://api.example.com`. Browser will first send preflight request (HTTPS
OPTIONS) to Cross Origin, and API Gateway sends preflight response back. Then,
the browser and API Gateway can now communicate with one another (i.e. now able
to send GET requests to https://api.example.com).

API Gateway -- Security
-----------------------

**IAM Permissions**

- create an IAM policy authorization and attach to User / Role.
- Authentication via IAM and Authorization via IAM Policy.
- Good to provide access within AWS (EC2, Lambda, IAM users...).
- uses `Sig v4` capability where IAM credential are in headers.

i.e. Client makes the REST API call to API Gateway where its header contains
the Sig v4. API Gateway will decrypt and check against IAM Policy to whether
the client is authorized to make the call.

**Resource Policies**

- Resource policies are similar to Lambda resource policy.
- allow for cross account access (combined with IAM Security).
- allow for a specific source IP address.
- allow for a VPC endpoint.

**Cognito User Pools**

- Cognito is a database of user pools.
- Cognito fully manages user lifecycle, token expires automatically.
- API Gateway verifies identity automatically from AWS Cognito.
- No custom implementation is needed.
- Authentication via Cognito User Pools and Authorization via API Gateway
  Methods.

i.e. Clent first authenticate to Cognito User Pools to retrieve the connection
token. Then, makes REST API request to API Gateway with the Token, and API
Gateway evaluates the Cognito token. If correct, it will allow access to the
backend.

**Lambda Authorizer**

- Token-based authorizer (bearer token).
- i.e. JWT (JSON Web Token) or Oauth.
- A _request paraemter-based Lambda authorizer)_ (headers, query string,
  stage variable).
- Lambda must return an IAM policy for the user, result policy is cached.
- Authentication via External and Authoriation via Lambda function.

i.e. Client first authenticate to third party auth-system to receive a token.
Then, it makes a request with bearer token or request parameters. API Gateway
looks at the context and token, or request params against Lambda Authorizer to
determine whether to allow access to backend. Lambda Authorizer will examine
and return IAM Principal and IAM Policy - this is done once and will be cached
in API Gateway.


API Gateway -- Security Summary
-------------------------------

**IAM**

- Good for users and roles already within your AWS account; and resource policy
  for across account.
- Handling authentication and authorization.
- Leaverges Sig v4.

**Custom Authorizer**

- Great for third party tokens.
- Very flexible in terms of what IAM policy is returned.
- Handle Authentication verification and Authorization in the Lambda function.
- Pay per Lambda function invocation; results are cached.

**Cognito User Pool**

- Manage your own user pool (can be backd by Fackbook, Google Login, etc).
- Do not require custom code.
- Must implement authroization in the backend.

API Gateway -- HTTP API vs REST API
-----------------------------------

**HTTP APIs**

- low-latency, cost-effective AWS Lambda proxy, HTTP proxy APIs and private
  integration (no data mapping).
- support OIDC and OAuth 2.0 authorization, and built-in support for CORS.
- No usage plans and API keys.

**REST APIs**

- All features available except Naitive OpenID Connect and OAuth 2.0.

API Gateway -- WebSocket API
----------------------------

**WebSocket?**

- It is a two-way interactive communication between a user's browser and
  a server.
- Server can push information to the client.
- This enables stateful application use cases.

- WebSocket APIs are often used in _real-time_ applications such as chat
  applications, collaboration platforms, multiplayer games and financial
  trading platforms.

- It works with AWS Services (Lambda, DynamoDB) or HTTP endpoints.





