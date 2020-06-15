# AWS Certificate Manager

- Amazon Certificate Manager (ACM) provision, manage, and deploy public and
    private SSL/TLS certifications for use with AWS services.

- In detail, ACM handles the complexity of creating and managing public SSL/TLS
    certificates for your AWS based websites and applications.

- It handles two kinds of certificates:

    1. Public - certificates provided by ACM (Free).

    2. Private - certificates you import (400$ / month).

- It can also handle multiple subdomains and wildcard domains.
    - i.e. Domain name 'website.co' '*.website.co'

- (***)ACM can be attached to the following AWS resources:

    - Elastic Load Balancer
    - CloudFront
    - API Gateway
    - Elastic Beanstalk (through ELB)

## ACM Example SSL Termination

- Terminating SSL at the Load Balancer

    - All traffic in-transit beyond the ALB is unencrypted; which is fairly safe
        as beyond ELB (or ALB) is your own network.
    - The advantage of doing this is that you can add as many EC2 instances to
        the ALB and you do not need to install certificates on each instace.

- Terminating SSL End-To-End

    - Traffic is encrypted in-transit all the way to the application. Even
        within own network, it will still be encrypted.
    - More complicated to maintain the certificates at instances level.


