AWS CloudFormation
==================

Infrastructure as a Code
------------------------

Thus far, we have been doing a lot of manual work which are hard to reproduce
to another region, account or even in the same account in case of recovery. It
would be much simpler to control our infrastructure with a code that is much
easier to create, update and delete.

CloudFormation
--------------

It is a declarative way of outlining your AWS infrastructure. For example,
within a CloudFormation template, you may specify:

- Provision a security group.
- Provision two EC2 instances with above security group attached.
- Request and attach two EIPs for these instances.
- Create a S3 bucket.
- Create an ELB to route trafic to these instances.

CloudFormation -- Benefits
--------------------------

**Infrastructure as a Code**

- No resources are manually created.
- All codes are version controlled.
- Changes to the infrastructure will go through code reviews and it will be
  transparent.

**Cost**

- Each resources within the stack is tagged with an identifier.
- Estimation of your reousrces cost using CLoudFormation template.
- For example, you can specifiy a schedule such that remove the templates at
  certain time, and resume later.

**Separation of Concerns**

- Can create many stacks for many apps and layers.
- VPC stacks.
- Network stacks.
- Application stacks.

**Leverage on existing solutions**

- There are already many templates to choose from with excellent documentation.

CloudFormation -- Process
-------------------------

Templates are uploaded to the S3, and is referenced by CloudFormation. To
update a template, we cannot edit the previous template but you must reupload
the new template.

Created stacks are identified by its name - and deleting a stack deletes every
single artifact that was created with CloudFormation.

CloudFormation -- Deploying Templates
-------------------------------------

Manually you can edit templates in the CloudFormation Designer, or use the
console to input parameter.

We can automate this by editing the templates in YAML, and use the CLI to
deploy the templates for us.

CloudFormation -- Building Blocks
---------------------------------

**Template Compoenents**

1. Resources : AWS resources needed (**mandatory**).
2. Parameters: dynamic inputs for your template.
3. Mappings: static variables for your template.
4. Outputs: references to what has been created.
5. Conditionals: list of conditions to perform resource creation.
6. Metadata

**Template Helpers**

- References
- Functions

CloudFormation -- Example of Template YAML File
-----------------------------------------------

```yaml
# provision EC2 instance
Resources:
    MyEC2Instance:
        Type:   AWS::EC2::Instance
        Properties:
            AvailabliltyZone: us-west-1a
            ImageId: ami-1234567890
            InstanceType: t2.micro
```

CloudFormation -- Resources
---------------------------

**Resources** are the basis of the CloudFormation template (it is
**required**). They are each different AWS resources that will be created and
configured in the template.

Resources are declared and can reference one another. For instance, you can
declare a security group and an EC2 instance; and have the security group
reference to EC2 instance.

> AWS::aws-service-name::data-type-name

