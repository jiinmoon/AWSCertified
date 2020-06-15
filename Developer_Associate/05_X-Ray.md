# X-Ray

- **AXS X-Ray** helps devs analyze and debug applications utilizing microservice
    architecture.
    - it is a distributed tracing system / performance monitoring system.

## Micro-service?

- Microservice is an architectural and organizational approach to software
    development where software is composed of small independent services that
    communicate over well-defined APIs.

- These services are owned by small, self-contained teams.

- This archtecture makes apps easier to scale and faster to develop, enabling
    innovation and accelerating time-to-market for new features.

- i.e. AWS is an example of Microservice architecture.
    - when using AWS, we use different services which are pieced together.

- Then, problem is, how do we debug or catch troubles that arises between these
    multiple services? X-Ray.

## X-Ray is Distributed Tracing System?

- **Distributed Tracing System** or distributed request racing is a method used
    to profile and monitor apps, especially those built using a microservices
    architecture.

- It helps to pinpoint where failures occur and what causes poor performance.

## Performance Monitoring?

- Monitoring and management of performance and availability of software apps.
    APM strives to detect and diagnose complex application performance problems
    to maintain an expected level of service.

## What are similar third-party services like X-Ray?

- Cloud Monitoring / Application Performance Monitoring Services (APM).

- DATADOG
- New Relic
- SignalFX
- lumigo

## Sum it up, X-Ray is Distributed Tracing System

- it collects data about requests that your app serves.
- views and filters collected data to identify issues and avenues for
    optimization.

- For any traced request to your application, you can see detailed information
    not only about the request and response, but also about calls that your app
    makes to downstream AWS resources, microservices, databases and HTTP web
    APIs.

## Componenets of X-Ray

- **X-Ray Console** is where we can visually see the interactions and monitor
    how the app is behaving.
- **X-RAY API** powers the console;
- **X-Ray Daemon** backs the API;
- **X-Ray SDK** backs the daemon;
- and other clients such as AWS SDK and AWS CLI can also interact with Daemon
    and API.

- **X-Ray SDK** is the main data collector that powers console. It allows:
    - interceptors to add to your code that can trace incoming HTTP requests.
    - client handlers to instrument AWS SDK clients that your app use to call
        other AWS services.
    - HTTP client to use to instrument calls to other internal/external HTTP web
        services.

## Hold on...What is instrumentation?

- Intrumenting refers to the ability to monitor or measure the leve lof a
    product's performance, to diagnose errors, and to write trace information.

- Example of intrumenting in code:
-
    """javascript
    var app = express();
    var AWSXRay = require('aws-xray-sdk');

    app.use(AWSXRay.express.openSegment('MyApp'));

    app.get('/', function (req, res) {
        res.render('index');
    });

    app.use(AWSXRay.express.closeSegment());
    """

- Between the open/close segment, information is captured such as duration.

## X-Ray Daemon

- the role of daemon is, instead of sending trace data directly to X-Ray, the
    SDK sends JSON segment documents to a daemon process listening for UDP
    traffic.

- i.e. X-Ray SDK and other clients would send the JSON documents to Daemon.

- Then, X-Ray Daemon buffers segments in a queue and uploads them to X-Ray in
    batches.
    - this is to prevent API from being overflooded.

- daemon is available for most OSs and is included within Elastic Beanstalk and
    Lambda platforms.

- X-Ray uses trace data from the AWS resources that power your cloud
    applications to generate a detailed service graph.

## X-Ray Concepts

- AWS X-Ray receives daya from services as **segments**.

- X-Ray then groups segments that have a common request into **traces**.

- X-Ray processes the traces to generate **service graph** that provides a
    visual representation of your app.

- Segments/Subsegments
- Service Graph
- Traces
- Sampling
- Tracing Header
- Filter Expressions
- Groups
- Annotations and Metadata
- Errors, Faults, and Exceptions

## Service Graph

- it shows the client, your front-end services and back-end services.

- it can visually identify bottlenecks, latency spikes and other issues.

## Segments

- The compute resources running your application logic send data about their
    work as segments.

- The segment contains following information:

    - **host** hostname, alias or IP addr
    - **request** method, client, addr, path, user agent
    - **response** status, content
    - **the work done** start and end times, subsegments
    - **issues** erros, faults, and exceptions

