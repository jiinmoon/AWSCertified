# Continuous Integration Continuous Delivery
## Continuous Integration

- Developers push code to a code repository.
- A test, build server will pull the changes made on the code base, and
  provides feedback to the developers.
- This way, we can find bugs faster and fix them sooner.
- Development cycle is always unblocked.

## Continuous Delivery

- Ensures that the softwre can be relased reliably whenever.
- We will deploy quickly and often.
- This implies automated deployment pipeline using different tools.

## Code Commit

- This is a Git repository for AWS.
- It is a private repository without a size limit.
- Fully managed and highly available.
- Secure (access control'd and encrypted).
- Integrated with other CI tools.

### Code Commit Security

- Interactions are done with Git.
- Authentication in Git:
    - SSH Keys can be configured by AWS users in IAM console.
    - HTTPS is doen with generating HTTPS credentials.
    - MFA can be enabled.
- Authorization in Git:
    - IAM policies manage user and roles rights to repositories.
    - i.e. permission to create new branch, delete a repo and so on.
- Encrytion:
    - Repos are encrytped at rest using KMS.
    - HTTPS or SSH to push changes.
- Cross Account access:
    - Use IAM roles and use STS AssumeRole API.

### Code Commit Notifications

- Trigger notifications via SNS or Lambda or CloudWatch Event rules.
- SNS/Lamda can be used to:
    - automatically delete unused branches.
    - detect pushes to the master branch and roll back if needed.
    - notify build system.
    - perform codebase analysis.
- CloudWatch Events can be used to:
    - trigger for pull request updates.
    - commit comment events.

## CodePipeline

- It is a continuous delivery service that provides visual workflow from build,
  testing, and deploying.

- It pulls from source (CodeCommit, GitHub, S3).
- It builds the code (CodeBuild, Jenkins).
- It performs testing.
- Then, it deploys with CodeDeploy, Beanstalk, CloudFormation, ECS...

- It is comprised of different stages.
    - Each satage can have sequential actions or parallel.
    - i.e. Build - Test - Deploy - etc.
    - Can set up manual approval to pass an stage.

- Each pipeline stage will generate an artifact.
- An artifact is temporary stored in S3 after each stage and pulled in before
  next stage.

### CodePipeline and CloudWatch

- CodePipeline state changes happen in CloudWatch Event and can publish SNS
  notifications (i.e. event for failed pipelines).
- If a pipeline has failed, the pipeline will stop and you can recieve
  inforamtion on the console.

- CloudTrail can be used to audit AWS API calls.
- If CodePipeline cannot perform an action, check IAM service role attached to
  it.

## CodeBuild

- It is a fully managed build service alternative to Jenkins.
- Continuous scaling - no servers to manage.
- Pay per usage (time that it takes to build).
- Leverages Docker udner the hood for reproducible builds.
- We can use our own base images if needed.
- Integrates with KMS for encryption of artifacts; IAM for build permissions;
  CloudTrail for API calls logging; and VPC for network security.

- It pulls source code from GitHub or CodeCommit or S3.
- Build instructions in `buildspec.yml` in the source directory.
- Outputs logs to S3 and CloudWatch Logs.
- Build statistics and metrics are made available.
- CloudWatch Alarms can detect failed builds and trigger notifications.
- SNS notifications.
- **Can use it locally for troubleshooting**.
- Can be used by itself or with CodePipeline.

### CodeBuild `buildspec.yml`

- This must exist in root of the code.
- It defines envrionment varaibles:
    - Plaintext variables
    - Secure secrets using SSM Parameter Store.
- Phases (commands to run):
    - Install (dependencies)
    - Prebuild (commands before build)
    - Build (actual build commands)
    - Post build (finishing touches)
- Artifacts are generated in the end and uploaded to S3 (encrypted with KMS).
- Can cache files (dependencies) to S3 for future builds.

### CodeBuild in VPC

- By default, CodeBuild containers are launched outside your VPC.
- Therefore, by default it cannot access resouces in a VPC.
- To specify a VPC configuration, provide VPC ID, Subnet ID, Security Group
  IDs.
- Then, CodeBUild can access resources in VPC (RDS, EC2, ...).
- i.e. you may have analytics tools and database inside the VPC that wants to
  analyze builds.

## CodeDeploy

- A way to deploy application automatically to many EC2 instances which are not
  managed by the Elastic Beanstalk.
- This is alternative to Ansible, Terraform, Chef, Puppet and etc.

- EC2 instances are grouped by deployment group.
- Flexible to define any kind of deployments.
- CodeDeploy can work with CodePipline and pull artifacts.
- Blue/Green only works with EC2 instances.
- Supports Lambda deployments.

### How it works

- Each EC2 machine (or on-premises) must have a CodeDeploy agent running.
- The agent is continuously polling CodeDeploy for work.
- CodeDeploy will send out `appsepc.yml`.
- Application is pulled from GitHub or S3.
- EC2 will execute the deployment instructions.
- Agent will report fail or sucess of the deployments.

### Components

- Application : unique name.
- Compute Platform : EC2 or on-premise or Lambda.
- Deployment Configuration: rules for success or failure
    - EC2 and on-premis can set min number of healthy instances for deployment.
    - Lambda specify how traffic is routed to your updated function versions.
- Deployment Group : group of tagged instances (deploy gradually)
- Deployment Type : in-place or Blue/Green
- IAM Instance Profile : EC2 instances require permission to pull from S3 and
  GitHub.
- Application Reivision : application code + `appsepc.yml`.
- Service Role : role for CodeDeploy to perform its task.
- Target revision : target deployment application version.

### `appsec.yml`

- It has File section:
    - describes how to source and copy from S3 or GitHub into the file system.

- It has Hooks:
    - set of instructions to do while deploying the new version.
    - Application Stop
    - DownloadBundle
    - BeforeInstall
    - AfterInstall
    - ApplicationStart
    - ValidateService

### Deployment Config

- Configs:
    - One a time : one instance at a time; one fails? stop.
    - Half at a time
    - All at once
    - Custom

- Failures:
    - Instances stay in its failed state.
    - new ployments will first take place on failed state instances.
    - roll back uses old deployment.

- Deployment Targets:
    - set of tagged EC2 instances
    - ASG (with or without tags)

### CodeDeploy to EC2

- To deploy, define how to deploy the application using `appspec.yml` and
  deployment strategy.
- It will do in-place update to EC2 instances.
- Can leverage on Hooks to verify the deployment.

### CodeDeploy to ASG

- In-place updates:
    - updates current existing EC2 instances
    - instances newly created by an ASG will also get automated deployments.

- Blue/Green:
    - a entirely new ASG is created.
    - can set how long to keep the old instances.
    - requires ELB to make it work.

### CodeDeploy Rollbacks

- Can specify automated rollback options.
- Can rollback when deployment fails.
- Can rollback when alarm threshold is met.
- Can disable rollback.

- When it happens, rolls back to the last known good revision as a new
  deployment.

## CodeStar

- It is an integrated solution that groups GitHub, CodeCommit, CodeBuild,
  CodeDeploy, CloudFormation, CodePipeline, CloudWatch.
- It quickly creates CICD ready projects for EC2, Lambda and Beanstalk.
- Supports C#, Go, HTML 5, Java, Node.js, PHP, Python, Ruby.
- Issue tracking integration with JIRA and GitHub Issues.
- One dashboard to view all your components
- Free; only pay for underlying resources.
- Limited in customization.



