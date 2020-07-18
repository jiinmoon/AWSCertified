AWS CICD 
========

CICD refers to following services: CodeCommit, CodePipeline, CodeBuild, and CodeDeploy.


CICD Introduction
-----------------

Thus far, we have covered how to create resources in AWS manually. We have
then used CLI to programmatically interact with AWS, and deploy codes to AWS
using Elastic Beanstalk.

_Minimize the number of steps = less like to commit mistakes._

We want to be able to push our code in the repository and have it deployed onto
the AWS automatically. This ensures automations, fits the right specs, and
tested before deployment. This also gives us to branch our code into different
stages or environments (dev, test, pre-prod, prod).

This is about CI; automating the deployment we have done so far while adding
increased safety measures. It is important as Dev exam has a section dedicated
to the CICD.

It is made of four services:

- **CodeCommit** : storing our codes.
- **CodePipeline** : automating our pipeline from code to ElasticBeanstalk.
- **CodeBuild** : building and testing our code.
- **CodeDeploy** : deploying the code to EC2 fleets (not Beanstalk).


Quick Intro: Continuous Integration
-----------------------------------

developers push the code to a code repository often
(GitHub/CodeCommit/BitBucket/GitLab/...); then a testing/build server detects
the commit, and checks the code as it is pushed (CodeBuild/Jenkins/...).
developer will be then able to get a quick feedback from tests/build status how
tests have pass/failed or build has been successful or failed.

Goal is 

    (1) find/fix bugs early; 

    (2) deliver faster as the code is tested; 

    (3) deploy often; 

    (4) happy developers - always unblocked.

Quick Intro: Continuous Delivery
--------------------------------

A process ensuring that the software can be released reliably whenever needed
and deployments are happenining as often as possible.

This implies automated deployment (use tools such as CodeDeploy / JenkinsCD
/ ...).

Once code is committed, it is built and deployment server takes in the built
package if it passes the tests, and automatically deploys it onto the server.

Tech Stack for CICD
-------------------

`Code > Build > Test > Deploy > Provision`

Code : CodeCommit / GitHub / ...

Build & Test : CodeBuild / Jenkins CI / ...

Deploy & Provision : Elastic Beanstalk or CodeDeploy > CloudFormation ...

Entire process is orchestrated by AWS CodePipeline.


---

CodeCommit Overview
-------------------

**Version Control** is the ability to understand the various changes that
happened to the code over time (and rollback if necessary).

All these are enabled by using a version control system such as Git.

A Git repository can be on any machine, but it is placed on a central online
repository for collaboration reasons. 

Benefits are:

- coolaborate with other developers.
- code is backed-up somewhere.
- make sure that it is fully visible and auditable.

CodeCommit
----------

- Git repositories can be expensive.
- The industry includes:
    - GitHub: free public repos, but paid private ones.
    - BitBucket
    - etc...

- AWS CodeCommit is a private Git repositories.
- No size limit - scales seamlessly.
- Fully managed; highl available.
- Code only lives within the AWS Cloud account.
    - Increased security and compliance.
- Added security (encryption, access control, etc...).
- Can be integrated with other CI tools; not just AWS.

CodeCommit Security
-------------------

The basic interaction with CodeCommit is done via Git command.

Authentication in Git:
    - SSH key: AWS Users can configure SSH keys in their IAM console.
    - HTTPS: Done through the AWS CLI Authentication helper or Generating HTTPS
      credentials.
    - MFA can be enabled for extra safety.

Authorization in Git:
    - IAM Policies, manage user / roles rights to repositories.

Encryption:
    - Repos are automatically encrypted at rest using KMS.
    - Encrypted in transit since we can only use either HTTPS or SSH.

Cross Acount acess:
    - to give others access to CodeCommit repo...
    - **Never share your SSH keys**.
    - **Never share your AWS credentials**.
    - Use IAM role in your AWS Account, and use AWS STS (with AssumeRole API).

CodeCommit vs GitHub
--------------------

Similarities:

- Both are git repos.
- Both support code reviews (pull requests).
- Both can be integrated with AWS CodeBuild.
- Both support HTTPS and SSH method of authentication.