## Subsegments

- Subsegments provide more granular timing information and details about
    downstream calls that your app made to fulfill the original request.

- It can contain additional details about a call to an AWS service, an external
    HTTP API, or an SQL database.

- Custom defined subsegments can be created and inserted into your code for
    monitoring.

    """javascript
    const AWS = require('aws-sdk')
    const xray - require('aws-xray-sdk-core')
    exports.handler = (event, context, callback) => {
        const segment = xray.getSegment()
        const subsegment = segment.addNewSubsegment('Mining')
        mining()
        subsegment.close()
        callback();
    }

    function mining() {
        console.log('mining for precious minerals')
    }
    """

- get the segment, and add the new subsegment to monitor.

## Traces

- A trace collects all segments generated by a single request.
    - as a request propagate through the services, we monitor how it affects
        different services.

- **Trace ID** tracks the path of a request through your application.

- The first supported service hat the HTTP request interacts with adds a trace
    ID header, and propagates it downstream to track the latency, dispositionm
    and other request data.

## Sampling

- X-Ray SDK uses a smapling algorithm to determine which requests get traced.

- By default, SDK records the first request each second, and 5% of any
    additional requests.
    - this is to avoid incurring service charges (thus low sampling).

- Can modify default sampling rule and add additional sampling rules.

- Samppling helps you to save money by reducing the amount of traces for
    high-volume and unimportant requests.

## Trace Header

- All requests are traced up to configured value.

- After reaching the minimum value, a percentage of requests are traced in order
    to avoid the unnecessary service cost.

- The sampling decision and trace ID are added to HTTP requests in tracing
    headers named X-Amzn-Trace-Id.

- The first X-Ray-integrated service that the request hits adda a tracing
    header.

## Filter Expressions

- Even with sampling, a complex application can generate a lot of data; this is
    when filtering comes in to narrow down specific paths or users.

- i.e. search for traces where http url contains '/api/user' may filter out for
    traces to find 'https://xxx.elasticbeanstalk.com/api/user', and so on.

## Groups

- Can assign a Filter Expression to a Group so that you can call group by name
    or by Amazon Resource Name (ARN) to generate its own service graph, trace
    summaries, and Amazon CloudWatch metrics.

- Updating group's filter expression does not change data already recorded!

---

# X-Ray Cheat Sheet

- X-Ray helps developers analyze and debug applications that utilize the
    microservice architecture.

- It is a Distributed Tracing System, it is a method used to profile and moitor
    apps, especially those built using microservices to pinpoint failures and to
    identify bottle necks.

- X-Ray Daemon is a software application that listens for traffic on UDP port
    2000, gathers raw segment data, and relays it to the AWS X-Ray API. Data is
    generally not sent to X-Ray API directly, but through the daemon which
    uploads it in bulk.

- Segments provides the resource's name, details about the request and details
    about the work done.

- Subsegments provide more granular timing information and details about
    downtream calls that your app made to fulfill the original request.

- Service Graph is a flow chart visualization of average response for
    microservices and to visually pinpoint failure.

- Traces pools the all segments generated by a single request so you can track
    the path of requests through multiple services.

- Sampling is an algorithm that decides which requests should be traced; by
    default X-Ray SDK records the first request each second and 5% of any
    additional.

- Tracing Header is named X-Amzn-Trace-Id and identifies a trace which passed
    along to downstream services.

- Filter Expression allows you to narrow down specific paths or users.

- Groups allow save Filter Expressions to quickly filter traces.

- Annotations and Metadata allow you to capture additional information in
    key-value pairs.
    - Annotations are indexed to use filter expressions on (limit 50).
    - Metadata are not indexed; use to store traces but not searching.

- Errors are 400/4xx errors
- Faults are 400/5xx errors
- Throttle is 429 too many requests
- X-Ray supports the following languages:
    - Go, NodeJs, Ruby, Java, Python, ASP.NET, PHP

- X-Ray support AWS Service Integrations with following:
    - Lambda, API Gateway, App Mesh, CloudTrail, CloudWatch, AWS Config, EB,
        ELB, SNS, SQS, EC2, ECS, Fargate.


