# Introduction to Elastic Beanstalk

- EB is used to quickly deploy and manage web-apps on AWS.
- Do not have to worry about underlying complexity nor infrastructure.
- It is a PaaS.
    - Platform as a Service?
        - it is a business model - that provides platform where customer can
            develop, run, and manage apps without the complexity of building and
            maintaining the infrastructure.
    - similar to Heroku.

- in short, we choose a plaform, upload our code, and it runs without having to
    worry about infrastructure.
    - not recommended for enterprise level applications.

- EB is powerd by a Cloud Formation template setups:
    - Elastic Load Balancer (ELB)
    - Autoscaling Groups (ASG)
    - RDS Database
    - EC2 Instance preconfigured platforms
    - Monitoring (CloudWatch, SNS)
    - In-place and Blue/Green deployment methodologies
    - Security (Password rotation)
    - also can run Dockerized containers/environments


- Provides two environments:
    - Web vs Worker Environment

- Web environment creates an ASG (manages EC2 instances) and an ELB.
- Worker Environment creates an ASG, SQS Queue, creates CloudWatch Alarm to
    dynamically scale instances. This is ideal for background apps.

- Web Environment also has two types:
    - Load Balanced Env
        - uses ASG and set to scale
        - uses ELB
        - more traffic? more instances created
    - Single-Instance Env
        - cost-effective (single server)
        - ASG is still used, but scaled at 1
        - no ELB - thus, public IP used to direct traffic to server


- Deployment Policies
    - All-at-once, Rolling, Rolling with additional batch, Immutable
    - Rolling and Rolling with additional batch is unavailable to Single
        Instance Env as it requires ELB to function.
        - ELB is used to attach/dettach instances when needed.

- Deployment Policies: All-at-Once
    - when app is updated, all instances are updated at the same time.
    - all instances will be out of service while updating.
    - high risk - in case of failure, we need to roll-back and re-deploy the
        original version again to all instances.

- Deployment Policies: Rolling
    - deploys new app version to a 'batch' of instances at a time.
    - the batch's instances will be out of service during deployment process.
    - reattaches updated instances.
    - moves onto next batches, detaching it from service.
    - in case of failure, need to perform an additional rolling update in order
        to roll back the changes.

- Deployment Policies: Rolling with additional batch
    - launch new instance that will be used to replace a batch.
    - deploy update app version to new batch.
    - attach the new batch and terminate the existing batch.
    - by doing this, the capacity is never reduced (none are out of service).
        - important where the availability is critical.
    - failure? need to perform an additional rolling update to roll back the
        changes.

- Deployment Policies: Immutable
    - create a new ASG group with EC2 instances.
    - deploy the updated version of the app on the new EC2 instances.
    - point the ELB to the new ASG and delete the old ASG which will terminate
        the old EC2 instances.
    - safest way to deploy for critical applications.
    - failure? terminate the new instances since the existing instances still
        remain.


- Deployment Methods (trade-offs)
    - All-at-once
        - failure results in downtime.
        - quickest to deploy. 
    - Rolling
        - failure results in single batch out of servicei (reduced capacity).
        - second quickest for deployment.
    - Rolling with additional batch
        - failure is similar to rolling (capacity stays same).
    - Immutable
        - replicating entire environment.
        - failure impact is non-exsitent.
        - takes longest time to deploy.
    - Blue/Green
        - uses DNS changes.
            - may cause problems - DNS is already populated out in the world;
                people may still go to old server instead of new.
        - minimal failed deployment impact.
        - takes long time to deploy.


- Deployment Comparison: In-place Vs Blue/Green
    - In-place and Blue/Green Deployment are not definitive in definition and
        context can change the scope.
    - EB by default performs in-place updates.

    - In-Place could mean within the scope of EB environment:
        - All the deployment policies provided by EB can be considered In-Place
            since they are within the scope of EB env.
        - (All-at-once, Rolling, Rolling with additional batch, Immutable).

    - In-PLace could mean within the scope of the same server:
        - Deployment policies which do not invlove the server being replaced.
        - i.e. All-at-once, Rolling.

    - In-Place could mean within the scope of an uninterruped server:
        - Traffic is never routed away from the server (taken-of-service).
        - Implements Zero-downtime deploys where Blue/Greens occurs on the
            server.
        - EB cannot achieve this. (Capistrano + Ruby on Rails + Unicorn is an
            example of such deployment method).

    - In exam, In-Place refers to the scope of the EB enviroment.


    - Blue/Green deployment in the context of Elastic Beanstalk:
        - Immutable (In-Place):
            - Even though swapping ASG is a Blue/Green deployment strategy our
                Environment bay the EB container and so the mechanism of our
                deploy is happening within our environment so its consider an
                In-Place deployment.
        - Swap ENV URL (Blue/Green):
            - EB defines Blue/Green as swapping EB environments and this occurs
                at the DNS level swapping out the ELBs.
            - Entire environment change.
        - Thus, Blue/Green in EB means that pointing at different EB
            environments all together.

    - Why use Blue/Green? due to database - external resources outside of EB.
    - if EB env goes down, we lose resources stored within that env as well.
    - does not mean you cannot use outside database with In-Place, but it is
        typically used with Blue/Green.

- Configureation Files
    - EB env can be customized using config files.
    - '/.ebextensions' is a hidden folder called at the root of your project
        which contains the config files.
    - '.config' is th eextension.
    - it can config:
        - option settings.
        - linux/windows server configs.
        - custom resources.

