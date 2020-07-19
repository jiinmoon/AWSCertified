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

Note that you do not have to know this to heart for exam.

Parameters can be controlled by these settings:

- Type:
    - String
    - Number
    - CommaDelimitedList
    - List<Type>
    - AWS Parameter (to help catch invalid values -- match against existing
      values in the AWS Account)
- Description
- Constraints
- ConstraintsDescription
- Min/MaxLength
- Min/MaxValue
- Defaults
- AllowedValues (array)
- AllowedPattern (regexp)

Referencing a parameter
-----------------------

To reference a parameter `Fn::Ref` function is used in YAML.

```yaml
DbSubnet1:
    Type:   AWS::EC2::Subnet
    Properties:
        VpcId:  !Ref MyVPC
```

More comprehensive example:

```yaml
# param declaration; it will be referenced later.
Parameters:
    SecurityGroupDescription:
        Description:    Security Group Description
        Type:           String

Resources:
    MyInstane:
        Type:   AWS::EC2::Instance
        Properties:
            AvailabilityZones: us-east-1a
            ImageId: ami-a4c7edb2
            InstanceType: t2.micro
            SecurityGroups:
                # referenced from below
                - !Ref SSHSecurityGroup
                - !Ref ServerSecurityGroup

    MyEIP:
        Type:   AWS::EC2::EIP
        Properties:
            InstanceId: !Ref MyInstance

    SSHSecurityGroup:
        Type:   AWS::EC2::SecurityGroup
        Properties:
            GroupDescription: Enable SSH access via port 22
            SecurityGroupIngress:
            - CidrIp: 0.0.0.0/0

...
```

Concept: Pseudo Parameters
--------------------------

AWS offers us pseudo parameters in any CloudFormation template, which can be
used at anytime and always enabled by default.

| Reference Value | Example Return Value |
| --- | --- |
| AWS::AccoundId | 1234567890 |
| AWS::NofiticationARNs | [arn:aws:sns:use-east-1:123456789012:MyTopic] |
| AWS::NoValue | Does not return a value. |
| AWS::Region | us-east-2 |
| AWS::StackId | arn:aws:cloudformation:us-east-1:123456789012:stack/MyStack/1c2fa620-... |
| AWS::StackName | MyStack |

CloudFormation Mappings
-----------------------

**Mappings** are fixed variables within your CloudFormation Template - they are
handy to differentiate between different environments (dev vs prod), regions
(AWS regions), AMI types, and etc...

Values are hard-coded within the template.

```yaml
Mappings:
    Mapping01:
        Key01:
            Name: Value01
        Key02:
            Name: Value02
        Key03:
            Name: Value03
```

Example of using different images for regions based on architecture:

```yaml
RegionMap:
    us-east-1:
        "32": "ami-6411e20d"
        "64": "ami-7a11e213"
    us-west-1:
        "32": "ami-c9c7978c"
        "64": "ami-cfc7978a"
```

When to use Mappings vs Parameters?
-----------------------------------

Mappings are better if you know the all the values beforehand; and that they
can be deduced from variables such as:

- Region
- AZ
- AWS Account
- Environment (dev vs prod)

Mappings allow for safer control over the template.

Use parameters when the values are really user specific.

Fn::FindInMap
-------------

`Fn:FindInMap` is used to access the mapping values; the named value from
a specific key.

Example:

```yaml
Mappings:
    RegionMap:
        us-east-1:
            "32": "ami-12312e11"
            "64": "ami-7171722e"
        us-west-1:
            "32": "ami-92932aq2"
            "64": "ami-c99128a1"

Resources:
    MyEC2Instance:
        Type: "AWS::EC2::Instance"
        Properties:
            # pseudo parameter AWS::Region is used to find current region
            ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", 32]
            InstanceType: t2.micro
```

CloudFormation -- Outputs
-------------------------

**Outputs** are optional outputs values that we can import into other stacks
(export first). Thus, allowing you to linking multiple CloudFormation
templates.

You can find the outputs in the AWS Console or using the CLI.

Use-case: define a network CloudFormation and output the variables such as VPC
ID and your Subnet IDs.

This is the best way to perform some collaboration between multiple stacks.

You cannot delete a CloudFormation stack if its outputs are being referenced by
another CloudFormation stack.

Example:

Here, we create an output that references security group.

```yaml
Outputs:
    StackSSHSecurityGroup:
        Description: The SSH Security Group for our Company
        Value: !Ref MyCompanyWideSSHSecurityGroup
        Export:
            Name: SSHSecurityGroup
```

Cross Stack Reference
---------------------

To **import** the outputs (exported references) of another template, we use
`Fn::ImportValue` function.

Suppose on the another second template:

```yaml
Resources:
    MySecureInstance:
        Type: AWS::EC2::Instance
        Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-a4c22e33
            InstanceType: t2.micro
            SecurityGroups:
                - !ImportValue SSHSecurityGroup
```

