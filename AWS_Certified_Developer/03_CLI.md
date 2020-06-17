# Command Line Interface follow-along

- This is a demo on how to interact with AWS using CLI, within the EC2 instance
    that we have created before.

- SSH into the running EC2 instance:

> ssh ec2-user@YOUR.EC2.Instance.IP -i YOUR-KEY-PAIR-NAME.pem

- Elevate to root privilege.
- We want to use the AWS command line, it follows by 'aws [name of AWS
    service]'.

> sudo su
> aws S3 ls

- For first time, this will return an error saying that it cannot locate
    credentials. To do so, run AWS confgure.

> aws configure

- Once done so, it will ask for access key and secret access key to use AWS CLI.
- To find them, let's go back to the console.
- Under IAM, we will create a new user.
    - This user will access AWS via Programmatic Access, using AWS API, CLI,
        SDK, or etc.
- Now, when first created, it will give us Access KeyID and Secret Access Key.
    - This is what you will use.
- After credentials have been configured, we can now access AWS services in CLI.
- Let's try to see the list of S3 buckets again.

> aws S3 ls

- It should be empty if S3 bucket is not used yet.
- We can create new bucket.

> aws S3 mb s3://YOUR-BUCKET-NAME

- It should have been made; which can be seen.
- Let's try uploading a file to this bucket.

> aws s3 cp PATH_TO_FILE s3://YOUR-BUCKET-NAME

- We can see it has been uploaded by ls on the specific bucket.

> aws s3 ls s3://YOUR-BUCKET-NAME


