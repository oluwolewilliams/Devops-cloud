## DOCUMENTATION FOR ANSIBLE-AUTOMATE PROJECT

In this project, we will be working on automating routine IT operations with [Ansible Configuration Management](https://www.redhat.com/en/topics/automation/what-is-configuration-management#:~:text=Configuration%20management%20is%20a%20process,in%20a%20desired%2C%20consistent%20state.&text=Managing%20IT%20system%20configurations%20involves,building%20and%20maintaining%20those%20systems.). The project will show how deploying Ansible will help to simplify complex tasks and streamline IT infastructure. We will also work with Jenkins to configure and execute build jobs whilst writing code using declarative language such as **`YAML`**.
 

### <br>Introduction to Ansible Configuration Management<br/>

The goal of this project is to demonstrate Ansible's powerful automation capabilities. Ansible is an open source, command line software application that is used for automating IT operations such as deploying applications, managing configurations scaling infrastructure and other activities involving many repetitive tasks. Ansible's main strengths are simplicity and ease of use. It lets IT professionals quickly and easily deploy multi tier apps. Rather than needing to write lengthy code to automate our systems, we simply list the tasks that require automation by writing a Playbook and Ansible figures out how to get our systems to the state we need them to be in. It is also important to note Ansible has a strong focus on security and reliability and as such it has very minimal moving parts. A great exapmle of this is that Ansible is agentless. This means that the devices or infrastructure it monitors do not require any proprietary software agent to be installed on them beforehand. Ansible has two major types of files: 

**1.** The Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.
   
**2.** Ansible Playbooks are lists of tasks that are automatically executed for the specified inventory or groups of hosts. One or more Ansible tasks can be combined to make a play — an ordered grouping of tasks mapped to specific hosts—and tasks are executed in the order in which they are written.

![0_sMSfIbPO8mH299to](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4134dac2-80be-4957-bdf5-9357889f51f5)

#### Ansible as a Jump Server

A [Jump Server](https://en.wikipedia.org/wiki/Jump_server) (sometimes also referred as [Bastion Host](https://en.wikipedia.org/wiki/Bastion_host)) is an intermediary server through which access to internal network can be provided. If you think about the current architecture we are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provides better security and reduces [attack surface](https://en.wikipedia.org/wiki/Attack_surface).

On the diagram below, the Virtual Private Network (VPC) is divided into [two subnets](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html) – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

![bastion host](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7edfe278-2217-45c4-b35d-ac05bfd8ff47)

Further on in our future projects, we will implement a proper Bastion Host. But for now, we will make do with developing Ansible scripts to simulate the use of a Jump Box/Bastion Host to access our Web Servers. Therefore, based on the infrastructure to be used and the goal of this project, it shall consist of three parts:

**1.** Install and Set-up Jenkins on EC2 Instance.

**2.** Install and Configure Ansible to act as a Jump Server/Bastion Host.

**3.** Begin Ansible Development, Set up an Inventory and Create a Simple Ansible Playbook to Automate Server Configuration.

### <br>Install and Set-up Jenkins on EC2 Instance<br/>

To begin our project we need to install and set up Jenkins. Jenkins is an open-source automation tool written in Java with plugins built for continuous integration. Jenkins is used to build and test software projects continuously making it easier for developers to integrate changes to the project, and making it easier for users to obtain a fresh build. It also allows developers to continuously deliver software by integrating with a large number of testing and deployment technologies. We deploy Jenkins by implementing the following steps:

#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Ubuntu Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e9c2dc00-0a59-433a-ab2d-6136183f190f)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Ubuntu Linux Server 22.04 LTS (HVM).

![Application and OS Image](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ae844641-2121-49de-99e3-67c8621c4027)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Open Port 8080<br/>

**Jenkins** uses TCP Port 8080 by default and as this will be our first installation, we will therefore need to open Port 8080 to allow traffic from anywhere. To implement this, we need to add a rule to the Security Group of our Web Server:

**i.** In the AWS  console navigation pane, we choose **Instances**.

![Instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75d2208f-d030-4f44-9667-23521332607f)

**ii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ee3841b0-95f1-43be-94b1-6210d7bea296)

**iii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**v.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **8080** 

+ In the space with the magnifying glass under **Source**, choose **Anywhere**.

+ Click on **Save rules** at the bottom right corner of the page.

![open port 8080](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d41b3f43-59c9-4c78-8a80-0fa40515399a)

#### <br>Step 3: Create and Allocate Elastic IP Address to Jenkins-Ansible Server<br/>

Considering that we'll be using Jenkins with Github and configuring Web Hooks in this project, it will make our job easier to create and allocate an elastic IP adress to our Jenkins-Ansible Server. This is beacuse everytime we stop/start the server, there will be a need to keep reconfiguring Github Web Hooks to a new IP address. Having an elastic IP address (which will not change when we stop/start the server) is the ideal way to overcome this issue.

**i.** From the EC2 cockpit, we click on **"Elastic Ips"**.

![elastic IPs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2bd8c559-b18f-4db9-9446-84eed4c34c81)

**ii.** In the next page displayed, we click on **"Allocate Elastic IP Address"**.

![Allocate elastic IP address](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/633bb57d-7414-425e-ba73-66faa341eeb1)

**iii.** In the elastic IP address settings page we ensure to choose the same network border group as our EC2 instance. Then we select the option to choose from Amazon's pool of IPv4 addresses. Then we click on **"Allocate"**.

![elastic IP address settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2b22c080-2bfa-4b8f-aa59-808fc7d1ba9f)

**iv.** In the next page which shows us that the elastic IP has been allocated successfully, we click on the **"Actions"** drop down tab and we select **"Associate Elastic IP address"**.

![associate elatic ip 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/00c69fbe-bea1-435c-ba5a-37323e19ccc7)

**v.** In the Associate Elastic IP Address page, we scroll down to **"Instance"** and we select our EC2 instance. After this, we click on the **"Associate"** button at the bottom of the page.

![Associate Elastic IP address](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a8ee57a4-991e-4505-9fb8-9b7132353d8a)

**vi.** As can be seen in the output image below, the elastic IP address was successfully asssociated with our EC2 instance.

![Elastic IP successfully associated](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a6dba97b-2953-485e-a13f-ecbe8ac62dd4)

#### <br>Step 4: Connect to the Jenkins-Ansible Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have opened the necessary port, we must next connect to the server via an SSH client. This will enable us to subsequently be able to run commands on the terminal of our server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Jenkins Installation and Set-up<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing other installations or configurations. We do this by executing the following command: 

**`$ sudo apt update -y`**

![sudo apt update -y](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5c7ab738-cee6-4385-b1f1-9ddff8f263af)

**ii.** Next, we run the following set of commands to [install dependencies for Jenkins](https://pkg.jenkins.io/debian-stable/):

```
$ sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \ https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

$ echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \ https://pkg.jenkins.io/debian-stable binary/ | sudo tee \ /etc/apt/sources.list.d/jenkins.list > /dev/null

$ sudo apt-get update

$ sudo apt-get install fontconfig openjdk-11-jre

```

![jenkins installation 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4e9df2c4-1a6c-41a8-9d51-18289cae592d)

**iii.** Then we execute the command below to install Jenkins:

**`$ sudo apt-get install jenkins`**

![jenkins installation 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4a53c019-5a7c-4d84-8999-4cb0ec297313)

**iv.** To ensure Jenkins is up and running, we enter the command below:

**`$ sudo systemctl status jenkins`**

![sudo systemctl jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d401bbae-7a7b-4e94-bbe6-08f8302dc06c)

**v.** Then we begin setting up Jenkins by accessing it via our browser using the following syntax: 

**`http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`**

![unlock Jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/730aadd0-bfb3-4774-b009-c7c7de539a2d)

**vi.** As shown in the output image above, we are prompted to provide an Administrator password to unlock Jenkins. To retrieve the password from the Jenkins Server, we enter the following command: 

**`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`**

![initial admin password](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b4b11c70-6079-4fef-a5bd-996b8959fc5a)

**vii.** Next, from the server, we copy the password as seen in the image above and we paste it in the dialogue box in the **"Unlock Jenkins"** page after which we click on **"Continue"**.

![unlock jenkins 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce0044ef-005e-498d-adc1-620b9a1f1e92)

**viii.** This brings us to the **"Customize Jenkins"** page. Here we select and click on **"Install Suggested Plugins"**.

![jenkins install suggested plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7b78280e-b378-4529-86db-be2804b09de0)

![installing plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4dd246cf-eb66-42e0-ac53-f038475ca8ca)

**ix.** As shown in the above image, we were able to successfully install the plugins. After completion, Jenkins prompts us to **"Create a First Admin User"** we can either fill in our details to do this or we just choose to **"Skip and continue as admin"**. We decide to go with the latter option.

![skip and continue as admin](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/82c59719-18a2-49b5-b1aa-610527076f30)

**x.** In the instance configuration page, we paste in our elastic ip address and click on **"Save and Finish"**.

![instance configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7a0e7627-7661-4af4-b270-d9efbca0207f)

**xi.** At this point the set-up is complete and Jenkins is ready to be used. We click on **"Start using Jenkins"** to move into the main Jenkins Environment.

![installation complete](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b725826c-ef8b-4906-b6ac-02d37cca0d43)

### <br>Install and Configure Ansible to act as a Jump Server/Bastion Host<br/>

The next phase of our project involves the installation and configuration of Ansible as a Jump Server/Bastion Host. An SSH Jump Server/Bastion Host is a regular Linux server, accessible from the Internet, which is used as a gateway to access other Linux machines on a private network using the SSH protocol. The purpose of an SSH jump server is to be the only gateway for access to our infrastructure reducing the size of any potential attack surface. We shall proceed to implement this with the following steps:

#### <br>Step 1: Create New Repository in GitHub<br/>

To create a new repository, we carry out the following steps:

**i.** We click on the plus sign **(+)** at the top right corner of our github account. A dropdown menu box appears and we select **"New repository"**.

![plus sign](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b8fdf5ae-6f7a-4396-9bbc-aadaea9bfd2d)

**ii.** We fill out the form by entering **`ansible-config-mgt`** as name for our repository. We enter a description and then we check the box to add a **README.md** file. Afterwards, we leave every other box or button in their default state.

![create new repository](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0f08b401-d40f-40a9-ae32-93757fa8c10d)

**iii.** We click the green **"Create repository"** button at the bottom of the page to create our repository.

![create repository button](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2672317f-6cac-4411-ac17-319f512e2e10)

**iv.** As can be seen in the image below, we were able to successfuly create the repository.

![succesful create repository](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8f358b9f-1417-4c5d-a5f3-d629a91651e6)

#### <br>Step 2: Ansible Installation<br/>

In this step, we are going to install Ansible on the same server (Jenkins-Ansible Server) where we have Jenkins installed.

**i.** First of all, we update the server machine if we have not already done so:

**`$ sudo apt update -y`**

![update machine](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d6bd2879-c06f-4cd5-b27a-38022f4d213b)

**ii.** Then we install Ansible by executing the command below:

**`$ sudo apt install ansible -y`**

![install ansible](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/99407114-aac0-4120-beca-d38584d79121)

**iii.** Next we check our ansible version by running the following command:

**`$ ansible --version`**

![ansible version](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3c1796d7-7424-423c-a0ce-b079ef6cf2c5)

#### <br>Step 3: Create and Configure Jenkins Freestyle Build Job<br/>

In the Jenkins web console, we create a new freestyle project that we will name **`ansible`** and point it to our GitHub **`ansible-config-mgt`** repository. We do this with the following steps:

**i.** From the Jenkins web console, we click on **"New item"**

![jenkins new item](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6dd5ad81-1d92-4908-813f-eecce8717871)

**ii.** In the next page under **"Enter an item name"** we type in **`ansible`**, then we select **"Freestyle project"** and we click on **"Ok"** at the bottom of the page.

![enter item name](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9677dbe2-4cf7-40a9-a4c3-7fe379ef3d11)

**iii.** We go to our GitHub repository page and we copy the URL of the webpage.

![project url](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0d2828b3-3690-467e-a720-898428f2617e)

**iv.** Then in the Jenkins configuration page, we click on the **"GitHub project"** check box and we paste in the **"project URL"**.

![github project](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/38961a2f-2d6d-4881-a479-e7f7844eaf03)

**v.** Next, we go to our GitHub repository and we obtain the remote link for our **`ansible-config-mgt`** repository by clicking the green **"Code"** button and copying the https link.

![copy repository URL](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca80b0ad-6ed9-43c2-bbf6-b9e95656863a)
  
**vi.** In the Jenkins configuration page, under **"Soure Code Management"** we click on **"Repository URL"** and we paste in the remote link for our GitHub repository.

 ![Repository URL](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8dc2a7c5-add4-4ec4-82e7-a757804d0f84)

**vii.** Under **"Branch Specifier"**, we change ***/master** to ***/main**

![branch specifier](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/431e7d41-41f8-4056-a56d-b892fa40fe64)

**viii.** Then we click on **"Apply"** and **"Save"** at the bottom of the page.

![apply and save](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4f559ba5-61b1-4118-a161-f18db45a0f84)

**ix.** In the Jenkins web console, we go to the left pane and click on **"Build Now"** and if the build is successful, we will see it under **"Build History"** as seen in the image below:

![build now](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f07b56f1-b67e-45b4-bbc8-773304e9fc33)

**x.** To view more details about the successful build, we click on the drop down icon beside the build and we select **"Console Output"**.

![console output 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1e66a2a1-1427-4011-962d-22752b912e60)

**xi.** The Console Output shows us the complete text log of output from the build execution:

![console output 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b820be16-8557-4255-b0b7-3519c696e679)

#### <br>Step 4: Configure Jenkins Build Job to Archive Repository Content everytime there are changes<br/>

After creating and configuring the **`ansible`** freestyle project, we had to trigger it manually for it to run. But we can go a step further. To enable our build run automatically whenever there is a change in our Git repository, we need to enable Webhooks in our GitHub repository settings and configure it to trigger **`ansible`** build:

**i.** From our GitHub account, we click on **"Repositories"** tab and then we select the **`ansible-config-mgt`** repository.

![repositories](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1c720509-1936-44cb-ab34-d109d12aaf50)

**ii.** Next on the repository page we click on the **"Settings"** tab.

![repository settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/79517652-2336-40be-b457-90a010c780be)

**iii.** On the Settings page, on the left panel, we click on **"Webhooks"**

![Webhooks](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/36403a01-81b6-44d2-b9fe-1da4378685bf)

**iv.** In the Webhooks page, we click on **"Add webhook"**

![add webhook](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4974fca2-0fba-44ca-8190-a56f2cfdf8a8)

**v.** On the Webhooks/Add webhook page, under **"Payload URL"**, we input our elastic IP address using the following syntax:

  **`http://<Jenkins server IP address>/github-webhook/`**

+ Under **"Content type"**, we select **"application/json"**

+ Then we click on the green **"Add webhook"** button at the bottom of the page.

![add webhook 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2fe8a0ec-06c5-48f6-bec1-a157cc1fd4cb)

**vi.** In Jenkins, on our **`ansible`** project page, we go to the left pane and click on **"Configure"**.

![configure](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b0896307-fd4c-495a-ba04-f2a1a3d0c2fc)

**vii.** Under **"Build Triggers"**, we check the box for **"GitHub hook trigger for GITScm polling"**

![build trigger](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/baa66a8c-bce9-4659-9258-19d6c6fa6254)

**viii.** Next to ensure Jenkins saves all files also known as **"Build Artifacts"**, we go under Post-build Actions, we click on Add post-build Action and we select **"Archive the artifacts"**

![archive the artifacts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/32e764f8-755d-4ac8-810f-6aca5df90a4c)

**ix.**  In the dialogue box under **"Files to archive"**, we simply enter __**__ which refers to all available paths in the working space.

![files to archive](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b4eb146f-ae55-4bda-9844-1ad3daba75d0)

**x.** Then we click on **"Apply"** and **"Save"** at the bottom of the page.

![apply and save 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/63f35033-c852-4ed0-b036-cd3219c9640f)
  
**xi.** To test our set up, we made some changes to the **README.md** file in our **`ansible-config-mgt`** GitHub repository.

 ![readme edit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75f1440b-879a-48e3-bd0d-162da9897e23)

**xii.** Our build started automatically and Jenkins saved the files (build artifacts). We enter the following commands to further confirm this:

```
$ ls /var/lib/jenkins/jobs/ansible/builds/5/archive/

$ sudo cat /var/lib/jenkins/jobs/ansible/builds/5/archive/README.md
```

![config confirmation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/41d11f65-fba5-40f7-8221-bf225c3f30f7)

#### <br>Step 5: Configure Jenkins to Connect and Copy Files to NFS Server <br/>

 In **Step 4** above, we concluded with Jenkins successfully saving our build artifacts. Now in this step, we will proceed to configure Jenkins to connect via SSH and copy files to the **`/mnt/opt`** directory in the NFS server we deployed for the [Tooling Website Solution](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project10.md)

**i.** To begin this implementation, we will require a plugin called **"Publish Over SSH"**. We install this by doing the following:

+ From the main Jenkins Environment, we click on **"Manage Jenkins"**.

 ![manage jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/490e07d7-4916-493e-bb16-f541914e99ff)

+ On the System Configuration page, we click on **"Plugins"**.

 ![plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9f23b0ef-46e5-43ae-a876-ffb499a7896d)

+ On the Plugins page, we click on Available plugins, then we type **"Publish Over SSH"** in the search box.

 ![Available plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c6d11722-53d4-42d1-8b39-9c7770004227)

+ From here, we can see the **"Publish Over SSH"** plugin we wish to install, so we check the **"Install"** checkbox beside it and we click on the **"Install"** button. After doing this we will be able to see the download progress page.

![installed plugin](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/15174ef3-7d8f-4300-90b2-18b9ac2928a5)

**ii.** The next step is to configure the **"Publish Over SSH"** plugin and our **`ansible`** job/project to copy our build artifacts to the NFS server. We do this by carrying out the following:

+ From the main Jenkins Environment, we click on **"Manage Jenkins"**.

 ![manage jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/490e07d7-4916-493e-bb16-f541914e99ff)

+ On the **"System Configuration"** page, we click on **"System"**.

 ![configure system](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/03afe719-f9d4-4b1f-a8a8-fc0ddf64e2b1)

+ Then we scroll down to the configuration section for the  **"Publish Over SSH"** plugin configuration section. 

![publish over ssh configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce6f319a-8f2e-4d2c-aaa6-f0653328bbb1)

+ Next, under **"Key"**, we put in the content of .pem file that we used in connecting to to the NFS server via SSH.

![publish over ssh ey](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f7168f5f-3400-4d3f-bc39-3f197f9de66f)

**iii.** Now we need to input configuration more configuration to enable connection to the NFS Server. Under **"SSH Servers"**, we click on the **"Add"** button and then we input the following: 

+ **Name**: This can be any arbitrary name.

+ **Hostname**: This will be the private IP address of the NFS server.

+ **Username**: For this we will use **ec2-user** which is the username used for Red Hat Enterprise Linux based servers such as our NFS Server.

+ **Remote directory**: Here, we will use the **`/mnt/opt`** directory which we specified we will be using for Jenkins in our [Tooling Website Solution project.](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project10.md)

+ Next is to test the configuration and ensure the connection returns **"Success"**. We should note that TCP Port 22 on our NFS server must be open to receive SSH connections.

![ssh servers](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6fab555f-e524-4cfe-9791-17fca96205f4)

+ As we can see from the image above, the connection is successful, so we save the configuration by clicking on **"Apply"** and **"Save"**.

**iv.** The next step is to add another Post-build Action to our **`ansible`** project. We do this by implementing the following:

+ From the main Jenkins Environment, we click on the project **`ansible`**.

 ![project ansible](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7c171b6b-efe8-49f8-9a12-aae9207939ac)

+ Then in the  **`ansible`** project page we click on **"Configure"** in the left hand pane.

 ![configure ansible](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/98c06eec-50f4-481c-a3e5-16ea8f0f2dc7)

+ In the Configuration page, we select **"Post-build Actions"** in the left hand pane, then we click on the **"Add Post-build Action"** drop down button and we select **"Send build artifacts over SSH"**.

 ![add post build action](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4b45b3b3-2969-4fea-8332-4d1ee9848dff)

+ To ensure that all files produced by the build are sent to the **`/mnt/opt`** directory, under **"Transfers"** section, we enter __**__ under **"Source Files"**.

 ![source files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/bb3bae51-b45c-46b0-a46c-36a907dc5241)

+ Then we click on **"Apply"** and **"Save"** at the bottom of the page.

 ![apply and save 3](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/47a5fc3e-cdfc-401f-ac1d-04028bebed9f)

+ To test our set up, we made some changes to the **README.md** file in our **`ansible-config-mgt`** GitHub repository.

 ![changes to readme](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f982026d-f9f8-4cf3-ba09-4b21cc5f3afd)

+ As we can see in the console output below, our build ran successfully. Connection was made with the NFS Server via SSH and the build artifacts were successfully copied.

![console output 3](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d83c17ff-e372-4af2-a89c-669e2e909a8c)

+ To also confirm that the files in **`/mnt/opt`** directory have been updated we connect to the NFS server via our SSH client and execute the following command:

**`cat /mnt/opt/README.md`**

![confirmation in NFS server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d14f6de4-2f4b-4e76-8dee-a20dd36514bb)

#### <br>Step 6: Deploy EC2 Instance to act as Load Balancer <br/>

To get our set up to look like the one in the diagram below, we will need to deploy a load balancer to distribute traffic across our web servers.

![nginx architecture](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5aa4052c-ae28-4e56-8050-46b4f6613295)

We begin by spinning up an EC2 Instance of Ubuntu Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our web server.

![Name and tags 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/af3ac793-67a3-46fa-bb19-b7dbc2255518)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM).

![Application and OS Image](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ae844641-2121-49de-99e3-67c8621c4027)
  
**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![KP](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d0c8f5c4-d0ca-46a1-9316-0438618551c8)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

**vi.** We will be running our Load Balancer on TCP Port 8080. We will therefore need to open Port 8080 to allow traffic from anywhere. To implement this, we need to add a rule to the Security Group of our Load Balancer. In the AWS  console navigation pane, we choose **Instances**.

![Instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75d2208f-d030-4f44-9667-23521332607f)

**vii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cd84ec18-4072-42c6-aee7-94921bf803e3)

