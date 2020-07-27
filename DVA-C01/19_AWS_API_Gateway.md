AWS API Gateway
===============

API Gateway is used to allow clients to interact with our applications using
REST APIs.

API Gateway -- Integrations
---------------------------

**Lambda functions**

- invokes the Lambda functions.
- easy way to expose REST API backed by AWS Lambda.

**HTTP**

- exposes HTTP endpoints in the backend.
- i.e. have internal HTTP API on-premises, ALB, etc.
- Reasons: add rate limiting, caching, user authentication, API keys, etc.

**AWS Service**

- exposes any AWS API through the API Gateway.
- i.e. start an AWS Step Function workflow, post a message to SQS, etc.

API Gateway -- Endpoint Types
-----------------------------

**Edge-Optimized (Default)**

- For global clients.
- Requests are routed through the CloudFront Edge Locations.
- API Gateway will still be in one region.

**Regional**

- For clients within the same region.
- Can manually combine with CloudFront (more control over caching strategies
  and distributions).

**Private**

- Can only be accessed from your VPC using an interface VPC endpoint (ENI).
- Uses aresource policy to define access.

API Gateway -- Deployment Stages
--------------------------------

- Making changes within the API Gateway does not "deploy" them.
- Changes are first deployed to "stages".
- Each stage has its own configuration parameters.
- States can be rolled back (stage history).

**API Gateway Stages v1 and v2**

- Suppose an API v1 stage is deployed and is working at
  `https://api.example.com/v1`.
- New v2 stage can be created at new URL `https://api.example.com/v2` that
  perhaps porxy request to a new Lambda function.

API Gateway -- Stage Variables
------------------------------

Stage Variables are like environment variables for API Gateway, which can be
used to reference often chaning configuration values such as:

- Lambda function arn
- HTTP endpoint
- parameter mapping templates

Use Cases:

- Configure HTTP endpoints your stages talk to DEV, TEST, PROD, ...
- Pass configuration parameters to Lambda through mapping template.

Stage variables are passed to the context object in Lambda.

API Gateway -- Stage Variables and Lambda Aliases
-------------------------------------------------

Suppose that we have a DEV stage pointing at DEV Lambda alias that directs 100
% of its traffic to $LATEST version of the function. We can have another TEST
stage that points to TEST Lambda alias that directs to another function
version.

Now, we may have PROD stage where it will point to PROD Lambda alias that
directs 90 % of its traffic to the stable Lambda function and 5 % to the
unstable test version of the function.

API Gateway -- Canary Deployment
--------------------------------

It is possible to enable canary deployments for any stage, and choose the % of
the traffic that the canary channel will receive.

API Gateway -- Integration Types
--------------------------------

**MOCK**

- API Gateway returns a response without actually sending the requests to the
  backend.

**HTTP and AWS services**

- Must configure both the inegration request and integration response.
- Setup data mapping using mapping templates for the request and response.

**AWS_PROXY**

- Incoming requests from the client is the input for Lambda.
- The function is responsible for the logic of request and response.
- No mapping template, headers, query string parameters; these will be a simple
  argument to the function.
- HTTP request is simply passed along to the backend directly.

API Gateway -- Mapping Templates
--------------------------------

Mapping templates are used to modify request and responses; it rename and
modify query string parameters, body content, and headers. It uses the Velocity
Template Language (VTL).

Example of Mapping: JSON to XML with SOAP

- SOAP API are XML based and REST API are JSON.
- Client send JSON payload via RESTful API to API Gateway.
- API Gateay uses mapping templates to translate the request.
- Message if then forwarded as a XML payload to SOAP API in the backend.
- Here, API Gateway does:
    - Extract data from the request.
    - Build SOAP message based on request data using mapping template.
    - Call SOAP service and receive XML response.
    - Transform XML reponse to desired format for the client.

Example of Mapping: Query String Parameters

Suppose an HTTP request is sent like such
`https://xyz.com/path?name=foo&other=bar`. Then, API Gate way can translate the
parameters into JSON format with mapping templates.

```json
{
    "myVar" : "foo",
    "otherVar" : "bar",
}
```

Notice that you can rename the variable names as well.

API Gateway -- Swagger and OpenAPI Spec
---------------------------------------

It is a common way of building and defining REST APIs, using API definition as
code. Import existing Swagger and OpenAPI 3.0 Spec to API Gateway:

- Method
- Method Request
- Integration Request
- Method response
- and AWS extensions for API Gateway and set-up every option

You can export current API as Swagger and OpenAPI spec. The Swagger can be
written in YAML or JSON.

API Gateway -- Caching Responses
--------------------------------

- Caching reduces the number of calls made to the backend;
- API Gateway when receives request from the client, it will fist check against
  the Gateway cache.
- Default TTL of 300 seconds (0 to 3600 seconds).
- **Caches are defined per stage**.
- Possible to override cache settings per method.
- Possible to encrypt the cache.
- Cache capacity is bettwen 0.5 to 237 GB.
- Cache is expensive; use for PROD.

API Gateway -- Cache Invalidation
---------------------------------

- Able to flush the entire cache immediately.
- Clients can invalidate the cache with `header: CacheControl: max-age=0` and
  proper IAM authorization.
