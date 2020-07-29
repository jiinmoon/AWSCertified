Notes on Practicing CICD on AWS
===============================

Original paper is found at [here](https://aws.amazon.com/whitepapers/).


The Challenge of Software Delivery
----------------------------------

Competitive landscape changes quickly - operations stability vs rapid feature
development. CICD is a practice that enables both while maintaining security.

To deliver software at high velocity, DevOps is born - it is a combination of
cultural pilosophies, practices, and tools that increses ability to deliver
applications and services with speed.

AWS offers CodeStar, CodeCommit, CodePipeline, CodeBuild and CodeDeploy.

CodeStar is used to orchestrate an end-to-end software release workflow
with these services.

For existing environment, CodePipeline integrates these services with existing
tools.

What is CICD?
-------------

**Continuous Integration**

CI is a software development practice where developers regularly merge their
code changes into a central repository, after which automated builds and tests
are run.

Key goal is to find and address bugs quickly, improve quailty, and reduce the
time it takes to validate and release new software updates.

CI focuses on smaller commits and smaller code changes to integrate. 

**Continous Delivery and Deployment**

CD is a software development practice where code changes are automatically
built, tested, and repared for production release. It expands on CI by
deploying all code changes to a testing, production or both environments after
build stage has been completed.

This can be a fully automated with workflow process or partially automated with
manual steps for critical checkpoints.

When done correctly, developes are always unblocked.

**Continous Delivery != Continuous Deployment**

Not every changes made makes it all the way to the production after passing
tests. The point of continuous delivery is not to apply changes to production
immediately, but to ensure that every chage is ready to go to production.

**Benefits of Continuous Delivery**

Automate the Sofwtware Release Process.

Improve Developer Productivity. No more time wasted on manual tasks, and solving
dependencies.

Improve Code Quailty. It helps to find bugs early in the process before it
becomes a larger problem.

Deliver Updates Faster. It makes team more responsive to customer feedbacks.

**Implementing CICD**

CICD is illurstrated as a pieple where new code is submitted on one end, tested
over various stages and then published as production ready code.

    Source -> Build -> Staging -> Production

i.e. Continuous Integration

Code is commited to the source control (CodeCommit). CodePipeline is triggered
to take the changes to the CodeBuild where it will run build and unit tests.
Then, it is deployed to the test envrionment to run integration tests, load
tests and others. If ready, it can now be deployed to the production
environment.

The first phase of CI involves the developers regularly commit their code to
a central repository (CodeCommit or GitHub) and merge all changes to a relase
branch for the app. 

Unit tests should be written early to discover problems before the code makes
to the central repository.

Cotinous delivery is the next phase where it involves deploying the app to
a staging envrionmen, which is a replica of the production stack and running
more functional tests. The staging envrionment could be a static environment
premade for testing, or etc.

After staging environment is setup as infrastructure as code, a production
envrionment can be built the same way.

The final phase is the continuous deployment. It includes full automation of
the software release process (+ deployment to production environment).

As organization matures, more improvements can be added:

- More staing envrionments for specific performance, compliance, security and
  user interface tests.
- Unit tests of inrastructure and configuration code along with the application
  code.
- Integration with other systems and processes such as code review, issue
  tracking, and event notification.
- Intgration with database schema migration.
- Additional steps for audits and business approval.

Teams
-----

Recommends three developer teams for implmenting CICD: an application team, an
infrastructure team, and a tools team. Each team is about 10-12 people. 

Application team creates the application - they own backlog, stories, and unit
tests; and they develop features based on a specified application target. Goal
is to minimize the time the developers spend on non-core application tasks.

Infrastructure Team writes the code that both creates and configures the
infrastructure needed to run the application. They use tools such as
CloudFormation, Chef, Puppet or Ansible. They are responsible for specifying
what resources are needed and works closely with the application team. The
teams should have skills in infrastructure provisioning methods such as
CloudFormation or HashiCorp Terraform; and configuration automation skills with
tools such as Chef,Ansible, Puppet, or Salt.

Tools team builds and manages the CICD pipeline, and is responsiblefor
infrastructure and tools the make up the pipeline. They create tools to be used
by other two teams in the organization. They must be skilled in building and
integrating all parts of the CICD pipeline. i.e. Building source control
repositories, workflow engines, build environments, testing frameworks, and
artifact repositories. Can implement CodeStar, CodePipeline, CodeCommit,
CodeDeploy, and CodeBuild with Jenkins, GitHub, Artifactory, TeamCenter and
etc. This is close to typical DevOps team but DevOps is sum of the people,
processes and tools in software delivery.

Testing Stages in CICD
----------------------

The three teams should incorporate testing to development lifecycle at various
stages of the pipeline - and should start early as possible. Unit tests are
cheapest and fast to run, followed by Service/Integration/Components,
Performance/Compliance, and UI.

Unit tests should make up of bulk of the testing strategy (~70 %) and should
have near-complete code coverages.

Services, component, and integration tests require detailed envrironments;
thus, they are more costly and slower to run. 

Performance and compliance tests require production-grade envrionments as well
as the UI tests.

Setting up the Source
---------------------

In the beginning of the project, it is essential to set up a source where we
can store the raw code and configuration and schema changes. Choose a source
code repository such as CodeCommit or GitHub.

Setting Up and Executing Builds
-------------------------------

Build automation is essential to the CI process. First, we need the right tool:
Ant, Maven, and Gradle for Java; Make for C/C++; Grunt for JS; and Rake for
Ruby. And we need to clearly define dependencies within the build scripts with
the build steps. Versionning is best practice.

In build stage, build tools take any changes to the source in the code
repository, build the software and run the unit tests.

Staging
-------

In this phase, full envrionments are created that mirrors the production.

Integration Testing - verifies the interfaces between componenets against
software design.

Component Testing - tests message passing between various componenets.

System Testing - tests system end-to-end.

Performance Testing - detemines the responsiveness and stability of a system
under a workload.

Compliance Testing - tests whether code meets the requirements to specs or
regulations.

User Acceptance Testing - validates end-to-end business flow.

Production
----------

Staging is repated in the production environment and **Canary test** is
performed.

---

Building the Pipeline
---------------------

CodeStar uses CodePipeline, CodeBuild, CodeCommit and CodeDeploy with an
integrated setup process, tools, templates, and dashboard.

CodePipeline is a CICD service that builds, tests, and deploys code whenever
there is a change. This is a pay-per-usage.

For CodePipline, source can be from GitHub, CodeCommit and S3. It works with
CodeBuild to set up a build step in the piepline that packages code and runs
unit tests. Else, it can work with Jenkins, Solano CI, and TeamCity.

After CI pipeline has been implemented, it is time to build continuous delivery
pipeline. It is characterized by Staging and Production steps, where the
Production step is performd after a manual approval.

CodeDeploy automates code deployments to EC2 instances and instances running
on-premises, and AWS OpsWorks which is a configuration management service helps
you operate applications using Chef and to Elastic Beanstalk.

CodeStar and CodePipeline integrates with Lambda to perform following tasks:

- Roll out changes to your environment by applying or updating an
  CloudFormation template.
- Create resources on demand in one stage of a pipeline using CloudFormation
  and elete them in another stage.
- Deploy application versions with zero downtime in EB with a Lambda function
  that swaps CNAME values.
- Deploy to EC2 ECS Docker instances.
- Back up resources before building or deploying by creating an AMI snapshot.
- Add integration with third-party products to your pipeline (i.e. post
  messages to an IRC client).

CodePipline lets you select CloudFormation as a deployment action.

For serverless applications, CodeStat, CodePipeline, CoudBuild and
CloudFormation works to build CICD pieplines.

CodeBuild is designed o enable your organization to build a highly available
build process with unlimited scale.

Deployment Methods
------------------

**Deploy in place**

- Has a downtime, but quickest to deploy.

**Rolling**

- Single batch will be down, reducing the capacity.

**Rolling with Additional Batch**

- No downtime as we create additional instances to deploy and replace old.

**Immutable**

- Create new instances to shift over.

**Blue/Green**

- No downtime as we create completely new environment and change DNS.

---

Summary of Best CICD Practices
==============================

**Do's**

Treat your infrastucture as code:

- Use version control for infrastructure code.
- Make use of bug tracking/ticketing systems.
- Have peers review changes before applying.
- Establish infrastructure code pattersn and designs.
- Test infrastructure changes like code changes.

Put developers into integrated tams of sized 10-12.

Have developers commit code frequently.

Consistently adopta build system Maven or Gradle; standardize builds.

Have developers to build unit tests that covers 100% of code coverage.

Ensure that unit tests make up 70% of all tests.

Ensure that unit tests are up-to-date.

Treat continuous delivery configuration as code.

Establish role-based security conrols (who can do what and when):

- Monitor/track every resource.
- Alert on services, availability, and response times.
- Capture, learn, and improve.
- Share access with everyone on the team.
- Plan metrics and monitoring into the lifecycle.

Keep and track standard metrics:

- Number of builds and deployments.
- Average time for changes to reach production.
- Average time from first stage to each stage.
- Number of changes reaching production.
- Average build time.

Use multiple distinct pipelines for each branch and team.

**Don't**

Have long-running branches with large complicated merges.

Have manual tests.

Have manual approval proceeses, gates, code reviews and security reviews.


