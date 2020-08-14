# AWS CLI, SDK, IAM Roles and Policies

- WHen interacting with the AWS, we can use the console, but we can also use
  any of the following:
    - AWS CLI on local or EC2 machines
    - AWS SDK on local or EC2 machines
    - AWS Instance Metadata Service for EC2

## AWS CLI
### CLI Configuration on Local

- We can configure our credentials to use with AWS CLI via command `aws
  configure` which will prompt you for access key and secret access key.

- This will generate a config file at `~/.aws/config` as well as
  `~/.aws/credentials`.

- You can set up multiple profiles if needed and specify them when executing
  aws commands with option `--profile`.

- To do so, use `aws configure --profile`.

### CLI Configuration on EC2

- We should not set up same approach as we did with the local setup.
- This is highly insecure to place your credentials directly on an EC2.
- The correct way is to attach an IAM Roles to the EC2 instance.

### CLI Dry-Run

- When you need to test whether the changed policy for an user took a place,
  you can supply an option `--dry-run` with the aws command to check whether
  the user has necessary permission to execute the action.

- This will not actually execute the commands so no costs.

### AWS CLI STS Decode

- When CLI commands fail, it will generate an error message that can only be
  decoded with STS command line:

- `sts decode-authorization-message`.

- It will return detailed information in JSON format.

### EC2 Instance Metadata

- This allows EC2 instances to "learn" about themselves without using IAM Role.
- This information is made available at URL
  `http://169.254.169.254/latest/meta-data`.

- Note that this cannot be used to retrieve the IAM policy attached to the EC2
  instance (only IAM ROle name).

### CLI with MFA

- To use MFA with CLI, it requires a temporary session via `aws sts
  get-session-token`.

- We provide serial number for the MFA device associated and the current token
  code that is generated on the MFA.

- This will return a temporary cretential for durations specified- This will
  return a temporary cretential for durations specified..

## AWS SDK

- SDK is a application library that allows your applications to make AWS API
  calls.

- For example, when intercting with DynamoDB with our app, SDK is a must.

- Note that if a region is unspecified, it will default to us-east-1.

## AWS Limits

- API Rate Limits:
    - These are hard set limits on how often you can call certain API.
    - i.e. s3::GetObject has a limit of 5500 per second per prefix.
    - For temp errors, try exponential backoff strategy.
    - If error is consistent, request AWS for throttling limit increase.

- Service Quotas:
    - There are hard set limits of the services provied.
    - i.e. On-Demand has a maximum 1152 vCPUs.
    - Request a service limit increase by opening a ticket.

### Exponential Backoffs

- It is a strategy used to mitigate THrottlingException errors.
- It is a retry mechanism included in the SDK.
- With each failture, double the wait period between retries.

## AWS CLI Credentials Provider Chain

- CLI look for credentials in following order:

1. Command line options.
2. Envrionment variables.
3. CLI credentials file.
4. CLI configuration file.
5. Container crendentials.
6. Instance profile crendentials.

## AWS SDK Crendentials Provider Chain

- SDK look for credentials in following order:

1. Environmental variables.
2. Java System Properties on runtime.
3. The default credential profiles file (`~/.aws/crendentials`).
4. ECS container crendentials.
5. INstance profile credentials.

## Signing AWS API Requests

- Every AWS HTTP API calls need to be signed with the user's key to that they
  can confirm the request came from the right person.
- Some requests does not need to be signed such as requests to S3.
- SDK and CLI both signs the HTTP requests automatically.

- This is done with SigV4 by setting the header with the signature.