- If youdo not impose an `InvalidateCache policy (or choose the `Require`
  authorization checkbox in the console), any client can invalidate the API
  cache.

API Gateay -- Usage Plans and API Keys
--------------------------------------

If you want to make an API availabe as an offering (charge users), then
following options are available:

**Usage Plan**

- Controls who can access one or more deployed API stages and methods.
- How much and how fast they can access them.
- Uses API keys to identify API clients and meter access.
- Configure throttling limits and quota limits that are enforced on individual
  client.

**API Keys**

- Alphanumeric sring values to distribute to your customers.
- Can be used with usage plans to control access.
- Throttling limits are applied to the API keys.
- Quota limits is the overall number of max requests.

API Gateway -- Correct Order for API Keys
-----------------------------------------

**To configure a usage plan:**

1. Create one or more APIs, configuring the methods to require an API key and
   deploy the PIs to sages.

2. Generate or import API keys to distribute to application developers (your
   customers) who will be using your API.

3. Create the usage plan with the desired throttle and quota limits.

4. Associate API stage and API keys with the usage plan.

Note that callers of the API must supply the assigned API key in the
`x-api-key` header in requests to the API.

API Gateway -- Logging and Tracing
----------------------------------

**CloudWatch Logs**

- Enable CLoudWatch logging at the stage level (with log level).
- Can override settings on per API basis (ERROR, DEBUG, INFO).
- Log contains information about request and response body.

**X-Ray**

- Enable tracing to get more information about requests in API Gateway.
- X-Ray API Gateway and Lambda will give you full picture.

API Gateway -- CloudWatch Metrics
---------------------------------

- Metrics are enabled by stage.
- `CacheHitCount` and `CacheMissCount` show efficiency of the cache.
- `Count` is the total number of API requests in a given period.
- `IntegrationLatency` is the time between when API Gateay relays a request to
  the backend and when it receives a response from the backend.
- `Latency` is the time between when API Gateway receives a request from
  a client and when it returns a response to the client.

- 4XX Error is client-side induced.
- 5XX Error is server-side induced.

API Gateway -- Throttling
-------------------------

**Account Limit**

- API Gateway throttles requests at 10,000 rps across all APIs.
- Soft limit that can be increased with request.

In case of throttling, it will return `429 Too Many Requests`. Set **Stage
Limit** and **Method Limit** to imporve the performance and set **Usage Plans**
to throttle per customer.

**Like Lambda concurrency, one API being overloaded can cause the other APIs to
be throttled**.

API Gateway -- Errors
---------------------

**4XX Client Errors**

- 400 : bad request
- 403 : access denied, WAF filtered
- 429 : quota exceeded, throttle

**5XX Server Errors**

- 502 : bad gateway exception; usually for an incompatible output returned from
  a Lambda proxy integration backend and occasionally for out-of-order
  invocations due to heavy loads.
- 503 : service unavailable exception.
- 504 : integration failure (i.e. Endpoint Request Timed-Out Exception).

API Gateway -- CORS
-------------------

- Corss-Origin Resourcei Sharing must be enabled when you receive API calls
  from another domain.
- `OPTIONS` pre-flight request must contain following headers:
    - `Access-Control-Allow-Methods`
    - `Access-Control-Allow-Headers`
    - `Access-Control-Allow-Origin`
- CORS can be enabled through the console.

Suppose a web browser has accessed the static website hosted on S3 bucket at
`https://www.example.com`. It directs the browser to access the cross origin at
`https://api.example.com`. The browser will first send pre-flight request to
cross origin, and API Gateway sned pre-flight response back. Then, they can
communicate with eacher (a browser can now make GET request to
`https://api.example.com`).

API Gateay -- Security
----------------------

**IAM permissions**

- Create an IAM policy authorization and attach to User and Role.
- Authentication via IAM and Authorization via IAM policy.
- Good to provide access within AWS services.
- Uses Sigv4 capability where IAM credentials are in headers.

Suppose a client makes the REST API calls to API Gateway where its header
contains the Sigv4. API Gateay will decrypt and check against IMA policy to
whether allow access.

**Resource Policies**

- Resource policies are similar to Lambda resource policy.
- Allow for cross account access (combined with IAM security).
- Allow for a specific source IP.
- Allow for a VPC endpoint.

**AWS Cognito User Pools**

- Cognito is a database of user pools.
- Cognito fully maanges user lifecycle, token expires automatically.
- API Gateway verifies identity automatically from Cognito.
- No custom implementation is needed.
- Authentication via Cognito User Pools and Authorization via API Gateay
  Methods.

**Lambda Authorizer**

- Token-based authorizer (bearer token).
- i.e. JWT (JSON Web Token) or OAuth.
- **A request parameter-based Lambda authorizer((.
- Lambda must return an IAM policy for the user, result policy is cached.
- Authentication via External and Authorization via Lambda function.

API Gateay -- Security Summary
------------------------------

**IAM**

- Good for users and roles already within your AWS account; and resource
  policy for across account.
- Handling authentication and authorization.
- Leverages Sigv4.

**Custom Authorizer**

- Great for thrid party tokens.
- Very lflexible in terms of what IAM policy is returned.
- Handle Authentication verification and Authorization in the Lambda function.
- Pay per Lambda function invocation and results are cached.

**Cognito User Pool**

- Manage your own user pool.
- Do not require custom code.
- Must implement authorization in the backend.

API Gateway -- HTTP API vs REST API
-----------------------------------

**HTTP APIs**

- Low-latency, cost-effective AWS Lambda proxy, HTTP proxy APIs and private
  integration (no data mapping).
- Support OIDC and OAUth 2.0 authorization, and built-in support for CORS.
- No usage plans and API keys.

**REST APIs**

- All features are avilable except OpenID Connect and OAuth 2.0.

API Gateway -- WebSocket API
----------------------------

WebSocket is a two-way interactive communication between a user's browser and
a sserver. This enables stateful application use cases.

WebSocket APIs work with AWS services or HTTP endpoints.
