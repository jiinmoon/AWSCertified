# EC2 with S3 Role

- Previously, we have SSH into the running EC2 instance and used AWS CLI.
- But to do so, we had to create an user and required Access key ID to use AWS
    API; we have used CLI to create buckets and so on.
- Instead, we will create Role with S3AdminAcess. Then, we will apply that role
    to the running EC2 instance!

---

- Let's log into the AWS Management Console.
- Goto IAM; we can see the list of users.
- Now, we will create new Role.
    - Again, IAM roles are a way to grant permissions to trusted entities.
    - i.e. app code running on EC2 instance that wish to interact with other
        AWS services such as S3 in this case.
- Create role for AWS ervice; we will choose EC2.
- Now, we will need to select permissions policies to attach to this Role.
    - think carefully before giving it a full AdminAcess; if your EC2 instance
        is compromised, it can now control over your entire AWS services.
    - hence, we will give it only S3FullAccess.
    - *know how to read JSON formated policies.
- Then, we will apply this role to our EC2 instance; let's head over to EC2.
- To double check, SSH into the EC2 instance and try to interact with AWS CLI;
    it should not work.
- On the running EC2 instance, attach the new role that we have created
    previously.
- Now, check back on the SSH to see whether we can now interact with S3 via AWS
    CLI.
- Just a side note: on running EC2 instance, once we switch to root and go to
    home directory, we should see the hidden directory '~/.aws'.
    - here, we can find 'config' and 'credentials' for AWS CLI.
    - so if required, we can change them manually if you would like.

---

# Exam Tips

- IAM Roles allow you to not use Access Key ID's and Secret Access Keys.

- Roles are preferred from a security perspective!

- Roles are controlled by policies.

- You can change a policy on the role and it takes immediate affect.

- Attaching/Detaching roles to running EC2 instances are possible without having
    to stop/terminate them.


