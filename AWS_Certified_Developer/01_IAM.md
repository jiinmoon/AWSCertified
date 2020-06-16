# Identity Access Management

---

## What is IAM?

- IAM allows you to manage users and their level of AWS Console. It is important
    to understand IAM for administrating a company's AWS account in real life.


## IAM functions are...

- Centralized control of your AWS account.
- Shared Access to your AWS acount.
- Granular Permissions - varying levels of access to users.
- Identity Federation.
- Multifactor Authentication.
- Provides temporary access for users/devices and services, as necessary.
- Allows you to set your own password rotation policy.
- Integrates with many different AWS services.
- Supports PCI DSS Compliance,

## Critial Terms

- Users :  End Users (people).

- Groups : A collection of users under one set of permissions.

- Roles : Create role sand assign them to AWS resources.

- Policies : A doc that defines one or more permissions.


## IAM Lab Follow-Along

- Log into the **AWS Management Console**.

- **IAM** can be found under Security, Identy & Compliance (or, simply search in
    Find Services).

- IAM console will show you IAM users sign-in link.
    - i.e. 'https://YOUR-ACCOUNT-NUMBER.signin.aws.amazon.com/console'.
    - URL can be customized.
    - Users can follow that link to access the console.

- Should never use/hand-out root account access to everyone.
    - Best practice is to create individual IAM users with permissions.
    - For example, a database manager may only need to access to database
        services; or, HR team may need S3 bucket access.

- Should activate MFA (Multi-Factor Authentication) for root-account:
    - Virtual MFA device.
    - Install a compatible authentication app on your device.
    - Type in two consecutive codes.

- Now, create individual IAM users:
    - Goto Users tab; then Click Add user.
    - There are two different AWS access types:
        - Programmatic access using AWS CLI (+ API, SDK, etc...), or
        - AWS Management Console access.
    - Then, we need to set permissions. There are 3 ways to do this:
        - Add users to group.
        - Copy permissions from another user.
        - Attach existing policies.
    - For now, let's choose Adding users to group.
    - Since this is our first time, need to create a group; let's create one
        named 'sysadmin'; and we need to select policies to apply.
        - You can view which policy to apply; and they have a JSON version to
            show what they are capable of.
        - Below is an example of **AdministratorAccess** policy.

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "*",
                "Resource": "*"
            }
        ]
    }
    ```

    - After assigning group to user, you may optionally add **tags** which is a
        key-value pairs include user information (such as e-mail addr, or job
        title). This is for purely organizational, tracking purposes.
    - When user is created, we can see user security credentials; and it can be
        view or downloaded. You may also forward new user by e-mail with
        instructions for signing in to the AWS Management Console.
        - You can see **Access key ID** and **Secret access key** required for
            terminal access (CLI).
        - Caution! This will be the only time to see these.

    - Continue same process to create another hypothetical user, adding to
        sysadmin.
    - However, we changed our mind and wish to assign him to a new group.
    - To do so, create a New Group; let's name it 'HR'. And we can attach policy
        to this group. Select AmazonS3ReadOnlyAccess.
    - Then, we should go to the hypothetical user, changing its Groups to 'HR'
        by add user to groups; Remember to delete him from sysadmin group.
    - Also, we can attach specific policy directly to grant permissions.

- Lastly, we should set a IAM password policy.
    - We can set minimum password length.
    - Require at least on uppercase letter.
    - etc...

- We can create **Roles** as well.
    - IAM roles are way to grant permissions to trusted entities such as
        - IAM user in another account;
        - App code running on an EC2 instance that needs to perform actions on
            AWS resource;
        - An AWS service that needs to act on resources in your account to
            provide its features.
    - As practice, let's create an Role that allows EC2 instance to have right
        to write to S3.

    - Create Role; select service - in this case, EC2.
    - Now, need to attach permissions policies; search for AmazonS3FullAcess.
    - This role can be then later apply to any EC2 instances created to give it
        a full access over AWS S3 service.

---

# IAM Summary

- IAM consists of:
    - Users
    - Groups (collectively apply policy to group of users)
    - Roles
    - Policy Documents (JSON format)

- **IAM is universal** - meaning that it is global; not regional.

- 'root account' refers to the very first account created when first setup AWS
    account; it has complete admin privileges.

- New users do not have any permissions when first created.

- New users are assigned Access Key ID & Secrey Access Keys upon creation.
    - This is not a password for AWS Management Console.
    - This is to access AWS via APIs and terminal.
    - Can only see this once; if lost, need to be regenerated.

- Should enable MFA for root account for best practices.

- Should also create password rotation policy.

