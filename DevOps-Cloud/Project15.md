AWS Cloud Solution For 2 Company Websites Using A Reverse Proxy Technology 
============================================

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company that uses WordPress CMS for its main business website, and a Tooling Website **(`https://github.com/QuadriBello/tooling`)** for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensures that infrastructure for both websites, WordPress and Tooling, is resilient to the Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a07fce80-4415-4e3c-961e-ff5e1af744cf)

### <br>Introduction to AWS VPC<br/>

An Amazon Virtual Private Cloud (VPC) is like your own private section of the Amazon cloud, where you can place and manage your resources (like servers or databases). You control who and what can go in and out, just like a gated community.
An AWS Cloud setup usually comes with a default VPC. The Default VPC is like a starter pack provided by Amazon for your cloud resources. It's a pre-configured space in the Amazon cloud where you can immediately start deploying your applications or services. It has built-in security and network settings to help you get up and running quickly, but you can adjust these as you see fit.

![VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0d8d5f0b-d623-4461-a24c-b5bfb7c32366)

A Default VPC, which Amazon provides for you in each region (think of a region as a separate city), is like a pre-built house in that city. This house comes with some default settings to help you move in and start living (or start deploying your applications) immediately. But just like a real house, you can change these settings according to your needs. A VPC can also have smaller sections called subnets.

Subnets are like smaller segments within a VPC that help you organize and manage your resources. Subnets are like dividing an office building into smaller sections, where each section represents a department. In this analogy, subnets are created to organize and manage the network effectively.

![subnet](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e95a3a4b-9b4d-4ce0-b264-80acd061d715)

### Project Prerequisites

There are few requirements that must be met before we begin our project implementation:

**1.** We properly configure our AWS account and Organization Unit [by following the steps here](https://youtu.be/9PQYCc_20-Q)

* Create an [AWS Master account](https://aws.amazon.com/free/). (*Also known as Root Account*)
* Within the Root account, create a sub-account and name it **DevOps**. (We will need another email address to complete this) 
* Within the Root account, create an **AWS Organization Unit (OU)**. Name it **Dev**. (We will launch Dev resources in there)
* Move the **DevOps** account into the ***Dev*** **OU**.
* Login to the newly created AWS account using the new email address.

![AWS organisations account](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f6845afd-f717-46aa-8932-38ffb8c0d153)

**2.** We create a free domain name for our fictitious company at Freenom domain registrar [here](https://www.freenom.com).

**3.** We create a hosted zone in AWS, and map it to our free domain from Freenom. [Watch how to do that here](https://youtu.be/IjcHp94Hq8A) 

***NOTE*** : As we proceed with configuration, we ensure that all resources are appropriately tagged, for example:

* Project: `<Give our project a name>`
* Environment: `<dev>`
* Automated: `<No>` (If we create a recource using an automation tool, it would be `<Yes>`)

#### <br>Setting Up Infrastructure<br/>

We need to always make reference to the architectural diagram and ensure that our configuration and infrastructure is aligned with it.

**1.** Create a [VPC:](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) 

**i.** We navigate to the VPC dashboard in AWS and the we click on **`Your VPCs > Create VPC`**. Next we enter the Name tag and the IPv4 CIDR and then we click on the "Create VPC" button.

![create VPC](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e909e16a-101f-4ac1-8696-ffe6e69fdea6)

![VPC created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c51ca9a2-41e3-401c-a718-cbf73666193b)

**ii.** As we can see in the image above, our VPC was successfully created. However, DNS hostnames is disabled. To enable this, we click on **`Actions > Edit VPC Settings`**, then we select the check box to Enable DNS hostnames and we click on Save.

![enable DNS hostnames](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/14831105-f414-47f8-b572-c7587a1f1423)

![dns hostname enabled](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/af236a80-35bd-4a2b-9cb2-db51779e2512)
 
 **2.** Create subnets as shown in the architecture:

 **i.** From the VPC dashboard we navigate to **`Subnets > Create subnet`**, then we select our VPC in the VPC dropdown box and as shown in the image below, we enter the details for subnet settings to create all our subnets.

![subnet settings](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/91a823d8-b5a2-4da9-9ffb-10dfccd9d4fc)

![subnets created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f1a5028d-39d3-44af-ba36-901591be7423)

**3.** Create a route table and associate it with public subnets:

 **i.** From the VPC dashboard we navigate to **`Route tables > Create route tables`**, then we enter the route table name and we select our VPC in the VPC dropdown box and click on "Create route table".

 ![create public route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/45287965-74c5-4b99-808a-bc6621a771a0)

 **ii.** Next we associate the route table with our public subnets by selecting the route table and navigating to **`Subnet associations > Edit subnet associations`**.and then we select our public subnets and we click on "Save associations".

![associate public subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/58472e11-c8bc-41a4-a117-5023204eb6f7)

**4.** Create a route table and associate it with private subnets:

 **i.** From the VPC dashboard we navigate to **`Route tables > Create route tables`**, then we enter the route table name and we select our VPC in the VPC dropdown box and click on "Create route table".

 ![create private route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b67f6412-9ad1-4358-b51a-b1faa1ff9ef8)

**ii.** Next we associate the route table with our private subnets by selecting the route table and navigating to **`Subnet associations > Edit subnet associations`**.and then we select our private subnets and we click on "Save associations".

![associate private subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c1e73208-f253-42c8-b09a-1b089649edcd)

**5.** Create an [Internet Gateway:](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

**i.** From the VPC dashboard we navigate to  **`Internet gateways > Create internet gateway`**, then we enter the name tag and click on "Create internet gateway".

  ![create internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1ad68088-d9a7-47b1-8cab-c575e41983a7)

**ii.** Next, we click on "Attach to a VPC" then in the resulting page, we select our VPC in the dropdown box and we click on "Attach internet gateway". 

![attach to a vpc](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bef044c6-ee0b-49d8-bf80-1759bfb932a1)

![attach to a vpc 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3d89a686-731b-4f64-9c4d-2031a71e110a)

**6.** Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet)

**i.** We select the public route table and we navigate to **`Actions > Edit routes`** then we click on "Add route" and then we edit and save changes as shown in the image below:

![edit route associate internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6769250d-a74c-4546-b855-c87f1e6899df)

**7.** Create 3 [Elastic IPs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

**i.** From the VPC dashboard we navigate to  **`Elastic IPs > Allocate Elastic IP address`**, then we give a name tag and we click on "Allocate".

![allocate elastic IP](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4b8f61f4-68ca-4f49-8cdc-06a4bd5b4c14)

**8.** Create a [Nat Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) and assign one of the Elastic IPs (*The other 2 will be used by [Bastion hosts](https://aws.amazon.com/quickstart/architecture/linux-bastion/))
  
**i.** From the VPC dashboard we navigate to  **`NAT gateways > Create NAT gateway`**, then we proceed to enter the NAT gateway settings as shown in the image below, and afterwards, we click on "Create NAT gateway".

 ![create NAT gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d8645cbf-cc0d-43e9-b715-2d1b4d72673a)

**ii.** Next we proceed to add the NAT gateway to our private route table.

![add NAT gateway to private route table](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/75478b8c-1100-4665-b895-844091a31425)

**9.** Create [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#CreatingSecurityGroups)

From the VPC dashboard we navigate to  **`Security groups > Create security group`**, then we proceed to create security groups in accordance with the following:

**i.** `Application Load Balancer`: Our external ALB will be available from the Internet.

![security group ALB nginx servers](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/03562318-36a3-4512-b804-c7821045c21b)

**ii.** `Bastion Servers`: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type `curl www.canhazip.com`

![security group for bastion](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/459a6079-26b1-4d3f-93c9-92bc1c800b92)

**iii.** `Nginx Servers`: Traffic access to Nginx reverse proxy servers should only be allowed from a [Application Load balancer (ALB)](https://aws.amazon.com/elasticloadbalancing/application-load-balancer/). However, we also allow SSH access from the bastion server.

![security group for nginx reverse proxy](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0177df9f-a9c5-4151-9c08-e6bbfa9a96fa)

**iv.** `Internal Application Load Balancer`: Our Internal ALB will only allow traffic from the Nginx reverse proxy servers.

![security group for internal application load balancer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8244f3b0-7507-44d0-b41f-9cdb73eb290d)

**v.** `Webservers`: Access to Webservers should only be allowed from the internal ALB servers. We also allow SSH access to the webservers from the bastion.

![security group for webservers](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/611d32b7-a401-4824-8a77-2c33eac6f951)

**vi.** `Data Layer`: Access to the Data layer, which is comprised of [Amazon Relational Database Service (RDS)](https://aws.amazon.com/rds/) and [Amazon Elastic File System (EFS)](https://aws.amazon.com/efs/) must be carefully desinged - only bastion and  webservers should be able to connect to **RDS**, while Nginx and Webservers will have access to **EFS Mountpoint**.

![security group for data layer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/189160b7-dc96-400a-9104-ebdbb39f4d58)

#### <br>Setup Amazon EFS<br/>

[Amazon Elastic File System (Amazon EFS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEFS.html) provides a simple, scalable, fully managed elastic [Network File System (NFS)](https://en.wikipedia.org/wiki/Network_File_System) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

**i.** Create an EFS filesystem: From the EFS dashboard we click on "Create file system". Then we enter the filesystem name and select the VPC, then we click on "customize".

![create file system](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ce4a13c2-504d-4544-8dae-db8441ff4c3b)

**ii.** In the following page, we add a Name tag and we click on the "Next" button.

![name tag](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ab3805db-2b2e-41b6-b5b0-114b27fd8aac)

**iii.** In the Network Access page, we need to specify our EFS Mount targets per Availability Zone in the VPC and associate it with our private subnets 1 and 2 according to our project diagram.

![network access](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a64a8734-7c00-43a4-8aea-c9300564b8d9)

**iv.** Associate the Security groups created earlier for data layer.

**v.** And in the next page, we review our configuration and we click on "Create".

![review and create](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/cfdf9ea4-45ff-41a0-b300-21e407e0e8e5)

**vi.** Next we create two access points for our Wordpress and Tooling web servers respectively. We navigate to **`ACS-filesystem > Access points > Create access point`** and then we configure the access points as shown in the images below:

![wordpress access point](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dba81c61-9de6-4773-9ebc-c041a7596bf2)

![tooling access point](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/03c62eaa-9bbe-4952-a647-5068f7fd8e67)

#### <br>Setup Amazon RDS<br/> 

***Pre-requisite***: Create a KMS key from [Key Management Service (KMS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) to be used to encrypt the database instance.

* From the AWS Key Management Service dashboard, we click on "Create a Key" and then we configure key as shown in the images below:

![configure KMS key](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a6456c21-f31f-49b3-8dfc-c0b194b02aa0)

![configure KMS key2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/20482dad-12a6-402a-ac0c-ed9880c74ae5)

![configure KMS key3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7a5d528a-fd50-4763-938b-ce7494da26a3)

[Amazon Relational Database Service (Amazon RDS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. *Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless*

To ensure that our databases are highly available and also have failover support in case one availability zone fails, we will configure a [multi-AZ](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html) set up of [RDS MySQL database](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html) instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution - this is a more advanced concept that will be discussed in following projects.

To configure RDS, we follow steps below:

**i.** Create a subnet group and add 2 private subnets (data Layer). From the Amazon RDS dashboard, we navigate to **`Subnet groups > Create DB subnet group`** and we configure as shown in the image below:

![create subnet group](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/315396ca-0f60-42f6-be9b-f85e0823377e)

**ii.** Create an RDS Instance for `mysql 8.*.*`. We navigate to **`Dashboard > Create database`**, then we configure and create our database as shown in the images below:

![RDS config 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/77769ba8-e532-4824-ad6e-2e7e28d52fb6)

**iii.** To satisfy our architectural diagram, we will need to select either **Dev/Test** or **Production** Sample Template. But to minimize AWS cost, we can select the free tier sample template then we configure other settings accordingly (*For test purposes, most of the default settings are good to go*). In the real world, we will need to size the database appropriately. We will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.

![RDS config 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/eb6b0317-7eb4-46c2-bdad-41a0bafc425f)

**iv.** Next, we configure VPC and security (ensure the database is not available from the Internet), we configure backups and retention and we encrypt the database using the KMS key created earlier.

![RDS config 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bcac1b17-6f14-4b54-ab89-d894004e130c)
 
**v.** Enable [CloudWatch](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/monitoring-cloudwatch.html) monitoring and export `Error` and `Slow Query` logs (for production, also include `Audit`)

#### <br>Creating Compute Resources<br/> 

Next, we need to set up and configure compute resources inside our VPC. The resources related to compute are:

**1.** [TLS Certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security)
   
**2.** [EC2 Instances](https://www.amazonaws.cn/en/ec2/instance-types/)
 
**3.** [Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html)
 
**4.** [Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

**5.** [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html)

**6.** [Application Load Balancers (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)

#### <br>TLS Certificates From [Amazon Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/)<br/>

We will need TLS certificates to handle secured connectivity to our Application Load Balancers (ALB). To obtain the certificates, we proceed with the following steps:

**i.** Navigate to AWS Certificate Manager

**ii.** Request a public wildcard certificate for the domain name you registered.

**iii.** Use DNS to validate the domain name

![AWS cert](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7699f683-c653-4829-97ac-01f8f64efbfd)

**iv.** Tag the resource and Click on Request.

**v.** Create DNS CNAME records in Amazon Route 53. 

![DNS CNAME record route53](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5957bed9-8aea-404d-95a4-e65e34c45bc8)

#### <br>Set Up Compute Resources for Nginx<br/>

We proceed to setup all compute resources for Nginx.

##### <br>Provision EC2 Instances for Nginx<br/>

**i.** We create an EC2 Instance based on CentOS [Amazon Machine Image (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) in any 2 [Availability Zones (AZ) in any AWS Region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) ([it is recommended to use the Region that is closest to our customers](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/RegionsAndAZs.html)). We use EC2 instance of [T2 family](https://aws.amazon.com/ec2/instance-types/t2/) (e.g. t2.micro or similar)

![launch ec2 instances](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2d0f17d8-3f95-4244-966a-496894c95309)
   
**ii.** Then we ssh into the instance and we ensure that it has the following software installed:
        
    * `python`
    * `ntp`
    * `net-tools`
    * `vim`
    * `wget`
    * `telnet`
    * `epel-release`
    * `htop`

```
$ yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

$ yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ yum install wget vim python3 telnet htop git mysql net-tools chrony -y

$ systemctl start chronyd

$ systemctl enable chronyd
```

![nginx ami installation 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/80a9e70d-e9aa-4a4d-8dc1-ca14e47a8255)

![nginx ami installation 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2ae5d5cd-1663-4079-89bc-98bd2eb54e82)

![nginx ami installation 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/23983287-e5f4-48a8-820b-785b6e431fe5)

![nginx ami installation 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/99b1bfae-5488-4aae-9dab-081dc14a52ff)

**iii.** Next, we configure selinux policies for the nginx server using the following commands:

```
$ setsebool -P httpd_can_network_connect=1

$ setsebool -P httpd_can_network_connect_db=1

$ setsebool -P httpd_execmem=1

$ setsebool -P httpd_use_nfs 1
```

![nginx ami installation 5](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/16a10e31-614a-41e8-8f80-01d0368c7064)

**iv.** Then, we use the following set of commands to install amazon efs utils for mounting the target on the Elastic file system.

```
$ git clone https://github.com/aws/efs-utils

$ cd efs-utils

$ yum install -y make

$ yum install -y rpm-build

$ make rpm 

$ yum install -y  ./build/amazon-efs-utils*rpm
```

![nginx ami installation 6](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a24b3a5a-162d-4402-8712-b3da294ad949)

![nginx ami installation 7](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e4fbb22a-125b-48d0-8a04-d12f4361dcaa)

![nginx ami installation 8](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9f928666-32f8-4582-9ada-fee396cb1d12)

![nginx ami installation 9](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dd586dcf-b0da-4f3d-a6ae-426219d2676f)


**v.** We proceed with setting up a self-signed certificate for the nginx instance.

```
$ sudo mkdir /etc/ssl/private

$ sudo chmod 700 /etc/ssl/private

$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt

$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

![nginx ami install certificate generation](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/27eda37b-d8d9-4c4b-a35a-084f9165ba5b)

![nginx ami install certificate generation 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3f2829ff-e54d-4c16-9bfe-3e1f78e05538)


**vi.** We [create an `AMI`](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html) out of the EC2 instance.

* Select Nginx instance, then navigate to **`Actions > Image and templates > Create image`** and then we configure and create the AMI as shown in the image below:

![create nginx ami](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/39d6c09b-a8fd-4c3b-8225-ec8b08ed2b5b)

###### Prepare Launch Template For Nginx (One Per Subnet)

**i.** Make use of the AMI to set up a launch template.

![create nginx launch template 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/73367a1a-5833-487d-8d0b-b53602731f59)

**ii.** Ensure the Instances are launched into a public subnet.

![create nginx launch template 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4679f508-1db7-4c58-b03a-9b05cdd16b4b)

**iii.** Assign appropriate security group.

![create nginx launch template 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/44e96f30-978f-4f3d-8c9b-cf9dc046f71d)

**iv.** Configure [Userdata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) to update `yum` package repository and install `nginx`.

![create nginx launch template 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/840c7142-0f68-440d-b323-1e38f847d0ac)

###### Configure Target Groups

* From the EC2 dashboard, we navigate to **`Target groups > Create target group`** and then we do the following:

**i.** Select Instances as the target type.

**ii.** Ensure the protocol `HTTPS` on secure TLS port `443`.

![create nginx target group](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2f94b2b5-ba9f-4e7f-a9d3-2ca1b12fd0cf)

**iii.** Ensure that the health check path is `/healthstatus`.

**iv.** Register `Nginx` Instances as targets.

**v.** Ensure that health check passes for the target group.

![target group healthy 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a38622c2-4c1f-499d-92c8-11bf8d5394f4)

###### Configure Autoscaling For Nginx 

* From the EC2 dashboard, we navigate to **`Auto Scaling Groups > Create Auto Scaling group`** and we do the following:

**i.** Select the right launch template.

**ii.** Select the VPC.

**iii.** Select both public subnets.

![ACS nginx autoscaling 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0feb9b39-6c63-41e0-822f-c7af60db9f68)

**iv.** Enable Application Load Balancer for the AutoScalingGroup (ASG).

**v.** Select the target group you created before.

**vi.** Ensure that you have health checks for both `EC2` and `ALB`.

**vii.** The desired capacity is 2.

**viii.** Minimum capacity is 2.

**ix.** Maximum capacity is 4.

![ACS nginx autoscaling 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6b142fcc-647a-4ce3-b699-667200979ee0)

**x.** Set [scale out](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroupLifecycle.html) if CPU utilization reaches 90%.

**xi.** Ensure there is an [`SNS`](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) topic to send scaling notifications.

![ACS nginx autoscaling 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/180c2525-3bfa-46b3-8c08-cbde7de61155)

#### <br>Set Up Compute Resources for Bastion<br/>

We proceed to set up all compute resources for Bastion.

##### <br>Provision the EC2 Instances for Bastion<br/>

**i.** We create an EC2 Instance based on RHEL Amazon Machine Image (AMI) per each Availability Zone in the same Region and the same AZ where we created Nginx server.

![launch ec2 instances](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2d0f17d8-3f95-4244-966a-496894c95309)
   
**ii.** Then we ssh into the instance and we ensure that it has the following software installed:

    * `python`
    * `ntp`
    * `net-tools`
    * `vim`
    * `wget`
    * `telnet`
    * `epel-release`
    * `htop`

```
$ yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

$ yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ yum install wget vim python3 telnet htop git mysql net-tools chrony -y

$ systemctl start chronyd

$ systemctl enable chronyd
```

![bastion ami installation 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0e4800d3-82ec-48ae-9e9f-725c833e1016)

![bastion ami installation 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/64b24662-5094-497d-a6e7-e563fc1344d4)

![bastion ami installation 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/50de294a-83e9-4737-989f-63d418e73de8)

![bastion ami installation 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dcea8c8a-5642-486f-b362-5c3c68136ac3)
    
**iii.** [Associate an Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html#using-instance-addressing-eips-associating) with each of the Bastion EC2 Instances
   
**iv.** Create an `AMI` out of the EC2 instance

* We select the Bastion instance, then navigate to **`Actions > Image and templates > Create image`** and then we configure and create the AMI as shown in the image below:

![create bastion ami](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f8a9205d-f44a-4904-a7b8-e94954f3e112)

###### Prepare Launch Template For Bastion (One per subnet)

* From the EC2 dashboard, we navigate to **`Launch Templates > Create launch template`** and then we do the following:

**i.** Make use of the AMI to set up a launch template.

![create bastion launch template 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b89161b4-e6d4-41c1-8d66-7fc402b87b5a)

**ii.** Ensure the Instances are launched into a public subnet.

![create bastion launch template 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/774463ea-d66f-4176-af71-f9ee5f00926d)

**iii.** Assign appropriate security group.

![create bastion launch template 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/24223bae-cd9a-4d3e-8c89-68e9c181ff3d)

**iv.** Configure Userdata to update `yum` package repository and install `Ansible` and `git`.

![create bastion launch template 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ab504009-75f1-4e3f-b637-8f7a2f6e0555)

###### Configure Autoscaling For Bastion 

* From the EC2 dashboard, we navigate to **`Auto Scaling Groups > Create Auto Scaling group`**
  
**i.** Select the right launch template.

**ii.** Select the VPC.

**iii**. Select both public subnets.

![ACS bastion autoscaling 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/385912a4-bc1d-47dd-82ba-c541b2d33106)

**iv.** Enable Application Load Balancer for the AutoScalingGroup (ASG).

**v.** Select the target group you created before.

**vi.** Ensure that you have health checks for both `EC2` and `ALB`.

**vii.** The desired capacity is 2.

**viii.** Minimum capacity is 2.

**ix.** Maximum capacity is 4.

**x.** Set scale out if CPU utilization reaches 90%.

**xi.** Ensure there is an `SNS` topic to send scaling notifications.

![ACS bastion autoscaling 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e0546b2f-c3db-493f-bd5f-53ad41a292dc)

#### <br>Set Up Compute Resources for Webservers<br/>

We proceed to set up all the compute resources for our Web servers.

##### <br>Provision the EC2 Instances for Webservers<br/> 

Now, we will need to create 2 separate launch templates for both the WordPress and Tooling websites

**i.** Create an EC2 Instance (`RHEL`) each for WordPress and Tooling websites per Availability Zone (in the same Region).

**ii.** Ensure that it has the following software installed

    * `python`
    * `ntp`
    * `net-tools`
    * `vim`
    * `wget`
    * `telnet`
    * `epel-release`
    * `htop`
    * `php`

```
$ yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

$ yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ yum install wget vim python3 telnet htop git mysql net-tools chrony -y

$ systemctl start chronyd

$ systemctl enable chronyd
```

![nginx ami installation 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/80a9e70d-e9aa-4a4d-8dc1-ca14e47a8255)

![nginx ami installation 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2ae5d5cd-1663-4079-89bc-98bd2eb54e82)

![nginx ami installation 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/23983287-e5f4-48a8-820b-785b6e431fe5)

![nginx ami installation 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/99b1bfae-5488-4aae-9dab-081dc14a52ff)

**iii.** Next, we configure selinux policies for the webserver using the following commands:

```
$ setsebool -P httpd_can_network_connect=1

$ setsebool -P httpd_can_network_connect_db=1

$ setsebool -P httpd_execmem=1

$ setsebool -P httpd_use_nfs 1
```

![nginx ami installation 5](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/16a10e31-614a-41e8-8f80-01d0368c7064)

**iv.** Then, we use the following set of commands to install amazon efs utils for mounting the target on the Elastic file system.

```
$ git clone https://github.com/aws/efs-utils

$ cd efs-utils

$ yum install -y make

$ yum install -y rpm-build

$ make rpm 

$ yum install -y  ./build/amazon-efs-utils*rpm
```

![nginx ami installation 6](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a24b3a5a-162d-4402-8712-b3da294ad949)

![nginx ami installation 7](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e4fbb22a-125b-48d0-8a04-d12f4361dcaa)

![nginx ami installation 8](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9f928666-32f8-4582-9ada-fee396cb1d12)

![nginx ami installation 9](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/dd586dcf-b0da-4f3d-a6ae-426219d2676f)


**v.** We proceed with setting up a self-signed certificate for the apache webserver instance.

```
$ sudo yum install -y mod_ssl

$ openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt
```

![webserver ami installation generate cert](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b25f4389-065c-4b75-a274-eaaa317dec4d)

* Then we edit the **`ssl.conf`** fie as shown in the image below.

**`$ sudo vi /etc/httpd/conf.d/ssl.conf`**

![webserver ami installation edit etc httpd conf](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9ce341b1-eba6-46b1-b0e9-e4da69950f8c)

**vi.** Create an `AMI` out of the EC2 instance

* Select Webserver instance, then navigate to **`Actions > Image and templates > Create image`** and then we configure and create the AMI as shown in the image below:

![create webserver ami](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8d76a2f4-b4a1-42d2-a440-f770f927a323)

###### Prepare Launch Template For Webservers (One per subnet)

**i.** Make use of the AMI to set up a launch template.

![create wordpress launch template 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c0de09a2-fa65-4556-b3b0-83129cbad384)

**ii.** Ensure the Instances are launched into a public subnet.

![create wordpress launch template 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c76a50a1-61a3-4272-8e41-015e186477d1)

**iii.** Assign appropriate security group.

![create wordpress launch template 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/607fd30c-77c9-44ed-9c71-e9017c8873ec)

**iv.** Configure Userdata to update `yum` package repository and install `wordpress` (*Only required on the WordPress launch template*).

![create wordpress launch template 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/28c20a5f-c7e7-405e-8439-a91bfd2cfebc)

###### Configure Target Groups for both Wordpress and Tooling servers

* From the EC2 dashboard, we navigate to **`Target groups > Create target group`** and then we do the following:

**i.** Select Instances as the target type.

**ii.** Ensure the protocol `HTTPS` on secure TLS port `443`

**iii.** Ensure that the health check path is `/healthstatus`

**iv.** Register targets accordingly.

![create wordpress target group](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f3f68f27-4166-4dd7-8bbc-13a92d4c58f3)

![create tooling target group](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0af43d8a-cf8b-4322-9dd6-d32ee8b1614d)

**v.** Ensure that health check passes for the target group.

![target group healthy 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f3779153-e61a-4561-8bf7-d16b1b11ef31)

![target group healthy 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/2a6d300b-66ce-4299-9406-4b0c55ee06bf)

###### Create Database for Wordpress and Tooling

Before creating Autoscaling Group for WordPress and Tooling, we need to SSH into our Bastion server and from the bastion we connect into RDS by copying the RDS endpoint as host. And after we gain access into the RDS instance, we create database for tooling and wordpress respectively.

```
$ mysql -h acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com -u ACSadmin -p

mysql> CREATE DATABASE wordpressdb;

mysql> CREATE DATABASE toolingdb;
```

![connect to RDS and create tooling and wordpress db](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/29090557-ac7e-4fc1-a8c6-9e80820faa7a)

###### Configure Autoscaling For Wordpress 

* From the EC2 dashboard, we navigate to **`Auto Scaling Groups > Create Auto Scaling group`** and we do the following:

**i.** Select the right launch template.

**ii.** Select the VPC.

**iii.** Select both public subnets.

![ACS wordpress autoscaling 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a82bb4b8-849f-49eb-a63d-f25e25088478)

**iv.** Enable Application Load Balancer for the AutoScalingGroup (ASG).

**v.** Select the target group you created before.

**vi.** Ensure that you have health checks for both `EC2` and `ALB`.

**vii.** The desired capacity is 2.

**viii.** Minimum capacity is 2.

**ix.** Maximum capacity is 4.

![ACS wordpress autoscaling 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c0128953-7061-418b-81c8-7989185311e1)

**x.** Set [scale out](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroupLifecycle.html) if CPU utilization reaches 90%.

**xi.** Ensure there is an [`SNS`](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) topic to send scaling notifications.

![ACS wordpress autoscaling 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/abdf7eb3-f568-4878-9fd1-76465d5a6568)

###### Configure Autoscaling For Tooling 

* From the EC2 dashboard, we navigate to **`Auto Scaling Groups > Create Auto Scaling group`** and we do the following:

**i.** Select the right launch template.

**ii.** Select the VPC.

**iii.** Select both public subnets.

![ACS tooling autoscaling 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/48a120c1-5a72-4920-9490-6dfe2c57d88a)

**iv.** Enable Application Load Balancer for the AutoScalingGroup (ASG).

**v.** Select the target group you created before.

**vi.** Ensure that you have health checks for both `EC2` and `ALB`.

**vii.** The desired capacity is 2.

**viii.** Minimum capacity is 2.

**ix.** Maximum capacity is 4.

![ACS tooling autoscaling 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c4281a0b-7ceb-4b3a-8d86-3205646a8713)

**x.** Set [scale out](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroupLifecycle.html) if CPU utilization reaches 90%.

**xi.** Ensure there is an [`SNS`](https://docs.aws.amazon.com/sns/latest/dg/welcome.html) topic to send scaling notifications.

![ACS tooling autoscaling 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f9441d5f-bc92-4d42-bb09-d8550f24c4f0)

#### <br>Configure Application Load Balancer (ALB)<br/>

We proceed to configure both External and Internal Application Load Balancers.

##### <br>External Application Load Balancer To Route Traffic To NGINX<br/> 

Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the **ALB** to Nginx servers across the 2 Availability Zones. We will also be able to [offload](https://avinetworks.com/glossary/ssl-offload/) SSL/TLS certificates on the **ALB** instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

From the EC2 dashboard, we navigate to **`Load Balancers > Create load balancer > Application load balancer > Create`** and then we do the following:

**i.** Create an Internet facing ALB.

**ii.** Ensure that it listens on `HTTPS` protocol (TCP port 443).

**iii.** Ensure the ALB is created within the appropriate `VPC | AZ | Subnets`.

**iv.** Choose the Certificate from ACM.

**v.** Select Security Group.

**vi.** Select Nginx Instances as the target group.

![create external application load balancer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/86ee1ad5-0c2b-4b3a-b6e4-6ce34446e6ee)

##### <br>Internal Application Load Balancer To Route Traffic To Web Servers<br/> 

Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic. 

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

From the EC2 dashboard, we navigate to **`Load Balancers > Create load balancer> Application load balancer > Create`** and then we do the following:

**i.** Create an Internal ALB.

**ii.** Ensure that it listens on `HTTPS` protocol (TCP port 443).

**iii.** Ensure the ALB is created within the appropriate `VPC | AZ | Subnets`.

**iv.** Choose the Certificate from ACM.

**v.** Select Security Group.

**vi.** Select webserver Instances as the target group.

![create internal application load balancer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8b6668bf-5715-483b-a157-bcd2e55a61bf)

* After the internal load balancer is created, we select it and navigate to **`Listeners and rules > Manage rules > Add rule `** and then we do the following:

**i.** Add a Name tag and define Rule conditions.

![define rule condition](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a8b3fd06-1798-4382-9f01-538f3aa12f30)

**ii.** Define Rule actions.

![define rule actions](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/59d03b25-87d7-451f-ac7f-7acbfc5396f9)

**iii.** Set rule priority.

![set rule priority](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d1d337be-eac5-4a4e-8957-6516459c9d0d)

**iv.** Review and Create rule.

![review and create rule](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5baf5d1b-ce16-4d9c-b546-30bf199d5b16)

***NOTE:*** This process must be repeated for both WordPress and Tooling websites.

#### <br>Configuring DNS with Route53<br/>

Earlier in this project we registered a free domain with Freenom and configured a hosted zone in [**Route53**](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html). But that is not all that needs to be done as far as DNS configuration is concerned. 

We need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

We create other records such as [`CNAME`, `alias` and `A` records](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/route-53-concepts.html).

***NOTE***: You can use either `CNAME` or `alias` records to achieve the same thing. But `alias` record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read [here](https://support.dnsimple.com/articles/differences-between-a-cname-alias-url/#:~:text=The%20A%20record%20maps%20a,a%20name%20to%20another%20name.&text=The%20ALIAS%20record%20maps%20a,the%20HTTP%20301%20status%20code) to get to know more about the differences.

![records in route 53](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/677cbb33-1646-478b-871e-ecf5726870b9)

**i.** Create an `alias` record for the root domain and direct its traffic to the `ALB` DNS name.

![route 53 hosted zones add records2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8f306d05-9383-427c-8005-a99a6626bfb3)

**ii.** Create an `alias` record for `tooling.<yourdomain>.com` and direct its traffic to the `ALB` DNS name.

![route 53 hosted zones add records](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/80183177-66a7-4219-a703-8519836343c3)

### <br>Conclusion<br/>

We have been able to successfuly use reverse proxy technology to build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a company that uses WordPress CMS for its main business website, and a Tooling Website **(`https://github.com/QuadriBello/tooling`)** for their DevOps team. As can be seen in the images below, we were able to successfully reach both Wordpress and Tooling websites via the browser.

![Welcome to wordpress](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a0e74ff3-7a53-4e85-b8ea-4c2cf87500c7)

![tooling webpage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1e62e0b7-4db3-4b31-9f00-6e79ee32f045)

The goal of for us at the beginning of this project was to rely on our project diagram and employ AWS cloud concepts and tools in building robust and secure cloud infrastructure and components to support our Wordpress and Tooling websites. We began by creating a domain name for our company and then created a hosted zone in AWS Route 53 and mapped it to our domain. Next, we created a Virtual Private Cloud (VPC). Within the VPC, we enabled DNS hostnames within VPC settings, created public and private subnets, created route tables and associated them with our subnets, created and attached Internet gateway, created NAT gateway and then also created security groups.

Next, we created [Amazon Elastic File System (Amazon EFS)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEFS.html) a simple, scalable, fully managed elastic [Network File System (NFS)](https://en.wikipedia.org/wiki/Network_File_System) for use with AWS Cloud services and on-premises resources; and  [Amazon Relational Database Service (Amazon RDS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html) a managed distributed relational database service by Amazon Web Services. After this, we deployed our compute resources which include: [TLS Certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security), [EC2 Instances](https://www.amazonaws.cn/en/ec2/instance-types/)for Nginx, Bastion and Web Servers, [Launch Templates](https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchTemplates.html), [Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html), [Autoscaling Groups](https://docs.aws.amazon.com/autoscaling/ec2/userguide/AutoScalingGroup.html) and [Application Load Balancers (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html).

We made sure to SSH into our Bastion server and from the bastion we connected into RDS and we created database for tooling and wordpress respectively. We also deployed the External Application Load Balancer to route traffic to NGINX and the Internal Application Load Balancer to route traffic to Web Servers. Next, we updated our DNS with route 53 and created alias records for our Wordpress and Tooling domains. Subsequently, we were able to access both the Wordpress and Tooling websites from our browser bringing us to the end of the project. Thank you.

  
