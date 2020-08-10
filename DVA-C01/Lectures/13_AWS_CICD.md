AWS Continous-Integration and Continuous-Delivery (CICD)
========================================================

In AWS, CICD refers to following services: CodeCommit, CodeBuild, CodeDeploy
and CodePipeline.

CICD -- Introduction
--------------------

We want to be able to push the code in the repository and have it deployed on
the AWS automaitcally - minimizing the number of steps where humans are
involved to introduce mistakes. This involves automations, tests, and
deployment.

CICD -- Continuous Integration
------------------------------

Devs push the code to the code repository (such as GitHub, CodeCommit, GitLab,
etc), then a test/build server detects the commit and checkout the latest code
(such as Jenkins, CodeBuild, etc). Devs will be able to receive a quick feeback
from automated tests and build statuses.

Goal of CI is ass follows:

1. Find and fix bugs early.
2. Deliver faster and deploy often.
3. Unblock the pipeline.

CICD -- Continuous Delivery
---------------------------

This is a process ensuring that the software can be released reliably wheneer
needed and deployments are happneing as often as possible - which implies using
automated systems such as Jenkins or CodeDeploy.

Once the code is committed, it is built and deployment server takes the
successfully built and tested package to the production envrionment.

CICD -- Technology Stack
------------------------

> Code > Build > Test > Deploy > Provision

- **Code** : CodeCommit, GitHub, GitLab, ...

- **Build and Test** : CodeBuild, Jenkins CI, ...

- **Deploy and Provision** : EB, CodeDeploy -> CloudFormation

- And entire process is orchestrated via CodePipeline.

---

CodeCommit
----------

**Version Control** is an ability to understand changes that happened to the
code over time, trace them and revert changes if necessary. Popular VCS of
choice is Git.

Git repositories can reside on anywhere, but the central online repository is
used to collaboration with other developers. There are GitHub, BitBucket, and
so on.

Git repositories can get expensive; hence, AWS offers its own private Git
repository service, CodeCommit.

- It has no size limit.
- Fully managed by AWS.
- Code only lives within the AWS Cloud account.
- Can add further security (encryption, access conrol, etc).

CodeCommit -- Security
----------------------

Authentication in Git:

- SSH Key: AWS users can configure SSH keys in their IAM console.
- HTTPS: done through AWS CLI authentication helper or generating HTTPS
  credentials.
- MFA

Authorization in Git uses IAM Policies to manage user and roles (grant access
to repos).

Encryption:

- Repositories are automatically encrypted at rest using KMS.
- In-flight encryption (as only way to upload is through HTTPS or SSH).

Cross Account access:

- Do not share AWS credentials with others.
- Use IAM role and use AWS STS (with `AssumeRole` API).

CodeCommit -- Notifications
---------------------------

Nofitications can be triggers for CodeCommit events using AWS Simple
Notification Service (SNS) or AWS Lambda or AWS CloudWatch Event Rules.

Use cases for SNS and AWS Lambda:

- Deletion of unnecessary branches.
- Trigger for pushes that happens on the master branch.
- Notify external build system.
- Trigger AWS Lambda function to perform codebase analysis (any credentials
  left within the code?).

Use cases for CloudWatch Event Rules:

- Trigger for pull request updates.
- Commit comment events
- CloudWatch Event Rules go into an SNS topic.

CodeCommit -- Authentication
----------------------------

SSH keys and Git credentials are found under IAM > Users > Security
credentials.

---

CodePipeline
------------

It is a visual tool for continuous delivery that glues together different
services from source, build, testing and deploy. It is composed of "stages":

- Each stage can have sequential actions and parallel actions.
- Stage example: BUILD, TEST, DEPLOY, TEST, etc.
- Manual approval can be defined at any stage.

CodePipeline -- Artifacts
-------------------------

Each pipeline stage can create "artifacts" which are stroed in S3 and passed
onto the next stage.

For example, when a source has been triggered, it will send code-artifacts down
to the S3 bucket. Then, it will be passd to BUILD stage as an input artifacts
to CodeBuild. Then, output of the CodeBuild is placed in S3 again, available as
an input to next stage.

CodePipeline -- Troubleshooting
-------------------------------

You can create vents for failed pipelines or cancelled stages. If it fails, the
pipeline will stop and log will be available.

AWS CloudTrail can be used to audit AWS API calls.

If the pipeline cannot performa an action, it is likely that improper IAM Role
has been assigned.

---

CodeBuild
---------

It is a fully managed build service which is alternative to other existing
tools such as Jenkins. It is made for continous scaling - there is no servers
to manage or provision.

Pay-per-usage: the time it takes to complete the builds.

It uses Docker under the hood for reproducible builds; so it is posbbile to
provide our custom images if needed.

It is secured through:

- integration with KMS for encryption of build artifacts.
- IAM for build permissions.
- VPC for network security.
- CloudTrail for logging API calls.

CodeBuild can pull from various sources (i.e. CodeCommit, GitHub).

Build instructions can be defined in code `buildspec.yml` file.

