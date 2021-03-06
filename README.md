# Project-15
# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

WARNING: This infrastructure set up is NOT covered by AWS free tier. Therefore, ensure to DELETE ALL the resources created immediately after finishing the project. Monthly cost may be shockingly high if resources are not deleted. Also, it is strongly recommended to set up a budget and configure notifications when your spendings reach a predefined limit. Watch this video to learn how to configure AWS budget.

Let us Get Started
You have been doing great work so far – implementing different Web Solutions and getting hands on experience with many great DevOps tools. In previous projects you used basic Infrastructure as a Service (IaaS) offerings from AWS such as EC2 (Elastic Compute Cloud) as rented Virtual Machines and EBS (Elastic Block Store), you have also learned how to configure Key pairs and basic Security Groups.

But the power of Clouds is not only in being able to rent Virtual Machines – it is much more than that. From now on, you will start gradually study different Cloud concepts and tools on example of AWS, but do not be worried, your knowledge will not be limited to only AWS specific concepts – overall principles are common across most of the major Cloud Providers (e.g., Microsoft Azure and Google Cloud Platform).

NOTE: The next few projects will be implemented manually. Before you begin to automate infrastructure in the cloud, it is very important that you can build the solution manually. Otherwise, programming your automation may become frustrating very quickly.

You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/<your-name>/tooling) for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.


![alt text](./images/a.PNG)
![alt text](./images/a2.PNG)



## Starting Off Your AWS Cloud Project
There are few requirements that must be met before you begin:

1. Properly configure your AWS account and Organization Unit Watch How To Do This Here

-Create an AWS Master account. (Also known as Root Account)
Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
Move the DevOps account into the Dev OU.
Login to the newly created AWS account using the new email address

2. Create a free domain name for your fictitious company at Freenom domain registrar here.

3. Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that here

NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

-Project: <Give your project a name>
-Environment: <dev>
-Automated: <No> (If you create a recource using an automation tool, it would be <Yes>)

![alt text](./images/ou.PNG)

![alt text](./images/ou1.PNG)



## SET UP A VIRTUAL PRIVATE NETWORK (VPC)
Set Up a Virtual Private Network (VPC)
Always make reference to the architectural diagram and ensure that your configuration is aligned with it.

1. Create a VPC
2. Create subnets as shown in the architecture
3. Create a route table and associate it with public subnets
4. Create a route table and associate it with private subnets
5. Create an Internet Gateway
6. Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)
7. Create 3 Elastic IPs
8. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)
9. Create a Security Group for:

-Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
-Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
-Application Load Balancer: ALB will be available from the Internet
-Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
-Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

![alt text](./images/vpc.PNG)

![alt text](./images/vpc1.PNG)

![alt text](./images/vpc2.PNG)

![alt text](./images/vpc3.PNG)

![alt text](./images/vpc4)

![alt text](./images/vpc5.PNG)

![alt text](./images/vpc6.PNG)

![alt text](./images/vpc7.PNG)

![alt text](./images/vpc8.PNG)

![alt text](./images/vpc9.PNG)

![alt text](./images/vpc10.PNG)

![alt text](./images/vpc11.PNG)


## Proceed With Compute Resources
You will need to set up and configure compute resources inside your VPC. The recources related to compute are:

-EC2 Instances
-Launch Templates
-Target Groups
-Autoscaling Groups
-TLS Certificates
-Application Load Balancers (ALB)


## Set Up Compute Resources for Nginx
Provision EC2 Instances for Nginx
1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)

2. Ensure that it has the following software installed:
python
ntp
net-tools
vim
wget
telnet
epel-release
htop

3.Create an AMI out of the EC2 instance

-Prepare Launch Template For Nginx (One Per Subnet)
-Make use of the AMI to set up a launch template
-Ensure the Instances are launched into a public subnet
-Assign appropriate security group
-Configure Userdata to update yum package repository and install nginx

Configure Target Groups
-Select Instances as the target type
-Ensure the protocol HTTPS on secure TLS port 443
-Ensure that the health check path is /healthstatus
-Register Nginx Instances as targets
-Ensure that health check passes for the target group

Configure Autoscaling For Nginx
-Select the right launch template
-Select the VPC
-Select both public subnets
-Enable Application Load Balancer for the AutoScalingGroup (ASG)
-Select the target group you created before
-Ensure that you have health checks for both EC2 and ALB
-The desired capacity is 2
-Minimum capacity is 2
-Maximum capacity is 4
-Set scale out if CPU utilization reaches 90%
-Ensure there is an SNS topic to send scaling notifications


Set Up Compute Resources for Bastion
Provision the EC2 Instances for Bastion
1. Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
2. Ensure that it has the following software installed
python
ntp
net-tools
vim
wget
telnet
epel-release
htop

3. Associate an Elastic IP with each of the Bastion EC2 Instances
4. Create an AMI out of the EC2 instance

Prepare Launch Template For Bastion (One per subnet)
-Make use of the AMI to set up a launch template
-Ensure the Instances are launched into a public subnet
-Assign appropriate security group
-Configure Userdata to update yum package repository and install Ansible and git