Differences:

- Security:
    - GitHub: GitHub Users
    - CodeCommit: AWS IAM users and roles

- Hosted:
    - GitHub: hosted by GitHub (3rd party - give your code).
    - GitHub Enterprise: self-hosted on your servers.
    - CodeCommit: managed & hosted by AWS.

- UI:
    - GitHub: UI is fully featured.
    - CodeCommit: UI is minimal.

CodeCommit Notifications
------------------------

You can trigger notifications in CodeCommit using **AWS SNS (Simple
Notification Service)** or **AWS Lambda** or **AWS CloudWatch** Event rules.

Use cases for SNS / AWS Lambda:

- Deletion of branches.
- Trigger for pushes that happens in master branch.
- Notify external Build System.
- Trigger AWS Lambda function to perform codebase analysis (is there
  credentials left in the code?)

Use cases for CloudWatch Event Rules:

- Trigger for pull request updates (created / updated/ deleted / commented)
- Commit comment events
- CloudWatch Event Rules goes into an SNS topic...

They can be set under the _Settings_ > _Notification_.

Under here, we can specify Notification Rules; sepcify various events that will
trigger notifications (Comments, Merged...). SNS or AWS Slack bot.

Also, there is a _Trigger_ which are more specific events happening to the
repository such as push, create branch, delete branch, and so on. The target
here can be SNS or Lambda.

CodeCommit Authentication
-------------------------

Under IAM > Users > Security credentials, you will find the SSH keys for AWS
CodeCommit and HTTPS Git credentials for AWS CodeCommit.

---

CodePipeline
------------

It is a visual tool for continous delivery.

Sources:        GitHub / CodeCommit / Amazon S3

Build:          CodeBuild / Jenkins / etc...

Load Testing:   ...

Deploy:         AWS CodeDeploy / Beanstalk / CloudFormation / ECS / ...

It is made of stages:

- each stage can hav sequential actions and/or parallel actions.
- stages example: build / test / deply / loadtest / etc..
- manual approval can be defined at any stage.

CodePipeline Artifacts
----------------------

- Each pipeline stage can create "artifacts".

- Artifacts are stored in S3 and passed on to the next stage.

- i.e. when a source (CodeCommit) has been triggered, it will send
  code-artifacts down to the S3 bucket. Then, it will move onto the Build stage
  as an input artifacts (CodeBuild). Again, the output of Build stage once
  again goes through the S3 bucket, and onto the next stage.

CodePipeline Troubleshooting
----------------------------

When state changes within pipeline in AWS CloudWatch Events, which can in
return create SNS notifications.

i.e. You may create events for failed pipelines or cancelled stages.

If CodePipeline fails, it will stop; and you can get information in the
console.

AWS CloudTrail can be used to audit AWS API calls.

If pipeline cannot perform an action, make sure `IAM Service Role` attached
does have enough permissions (IAM Policy).

CodePipeline Hands-On
---------------------

CodePipline has to orchestrate multiple services - so it should get a service
role (or use existing one).

You will add source stage - where you can find the stored input artifacts for
pipeline (it can be CodeCommit, ECR, S3, GitHub...). Once selected, we choose
repository name and branch.

There are two ways for CodeCommit can be tracked: CloudWatch Events or
CodePipeline.

Build stage is optional - you can choose CodeBuild or Jenkins.

Deploy stage has a target - where to deploy the code to (Beanstalk, ECS,
CloudFormation, etc...).

Once pipeline is created, it will deploy automatically. Here, we can edit and
add in as many stages as we would like. For example, we may add in stage to
manually approve the changes before it can be deploy on production Beanstalk
environment.

---

CodeBuild Overview
------------------

It is a fully managed build service - alternative to other build tools such as
Jenkins. Made for continuous scaling (no servers to manage or provision - no
build queue).

Pay for usage: the time it takes to complete the builds.

It uses Docker under the hood for reproducible builds - so, it is possible to
provide our own custom base images if needed.

Secure:

- integrated with KMS for encryption of build artifacts.
- IAM for build permissions.
- VPC for network security.
- CloudTrail for API calls logging.

It pulls source code from various places (GitHub / CodeCommit / CodePipeline
/ S3 / ...).

Build instructions can be defined in code `buildspec.yml` file.

Output logs to S3 and AWS CloudWatch Logs.

Metrics to monitor CodeBuild statistics.

Use CloudWatch Alarms to detect failed builds and trigger notifications.

Use CloudWatch Event / AWS Lambda as a Glue.

Trigger SNS Notifications.

Can reproduce CodeBuild locally to troubleshoot in case of errors.

Builds can be defined within CodePipeline or CodeBuild itself.

CodeBuild Support environment
-----------------------------

By default it supports following:

- Java
- Ruby
- Python
- Go
- Node.js
- Android
- .NET Core
- PHP
- Docker: extend any environment you like.

How CodeBuild Works
-------------------

Source Code (CodeCommit) has a `buildspec.yml` at the root.

Build Docker image (AWS managed or custom).

There will be CodeBuild Docker Container - running on Build Docker Image and
running the instructions from `buildspec.yml`. Optionally we can add S3 Cache
Bucket if we do multiple builds or complicated builds.

The created build result will be put into the S3 Bucket Artifacts; and logs
will be saved on CloudWatch or S3.

CodeBuild BuildSpec
-------------------

`buildspc.yml` must exist at the root of your code.

We can define environment variables:

- plaintext variables
- secure secrets: use SSM Parameter store; pull in secrets from secure source

Phases specify commands to run:

- install: install any dependencies.
- prebuild: final commands to execute before build.
- build: actual build commands.
- postbuild: finishing touches (package as zip file for example).

In the end, we get Artifacts to upload to S3 (encrypted with KMS).

Cache: Files to cache (usually dependencies) to S3 for future build speedup.

CodeBuild in VPC
----------------

By default, CodeBuild containers are launched outside of VPC - thus, it will
not be able to access resorces within a VPC.

Suppose you have a RDS database on a private subnet, CodeBuild container will
not be able to communicate to it.

This can be fixed by specifying a VPC configuration:

- VPC ID
- Subnet IDs
- Security Group IDs

Then, your build can access resources in your VPS (RDS, ElastiCache, EC2,
ALB...). CodeBuild Container will run within your VPS.

Use cases: integration tests, dta query, internal load balancers...

VPC options are accessed in the CodeBuild > Edit Environment.

---

CodeDeploy
----------

We want to deploy our app automatically to many EC2 instances. But these
instances are not managed by Elastic Beanstalk - we need to manage ourselves
using many open source tools such as Ansible, Terraform, Chef, Puppet, etc...).

CodeDeploy -- Steps
-------------------

Each EC2 Machine (or On-Premise machine) must be running the CodeDeploy Agent.

The agent is continously polling AWS CodeDeploy for work to do - it will send
`appsepc.yml` file.

Application is pulled from GitHub or S3.

EC2 will run the deployment instructions. And in the end, CodeDeploy Agent will
report success or failure of deployment on the instance.

Source code will include `appspec.yml` file in the root directory, and it will
be pushed onto the GitHub or AWS S3. This will trigger deployment on AWS
CodeDeploy. EC2 instances which are running the CodeDeploy Agent will poll and
will be notified of trigger, and download the new code + `appspec.yml` to
deploy.

CodeDeploy -- Others
--------------------

EC2 instances are grouped by deployment group (dev / test/ prod).

Lots of flexibility to define any kind of deployments.

CodeDeploy can be chained into CodePipeline and use artifacts.

CodeDeploy can re-use existing setup tools, works with any application, auto
scaling integration.

Blue / Green Deployment works with EC2 instances (not on-premise).


Support for AWS Lambda deployments.

Note that CodeDeploy does not provison resources - it assumes instances already
exist.

CodeDeploy Primary Components
------------------------------

CodeDeploy consists of following:

- Application: Unique Name
- Compute platform: EC2/On-Premis or Lambda
- Deployment configuration: Deployment rules for success / failures
    - EC2/On-Premis: you can specify min number of healthy instances for
      deployment.
    - AWS Lambda: specify how traffic is routed to your updated Lambda function
      versions.
- Deployment group: group of tagged instances (allows to deploy gradually)
- Deployment type: In-place or Blue / Green deployment
- IAM instance profile: need to give EC2 the permission to pull from S3 or
  GitHub.
- Application Revision: app code + `appsec.yml` file.
- Service Role: Role for CodeDeploy to perform what it needs to do.
- Target revision: Target deployment application version.

CodeDeploy AppSpec
------------------

- File section: how to source and copy from S3 / GitHub to filesystem.

- Hooks: set of instructions to do deploy the new version (hooks can have
  timeouts). The order is:

    - ApplicationStop
    - DownloadBundle
    - BeforeInstall
    - AfterInstall
    - ApplicationStart
    - **ValidateService** : making sure app is working.

CodeDeploy Deployment Config
----------------------------

- Configs:
    - One a time: one instance at a time, one instance fails -> deploy stops.
    - Half at a time: 50 %.
    - All at once: quick; but no healthy host and downtime. Good for dev.
    - Custom: i.e. num healthy host >= 75%.

- Failures:
    - Instances stay in "failed state".
    - New deployments will first be deployed to failed state instances.
    - To rollback, redeploy old deployment or enable automated rollback for
      failures.

- Deployment Targets:
    - Set of EC2 instances with tags
    - Directly to an ASG
    - Mix of ASG + tags
    - Customize in script with `DEPLOYMENT_GROUP_NAME` env variables.

In-Place Deployment - Half at a time
------------------------------------

Suppose we have four instances running app version 1. We take half of the
instances down (2), and deploy new app. Repeat for other half.

Blue/Green Deployment
---------------------

Load Balancer is attached to the ASG. New ASG is created with new applications
deployed on its instances. Load Balancer is redirected to the new ASG; and
terminate the old ASG.

CodeDeploy Hands-On
-------------------

Start with creating CodeDeploy Role in IAM (there already is an template for
CodeDeploy Roles).

Create EC2 instance where deployment will happen - attach appropriate role
(EC2InstanceRoleForCodeDeploy). SSH into the instance and install CodeDeploy
Agent - update and install. Start the service.

Tag the instance (i.e. "Environment": "Dev").

CodeDeploy environment config will have ASG, EC2 instances or On-Premise. When
EC2 instance is selected, we can choose instances by its tags. So, chooseing
"Environment" as key and "Dev" as value will select all instances tagged as
such - creating a deployment group.

CodeDeploy to EC2
-----------------

Need to define how to deploy the application using `appspec.yml` + deployment
strategy. Will do in-place update to your fleet of EC2 instances.

Can use hooks to verify the deployment after each deployment phase.

CodeDeploy to ASG
-----------------

**In-place updates**:

- Updates currently existing EC2 instances.
- Instances newly created by an ASG will also get automated deployments.

**Blue/GReen deployment**:

- required to use Load Balancer to work.
- whole new ASG with new instances are used for deployment.
- choose how long to keep the old instances.

CodeDeploy -- Roll backs
------------------------

Suppose we want to roll back to the previous versions; there is a way to
specify automated rollback options.

You can rollback based on different triggers:

- rollback when a deployment fails
- rollback when alarm thresholds are met
- disable rollbacks -- do not perform rollbacks for this deployment

- **If a roll back happens, CodeDeploy redeplos the last known good revision as
  a new deployment**.

CodeStar
--------

CodeStar is an integrated solution that regroups: GitHub, CodeCommit,
CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CloudWatch.

Helps to create "CICD-ready" projects for EC2, Lambda, Beanstalk very quickly.

Supports many typical languages (C#, Go, HTML5, Java, Node.js, PHP, Python,
Ruby).

Issue tracking integration with JIRA/GitHub Issues.

Can integrate with Cloud9 to obtain a web IDE (not available in all regions).

One dashboard to view all your components.

It is free - only pay for underlying provisioned services.

Limited in customization.