- Env Manifest
    - 'env.yml' is stored at the root of your project.
    - when EB env is first created, it looks for this manifest file to configure
        defaults.
    
    """
    AWSConfigureationEmplateVersion: 1.1.0.0
    EnviromentName: exapro-prod+
    SolutionStack: Ruby
    EnvironmentLinks:
        "WORKERQUEUE": "worker+"
    OptionSettings:
        aws:elb:loadbalancer:
            CrossZone: true
    """

- Linux Server Configuration

    """
    packages:
        yum:
            libmemcached: []
            ruby-devel: []

    users:
        andrew:
            groups:
                - groupAdmin
            uid: 87
            homDir: "/andrew"

    groups:
        groupAdmin: {}
        groupDev:
            gid: "12"

    files:
        "/home/ec2-user/app.yml":
            mode: "000755"
            owner: root
            group: root
            content: |
                SECRET: 000destrcut0

    commands:
        1_project_root:
            command: mkdir /var/www/app
        2_link:
            command: ln -s /var/www/app /app

    services:
        sysvinit:
            nginx:
                enabled: true
                ensureRunning: true

    container_commands:
        0_collectstatic:
            command: "django-admin.py collectstatic --noinput"
        1_syncdb:
            command: "django-admin.py syncdb --noinput"
            leader_only: true
        2_migrate:
            command: "django-admin.py migrate"
            leader_only: true
        3_customize:
            command: "scripts/customize.sh"
    """

    - packages: donwload and install prepackaged apps/components
    - groups: create Linux/UNIX groups and to assign group IDs
    - users: create Linux/UNIX users
    - files: create files on the EC2 instance
    - commands: execute commands on the EC2 instance before app is setup
    - services: define which services should be started or stopped when the
        instance is launched
    - container commands: exectue commands that affect your app source code
        - Not docker commands!

- EB CLI
    - CLI is hosted on Github; to install...
    - 'git clone https://github.com/aws/aws-elastic-beanstalk-cli-setup.git'
    - './aws-elastic-beanstalk-cli-setup/scripts/bundled_installer'

    - commands:
        - 'eb init'
            configure project directory and EB CLI
        - 'eb create'
            create first env
        - 'eb status'
            see current status of env
        - 'eb health'
            view health info about the instances and the state of overall env
        - 'eb events'
            see a list of events output by EB
        - 'eb logs'
            pull logs from an instance in your env
        - 'eb open'
            open your env's website in a browser
        - 'eb deploy'
            once the env is running, deploy an update
        - 'eb config'
            take a look at the config options
        - 'eb terminate'
            delete the environment

- EB Custom Images
    - when creating an EB environment, we can specify an AMI to use instead of
        standard EB AMI.
    - A custom AMI can improve provisioning times when instance s are launched
        in your environment.
    - If you need to install a lot of software that isn't included in the
        standard AMIs.
    - To create one,
        - AWS Docs - get platform information.
        - Use CLI 'describe-platform-version' - returns an imageID
        - using this AMI ID, launch an new EC2 instance of that image.
        - configure by installing packages manually.
        - new AMI is created - use it to update ENV.

- Configuration RDS
    - Database can be added inside or outside the EB env.

    - Inside EB ENV
        - intended for generally development envs.
        - create the database within EB; thus terminated along with env.

    - Outside EB ENV
        - intended for production envs.
        - create the database from RDS separate for EB; database remains
            regardless of env.

---

### EB Cheat Sheet

- What is EB?
    - It handles the deployment, from capacity provisioning, load balancing,
        auto-scaling to app health monitoring.
    - Used to run a web-app without worrying about underlying infrastructure.

- EB is free to use.
    - Only the reousrces it provisions costs $ (such as RDS, ELB, EC2).

- Recommended for test or development apps (not production use).
    - For small business? it is ok to use.

- Can use various preconfigured platforms: Java, .NET, PHP, Node.js, Python,
    Ruby, Go, and Docker.

- Can run containers on EB as in Single-container or Multi-container - these
    containers will run on ECS instead of EC2.

- There are two environemnts: Web or Worker.
    - Web Env comes in two types: Single-Instance or Load Balanced.
        - Single-Instance ENV launches a single EC2 instance, an EIP is assigned
            to the EC2.
        - Load Balanced Env launches EC2s behind an ELB managed by an ASG.
    - Woker Environments create an SQS queue, install the SQS daemon on the EC2
        instances, and has ASG scaling policy which will add or remove instances
        based on the queue size.

- EB Deployment Policies (***):
    - All-at-Once: takes all servers out of service, applies changes, puts
        server back online; fast; but has downtime.
    - Rolling: replaces servers in batches; reduces the capacity while some
        batches are taken out of service.
    - Rolling with additional batch: adds new servers in batches to replace the
        old; maintains same capacity.
    - Immutable: creates the same amount of servers, and switches all at once to
        new servers; removing old.
    - Rolling deployment policies require ELB so cannot be used with
        Single-Instance Web Environments.

    - In-Place deployment is when deployment occurs within the environment, all
        deployment policies are In-Place.
    - Blue/Green is whe ndeployment swaps the environments (outside an env);
        when you have external resources (i.e. RDS) which cannot be destroyed -
        it is suitable for Blue/Green.

- '/.ebextensions' storea all config files.

- Custom Images can be provided to EB to improve the provisioning times.

- RDS can be created inside or outside of EB; if created in-side, it will delete
    the database along with env. It is insteded for development and test
    environments.

- 'Dockerrun.aws.json' is similar to a ECS Task Definition files and defines
    multi container config.



