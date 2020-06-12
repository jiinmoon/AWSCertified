# Hands on EB Follow Along

- Use AWS Cloud9 enviroment.
    - Create a new Cloud9 Environment.
    - Create a simple web-app for testing.
        - example uses Node.js express.
        - (github.com/ExamProCo/TheFreeAWSDeveloperAssociate)

- To preview?
    - go to EC2 instance; there should be running C9 instance.
    - under Security Groups -> Inbound rules.
    - Edit inbound rules - add new (port: 8080, source: myIP...).

    - To do this in terminal/EB CLI...
    - need to find MAC Addr for EC2 instance - then use MAC Addr to get security
        group id; use CLI to update new rules.
    - 'curl -s http://169.254.169.254/latest/meta-data'
        - EC2
        - find mac addr
    - 'curl -s http://169.254.169.254/latest/meta-data/mac'
    - 'curl -s
        http://169.254.169.254/latest/meta-dtaa/network/interfaces/macs/**:**:**:**:**:**/security-group-ids'
    - fetches the security group id (or ids if multiple security groups).
    - 'aws ec2 authorize-security-group-ingress --group-id ************* --port
        8080 --protocol tcp --cidr **.**.**.**/32'
        - checkip.amazonaws.com
    - check whether new inbound rules have been added via AWS console.
        - should be Custom TCP rule added at port 8080, source with cidr
            provided.
    - can also confirm this via CLI
        - 'aws ec2 describe-security-groups --group-ids *********** --output
            text --filters Name=ip-permission.to-port,Values=8080'
        - usually returns in json; but for human eyes, --output and --filters
            are useful.

    - 'curl -s http://169.254.169.254/latest/meta-data/public-ipv4'
        - public ip to access our web app.

    - start the web app at port 8080 (set env var PORT=8080).
    - 'npm start'
    - web app should be made available at http://EC2_pubipv4:8080

- Initialize Git Repo
    - '.gitignore' is always a good to have.
        - used to ignore Node modules.
    - configure git:
        - `git init` : start new repo
        - 'git config' : user name/email.
    - get EB CLI (from github.com/aws/aws-elastic-beanstalk-cli-setup)
        - git clone/then run installer.

