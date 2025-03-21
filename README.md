# Lift-And-Shift-Application-Workload
Multi Tier Web Application Stack (VProfile), Host and Run on AWS Cloud for Production and Lift and Shift Strategy



# Scenario
Application Services running on Physical/Virtual machines.
Workload in Datacenter.
Virtualization team, DC OPS Team, Monitoring team, Sys Admins etc are involved.


# Problem that can be solved through this process
1. Complex Management.
2. Scale Up/Down complexity.
3. Upfront CapEx & Regular OpEx.
4. Manual Process.
5. Difficult to automate.
6. Time Consuming.
 
# Solution For Cloud SetUp
1. PayAsUGo.
2. IAAS.
3. Flexiblity.
4. Ease of Infra Management.
5. Automation

# AWS Services that will be Use
1. EC2 Instances : VM for TomCat, RabbitMQ, MEMCACHE, MySQL
2. ELB [Load Balancer]: NGINX LB Replacement.
3. Autoscaling: Automation For VM Scaling
4. Route 53: Private DNS Service
5. S3/EFS Storage: Shared Storage

# Objective
1. Flexible Infra
2. No Upfront Cost
3. Modernize Effectively
4. Ifra As A Code


  ![infra](https://github.com/user-attachments/assets/914709b9-e915-4556-90ba-3e168b36bb29)


# Localhost Prerequisites

    Github — choco/brew install git -y
    Maven — choco/brew install maven -y
    AWS CLI — choco/brew install awscli -y
    Vscode — choco/brew install vscode -y


# Architecture of AWS services for the project

1. 4 EC2 Instances, for tomcat(app01), rabbitmq(rmq01), mysql(db01), memcache(mc01).
2. ELB[load balancer] replacement for nginx LB
3. Autoscaling
4. S3 to store software artifact
5. EFS Storage
6. Route 53 for DNS zones
7. Amazon Certificate Manager (ACM) for https
8. IAM
9. EBS
10. 3 Security groups

    ![infra 2](https://github.com/user-attachments/assets/f3ccbb31-be51-4d67-95de-6209e5d22420)


# Architecture explanation on AWS

 - Users will access our website by using the URL and that URL will be pointed to an endpoint which its entry will be mentioned in godaddy DNS.
 - Users of the app will access it throuhg a secured endpoint url(https://) and each request will be recieved by a load balancer which only accepts https traffic.
 - The secured certficate ssl will be gotten from ACM.
 - The ALB will route the requst to tomcat instances(app).
 - The Tomcat will the running on ec2 instances which will be managed by auto scaling group. Depending on the traffic load from users, the ec2 instances will be scaled    out or scaled in.
 - The Tomcat ec2 servers will be in a separete security group which will only allow traffic on port 8080 from only the application load balancer.
 - The application needs backend servers, which are mysql,memchace and rabbitmq.
 - Information of the backend server or the backend server IP will be placed in a route 53 private dns zone.
 - So Tomcat instance will access backend servers with the name which is gotten from route 53 private dns with the private ip addr will be gotten as well.
 - The backend ec2 instances server will (rmq,mec,mysql) will all be in placed in another sercurity group.

# Flow of Execution

    - Login to AWS Account.
    - Create Security groups.
    - Create key pairs.
    - Lunch Instances with user data [BASH SCRIPTS].
    - Update backend servers IP addr to name mapping in route 53.
    - Build Application from source code.
    - Upload to S3 bucket.
    - Download artifact to Tomcat EC2 instance.
    - Setup ELB with HTTPS [Cert from Amazon Certicate Manager].
    - Map ELB Endpoint to website name in Godaddy DNS.
    - Verify!
    - Build Autoscaling group for tomact instances.  

# Setup

  1. Login to AWS Account
  2. Create Security groups :

  - First we create a security group for our load balancer: Go to EC2, security groups , click on create SG, give a name vprofile-ELB-SG , give description Security group for vprofile prod Load Balancer
    Give inbound rules of

                        http , port: 80 , allow : anywhere
                        https , port : 443 , allow : anywhere

  - Second create a security group for tomcat server: name vprofile-app-SG , give description Security group for tomcat instances
    Give inbound rules of

                  Custom TCP, port: 8080 , allow : only from SG of load balancer , description : allow traffic from ELB.

                  Custom TCP, port: 22 , allow : only from my ipp addr , description : allows ssh from me .Custom TCP, port: 8080 , allow : only from my ipp addr ,                      description : allows ssh from me .

   - Third ,create a security group for the backend services(rmq,mc,mysql): name vprofile-db-sg , give description Security group for vprofile backend serivces
     Give inbound rules of

                  Custom TCP, port: 22 , allow : only from my ipp addr , description : allows ssh from my ip .

                  MYSQL/Aurora, port: 3304 , allow : only from SG of app , description : allow 3306 traffic from application server.Custom TCP, port: 11211 , allow :                    only from SG of app , description : allow tomcat to connect to mmigemcache.Custom TCP, port: 5672 , allow : only from SG of app , description : allows                    tomcat to connect to rabbitmq.All taffic, port: ALL , allow : only from SG of app , description : allow internal traffic to flow on all ports.*here                    all backend servers are to interact with each other so we open port `All`*

      note: we need to update our ip regularly if our public ip is changing regularly

      save it

 3. Create key pairs : go to key pair, create keypair and name to vprofile-prod-key and download it to our system

 4. Lunch Instances with user data [BASH SCRIPTS]

    go to this (github-link)[https://github.com/Rietta1/migeration.git] and clone it locally, run git checkout aws-LiftAndshift to change branch...
    go to the folder userdata and we will see the scripts to implement each service on the ec2 instance
    lunch 4 ec2 instance , add the bash scripts to the lunch configuration and lunch

tomcat_ubuntu.sh for app01 server. (ubuntu 22)

     #!/bin/bash
     sudo apt update
     sudo apt upgrade -y
     sudo apt install openjdk-11-jdk -y
     sudo apt install tomcat9 tomcat9-admin tomcat9-docs tomcat9-common git -y

   mysql.sh for db01 (almalinux 9)

     #!/bin/bash
     DATABASE_PASS='admin123'
     sudo yum update -y
     sudo yum install epel-release -y
     sudo yum install git zip unzip -y
     sudo yum install mariadb-server -y

    # starting & enabling mariadb-server
    sudo systemctl start mariadb
    sudo systemctl enable mariadb
    cd /tmp/
    git clone -b main https://github.com/Rietta1/migeration.git
    #restore the dump file for the application
    sudo mysqladmin -u root password "$DATABASE_PASS"
    sudo mysql -u root -p"$DATABASE_PASS" -e "UPDATE mysql.user SET Password=PASSWORD('$DATABASE_PASS') WHERE User='root'"
    sudo mysql -u root -p"$DATABASE_PASS" -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1')"
    sudo mysql -u root -p"$DATABASE_PASS" -e "DELETE FROM mysql.user WHERE User=''"
    sudo mysql -u root -p"$DATABASE_PASS" -e "DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%'"
    sudo mysql -u root -p"$DATABASE_PASS" -e "FLUSH PRIVILEGES"
    sudo mysql -u root -p"$DATABASE_PASS" -e "create database accounts"
    sudo mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'localhost' identified by 'admin123'"
    sudo mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123'"
    sudo mysql -u root -p"$DATABASE_PASS" accounts < /tmp/vprofile-project/src/main/resources/db_backup.sql
    sudo mysql -u root -p"$DATABASE_PASS" -e "FLUSH PRIVILEGES"# Restart mariadb-server
    sudo systemctl restart mariadb
    
    #starting the firewall and allowing the mariadb to access from port no. 3306
    sudo systemctl start firewalld
    sudo systemctl enable firewalld
    sudo firewall-cmd --get-active-zones
    sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
    sudo firewall-cmd --reload
    sudo systemctl restart mariadb

memcache.sh for mc01 (almalinux 9)

    #!/bin/bash
    sudo dnf install epel-release -y
    sudo dnf install memcached -y
    sudo systemctl start memcached
    sudo systemctl enable memcached
    sudo systemctl status memcached
    sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
    sudo systemctl restart memcached
    firewall-cmd --add-port=11211/tcp
    firewall-cmd --runtime-to-permanent
    firewall-cmd --add-port=11111/udp
    firewall-cmd --runtime-to-permanent
    sudo memcached -p 11211 -U 11111 -u memcached -d

rabbitmq.sh for rmq01 (almalinux 9)

    #!/bin/bash
    sudo yum install epel-release -y
    sudo yum update -y
    sudo yum install wget -y
    cd /tmp/
    dnf -y install centos-release-rabbitmq-38
    dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
    systemctl enable --now rabbitmq-server
    firewall-cmd --add-port=5672/tcp
    firewall-cmd --runtime-to-permanent
    sudo systemctl start rabbitmq-server
    sudo systemctl enable rabbitmq-server
    sudo systemctl status rabbitmq-server
    sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
    sudo rabbitmqctl add_user test test
    sudo rabbitmqctl set_user_tags test administrator
    sudo systemctl restart rabbitmq-server

   - verify what we have installed curl http://169.254.169.254/latest/user-data to get the details of the scripts installed
   - Check our database installation

    sudo -i
    systemctl status mariadb
    mysql -u admin -padmin123 accounts
    show tables;

   - check rabbitmq and tomcat server

         ss -tunlp | grep 11211
         systemctl status rabbitmq-server
         systemctl status tomcat9
         ls /var/lib/tomcat9

3. Update IP to name mapping in route 53

   - go to route53 on aws, then create a private hosted zone named vprofile-project.in , add the vpc of the same region (us-east-1) where our servers are placed , click on create      hosted zone
   - create record, click simple routing, subdomain : db01 [vprofile-project.in] , A- records, place our db01 private ip address.
   - Do same for mc01 and rmq01
   - add vprofile-project.in to src/main/resources/application.properties

Go to rc/main/resources/application.properties and add the .vprofile-project.in to the backend names   
![infra 3](https://github.com/user-attachments/assets/fa46c0e3-4b59-4d50-a118-ec090c5058fa)


# Build Artifact

1. Build Application from source code

- We would build the artifact locally on our vscode, push it to s3 bucket and then deploy it to our tomcat server
- go tho the folder with pom.xml and build our artifact with maven
- run mvn install to build the artifact
- A target file is created with a vprofile.v2.war inside.

2. Push the artifacts into s3 bucket

- Go to IAM on the console and the USERS,=> click on ADD USERS,=> create a user called s3admin,=> attach policy directly amazons3fullaccess and create user
- Go to USERS,=> SECURITY,=> CREATE ACCESS KEY,=> use CLI interface, create access key and download it.
- Configure our aws cli in our vscode aws configure add the keys details.
- list buckets aws s3 ls
- now create a bucket from the cli aws s3 mb s3://vpro-arts remember bucket names are like domain names, one name can be owned by one person world wide
- copy the artifacts to the bucket aws s3 cp target/vprofile-v2.war s3://vpro-arts/

 We could check our s3 bucket and see the artifacts there was the upload is completed

- Now download the artifact into our tomcat ec2 instance

 We can use IAM access keys configuration in the instance, but there is a better way

 - we create an IAM ROLE, just like we created a user , and attach this role to our instance
 - go to IAM,create IAM ROLE,=> AWS service, => EC2 , search for the policy amazons3fullaccess => give role a name vprof-s3, then create role.
 - go to the instance , ACTION => SECURITY => MODIFY IAM ROLE, select the role created vprof-s3, => UPDATE IAM ROLE
 - ssh into the tomcat server ,

       sudo -i
       apt update 
       install awscli -y

       aws s3 ls#copy vprofile-v2.war from the s3 bucket to the tmp folder in the instance
       aws s3 cp s3://vpr-arts/vprofile-v2.war /tmp/systemctl stop tomcat9
       rm -rf /var/lib/tomcat9/webapps/ROOT
       cp /tmp/vprofile-v2.war /var/lib/tomcat9/webbapps/ROOT.war
       systemctl start tomcat9
       ls /var/lib/tomcat9/webapps
       cat /var/lib/tomcat9/webapps/ROOT/WEB-INF/classes/application.properties

# Configure Load-Balancer and DNS

 1. DNS / Amazon Certificate Manager (ACM)

 - Implementation of the SSL and configuration of Domain with Route53 Create a hosted zone and copy the namesavers details to where your domain name ns
 - Go to AWS Certificate Manager and create an ssl certificate domain name = rietta.online click on add another name to this certificate = *.rietta.online

 2. Load Balancer

We are going to create an Application load balancer, so we create a target group first.

  - go to LOAD BALANCER, => TARGET GROUP, => NAME: vprofile-app-TG, => PROTOCOL; HTTP:8080 , => HEALTH CHECK: /login , => click OVERRIDE : 8080, => HEALTHY TRESHOLD:3, => select the APP01 instance, => click INCLUDE AS PENDING BELOW, => create target group.
  - go to LOAD BALANCER and create, select APPLICATION LOAD BALANCER, => create, => NAME: vprofile-prod-ELB , => INTERNET FACING, => IPV4, => select ALL THE ZONES, SECURITY GROUP: vprofile-ELB-sg , => LISTENER; HTTPS:443:vprofile-app-TG, select the ACM CERT:rietta.online, => add another LISTENER; HTTP:80:vprofile-app-TG, then create load balancer

3. DNS

- copy the endpoint DNS NAME and go to ROUTE53 , create a record with record type as CNAME, => SUBDOMAIN:migapp(.rietta.online) and add the DNS NAME copied
- ping migapp.rietta.online, curl migapp.rietta.online, in the terminal to check
- go to the url and login, username:admin-vp , password: admin-vp

# Auto Scaling

   1 Create an AMI

  - go to the ec2 instance of the TOMCAT SERVER(app01), ACTION => IMAGE => CREATE IMAGE. give it a name mig-app-image
  - Create lunch template, go to AUTO SCALING GROUP, => NAME: mig-app-LC, select the AMI created, => INSTANCE TYPE: t2.micro, => IAM ROLE: use the one created, enable monitoring, use mig-app-SG, add key and create the lunch template
  - Create auto scaling group , select the launch template created , => select all the subnets, => enable load balancer and its health check, => capacit, 1 ,1, 4, => Target tracking scaling policy, 50 => enable instance scale-in protection, => add notifications , => tags:mig-app, project :mig, owner:loretta, => create
  - terminate previous tomcat server(app01)
   - verify again by login in , username:adim-vp , password:admin-vp