**viii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**ix.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**x.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **8080** 

+ In the space with the magnifying glass under **Source**, choose **Anywhere**.

+ Click on **Save rules** at the bottom right corner of the page.

![port 8080](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/183e37d3-2b35-430c-b22b-dc7d7e19b029)

**xi.** After we have provisioned our server and we have opened the necessary port, we must next connect to the server via an SSH client.  Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**xii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 7: Configure Load Balancer to distribute traffic across Web Servers <br/>

**i.** The subsequent step is to install Nginx and configure it to effectively distribute traffic across our three Web Servers. But we need to first of all update the server. So we concatenate the update and installation actions by executing the following command:

**`$ sudo apt update -y && sudo apt install nginx -y`**

![install nginx](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/beae01c8-cb86-4b8e-8c97-d9d9926b4ad2)

**ii.** To verify that Nginx is installed and active, we run the command below:

**`$ sudo systemctl status nginx`**

![systemctl status nginx](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b00ddbaf-1ec4-4a32-b76a-70f3e9c16e4a)

**iii.** Our next step is to open the Nginx configuration file with the following command:

**`$ sudo vi /etc/nginx/conf.d/loadbalancer.conf`**

+ We copy and paste in the configuration file below to configure nginx to act as a load balancer. As can be seen in the file, necessary information like Public IP and Port Number for our three web servers are provided. We also need to provide the Public IP address of our Nginx Load Balancer.

