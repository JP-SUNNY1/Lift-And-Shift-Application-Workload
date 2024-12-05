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