Output will log to S3 bucket and AWS CloudWatch Logs.

Metrics are available to monitor CodeBuild stats.

It uses CloudWatch Alarms to detect failed builds and trigger notifications.

CodeBuild -- Support Envrioment
-------------------------------

It supports following:

- Java
- Ruby
- Go
- Python
- etc
- Docker : extends the envrionment to include any platform.

CodeBuild -- Build Process
--------------------------

Source code will have a `buildspec.yml` at the root directory. 

There will be a CodeBuild Docker container running on Build Docker image and
running the instructions within `buildsepc.yml` - or optionally, we can add S3
cache bucket if we do multiple builds or complicated builds.

The packaged result will be placed into the S3 bucket as artifacts and logs
will be saved on CloudWatch or S3.

CodeBuild -- BuildSpec
----------------------

`buildspec.yml` must exist at the root of your code.

We can define environment variables:

- plaintext vvariables.
- secure secrets: use AWS SSM Parameter Store to pull in secrets from a secure
  source.

Phases specify commands to run:

- install
- prebuild : final commands to execute before the build.
- build
- postbuild : finishing touches (i.e. package as .zip).

The final result will be an Artifact that will be uploaded to S3.

It is possible to cache necessary dependencies and files to S3 to speed up the
future build processes.

CodeBuild -- VPC
----------------

By default, CodeBuild containers are launched outside of VPC; so it is possible
that it will not be able to access resources located within a VPC.

For example, you have a RDS database on a private subnet, CodeBuild container
will not be able to communicate with it. This is fixed by sepcifying a VPC
configuration:

- VPC ID
- Subnet IDs
- Security Group IDs

---

CodeDeploy
----------

We want to deploy our app automatically to many EC2 instances, but these
instances are not managed by EB - we need to manage ourselves using many open
source tools such as Ansible, Terraform, Chef, Puppet, etc.

CodeDeploy -- Process
---------------------

Each EC2 machine (or On-Premise machine) must be running the CodeDeploy Agent.
This agent will continously polling for work, looking for `appsepc.yml` file.

Application is pulled from GitHub or S3.

EC2 will run the deployment instructions and in the end, CodeDeploy agent will
report success or failure of deployment on the instance.

Source code will include `appsepc.yml` file in the root directory; and it is
pushed onto the Github or AWS S3. This will trigger deployment on AWS
CodeDeploy. EC2 instances which are running the CodeDeploy agnet will poll and
will be notified of trigger and downloadthe new code and `appsepc.yml` to
deploy.

CodeDeploy -- Primary Components
---------------------------------

CodeDeploy consists of following:

- Application : unique name
- Compute Platform : EC2 or On-Premise or Lambda
- Deployment configurations: deployment rules for determining success
    - EC2 or On-Premise : healthy instance?
    - Lambda : specify how traffic is routed to your updated Lambda function
- Deployment group: group of tagged instances
- Deployment type: in-place or BLUE/GREEN?
- IAM instance profile : required to give EC2 instances the permision to pull
  from S3 or GitHub
- Application Revision : code + `appspec.yml`
- Service Role : Role for CodeDeploy to perform the task
- Target revision : Target deployment application version

CodeDeploy -- appsepc.yml
-------------------------

File section specifies how to source and copy from S3 or GitHub to filesystem.

Hooks are set of instructions to do deploy the new version. Here are ordering:

- ApplicationStop
- DownloadBundle
- BeforeInstall
- AfterInstall
- ApplicationStart
- ValidateService

CodeDeploy -- Deployment Configuration
-------------------------------------

Configs

- One-a-time : one instance at a time; if one fails, stops deployment.
- Half-at-a-time : replace by half of instances each.
- All-at-once : replaces all at once.
- Custom : i.e. number of healthy host >= 75%.

Failures:

- Instances stay in "failed" state.
- New depolyments will first be deployed to failed state instances.
- To rollback, redeploy old deployment or enable automated rollback for
  failures.

Deployment Targets:

- Set of EC2 instances which are tagged.
- Directly to an ASG.
- Mix of both.

CodeDeploy -- to EC2
--------------------

Need to define how to deploy the applciation using `appspec.yml` and choose the
deployment strategy. It will do in-place update to your fleet of C2 instances.

Hooks can be used to verify he deployment after each phase.

CodeDeploy -- to ASG
--------------------

**In-place Updates**

- Updates currently existing EC2 instances.
- Instances newly created by an ASG will also get automated deployments.

**BLUE/GREEN Deployment**

- Load Balancer is required to make it work.
- New ASG with new instances will be used for deployment.

COdeDeploy -- Rollbacks
-----------------------

You can rollback based on following triggers:

- Rollback when a deployment fails.
- Rollback when an alarm thresholds are met.
- Disable rollback.

---

CodeStar
--------

CodeStar is a integrated solution that regroups GitHub, CodeCommit, CodeBuild.
CodeDeploy, CloudFormation, CodePipeline and CloudWatch.

It helps to creae CICD-ready projects for EC2, Lambda, and EB quickly.

