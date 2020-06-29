# AWS CLI

- Thus Far, we have interacted with services manually and they exposed standard
    information for clients:
    - EC2 exposes a standard Linux machine we can use any way we want.
    - RDS exposes a standard database we can connect to using a URL.
    - ElastiCache exposes a chace URL we can connect to using a URL.
    - ASG / ELB are automated and we do not have to program against them.
    - Route53 has an interactive setup manual.

- Developing against AWS has two components:
    - How to perform interactions with AWS without using the Online Console?
    - How to interact wit hAWS Proprietary services?

- AWS CLI is programmatic access to the AWS Services.

- Developing, and performing AWS tasks against AWS can be done in several ways:
    - using the AWS CLI on our local computer or EC2 machines.
    - using the AWS SDK on our local computer or EC2 machines.
    - using the AWS Instance Metadata Service for EC2.

---

## CLI Set-up (in Linux)

- Simply grab aws cli zip file, unzip, then run installation binary.

- Troubleshooting (Exam question)
    - If after installing the AWS CLI and use it only to get an error "aws:
        command not found". Then, check whether aws executable is in the PATH
        envrionment variable.

## AWS CLI Configuration

- CLI will access AWS network over www.
- To do so, it requires credentials.

- Head to IAM and create an user; this is the only time you have an access to
    your secret access key, which is used to connect to AWS services.
    - Download .csv file and store it in secure place.

- Now, we configure the AWS CLI with our credentials.

    > aws configure

- It will prompt you to enter your Access Key and Secret Access Key.
- Alternatively, you can create an ~/.aws directory, and store your config and
    credentials file there.

---

## AWS CLI on EC2 -- BAD WAY

- we could run `aws configure` on EC2 as we did on the local machine.
    - However, this is insecure.

- **Never put personal credentials on EC2.**
- EC2 can be compromised, and your entire AWS account as well.

## AWS CLI on EC2 -- CORRECT WAY

- Assign IAM Roles to the EC2 instances.
- IAM Roles can come with a policy authorizing exactly what the EC2 instance
    should be able to do.
- EC2 instances can then use these profiles automatically without any additional
    configurations.
- EC2 instances can use the profiles automatically without any additional
    configs!

## AWS CLI practice

- read up on S3 CLI commands on official doc.
- in fact, there are more CLI commands.

---

## AWS IAM Roles & Policies

- To each created role, we can either assign pre-assigned policy, or create a
    new policy to attach to the user, group or role (in this case, role).

- While Policies are in JSON format, there is an easy AWS Policy Generator.

## AWS Policy Simulator

- A tool to test your policies!

- It simuates your created policy by simulating service and action performed to
    see the results.

---

## AWS CLI Dry Runs

- Sometimes, we would like to make sure we have permissions; but not acutally
    run the command.

- Some AWS CLI commands can become expensive if they succeed - creating an EC2
    instance out of blue.
- Some AWS CLI commands thus supply you with a --dry-run option to simulate API
    calls!

- If permission is allowed, it would return Request would have succeeded, but
    DryRun flag is set message.

---

## AWS CLI STS Decode Errors

- When API calls fail, we may get a long error message.
- The error message can be decoded using **STS** command line.
- sts deconde-authorization-message

    > aws sts deconde-authorization-message --enconded-message <value>

## AWS EC2 Instance Metadata

- AWS EC2 Instance can learn about itself without using an IAM Role.
- URL : `http://169.254.169.254/latest/meta-data`.
    - This is internal; and only work within the EC2 instance.

- You can retrieve the IAM Role name from the metadata, but you CANNOT retrieve
    the IAM Policy.
- Note : metadata here is the info about EC2 instance.
    - User Data was an launch script.

- i.e.

    > curl http://169.254.169.254/latest/meta-data/iam/security-credentials

    - this should return list of roles attached to the EC2 instance.

## AWS CLI Profiles

- How do you manage multiple AWS Accounts? the credtentials folder only holds a
    access key and secret access key for one account.

- To resolve this, we use profile.

    > aws configure --profile profile-name

- This will prompt you to enter an access key, secret access key, region...
- After finished, when you view the credentials file, you can see that new
    profile has been created `[profile-name]`.

- Now, to perform AWS CLI commands on aws account associated with profile-name,
    we use `--profile profile-name` to our commands.

## AWS MFA with CLI

- To use MFA with CLI, you need to create a temporary session.
- To do so, run the **STS GetSessionToken** API call.
- i.e. `aws sts get-session-token --serial-number arn-of-the-mfa-device
    --token-code code-from-token -duration-seconds 3600`.

- This will return a JSON of credentials: a new secret access key, session
    token, access key id, and date of expiration.
- This can be used to call AWS API.

- Goto IAM -> users. You will find MFA device to add; google-authenticator is
    one of them.
- The registered MFA device will get the arn.
- Now, to use AWS CLI, we can use get-session-token.
- Good idea to use `aws configure --profile mfa`.
- Only change is that you have to additionally add another line with
    `aws_session_token = received-token`.

---

## AWS SDK Overview

- We may want to interact with AWS Services through our applications and code.
- To do this, we use SDK.
- SDK is supported on most of major language platforms:
    - i.e. Java, .NET, nodejs, PHP, Pyhon, Go, Ruby, C++

- As a matter of fact, AWS CLI is just a python program - using SDK to power it.

- We have to use SDK whe ncoding against AWS Services (i.e. DynamoDB).

- Exam expects you to know when to use SDK.
    - more practice with Lambda.

- Tip: if no region is specified, default is us-east-1.