Configure Target Groups
-Select Instances as the target type
-Ensure the protocol is TCP on port 22
-Register Bastion Instances as targets
-Ensure that health check passes for the target group

Configure Autoscaling For Bastion
-Select the right launch template
-Select the VPC
-Select both public subnets
-Enable Application Load Balancer for the AutoScalingGroup (ASG)
-Select the target group you created before
-Ensure that you have health checks for both EC2 and ALB
-The desired capacity is 2
-Minimum capacity is 2
-Maximum capacity is 4
-Set scale out if CPU utilization reaches 90%
-Ensure there is an SNS topic to send scaling notifications

## Set Up Compute Resources for Webservers
Provision the EC2 Instances for Webservers
Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites
-Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

-Ensure that it has the following software installed
python
ntp
net-tools
vim
wget
telnet
epel-release
htop
php

3. Create an AMI out of the EC2 instance

Prepare Launch Template For Webservers (One per subnet)
-Make use of the AMI to set up a launch template
-Ensure the Instances are launched into a public subnet
-Assign appropriate security group
-Configure Userdata to update yum package repository and install wordpress 
(Only required on the WordPress launch template)



![alt text](./images/webserver.PNG)

![alt text](./images/webserver1.PNG)

![alt text](./images/webserver2.PNG)

![alt text](./images/webserver3.PNG)


![alt text](./images/nginx.PNG)


![alt text](./images/nginx1.PNG)

![alt text](./images/nginx2.PNG)

![alt text](./images/nginx3.PNG)


![alt text](./images/bastion.PNG)

![alt text](./images/ami.PNG)






## TLS Certificates From Amazon Certificate Manager (ACM)
You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).
-Navigate to AWS ACM
-Request a public wildcard certificate for the domain name you registered in Freenom
-Use DNS to validate the domain name
-Tag the resource


## CONFIGURE APPLICATION LOAD BALANCER (ALB)
Application Load Balancer To Route Traffic To NGINX
Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

-Create an Internet facing ALB
-Ensure that it listens on HTTPS protocol (TCP port 443)
-Ensure the ALB is created within the appropriate VPC | AZ | Subnets
-Choose the Certificate from ACM
-Select Security Group
-Select Nginx Instances as the target group

Application Load Balancer To Route Traffic To Web Servers
Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

-Create an Internal ALB
-Ensure that it listens on HTTPS protocol (TCP port 443)
-Ensure the ALB is created within the appropriate VPC | AZ | Subnets
-Choose the Certificate from ACM
-Select Security Group
-Select webserver Instances as the target group
-Ensure that health check passes for the target group

NOTE: This process must be repeated for both WordPress and Tooling websites.

## Setup EFS
Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

-Create an EFS filesystem
-Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
-Associate the Security groups created earlier for data layer.
-Create an EFS access point. (Give it a name and leave all other settings as default)


## Setup RDS
Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless

To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution – this is a more advanced concept that will be discussed in following projects.

To configure RDS, follow steps below:

-Create a subnet group and add 2 private subnets (data Layer)
-Create an RDS Instance for mysql 8.*.*
-To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
-Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
-Configure VPC and security (ensure the database is not available from the Internet)
-Configure backups and retention
-Encrypt the database using the KMS key created earlier
-Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)


# Note This service is an expensinve one. Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)


## Configuring DNS with Route53
Earlier in this project you registered a free domain with Freenom and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.

You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

Create other records such as CNAME, alias and A records.

NOTE: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read here to get to know more about the differences.

Create an alias record for the root domain and direct its traffic to the ALB DNS name.
Create an alias record for tooling.<yourdomain>.com and direct its traffic to the ALB DNS name.

![alt text](./images/targetgroup.PNG)

![alt text](./images/targetgroup1.PNG)

![alt text](./images/targetgroup2.PNG)

![alt text](./images/targetgroup3.PNG)

![alt text](./images/targetgroup4.PNG)

![alt text](./images/targetgroup5.PNG)


![alt text](./images/targetgroup6.PNG)

![alt text](./images/loadbalancer.PNG)

![alt text](./images/loadbalancer1.PNG)


![alt text](./images/loadbalancer2.PNG)

![alt text](./images/launchtemplate.PNG)

![alt text](./images/launchtemplate1.PNG)


![alt text](./images/efs.PNG)

![alt text](./images/efs1.PNG)


![alt text](./images/rds.PNG)


![alt text](./images/sql.PNG)


![alt text](./images/sql1.PNG)

![alt text](./images/dns.PNG)

![alt text](./images/autoscalling.PNG)

![alt text](./images/server.PNG)

![alt text](./images/server1.PNG)

![alt text](./images/wordpress.PNG)


![alt text](./images/wordpress1.PNG)

![alt text](./images/wordpress3.PNG)


![alt text](./images/lastpix.PNG)

![alt text](./images/lastpix2.PNG)


# Congratulations!


You have just created a secured, scalable and cost-effective infrastructure to host 2 enterprise websites using various Cloud services from AWS. At this point, your infrastructure is ready to host real websites’ load. Since it is a pretty expensive infrastructure to keep it up and running, we are going to start using Infrastructure as Code tool Terraform to easily provision and destroy this set up.

Move on for more amazing and challenging projects ahead!




