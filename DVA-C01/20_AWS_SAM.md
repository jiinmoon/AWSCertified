AWS Serverless Application Model (SAM)
======================================

SAM is a framework for developing and deploying serverless applications. All
configuration is YAML code and generate complex CloudFormation.

It supports anything from CloudFormation: Outputs, Mappings, Parameters,
Resources and so on.

Only needs two commands to deploy to AWS or use CodeDeploy to deploy Lambda
functions.

It helps you to run Lambda, API Gateway, DynamoDB locally.

AWS SAM -- Recipe
-----------------

**Transform Header**

- This indicates it is a SAM template.
- `Transform: "AWS::Serverless-2020-10-10"`.

**Write Code**:

- `AWS::Serverless::Function`
- `AWS::Serverless::Api`
- `AWS::Serverless::SimpleTable`

**Package and Deploy**:

- `aws cloudformation package` or `sam package`
- `aws cloudformation deploy` or `sam deploy`