```
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server 51.20.82.142:80; # public IP and port for webserver 1
            server 51.20.131.231:80; # public IP and port for webserver 2
            server 51.20.122.38:80; # public IP and port for webserver 3

        }

        server {
            listen 8080;
            server_name 51.20.190.36; # provide your load balancers public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
```

![load balancer configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/87f67b78-e276-4f71-a664-a93ca0175d12)

The following serves to break down the configuration file above and explain in more detail:

+ _**upstream backend_servers**_ defines a group of back end servers (our two web servers).

+ The **server** lines inside the **upstream** block lists the ports and public IP addresses of both of our backend webservers.

+ **proxy_pass** inside the location block sets up the load balancing, passing the request to the back end servers.

+ The **proxy_set_header** lines pass necessary headers to the backend servers to correctly haandle the requests.

+ Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**iv.** We proceed to execute the command below to test our configuration:

**`$ sudo nginx -t`**

![test configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e5f7b940-33c7-487d-b6fb-5bb7bb3ec4c6)

**v.** Provided there are no errors, we execute the following command to restart the Nginx service and load our new configuration.

**`sudo systemctl restart nginx`**

**vi.** Then the final step is to go to our browser to paste in the public IP address of our Nginx loadbalancer (syntax is: **`http://<Public-IP-Address>:8080`**). In our own use case, we enter the following url in our browser:

**`http://51.20.190.36:8080`**

![nginx browser](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b7d03bd3-5e47-4630-a5fb-de6f8a679d3c)

**vii.** As shown in the output image above, our load balancer is able to serve content from the tooling website on our web servers so we have somewhat acheived the setup architecture shown in the diagram below:

![nginx architecture](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5aa4052c-ae28-4e56-8050-46b4f6613295)

#### <br>Step 8: Prepare Development Environment <br/>

The first part of **"DevOps"** is **"Dev"**, which means we will be required to write some codes and to make coding and debugging comfortable, we will need an Integrated development environment (IDE) or Source-code Editor. There is a plethora of different IDEs and Source-code Editors for different languages with their own advantages and drawbacks. We however decided to use one free and universal editor that will fully satisfy our needs – **Visual Studio Code (VSC)**.

**i.** To proceed, we download and install [Visual Studio Code (VSC)](https://code.visualstudio.com/download)

**ii.** After successfully installing VSC, we configure it to connect to our newly created GitHub repository that we named **`ansible-config-mgt`**. We do this by implementing the following:

+ We open the VS code application and in on the left hand pane we click on **"Source Control"**

![clone repo](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/73c22477-db2b-4944-82c0-d5e5e4ccb37f)

+ Next, we click on the **"Clone Repository"** button and under the search bar, we click on **"Clone from Github"**. 

![clone from github](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/44aea80c-0294-4333-9209-ffb1bd35e80b)

+ This takes us to our browser and we get a prompt to log into our GitHub account so we click on **"Sign in"**.

![github sign in](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ff9b4cf1-b93a-4de2-b487-0a4e9ae51229)

+ Next, we get to the Authorization page where we **"Authorize GitHub for VS Code"**.

 ![Authorize github for vscode](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/be4717f2-efd5-408d-8b86-c66895f1e6d8)

+ Then back in VSC, we select our **`ansible-config-mgt`** repository.

![vsc ansible-config-mgt](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/90488c4d-ac0e-441a-8d65-64f2b494124f)

+  This prompts us to select a folder to use as the repository destination. So we create a folder for our repo and select it.

![create new folder](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0d0cec14-95da-4e2c-91db-e103149b03c9)

+  Then we get a dialogue box to open the cloned repository.

 ![open cloned repo](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8d6ba6b9-4c13-47e6-bd7e-77650efcc756)

+  As we can see in the image below, the **`ansible-config-mgt`** repository has been cloned to VS Code.

![repo cloned](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f7d6b699-d42f-4f63-9b36-a94bf0f0eda5)


**iii.** Next, we clone down the **`ansible-config-mgt`** repository to our Jenkins-Ansible instance with the following command:

**`git clone <ansible-config-mgt repo link>`**

![git clone](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a014233a-1034-4c74-ad0c-8fa2335d59c8)

### <br>Begin Ansible Development, Set up an Inventory and Create a Simple Ansible Playbook to Automate Server Configuration<br/>

#### <br>Step 1: Begin Ansible Development <br/>

**i.** To begin Ansible development we go to our ansible-config-mgt GitHub repository in VS Code and we create a new branch **`new-feature`** that will be used for development of a new feature. We do this by following the steps below:

+ From the VS Code environment we go to the bottom of the page and we click on **`main`**.

+ Next, under the dialogue box, we select **"Create new branch from.."** and we choose **`main`**.

![create new branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/659f1c32-4fa4-4d80-a97f-1c53bb1d70a1)

+ Then we subsequently click on the **"Publish Branch"** button.

 ![publish branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f3a6dad0-134c-40dd-a3b8-f7c37dba12ac)

+ Afterwards, we enter the new branch name **`new-feature`** and we press **Enter**.

![branch name](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ea9cf21d-a8ed-4abd-80ab-bfb8db0c09ed)

**ii.** Next, we enter the command below to checkout the newly created branch **`new-feature`** to our local machine and start building our code and directory structure:

**`$ git checkout new-feature`**

![git checkout](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/dfccdbc5-a0a9-4df0-9604-207c67fa5711)
 
**iii.** We use the following command to create a directory that will be used to store all our playbooks files and we name it **`playbooks`**.

**`$ mkdir playbooks`**

**iv.** We also create a directory that will be used to keep our hosts organised and we name this **`inventory`**. 

**`$ mkdir inventory`**

![mkdir playbooks inventory](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3997bcc9-6095-449e-bd05-1c43531ec6c4)

**v.** Within the playbooks folder, we create our first playbook and name it **common.yml**:

```
$ cd playbooks

$ touch common.yml
```

![create common-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/06666aab-0de2-48b5-8138-bb599473de78)

**vi.** And within the inventory folder, we create **`dev.yml`**, **`prod.yml`**, **`staging.yml`** and **`uat.yml`** for development, production, staging and user acceptance testing environments respectively.

```
$ cd inventory

$ touch dev.yml

$ touch prod.yml

$ touch staging.yml

$ touch uat.yml
```

![devprodstaginguat-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8ff6d91c-1875-4e9e-a48d-487e5949c944)

#### <br>Step 2: Setting up Ansible Inventory <br/>

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs, it is important to have a way to organize our hosts in such an inventory. We shall save the below inventory structure in the **`inventory/dev`** file to start configuring our development servers. We will also ensure to replace the IP addresses according to our own setup.

**i.** Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from the Jenkins-Ansible host. For this, we implement the concept of ssh-agent. To install and configure Openssh server and agent for Windows 11, we follow these [instructions.](https://windowsloop.com/install-openssh-server-windows-11/)

**ii.** Next, we need to enable the ssh agent for the current session:

**`$ eval `ssh-agent -s``**

![eval ssh agent](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/12190653-f8a0-4ef9-8b56-5f1c2c26e560)

**iii.** Then we import our key into ssh-agent by executing the following command:

**`$ ssh-add <path-to-private-key>`**

![ssh-add](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5fea854d-2c76-49ff-b134-1025bebb8c73)

**iv.** Afterwards, we confirm the key has been added with the command below:

**`$ ssh-add -l`**

![ssh-add-l](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8b7898a9-6ece-46ee-ba42-caf0ee1a68a7)

**v.** Now we proceed to ssh into our **Jenkins-Ansible** server using the ssh-agent:

**`ssh -A ubuntu@public-ip`**

![ssh -A ubuntu](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/674f0c64-3c4f-47aa-a3b3-88ef15133c0a)

**vi.** Then we update our **`inventory/dev.yml`** file with the following lines of code:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user=ec2-user

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user=ec2-user
<Web-Server2-Private-IP-Address> ansible_ssh_user=ec2-user
<Web-Server3-Private-IP-Address> ansible_ssh_user=ec2-user

[db]
<Database-Private-IP-Address> ansible_ssh_user=ec2-user 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user=ubuntu
```

![input code dev-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b4b5992e-5f92-4d5a-9d46-49447265ce91)

#### <br>Step 3: Create a Common Playbook <br/>

Ansible Playbooks are lists of tasks that automatically execute for a specified inventory or groups of hosts. Now we proceed to give Ansible the instructions on what we need to be performed on all servers listed in **`inventory/dev`**. In the **`common.yml`** playbook, we will write configuration for repeatable, re-usable, and multi-machine tasks that are common to systems within the infrastructure. We begin by updating our **`playbooks/common.yml`** file with following code:

```
---
- name: update web and nfs servers
  hosts: webservers, nfs
  become: yes
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest
   

- name: update LB and db servers
  hosts: lb, db
  become: yes
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest
```

![playbook common-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7708b464-d2c1-4695-98f3-7d2d2fabf85c)

+ The code as seen in the playbook image above is divided into two parts with each of them intended to perform the same task: install wireshark utility (or make sure it is updated to the latest version) on our RHEL 8 (Web, NFS) and Ubuntu (LB, DB) servers. It uses root user to perform this task and respective package manager: **`yum`** for RHEL 8 and **`apt`** for Ubuntu.

#### <br>Step 4: Update GIT with latest Code <br/>

At this point, our directories and files are on our local machine so we need to push all the changes we made locally to Github.

**i.** We use the following commands to add commit and push our branch to GitHub:

```
$ git status

$ git add <selected files>

$ git commit -m "commit message"
```

![git commit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/750833cf-5068-4201-8a32-f217fd763d1e)

**ii.** The next thing we do is to create a **Pull Request** in GitHub by following [these steps:](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 

+ From the **`ansible-config-mgt`** repository page, we click on the **"Pull requests"** tab and then in the next page we click on the **"New pull request"** button.

![pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a27f564c-53f7-49c8-9905-aa7e440fc17b)

![new pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/46e0162b-0cb7-4ff4-8514-6fd0c8ab0a7c)

+ This takes us to the **"Compare changes"** page where we choose the **`new-feature`** branch to set up a comparison with the **`main`** branch.

![compare changes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d8994c81-ac90-49bd-8f0e-f0b2784928df)

+ Once we set up the comparisons between the **`main`** and the **`new-feature`** branch, we then proceed to click on the **"Create pull request"** button.

![compare changes able to merge create pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3e769f77-05a9-4e0e-a21b-44b4a6e9a9e0)

+ In the next page, we input a pull request message inside the dialogue box and we click on the  **"Create pull request"** button.

![create pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/227acfa7-d1e0-471b-8f21-11804afa65e1)

**iii.** Now as shown in the image below, we act as a reviewer and we examine the changes in the **`new-feature`** branch and check for conflicts with the **`main`** branch.

![merge pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9c3dd35f-af53-4f69-ae7a-49f49c2fa639)

**iv.** As we are satisfied and happy with the changes made in **`new-feature`**, we click on **"Merge pull request"** and then we click on **"Confirm merge"**

![confirm merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/602e25e2-635a-4eb1-b17d-4291a95a749a)

**v.** This takes us to the next page which shows that **`new-feature`** has been successfully merged to **`main`** branch.

![successful merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8d033c58-fe3b-4342-a966-b7181d9bef2f)

**vi.** Now we head back to the terminal, and we checkout from the **`new-feature`** branch into the **`main`** branch, and then we pull down the latest changes.

```
$ git checkout main

$ git pull
```

![checkout and git pull](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/66fe3dd6-e712-41e4-b4e9-19b14a128dc4)

**vii.** As seen in the image below, once our code changes appear in the **`main`** branch, Jenkins does its job and saves all the build artifacts (files) to **`/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`** directory on the **`Jenkins-Ansible`** server.

![ansible builds](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eac4dd3b-4763-46d3-863c-2ac56baa5fbe)

**viii.** For further confirmation we go to the terminal for the **`Jenkins-Ansible`** server and we enter the following commands:

```
$ sudo ls /var/lib/jenkins/jobs/ansible/builds/7/archive/playbooks

$ sudo cat /var/lib/jenkins/jobs/ansible/builds/7/archive/playbooks/common.yml

$ sudo ls /var/lib/jenkins/jobs/ansible/builds/7/archive/inventory

$ sudo cat /var/lib/jenkins/jobs/ansible/builds/7/archive/inventory/dev.yml
```

![inventory and playbooks confirmation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3cb20380-8dc6-4568-8391-aea92937520f)

#### <br>Step 5: Run First Ansible Test <br/>

Now, it is time to execute the **`ansible-playbook`** command and verify if our playbook actually works. We proceed by implementing the following steps:

**i.** We connect to our **`Jenkins-Ansible`** server via VScode:

**`$ ssh -A ubuntu@16.171.91.213`**

![connect to jenkins ansible](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/513d25c1-57bf-424d-b294-a00098af6a1e)

**ii.** Then we change to the **`ansible-config-mgt`** directory and we run our **`ansible-playbook`** with the following command:

```
$ cd ansible-config-mgt

$ ansible-playbook -i inventory/dev.yml playbooks/common.yml
```

![ansible playbook success](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/96d91b5f-1bb6-45b5-b19f-12092d4d5edd)

**iii.** As shown in the output image above, the **`ansible-playbook`** ran successfully and deployed **`wireshark`** on all the servers specified in the playbook. However, for further confirmation, we go to each one of our servers and we run the following commands:

```
$ which wireshark

$ wireshark --version
```

**iv.** The output images below reflect that our playbook deployed wireshark on the Load Balancer, NFS server, DB Server, and Web Servers 1, 2 and 3.

![wireshark loadbalancer](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/486ab3a8-b109-4a09-93b8-4be5c3b3cade)

![wireshark nfs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eea1071d-258f-4aaf-b63f-9c1cb203c481)

![wireshark db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/067e6558-0a77-4ca2-b3ae-6c385e9eef4d)

![wireshark web server 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cdc933e2-b6ff-444e-aa8f-a8de77a27063)

![wireshark web server 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eac63571-5fdb-4755-9de2-5cfe9c5e3ad5)

![wireshark webserver 3](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/94167d29-6375-4d75-b313-fc0767f7e4e3)

### <br>Conclusion<br/>

We have been able to successfuly automate routine tasks by completing the implementation of our first Ansible project and our setup looks somewhat like what we have in the image below.

![complete architecture](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c98a71ff-b96d-4481-a5fc-0d3876402fb7)

The goal of this project was to demonstrate Ansible's powerful automation capabilities and we have been able to do just that. We began the project by installing and setting up Jenkins which we would be using for running build jobs. And then in GitHub, we created a new repository **`ansible-config-mgt`**. After this, we installed and configured Ansible (on the same server as Jenkins) to serve as a Jump Server/Bastion Host. Next, we created a freestyle job and also used GitHub webhooks to configure a Jenkins build job to archive content to a our **`ansible-config-mgt`** repository any time there are changes. 

With our server's new status as a **`Jenkins-Ansible`** instance, we made sure to create and allocate an Elastic IP address to it so that everytime we stop/start the server, there will be no need to keep reconfiguring Github Web Hooks to a new IP address. The next step was to configure Jenkins to connect via SSH and copy files to the /mnt/opt directory in the NFS server we deployed for the [Tooling Website Solution.](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project10.md) Subsequently, to get our set up to the desired state, we needed to deploy a load balancer to distribute traffic across our web servers. Our next step involved preparing Visual Studio Code (VSC) as our development environment. We successfully installed and configured VS Code and then we cloned down our **`ansible-config-mgt`** repository to the **`Jenkins-Ansible`** server.

The final phase of this project was where we commenced Ansible Development. We created our playbook directory with the **`common.yml`** file inside it and then our inventory directory along with the relevant files for development, staging, testing and production. Next, we set up the Ansible inventory file to define the hosts and groups of hosts upon which the commands and tasks in the playbook would operate. We then proceeded to create a playbook to give Ansible the instructions on what we need to be performed on all servers listed in the Ansible inventory. After this, we pushed all the changes we made locally to our GitHub repository to ensure GIT remains updated with the latest code. We did this by creating a pull request and merging the code changes to the **`main`** branch. To complete the project, we set up VS Code to connect to our instance via SSH and we ran our playbook. Afterwards, we were able to confirm that the playbook task (which was to install the latest version of **`wireshark`** on all the servers specified in the inventory file) was carried out successfully. Thank you.



