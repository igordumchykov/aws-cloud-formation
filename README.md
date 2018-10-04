# aws-cloud-formation
Ansible playbook with AWS cloudformation integration for developing infrastructure and provisioning Nexus repository on EC2 instances.

Steps to run:
1. go to ./nexus directory
2. add aws credentials and key pair (*.csv, *.pem)
3. run ansible-playbook main.yml

Basic requirements:
1. Environment deployment should be automated
2. Environment should reside in isolated network
3. Environment access has to be restricted to "customer network" using some kind of VPN/routing (assume customer network is 172.16.10.0/24)
4. Only required ports should be opened
5. Nexus service has to be fault-tolerant. It should survive EC2 instance failure, Nexus application failure and even entire AZ failure. Nexus service has to be automatically re-deployed if it becomes unavailable.
6. Nexus service has to be available by DNS name
7. Some service downtime (in a case of failures) is acceptable, but the data has to remain available and consistent
8. Nexus service has to have an Internet access, but it shouldn't be accessible from the Internet

Steps:
1. Get familiar with Nexus OSS (latest version) installation procedure:
    - https://www.sonatype.com/nexus-repository-sonatype
    - https://help.sonatype.com/learning/repository-manager-3/proxying-maven-and-npm-quick-start-guide
2.  Develop architecture of AWS infrastructure required for Nexus service deployment:
    - manually create a user who will be responsible of running CF stack. 
    To launch an instance with a role, the developer must have permission to launch EC2 instances and permission to pass IAM roles.
    The following sample policy allows users to use the AWS Management Console to launch an instance with a role. The policy includes wildcards (*) to allow a user to pass any role and to perform all Amazon EC2 actions. The ListInstanceProfiles action allows users to view all of the roles that are available in the AWS account.
    
    Example policy that grants a user permission to use the Amazon EC2 console to launch an instance with any role
     {
       "Version": "2012-10-17",
       "Statement": [{
         "Effect": "Allow",
         "Action": [
           "iam:PassRole",
           "iam:ListInstanceProfiles",
           "ec2:*"
         ],
         "Resource": "*"
       }]
     }
     
     For more details, ref. 
     https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html
     https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html
     
     - create CF stack in YAML/JSON format
     - test CF stack from AWS Console
     
3. Automate the creation of AMI (containing Nexus binaries and config) using Ansible
    - install Ansible, boto, boto3, botocore
    - create ansible playbook file
    - run: ansible-playbook <playbook_file.yaml>
    - to run playbook, use created user credentials that you saved during user creation in csv format 
    (create externa_vars.yaml file, add ec2_access_key and ec2_secret_key from credentials, add this file to vars_files 
    section in ansible playbook)
    
4. Automate the creation of AWS infrastructure and Nexus instances with AWS CloudFormation
5. Automatically deploy all required AWS infrastructure and Nexus EC2 instances using created CF stack(s)

    Algorithm:
    - in Ansible playbook detect client IP address and register as variable
    - upload ansible playbook that will be responsible for provision EC2 instances to S3 bucket
    - run cloudformation task from ansible playbook that is responsible for AWS infrastructure creation
    
    
6. Make Nexus service available via DNS name


7. Configure Nexus to proxy "Maven central" repository


8. Test if artifacts are downloadable from deployed Nexus service
     I suggest that you use Maven for it, e.g.
       mvn dependency:get -DremoteRepositories=http://localhost:8081/repository/maven-public 
       -DgroupId=org.apache.commons -DartifactId=commons-lang3 -Dversion=3.6 -Dtransitive=false


9. Terminating different pieces of this solution (Nexus application/EC2 instances), 
verify failover procedures and check that you still can download the artifact in each case.

     
Testing Nexus provisioning using Vagrant

1. Install virtual box.
2. Initialize vagrant.
3. Change Vagrant file described here.
4. Install ansible.
5. Create ansible playbook described in main.yml.
6. Add ansible_hosts and ansible.cfg (not required because vagrant`s ansible provisioner creates it`s own)
7. Run vagrant up
8. Login to VM: vagrant ssh
9. Proxy maven repository explained below and test it.

For more details please refer:
https://www.techwalla.com/articles/how-to-set-java-home-on-centos
https://help.sonatype.com/learning/repository-manager-3/proxying-maven-and-npm-quick-start-guide
https://tecadmin.net/install-apache-maven-on-centos/

Example of Nexus Configuration as a standalone service:
https://devopscube.com/how-to-install-latest-sonatype-nexus-3-on-linux/

Example of Nexus Configuration as a docker container:
https://hub.docker.com/r/cavemandaveman/nexus/

Warning:
If you will run container/standalone service on machine with RAM = 1024, you will need to create swap file,
because Nexus requires 1200M by default.
Add swap file steps:
https://unix.stackexchange.com/questions/294600/i-cant-enable-swap-space-on-centos-7
https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-centos-7

Nexus Logs:
Nexus log you can find under NEXUS_HOME/data/log/nexus.log