CloudFormation Conditions
-------------------------

Used to control the creation of resources or outputs based on a condition.

Conditions can be anything; here are few examples:

- environment? (dev, test, or prod)
- AWS region?
- Any parameter value

Each condition can reference another condition, parameter value or mapping.

```yaml
Conditions:
    # Following condition evals to True iff EnvType == prod
    CreateProdResources: !Equals [ !Ref EnvType, prod ]
```

Conditions are applied to resources, outputs and etc.

```yaml
Resources:
    # MountPoint resources is created iff CreateProdResources == True
    MountPoint:
        Type: "AWS::EC2::VolumeAttachment"
        Condition: CreateProdResources
```

CloudFormation -- Intrinsic Functions
-------------------------------------

Here are functions that you must know how it works:

- `Ref`
- `Fn::GetAtt`
- `Fn::FindInMap`
- `Fn::ImportValue`
- `Fn::Join`
- `Fn::Sub`
- Conditions (`Fn::If`, `Fn::Not`, `Fn::Equals`, ...)

`Ref`
-----

`Fn::Ref` function is for _reference_:

- Paremeters, which returns the value of the parameter
- Resources, which returns the physical ID of the underlying resource (i.e. EC2
  instance ID)

```yaml
Resources:
    DbSubnet1:
        Type: "AWS::EC2::Subnet"
        Properties:
            VpcId: !Ref MyVPC
```

`Fn::GetAtt`
------------

_Attributes_ are attached to any resources we create - to see all the
attributes available for each resources, refer to the documentation.

i.e. to access the AZ of an EC2 machine:

```yaml
Resources:
    MyEC2:
        Type: "AWS::EC2::Instance"
        Properties:
            ImageId: ami-1234567
            InstanceType: t2.micro

    MyVolume:
        Type: "AWS::EC2::Volume"
        Condition: CreateProdResources
        Properties:
            Size: 100
            AvailabilityZone:
                !GetAtt MyEC2.AvailabilityZone
```

`Fn::FindInMap`
---------------

To find the a named value from a specific key in previously defined Mapping.

    !FindInMap [ MapName, TopLevelKey, SecondLevelKey ]

`Fn::ImportValue`
-----------------

Import values that are exported in other templates; `Fn::ImportValue`.

`Fn::Join`
----------

Join values with a delimiter.

    !Join [ delimiter, [ comma-delimited list of values ] ]


i.e. following will create a return value of "a:b:c"

    !Join [ ":", [ a, b, c ] ]

`Fn::Sub`
---------

It is a _substitute_ function usefuly for further customizing your templates.

i.e. you may combine `Fn::Sub` with references or pseudo-variables.

**String** must contain ${VariableName} - and it will be substituted.

    !Sub
     - String
     - { Var1Name: Var1Value, Var2Name: Var2Value }

Conditions
-----------

```yaml
Conditions:
    CreateProdResources: !Equals [ !Ref EnvType, prod ]
```

Functions can be `Fn::And`, `Fn::Equals`, `Fn::If`, `Fn::Not`, `Fn::Or`...

---

CloudFormation Rollbacks
------------------------

Stack Creation Fails?

- Default: everything rolls back (gets deleted); log is available.
- Optionally we can disable rollback and troubleshoot.

Stack Update Fails?

- stack automatically rolls back to the previous known working state.
- ability to see in the log what happened and error messages.

CloudFormation ChangeSets
-------------------------

when updating a stack, we need to know what changes will take place for greater
confidence - it is like testing.

ChangeSets does not gurantee whether the update will be successful.

Original Stack will create Change set, and view the changes; if we are fine,
then we can execute the change set.

CloudFormation Nested stacks
----------------------------

Nested stacks are stacks as part of other stacks. It allows you to isolate
repeated patterns / common components in separate stacks and cal them from
other stacks.

i.e. Load Balancer configurations or Security Groups will mostly likely to be
used same throughout the stacks.

Nested stacks are considered best practice.

To update nested stacks, must update the parent first.

Cross Stack vs Nested Stacks
----------------------------

**Cross Stacks**

- helpful when stacks have different lifecycles.
- use outputs export and `Fn::ImportValue`.
- when you need to pass export values to many stacks.
- i.e. we have a VPC stack, and its VPC ID is exported to many other stacks.

**Nested Stacks**

- helpful when componenets must be re-used.
- i.e. it makes sense to reuse the properly configured Application Load
  Balancer instead of creating a new resource each time with app.
- the nested stack only is import to the higher level stack (it is not shared).

StackSets
---------

create, update, or delete stacks across _multiple accounts and regions_ with
a single operation.

- administrator account to create StackSets.
- trusted accounts to create, update, delete stack instances from StackSets.
- when you update a stack set, all associated stack instances are updated
  throughout all accounts and regions.


