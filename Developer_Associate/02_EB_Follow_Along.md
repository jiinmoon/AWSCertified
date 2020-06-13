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

- EB CLI
    - 'eb init' : creates eb env / git commit.
    - can use codecommit
        - should be seen in developer tools console under CodeCommit
    - should see .elasticbeanstalk directory created.
    - inside should be config.yml.

- Initialized and commited on CodeCommit as well.
- now, we should modify config for the environment.
    - make .ebextensions directory.
    - inside, we can set different config files.
    - i.e. create '001_envar.config'

        '''
        option_settings:
            aws:elasticbeanstalk:application:environment:
                PORT: 8081
                NODE_ENV: production
        '''
    - git push changes

- We can create EB environment now.
    - 'eb create --single' : Single-Instance
        - without single, it creates ELB.
    - funny IAM role issue - fixed by creating a temp test environment in the
        console.

- Immutable deploys.
    - make another deploy.config in .ebextensions.
        
        '''
        option_settings:
            aws:elasticbeanstalk:command:
                DeploymentPolicy: Immutable
                HealthCheckSuccessThreshold: Warning
                IgnoreHealthCheck: true
                Timeout: "600"
        '''

    - now, whenever change is made on code, and pushed on CodeCommit; 'eb deploy' 
         will deploy the new changes using the immutable deployment policy.

- Blue/Green deploys.
    - create another environment.
    - we can swap to that new environment.
    - App - 'study-sync'
        - envs
            - 'study-sync-dev'
            - 'study-sync-prod'
    - new changes made - how to deploy to new environment?
        - supply specific env name.
        - i.e. latest change has been committed and pushed. we can deploy that
            change to study-sync-dev by 'eb deploy study-sync-dev'.
    - now to change, swap environment URLs in console, or use eb cli.
        - 'eb swap study-sync-dev -- destination study-sync-prod'.

- Deploying Single-Container Dockerfile
    - dockerize the project with a Dockerfile.
    - first, create a `Dockerfile`: this contains all commands to build the
        docker image.

        """
        FROM node:10
        RUN mkdir -p /app
        WORKDIR /app
        COPY . .
        RUN npm install
        CMD ["npm", "start"]
        EXPOSE 8080
        """    

    - this is not a Docker tutorial; but please do refresh your memory by going
        through the tutorial later.
    - also, create a `dockerignore` file.

    - now, we need to change the EB configuration to deploy docker images by
        running `eb platform select`.
    - to build the dokcer image, 'docker build --tag study-sync:1.0'.
    - to see all docker images, 'docker images'.
    - to run the container locally, 'docker run --env PORT=8080 --publish
        8080:8080 study-sync:1.0'.

    - EB takes care of all the mess behind; otherwise, we would had to manage
        and build the docker images, and have it on the docker repository.
        - ECR is the repo.

    - ECR:
        - once docker image is built, we upload on ecr.
        - but first authenticate on aws ecr,
            - 'aws ecr get-login-paasword | docker login --username AWS
                --password-stdin
                your-aws-account-#.dkr.ecr.us-east-1.amazonaws.com'
            - helps crendentials - creates token inside the .docker/config.json
        - tag the docker image, 'docker tag IMAGEID
            aws-account-#.dkr.ecr.us-east-1.amazonaws.com/app-name'.
        - then push onto repo, 'docker push'
            - should create the repo on ECR beforehand.
        - (this is akin to github).
        - suppose we create another directory now to have changes.
        - create file 'Dockerrun.aws.json'
        - should have them in git repo as well.
            - add .gitignore and .ebextensions (+ eb config).

        """
            {
                "AWSEBDockerrunVersion": "1",
                "Image" : {
                    "Name" : "IMAGE URL"
                },
                "Ports" : [{
                    "ContainerPort" : 8080,
                    "MostPort" : 8080
                }]
            }
        """

        - 'eb init'; 'eb create --single'
        - should create env with docker image running at IMAGE URL.
            - where is code? code is inside the docker image hosted at ECR.
    - This will need a change in the IAM role permissions.
        - assign AmazonEC2ContainerRegistryReadOnly permission to
            aws-elasticbeanstalk-ec2-role.

- Clean Up process.
    - just terminate envs/delete apps.
