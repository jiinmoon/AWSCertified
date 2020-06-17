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

- In IAM console, we will remove the access key for this user now. We can create
    new access keys for users anytime.
- Now, when we go back to terminal, and try to use aws CLI, it will give an
    InvalidAccesskeyId error.

- To learn more about AWS CLI, look for doc about AWS CLI Command Reference.

---

## CLI Exam Tips

- Practice a good security principle; in particular, **Least Privilege**:
    - give users the minimum required access.

- Always create groups:
    - Assign users to groups; users will then automatically inherit the
        permissions of the group.
    - Permissions are assigned using policy docs (JSON format).

- **Secret Access Key** is shown only once. Remember to save it. If lost, ask
    root account holder or admin to create a new one for you. Then, to use CLI,
    need to configure credentials using 'aws configure'.

- Never use one access key and share amongst the devs.
    - i.e. a dev can leave the company, in which case we need to delete the
        access key and then redistribute.

- Never upload your personal credentials to GitHub or etc. This is a basic
    stuff.

- Use CLI on your PC; i.e. treat S3 as a personal cloud storage!
    - Install details can be found in AWS CLI GitHub page.

