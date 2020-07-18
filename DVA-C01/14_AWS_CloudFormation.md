CloudFormation
==============

_Managing Infrastructure as code_.

Infrastructure as Code
----------------------

- Thus far, we have been doing a lot of manual work.
- These manual works are hard to reproduce:
    - in another region;
    - in another AWS account;
    - within same region if everything was deleted.

- It would be much better if we can view our infrastructure as code and manage
  it as such.

- Code would be deployed and create / update / delete our infrastructure.

CloudFormation?
---------------

It is a declarative way of outlining your AWS Infrastructure, for any resources
(most of them are supported).

i.e. within a CloudFormation template, you may specify:

- get a security group
- provision two EC2 machine with this security group
- get and assign two EIPs for these EC2 machines
- get S3 bucket
- grab ELB to route traffic to these instances

Then, CloudFormation carries these out in the right order with configuration
settings that you specify.

Benefits of CloudFormation
--------------------------

Infrastructure as code

- No resources are manually created, which is excellent for control.
- All codes are version controlled (i.e. in Git).
- Changes to the infrastructure will be code reviewed and transparent.

Cost

- Each resources within the stack is tagged with an identifier.
- Estimation of your resources cost using CloudFormation template.
- Saving strategy: In dev, you could automation deletion of templates at 5 PM
  and recreated at 8 AM safely.

Productivity

- Availity to destroy and re-create an infrastructure on the cloud on the fly.
- Automated generation of Diagrams for your templates.
- Declarative programming (no need to figure out ordering and orchestration).

Seperation of concern

- Can create many stacks for many apps and many layers.
- VPC stacks
- Network stacks
- App stacks

Don't try to re-invent the wheel

- there are already many templates to choose from.
- excellent documentation.

How does CloudFormation work?
-----------------------------

Templates have to be uploaded in S and then referenced in CloudFormation.

To update a template, we cannot edit previous ones; we have to reupload a new
version of the template to AWS.

Created stacks are identified by a name. Deleting a stack deletes every single
artifact that was created by CloudFormation.

Deploying CloudFormation templates
----------------------------------

Manual Way

- Editing templates in the CloudFormation Designer
- Using the console to input parameters...

Automated Way

- Editing templates in a YAML file
- Using the AWS CLI (Command Line Interface) to deploy the templates

CloudFormation Building Blocks
------------------------------

**Templates components (one course section for each)**:

1. Resources: AWS resources declared in the template (mandatory).
2. Parameters: dynamic inputs for your template
3. Mappings: static variables for your template
4. Outputs: references to what has been created
5. Conditionals: list of conditions to perform resource creation
6. Metadata

**Templates helpers**:

- References
- Functions

Sample CloudFormation template YAML file
----------------------------------------

```yaml
# declare resources: here we want EC2 instance
Resources:
    MyInstance:
        Type:   AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-a4c7edb2
            InstanceType: t2.micro
```

YAML Crash Course
-----------------

YAML and JSON are the languages used for CloudFormation. JSON tends to be
unreadable, thus YAML format is much more preferred.

YAML consists of Key-Value pairs; nested objects; supports arrays; multi-line
strings; and can include comments.

Learn more on YAML format.

CloudFormation Resources
------------------------

**Resources** are the basis of the CloudFormation template, and it is
_required_. They are different AWS components that will be created and
configured.

Resources are declared, and can reference one another. For example, you can
declare security group and EC2 instance; then, have it referene each other so
that you can attach security group to that instance.

AWS figures out creations, updates, and deletes of resources for us.

    AWS::aws-product-name::data-type-name

[Documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

You cannot create a dynamic amount of resources. Everything in the
CloudFormation template has to be declared. You can't perform code generation.

CloudFormation Parameters
-------------------------

Parameters are a way to provide inputs to your AWS CloudFormation template.
This is import if you wish to reuse the template or certain inputs cannot be
determined ahead of time.

Parameters are powerful, controlled, and can prevent errors from happening in
your templates due to _types_.

```yaml
Parameters:
    SecurityGroupDescription:
        Description: Security Group Description (Simple parameter)
        Type: String
```

Parameters are used when CloudFormation resource configuration is likely to change in the future.

This way, you do not have to re-upload a template to change its content.

Parameter Settings
------------------

Parameters can be controlled by these settings:

- Type:
    - String
    - Number
    - CommaDelimitedList
    - List<Type>
    - AWS Parameter (to help catch invalid values -- match against existing
      values in the AWS Account)


