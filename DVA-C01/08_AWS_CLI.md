AWS Command Line Interface (CLI)
================================

AWS CLI is a programmatic access to the AWS services.

CLI -- Set-UP (for Linux)
-------------------------

- Comphrensive guide as available at
  [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html).

- Make sure that PATH is correctly set to point at the executable.

CLI -- Configuration
--------------------

CLI will access the AWS network over www and will require AWS credentials. This
is a secret access key that was created when the user is first generated. The
credential is configured with following command:

        $ aws configure

It will prompt you to enter your access keys and secret access key;
alternatively, you may create `~/.aws` directory and store your config and
credentials.

CLI -- EC2
----------

It is possible to repeat above steps, setting up with `aws configure` on the
EC2 instances. However, this is highly discouraged as you should not store
personal crendentials on EC2 instances.

**Instead, assign IAM Roles to the EC2 instances** so that it can carry out the
tasks with CLI.

Read more at <https://aws.amazon.com/cli>.

CLI -- Dry Runs
---------------

Sometimes we would like to make sure that we have the right permissions to
carry out a command, but not actually execute it. This is done by supploy
`--dy-run` option at the end of the command when simulating the API calls.

CLI -- STS Decode Errors
------------------------

When API calls fail, they generate a long error messages which can be decoded
by using `STS` command line:


        $ aws sts decode-authorization-message -encoded-message <error-message>

CLI -- EC2 Instance Metadata
----------------------------

The running EC2 instances can learn about itself without using an IAM Role by
sending GET request to `http://169.254.169.254/latest/meta-data`. This only
works within the EC2 instance.

You can retrieve the IAM Role name from the metadata but cannot retrieve the
IAM Policy.

CLI -- Profiles
---------------

In order to manage multiple AWS accounts, we use `--profile` option with our
aws configuration options to manage our different credentials.


        $ aws configure --profile <profile-name>

To perform an API call under a specific profile, provide `--profile
<profilename>` option to the commands.

CLI -- MFA
----------

To use MFA with CLI, need to create a temp session which requires to call `STS
GetSessionToken` API.

        $ aws sts get-session-token --serial-number <arn-of-mfa-device> --tokencode <code-from-token> --duration-seconds 3600


This will return a crendential in JSON format: a new secret access key, session
token, access key id and date of expiration which are used to call AWS APIs.

You can goto IAM Users console and add a MFA device such as
google-authenticator; once registered, the device will get an arn. To use the
WS CLI, we can use the `get-session-token` as above.

---

AWS Software Development Kit (SDK)
----------------------------------

AWS SDK is used to interact with AWS Services through our codes. It supports
major languages such as Java, .NET, PHP, Node.js, Python, GO and etc. In fact,
CLI is just a Python program that uses SDK as its dependency.

AWS Servicing Limits
--------------------

**API Rate Limits**:

- **There is a limit on how many request you can make to an API in a period**.
- `Intermittent Error` is raised when exceeds the limit; in this case, we use
  Exponential Backoff strategy to retry.
- If we get consistent errors, request an API throttling limit increase.

**Service Quotas**:

- This is a limit on services itself.
- i.e. Running On-Demand Instances can have upto 1152 vCPUs.
- You may request a service limit increase via opening a ticket.

Exponential Backoff
-------------------

It is a strategy used in the case of `ThrottlingException` which is a builtin
retry mechanism within the SDK. Simply put, if an error is raised, the
wait-time is doubled before retrying (1, 2, 4, 8, 16, ... ).

CLI -- Credentials Provider Chain
---------------------------------

CLI will look for its credentials in following order:

1. Command-line options `--region`, `--output` and `--profile`.
2. Environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and
   `AWS_SESSION_TOKEN`.
3. CLI credential file at `~/.aws/credentials`.
4. CLI configuration file at `~/.aws/config`.
5. Container credentiasl for AWS Elastic Container Service (ECS) tasks.
6. Instance Profile credentials for EC2 Instances.

SDK - Credentials Provider Chain
--------------------------------

SDK will look for its credentials in following order:

1. Environment varaibles `AWS_ACCESS_KEY` and `AWS_SECRET_ACCESS_KEY`.
2. Java System Properties `aws.accessKeyId` and `aws.secretKey`.
3. Default credential profiles file at `~/.aws/credentials`.
4. ECS Container credentials (ECS).
5. Instance Profile credentials (EC2).

AWS Credentials Scenario
------------------------

Suppose an application is deployed on an EC2 instance where it is using
environment variables with credentials from an IAM user to call the S3 API.

IAM user has `S3FullAccess` permissions.

The app only uses a single S3 bucket, so best practice would be to:

- define IAM role and EC2 instance profile.
- assign the role with minimum permissions allow to access just that bucket.

However, it was discovered that the EC2 instance still has access to all S3
buckets. The most likely reason would be that inside the EC2 instance, the
credentials were saved in there which took the precedence.

AWS Credentials Best Practices
------------------------------

_Never hardcode your credentials on code_. Either inherit it from credentials
chain or if working within AWS, use IAM Roles. Outside of AWS, use environment
variables and named profiles.

Signing AWS API Requests
------------------------

When HTTP API call is made to AWS, you can sign the request so that AWS can
identify you using AWS Credentials. If using SDK or CLI, the requests are
already signed. If not, then you must sign it using SigV4.