[Documentation](http://docs.aws.amazon.com/AWSCLoudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

You may not create a dynamic amount of resources.

CloudFormation -- Parameters
----------------------------

**Parameters** are a way to provide inputs to your AWS CloudFormation template.
This is to import previously used template or inputs that cannot be determined
ahead of time.

```yaml
Parameters:
    SecurityGroupDescription:
        Description: Security Group Description
        Type: String
```

Parameters are used when CloudFormation resource configuration is likely to
change over time; this way, we do not have to re-upload everytime a config
changes.

To reference a parameter, use `Fn::Ref` function:

```yaml
DbSubnet1:
    Type:   AWS::EC2::Subnet
    Properties:
        VpcId:  !Ref MyVPC
```

**Pseudo-parameters** are AWS provided parameters which are available to us
even without having to define them in the parameters.

| Reference Value | Example Return Value |
| --- | --- |
| AWS::AccountId | 12367890 |
| AWS::NofiticationARNs | [arn:aws:sns:us-west-1:1234567890:myTopic] |
| AWS::NoValue | |
| AWS::Region | us-west-1 |
| AWS::StackId | arn:aws:cloudformation:us-west-1:1234567890:stack/myStack/1234 |
| AWS::StackName | myStack |

CloudFromation -- Mappings
--------------------------

**Mappings** are used for fixed variables within the template.

Mappings are used over parameters when the values are known beforehand.

`Fn:FindInMap` is used to access the mapping values. For example:

```yaml
RegionMap:
    us-west-1:
        "32": "ami-0000001",
        "64": "ami-0000002"
    us-west-2:
        "32": "ami-0000003",
        "64": "ami-0000004"

Resources:
    MyEC2Instance:
        Type:   "AWS::EC2::Instance"
        Properties:
            ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", 32]
            InstanceType: t2.micro
```

CloudFormation -- Outputs
-------------------------

**Outputs** are optional output values that we can import into another stack.
This allows linking of multiple CloudFormation templates.

You cannot delete a CloudFormation stack if its outputs are being referenced by
another CloudFormation stack.

```yaml
Outputs:
    StackSSHSecurityGroup:
        Description: SSH Security Group for company
        Value: !Ref MyCompanySSHSecurityGroup
        Export:
            Name: SSHSecurityGroup
```

CloudFormation -- Cross-Stack Reference
---------------------------------------

We **import** the exported outputs from another template using
`Fn::ImportValue`.

```yaml
Resources:
    MySecureInstance:
        Type:   "AWS::EC2::Instance"
        Properties:
            AvailabilityZone:   us-west-1a
            ImageId:            ami-1234567890
            InstanceType:       t2.micro
            SecurityGroups:
                - !ImportValue SSHSecurityGroup
```

CloudFormation -- Conditions
----------------------------

```yaml
Conditions:
    # create new resource if environment is PROD
    CreateNewResource: !Equals [ !Ref EnvType, PROD ]
```

CloudFormation -- Intrinstic Functions
--------------------------------------

The important template functions to know are as follows:

- `Ref`
- `Fn::GetAtt`
- `Fn::FindInMap`
- `Fn::ImportValue`
- `Fn::Join`
- `Fn::Sub`
- Conditions (`Fn::If`, `Fn::Not`, `Fn::Equals`)

CloudFormation -- `Ref`
-----------------------

`Ref` references either

- Parameters
- Rsources (returns physical ID)

```yaml
Resources
    DbSubnet1:
        Type:   "AWS::EC2::Subnet"
        Properties:
            VpcId:  !Ref MyVpcId
```

CloudFormation -- `Fn::GetAtt`
------------------------------

Attributes are attached to any resources - refer to documentations for all the
available attributes. For example, to access the AZ of the EC2 machine:

```yaml
Resources:
    MyEC2Instance:
        Type:   "AWS::EC2::Instance"
        Properties:
            ImageId:    ami-1234567890
            InstanceType:   t2.micro
    MyVolume:
        Type:   "AWS::EC2::Volume"
        Condition:  CreateProdResources
        Properties:
            Size: 100
            AvailabilityZone:
                !GetAtt MyEC2Instance.AvailabilityZone
```

CloudFormation -- `Fn::FindInMap`
---------------------------------

        !FindInMap [ MapName, TopLevelKey, SecondLevelKey ]

CloudFormation -- `Fn::ImportValue`
-----------------------------------

Import values that are exported from other templates.

CloudFormation -- `Fn::Join`
----------------------------

Join values with a delimiter:

        !Join [ delimiter, [ commadelimited list of values ] ]

CloudFormation -- `Fn::Sub`
---------------------------

It is a substitue function usefully for further customizing your templates.

i.e. You may combine `Fn::Sub` with reference or pseudo-variables.

        !Sub
        - String
        - { Var1Name: Var1Value, Var2Name: Var2Value }

CloudFormation -- Rollbacks
---------------------------

Stack Creation Fails?

- By default, every rolls back.
- Optionally we can disable rollback to troubleshoot.

Stack Update Fails?

- Stack automatically rolls back to the previous known working state.
- Ability to see in the log what happened and error messages.

CloudFormation -- ChangeSets
----------------------------

When updating a stack, we need to know what changes will take place for greater
confidence. ChangeSet does not gurantee whether the update will be successful.

CloudFormation -- Nested Stacks
-------------------------------

Nested stacks are stacks as part of other stacks. It allows you to isolate
repeate patterns and common componenets in separate stacks and call them from
othe stacks.

i.e. Load balancer configurations or Security Group is likely to remain same
over the deployments.

CloudFormation -- Cross-Stack vs Nested Stacks
----------------------------------------------

**Cross-Stack**

- helpful when stacks have different lifecycles.
- use **outputs** exported by one template, and imported on another with
  `Fn::ImportValue`.

**Nested Stacks**

- helpful when components are re-used.
- nested stack only imports to the higher level stack.

CloudFormation -- StackSets
---------------------------

You can create, update, or delete stacks across multiple accounts and regions
with a single operation.

- Admin account is needed to create StackSets.
- Trusted accounts can perform create, update, delete stack instances from
  StackSets.
- When a StackSet is updated, all other stack instances are updated across
  accounts and regions.
