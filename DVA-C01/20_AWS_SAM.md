AWS Serverless Application Model (SAM)
======================================

SAM is a framework for developing and deploying serverless applications. All
the configuration is `YAML` code - and generate complex CloudFormation from
simple SAM YAML file.

It supports anything from CLoudFormation: Outputs, Mappings, Parameters,
Resources and so on.

Only needs two commands to deploy to AWS or use CodeDeploy to deploy Lambda
functions.

It helps you to run Lambda, API Gateway, DynamoDB locally.

AWS SAM -- Recipe
-----------------

- **Transform Header**:
    - This indicates it is a SAM template.
    - `Transform: 'AWS::Serverless-2016-10-31'`
    - Notice the `Transform`.

- **Write Code**:
    - `AWS::Serverless::Function`
    - `AWS::Serverless::Api`
    - `AWS::Serverless::SimpleTable`

- **Package and Deploy**:
    - `aws cloudformation package` or `sam package`
    - `aws cloudformation deploy` or `sam deploy`


