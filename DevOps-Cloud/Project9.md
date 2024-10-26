## DOCUMENTATION FOR IMPLEMENTING WORDPRESS WEBSITE WITH LVM STORAGE MANAGEMENT

This project demonstrates how to build and manage a scalable WordPress website using AWS EC2 and LVM (Logical Volume Management) storage. The project will delve into the intricacies of LVM storage management on Ubuntu such as the processes of creating logical volumes, managing disk space and accomodating changing storage requirements. And it will also cover performance optimization techniques and best practices for managing and securing WordPress installation in the AWS cloud.

### <br>Introduction to Implementing WordPress Website with LVM Storage Management<br/>

This project consists of two parts:

1. Configure storage infastructure on two Linux OS based (Web and Database) servers.
   
2. Deploy the Web and Database tiers of a basic web solution by installing WordPress and connecting it to a remote MySQL database server.

WordPress is a free and open-source content management system written in **PHP** and paired with **MySQL** or **MariaDB** as its backend Relational Database Management System (RDBMS)

In terms of deploying tiers of a web solution, the **Three-tier Architecture** is generally used when implementing a web or mobile solution.

The **Three-tier Architecture** is a client-server architecture pattern that consists of three seperate layers.

![3 tier](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1ec51fa2-569f-4155-8310-1a24f9e80ef6)

**1. Presentation Layer (PL)**: This is the user interface and communication layer of the application, where the end user interacts with the application. This can for instance be the client server or browser on your machine. Its main purpose is to display information to and collect information from the user.

**2. Business Layer (BL)**: Also known as the Application Layer or the Logic Layer. This is the heart of the application. In this layer, information collected in the presentation tier is processed - sometimes against other information in the data layer - using business logic, a specific set of business rules.

**3. Data Access Layer (DAL)**: Also known as the Management Layeror Database Layer. This is where the information processed by the application is stored and managed. This can be a Database server such as PostgreSQL, MySQL, MariaDB etc. or  a File System Server such as FTP Server, NFS Server etc.

In the execution of this project, our three tier architecture shall be:

1. A Laptop or PC to serve as a Client.
   
2. An EC2 Linux Server to serve as a Web Server that will host our WordPress website.
   
3. An EC2 Linux Server to serve as our Database Server.

### <br>Implementing LVM on Linux Web Server<br/>

To begin our project we need to deploy and configure LVM for our Linux based Web server. We do this by implementing the following steps:

#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Red Hat Linux that will serve as our Web Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our web server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a121b805-be6a-4e01-8be2-698e194d8909)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 9 (HVM).

![Application and OS Images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/86ef4789-366a-4319-b4f2-709f305fa7f1)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Create EBS Volumes<br/>

The next course of action is to create 3 EBS volumes of 10GB each in the same Availability Zone as our EC2 Linux Web Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Click on **"Create Volume"** at the top right hand corner of the **"Volumes"** page.

![Create Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e710712-916e-4543-8b37-b2dc3a357c12)

**iii.** On the **"Volume Settings"** page, we select the **"Volume Type"**, set **"Size"** as 10GB, we select the **"Availability Zone"** of our EC2 Web Server and the we click on **"Create Volume"** at the bottom right corner of the page. 

![Volume Settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc5ac363-e6f6-45af-9e5a-19cf3342a6ed)

**iv.** We repeat **i-iii** above twice to create two more Elastic Block Store (EBS) Volumes.

#### <br>Step 3: Attach EBS Volumes to EC2 Web Server Instance<br/>

After creating the 3 EBS volumes we proceed to attach them to our EC2 Web Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Select the Volume you wish to attach under the Volumes list then right click and select **"Attach Volume"**.

![Attach Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/32ec447b-5d02-4328-8582-528573bb3667)

**iii.** In the **"Attach Volume"** page, under **"Instance"**, we select our EC2 Linux Web Server Instance, under **"Device name"** the name can be changed from **/dev/sdf** through **/dev/sdp** depending on preferences, then we click on **"Attach Volume"** at the bottom right corner of the page.

![Attach Volume 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/36039f97-ae1c-4eb8-85ae-8239b16f5e32)

**iv.** We repeat **i-iii** above twice to attach the remaining two Elastic Block Store (EBS) Volumes to our EC2 Linux Web Server Instance.

#### <br>Step 4: Connect to the Web Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have created and attached our EBS volumes, we must next connect to the web server via an SSH client. This will enable us to subsequently be able to run commands and begin configuration on our web server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Update Webserver and Inspect Attached Block Devices<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![sudo yum update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/67ea0933-1e20-4e1d-aede-a1e29e51fe67)

**ii.** Subsequently, we inspect what block devices are attached to the web server with the following command:

**`$ lsblk`**

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are **/dev/nvme0n1**, **/dev/nvme1n1**, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b312e15c-f0cc-4813-9f6c-e057471f90e9)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage device attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

**iii.** To see all mounts and free space on our Web Server, we run the command below.

**`$ df -h`**

![df -h](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/63742891-1c19-4932-8d8a-d5f4b98c7876)

**iv.** We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

![ls dev folder](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/958f72cb-9144-4a5d-bcc2-f26e8ad15795)

As can be seeen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.

**v.** We can also obtain more important information such as name, serial number, size and LBA format about all the NVMe devices attached to our machine. However, a prerequisite for this is to install the NVMe command line package, **`nvme-cli`** by executing the following command:

**`$ sudo dnf install nvme-cli -y`**

![nvme cli](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/48eb3eb1-8794-4c45-84ec-97364bfd9089)

**vi.** Then we subsequently run the command below to see additional information about the EBS volumes attached to our Linux Machine:

**`$ sudo nvme list`**

![nvme list](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/633d4425-ed2c-4d0f-870f-b88f77e61023)

#### <br>Step 6: Partition Disks and Install lvm2 Package<br/>

**i.** In this step, we proceed to create a single partition on each of the three (3) Disks using the **`gdisk`** utility. We partition **/dev/nvme1n1** by executing the following command: 

**`$ sudo gdisk /dev/nvme1n1`**

As shown in the output image below, we enter **`?`** to list out all the available commands in the gdisk console, we enter the **`p`** command to provide information about available space in hard disk to create a new partition. Subsequently we enter the **`n`** command to create a new partition, we follow through with the prompts and then we enter **`w`** to write the partition table to disk and exit the gdisk console. When the system requests for input with **`Do you want to proceed? (Y/N):`**,  we enter **`y`** to confirm our earlier operation.

![gdisk commands](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a3aa2185-2005-4604-9af3-7ff03a426b5f)

We repeat the same process above to create a single partition on **/dev/nvme2n1** and **/dev/nvme3n1**.

**ii.** Afterwards, we run the **`lsblk`** utility to view the newly configured partition on each of the three (3) disks.

![gdisk partition](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/47df48a1-9c8b-4dbc-b1d9-1d0a8a2ff9a0)

As can be seen in the image above, we have our newly configured partitions on each of the three (3) disks listed as **`nvme1n1p1`**, **`nvme2n1p1`** and **`nvme3n1p1`**

**iii.** The next course of action is to install the **`lvm2`** package using the following command:

**`$ sudo yum install lvm2 -y`**

![lvm2 installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cc915321-9fab-4aaa-9d95-66c5b13dbdcd)

**iv.** Then we run the command below to check for available partitions:

**`$ sudo lvmdiskscan`**

![sudo lvmdiskscan](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/caa1ef9d-5f06-42d2-8cc8-815dbad6037b)

#### <br>Step 7: Create Physical and Logical Volumes<br/>

**i.** For the next step, we use the **`pvcreate`** utility to mark each of our three (3) partitioned disks as physical volumes (PVs) to be used by LVM:

```
$ sudo pvcreate /dev/nvme1n1p1
$ sudo pvcreate /dev/nvme2n1p1
$ sudo pvcreate /dev/nvme3n1p1
```

![creating physical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4071f3c7-0ecf-454d-83e2-7840f50bd764)

**ii.** Then afterwards, we verify that the physical volumes (PVs) have been created by executing the command below:

**`$ sudo pvs`**

![sudo pvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cc0aacd8-ac5f-415f-acfb-9a4d96c290eb)

**iii.** After this, we proceed to use the **`vgcreate`** utility to add all three (3) physical volumes (PVs) to a volume group (VG) that we will be naming **webdata-vg**.

**`$ sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create volume group](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/658856e7-46dc-4644-829a-43ad07cd3296)

**iv.** And then we confirm that our volume group (VG) has been created by successfully executing the following command:

**`$ sudo vgs`**

![sudo vgs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7c5a00c5-b2ce-477c-b8f2-c9cfd963f6ab)

**v.** The next step is to create two (2) logical volumes (LVs) **apps-lv** and **logs-lv** using the **`lvcreate`** utility. For **apps-lv**, we will be using half of the PV size and it will be used to store data for the website while for **logs-lv** we will be using the remaining space left of the PV size and it will be used to store data for logs.

```
$ sudo lvcreate -n apps-lv -L 14G webdata-vg
$ sudo lvcreate -n logs-lv -L 14G webdata-vg
```

![create logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1f653f4e-1f3b-464c-83be-d0526683f467)

**vi.** And then we confirm that our logical volumes have been created by successfully executing the following command:

**`$ sudo lvs`**

![sudo lvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d14ad131-29a7-42b9-950c-e9dda18377e8)

**vii.** Then we verify our entire setup of Volume Group (VG), Physical Volumes (PV) and Logical Volumes (LV) with the following commands:

**`$ sudo vgdisplay -v`**

![sudo vgdisplay](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5168c6ad-e71b-4d9f-868c-ec2728ec95ed)

**`$ sudo lsblk`**

![verify entire setup](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/03a9e4c8-6ac9-486a-a81c-6626f91e5bbc)

**viii.** To complete the process, we use **`mkfs.ext4`** to format the logical volumes (LVs) with **ext4** filesystem.

```
$ sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
$ sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

![format logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/dfc6a0a3-51e4-4187-80c3-acec38b92412)

#### <br>Step 8: Create Directories and Mount on Logical Volumes<br/>

In this step, we need to create the directory to hold our website files and then another directory to store backup of log data after which we will respectively mount these directories on the created logical volumes **apps-lv** and **logs-lv**.

**i.** We use the following command to create the **/var/www/html** directory to store our website application files:

**`$ sudo mkdir -p /var/www/html`**

**ii.** We use the command below to create the **/home/recovery/logs** directory to store backup of log data:

**`$ sudo mkdir -p /home/recovery/logs`**

**iii.** Then we execute the following command to mount **/var/www/html/** on **apps-lv** logical volume:

**`$ sudo mount /dev/webdata-vg/apps-lv /var/www/html/`**

**iv.** **/var/log** is the default directory where Linux stores all log files. This is the directory that we need to mount on our **logs-lv** volume. However, mounting this directory will delete all the files contained in it so before we carry out this action, we need to use the **`rsync`** utility to backup all the files in the log directory **/var/log** into the **/home/recovery/logs** directory we created. We do this by executing the command below:

**`$ sudo rsync -av /var/log/. /home/recovery/logs/`**

![backup log files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1c8fabac-607a-49fd-a538-8abdb294d780)

**v.** Then we enter the command below to mount **/var/log** on **apps-lv** logical volume:

**`$ sudo mount /dev/webdata-vg/logs-lv /var/log`**

**vi.** Afterwards, we restore the log files back into the **/var/log** directory.

**`$ sudo rsync -av /home/recovery/logs/. /var/log`**

![restore log files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/da7a0e22-0a8f-4729-8621-f9665e9d6167)

**vii.** The next step is to use the universally unique identifier (UUID) of the device to update the **/etc/fstab** file so that the mount configuration will persist after the restart of the server. We check the UUID of the device by entering the command below:

**`$ sudo blkid`**

![uuid](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e8ce173d-4db6-4f0c-b7bb-b0b56349af4d)

**viii.** We copy the UUID as shown in the above image and we open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**ix.** We paste in the copied UUID whilst removing the leading and ending quotes and update the **/etc/fstab** file as shown in the image below:

![mounts for wordpress server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc378217-afc5-41ea-ac3a-306d2390f186)

**x.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**xi.** We test our mount configuration with the following command:

**`$ sudo mount -a`**

**xii.** Then we reload the daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**xiii.** To complete our configuration process, we verify our entire setup by executing the following command:

**`$ df -h`**

![df -h final output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5ca686a1-58f5-47e7-a41f-cffcab726981)

The output must look like what we have in the image above.

### <br>Implementing LVM on Linux Database Server<br/>

The next phase of this project involves preparing the Database Server. We shall launch a second RedHat EC2 instance that will have a role as our **‘DB Server’**. We shall be repeating the same steps as for the Web Server, but instead of **`apps-lv`** we will create **`db-lv`** and mount it to **`/db`** directory instead of **`/var/www/html/`**.

#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Red Hat Linux that will serve as our Database Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our Database server.

![Name and tags DB server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4a139fcb-1ace-478f-abb9-7e27c3e48704)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 9 (HVM).

![Application and OS Images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/86ef4789-366a-4319-b4f2-709f305fa7f1)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Create EBS Volumes<br/>

The next course of action is to create 3 EBS volumes of 10GB each in the same Availability Zone as our EC2 Linux Database Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Click on **"Create Volume"** at the top right hand corner of the **"Volumes"** page.

![Create Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e710712-916e-4543-8b37-b2dc3a357c12)

**iii.** On the **"Volume Settings"** page, we select the **"Volume Type"**, set **"Size"** as 10GB, we select the **"Availability Zone"** of our EC2 Database Server and the we click on **"Create Volume"** at the bottom right corner of the page. 

![Volume Settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc5ac363-e6f6-45af-9e5a-19cf3342a6ed)

**iv.** We repeat **i-iii** above twice to create two more Elastic Block Store (EBS) Volumes.

#### <br>Step 3: Attach EBS Volumes to EC2 Database Server Instance<br/>

After creating the 3 EBS volumes we proceed to attach them to our EC2 Database Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Select the Volume you wish to attach under the Volumes list then right click and select **"Attach Volume"**.

![attach volume db server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/391a7639-23ef-41f4-8689-4eba74456031)

**iii.** In the **"Attach Volume"** page, under **"Instance"**, we select our EC2 Linux Database Server Instance, under **"Device name"** the name can be changed from **/dev/sdf** through **/dev/sdp** depending on preferences, then we click on **"Attach Volume"** at the bottom right corner of the page.

![attach volume 2 DB server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6156c4d1-065d-4304-99b0-71b8f68f40ce)

**iv.** We repeat **i-iii** above twice to attach the remaining two Elastic Block Store (EBS) Volumes to our EC2 Linux Database Server Instance.

#### <br>Step 4: Connect to the Database Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have created and attached our EBS volumes, we must next connect to the Database server via an SSH client. This will enable us to subsequently be able to run commands and begin configuration on our Database server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Update Database Server and Inspect Attached Block Devices<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![sudo yum update db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6d9eed39-54cb-4243-b7bb-2a7bb08727ca)

**ii.** Subsequently, we inspect what block devices are attached to the Database server with the following command:

**`$ lsblk`**

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are **/dev/nvme0n1**, **/dev/nvme1n1**, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/91272424-2c1f-47c5-9041-4f89a76e0ea1)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage device attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

**iii.** To see all mounts and free space on our Database Server, we run the command below.

**`$ df -h`**

![df-h db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2302ce7d-a7bd-45cd-b600-95cc1804436c)

**iv.** We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

![ls dev folder db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/55cde77f-f5a7-4581-be52-1c3b61ce979f)

As can be seen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.

**v.** We can also obtain more important information such as name, serial number, size and LBA format about all the NVMe devices attached to our machine. However, a prerequisite for this is to install the NVMe command line package, **`nvme-cli`** by executing the following command:

**`$ sudo dnf install nvme-cli -y`**

![sudo dnf install db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/874557d2-cd22-4753-905f-41d7780cc608)

**vi.** Then we subsequently run the command below to see additional information about the EBS volumes attached to our Linux Machine:

**`$ sudo nvme list`**

![sudo nvme list db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2a6be523-3dab-4d1f-848b-7a78e200b5fe)

#### <br>Step 6: Partition Disks and Install lvm2 Package<br/>

**i.** In this step, we proceed to create a single partition on each of the three (3) Disks using the **`gdisk`** utility. We partition **/dev/nvme1n1** by executing the following command: 

**`$ sudo gdisk /dev/nvme1n1`**

As shown in the output image below, we enter **`?`** to list out all the available commands in the gdisk console, we enter the **`p`** command to provide information about available space in hard disk to create a new partition. Subsequently we enter the **`n`** command to create a new partition, we follow through with the prompts and then we enter **`w`** to write the partition table to disk and exit the gdisk console. When the system requests for input with **`Do you want to proceed? (Y/N):`**,  we enter **`y`** to confirm our earlier operation.

![gdisk commands db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/161cbb61-3082-4c8b-98c6-e4394c1af797)

We repeat the same process above to create a single partition on **/dev/nvme2n1** and **/dev/nvme3n1**.

**ii.** Afterwards, we run the **`lsblk`** utility to view the newly configured partition on each of the three (3) disks.

![gdisk partition db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0c080c79-3d60-4bfc-bf58-14702b18224f)

As can be seen in the image above, we have our newly configured partitions on each of the three (3) disks listed as **`nvme1n1p1`**, **`nvme2n1p1`** and **`nvme3n1p1`**

**iii.** The next course of action is to install the **`lvm2`** package using the following command:

**`$ sudo yum install lvm2 -y`**

![lvm2 installation db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b78d8cec-9596-48a9-903e-1d90269618b2)

**iv.** Then we run the command below to check for available partitions:

**`$ sudo lvmdiskscan`**

![sudo lvmdiskscan db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/578ac557-94ef-4091-ba56-5c6f1061557c)

#### <br>Step 7: Create Physical and Logical Volumes<br/>

**i.** For the next step, we use the **`pvcreate`** utility to mark each of our three (3) partitioned disks as physical volumes (PVs) to be used by LVM:

**`$ sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![creating physical volumes db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/846dab7e-6029-46b9-8c24-181c75609f91)

**ii.** Then afterwards, we verify that the physical volumes (PVs) have been created by executing the command below:

**`$ sudo pvs`**

![sudo pvs db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eb86fd81-9823-4244-aa23-0776c7f72a56)

**iii.** After this, we proceed to use the **`vgcreate`** utility to add all three (3) physical volumes (PVs) to a volume group (VG) that we will be naming **dbdata-vg**

**`$ sudo vgcreate dbdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create volume group db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/55905165-0d23-457c-9ef1-01aeb1c3e50c)

**iv.** And then we confirm that our volume group (VG) has been created by successfully executing the following command:

**`$ sudo vgs`**

![sudo vgs db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5214ffc4-682a-4689-a772-ee96158c0c83)

**v.** The next step is to create two (2) logical volumes (LVs) **db-lv** and **logs-lv** using the **`lvcreate`** utility. For **db-lv**, we will be using half of the PV size and it will be used for database storage while for **logs-lv** we will be using the remaining space left of the PV size and it will be used to store data for logs.

```
$ sudo lvcreate -n db-lv -L 14G dbdata-vg
$ sudo lvcreate -n logs-lv -L 14G dbdata-vg
```

![create logical volumes db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/131a20e5-2b3b-407b-b59c-537e1888642d)

**vi.** And then we confirm that our logical volumes have been created by successfully executing the following command:

**`$ sudo lvs`**

![sudo lvs db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/56ba9b5e-4487-465e-92bd-68d3251524e0)

**vii.** Then we verify our entire setup of Volume Group (VG), Physical Volumes (PV) and Logical Volumes (LV) with the following commands:

**`$ sudo vgdisplay -v`**

![sudo vgdisplay db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/55f8718d-2de8-44c2-a57d-afdf68310a9d)

**`$ sudo lsblk`**

![verify entire setup db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/257574ab-ca72-4cf9-a17a-82f252bedd05)

**viii.** To complete the process, we use **`mkfs.ext4`** to format the logical volumes (LVs) with **ext4** filesystem.

**`$ sudo mkfs -t ext4 /dev/dbdata-vg/db-lv && sudo mkfs -t ext4 /dev/dbdata-vg/logs-lv`**

![format logical volumes db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/33c81e0f-b165-4d2a-8d8f-05da70479146)

#### <br>Step 8: Create Directories and Mount on Logical Volumes<br/>

In this step, we need to create the directory to hold our database files and then another directory to store backup of log data after which we will respectively mount these directories on the created logical volumes **db-lv** and **logs-lv**.

**i.** We use the following command to create the **/db** directory to store our database files:

**`$ sudo mkdir -p /db`**

**ii.** We use the command below to create the **/home/recovery/logs** directory to store backup of log data:

**`$ sudo mkdir -p /home/recovery/logs`**

**iii.** Then we execute the following command to mount **/db** on **db-lv** logical volume:

**`$ sudo mount /dev/dbdata-vg/db-lv /db`**

**iv.** **/var/log** is the default directory where Linux stores all log files. This is the directory that we need to mount on our **logs-lv** volume. However, mounting this directory will delete all the files contained in it so before we carry out this action, we need to use the **`rsync`** utility to backup all the files in the log directory **/var/log** into the **/home/recovery/logs** directory we created. We do this by executing the command below:

**`$ sudo rsync -av /var/log/. /home/recovery/logs/`**

![backup log files db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3431073b-23ba-4c14-9624-e92d602b6b29)

**v.** Then we enter the command below to mount **/var/log** on **logs-lv** logical volume:

**`$ sudo mount /dev/dbdata-vg/logs-lv /var/log`**

**vi.** Afterwards, we restore the log files back into the **/var/log** directory.

**`$ sudo rsync -av /home/recovery/logs/. /var/log`**

![restore log files db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/39277e2e-f067-4c1b-bc61-42aade92704f)

**vii.** The next step is to use the universally unique identifier (UUID) of the device to update the **/etc/fstab** file so that the mount configuration will persist after the restart of the server. We check the UUID of the device by entering the command below:

**`$ sudo blkid`**

![uuid db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/10b6f02e-7d5d-405e-84c7-c9533cbd694e)

**viii.** We copy the UUID as shown in the above image and we open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**ix.** We paste in the copied UUID whilst removing the leading and ending quotes and update the **/etc/fstab** file as shown in the image below:

![Mounts for Database Server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/dbac1c4c-5178-4824-b00d-82418309dd4c)

**x.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**xi.** We test our mount configuration with the following command:

**`$ sudo mount -a`**

**xii.** Then we reload the daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**xiii.** To complete our configuration process, we verify our entire setup by executing the following command:

**`$ df -h`**

![df -h final output db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/00cc610d-137e-4eab-b98f-1f6ef2df09e0)

The output must look like what we have in the image above.

### <br>WordPress Installation and Configuration to Use MySQL Database<br/>

To deploy WordPress, we will need to first of all install Apache Web Server software on our Web Server machine and then install MySQL on our Database machine after which we will proceed to configure our database to work with WordPress. We do this with the following steps:

#### <br>Step 1: Install WordPress on EC2 Web Server Instance<br/>
 
**i.** We begin this part of our project by updating the repository if we have not already done so:

**`$ sudo yum -y update`**

![sudo yum update 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/648f1b2d-cad9-49b0-8d37-e78b7a58f16f)

**ii.** Then we proceed to install wget, Apache and its dependencies by executing the following command:

**`$ sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`**

![install wget apache dependencies](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/798aea09-ab60-41fc-bc2e-b1e074f76bb8)

**iii.** After successfully completing the installation, we start the apache service with the commands below:

```
$ sudo systemctl enable httpd
$ sudo systemctl start httpd
```

![enable and start apache httpd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9be56846-0af1-4c00-baa5-5efd984569fd)

**iv.** The next course of action is to install PHP and its dependencies with the following set of commands:

```
$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
$ sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
$ sudo yum module list php
$ sudo yum module reset php
$ sudo yum module enable php:remi-7.4
$ sudo yum install php php-opcache php-gd php-curl php-mysqlnd
$ sudo systemctl start php-fpm
$ sudo systemctl enable php-fpm
$ sudo setsebool -P httpd_execmem 1
```

#### BLOCKER❗

+ After concatenating and running the nine (9) commands above, the first command ran and the installation of **epel-release-latest-8.noarch.rpm** was completed successfully. However, as shown in the image below, we encountered an error when the second command was being executed:

![blocker 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/389f0ef7-d37a-49ed-a52b-065006f12cfa)

+ As shown in the error message in the above image, **system-release(releasever) = 8 needed by remi-release-8.8-1.el8.remi.noarch** is basically saying that the dependency we were trying to install **remi-release-8.rpm** encountered an error because our Operating system version is **Red Hat Enterprise Linux 9** whereas it requires **Red Hat Enterprise Linux 8** as referenced by **system-release(releasever) = 8**

+ We surmised that we had two options to resolve this issue:

**1.** We attempt to downgrade our Operating System version to Red Hat Enterprise Linux 8.

**2.** We install the latest version of the remi-release dependency that is compatible with Red Hat Enterprise Linux 9.

+ remi-release RPM repository is a free and stable YUM repository mainly for the PHP stack. It contains packages for the latest versions of PHP.

+ We decide to go with option **2** by executing the following command: 

**`$ sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm`**

+ However, we encountered another error as shown in the image below:

![remi-release error1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c21f8966-2f1e-4a8a-9131-72a29832fa8b)

+ **epel-release = 9 needed by remi-release-9.2-1.el9.remi.noarch** indicates that the version 9 of the remi-release dependency requires an equivalent version 9 of epel-release and we are getting the error because the command that successfully ran in **iv** above **`$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpmdependency`** installed version 8 of epel-release instead.

+ epel release is a repository that is well known for bringing a high-quality set of additional packages for Red Hat Enterprise Linux and other Linux flavours.

+ So we decide to install the latest version 9 of epel-release by executing the following command:

**`$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm`**

![epel-release successful installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c9f114be-046f-4966-96ed-830192cff584)

+ As can be seen in the above image, the installation was successful.

+ Now that the compatible epel-release dependency has been installed, we proceed to install the latest version of the remi-release dependency with the following command:

**`$ sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm`**

+ But as shown in the image below, we again encounter an error albeit a different one this time.

![remi-release killed](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5b4a1b39-9210-48f2-8f97-39ba34a191fe)

+ As can be seen in the image above, the kernel sent a SIGKILL signal to kill off our process. So we decide to investigate the relevant logs with the use of the **`dmesg`** utility:

**`$ sudo dmesg | tail -7`**

![dmesg utility](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2e1b51f8-1ae3-42a2-889f-fb36ea50e5d6)

+ As shown in the output image above, OOM sent a signal to kill the process employed by **`yum`** which is the tool we were using to install the remi-release dependency. The OOM Killer or Out Of Memory Killer is a process that the linux kernel employs when the system is critically low on memory. This situation occurs because the linux kernel has over allocated memory to its processes.
As can be seen in the above image, **anon-rss:473708kB** indicates that **`yum`** was using 473.708MB of RAM.

+ To investigate further we execute the **`top`** command which is used for memory monitoring in Linux.

![top memory monitoring command](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/89761542-ab0b-418f-a7b6-42bb3fe93103)

+ From the above image we can see that we have 419MB of free memory and 425.3MB of total available memory. 
We also use the **`free -m`** command which gives us information gives us information about used and unused memory usage.

![free -m memory command](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca3a2b2d-ffb1-40da-971c-c32c9563327c)

+ As seen in the above image, the command confirms that we have 425MB of free memory. For context, our EC2 Red Hat Linux instance has a total of 1GB RAM. So we can see the reason OOM Killer swung in to action as **`yum`** was using 473.708MB of RAM which is more than the free or available space as indicated in our memory diagnostics.

+ Considering the situation we were presented with, we surmised that we had two options to resolve this issue:

**1.** We add additional RAM to our Red Hat Enterprise Linux EC2 instance.

**2.** We create and enable **SWAP** on our Red Hat Enterprise Linux EC2 instance.

+ After discovering that the free tier AWS instance is limited to 1GB of RAM and adding additional RAM will incur expensive costs, we decided to go with option **2**. **SWAP**, also known as virtual RAM, is used in Linux to support storing data in hard disk memory when when a system is running out of physical memory (RAM).

+ To begin the process of creating **SWAP**, we check for free disk space on our default EBS volume by running the following command:

**`$ sudo df -h`**

![check for free space](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3256c75a-a4b1-4a9f-b602-0655700cbe3a)

+ Next, we create a 1GB swap file with the use of the **`dd`** command:

**`$ sudo dd if=/dev/zero of=/mnt/swapfile bs=1024 count=1048k`**

![create swap file sudo dd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f0751ee6-1a5f-4550-96be-293adfaf915b)

+ After this, we create a swap partition by executing the following command:

**`$ sudo mkswap /mnt/swapfile`**

![create swap partition](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/db17a1c5-5014-4cf1-b13f-e15e84af3fe0)

+ Then we enable **SWAP** and check its status with the commands below:

```
sudo swapon /mnt/swapfile
swapon -s
```

![enable swap](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2c3e70a0-4ce0-4c94-b995-fa63f1e07af6)

+ Next, we need to set up the Swap partition to automatically activate after rebooting the system by updating **/etc/fstab**

**`$ sudo vi /etc/fstab`**

![mount for swap partition](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d575846e-5a96-4427-997e-e572b7bb21e6)

+ To complete the process  we need to configure Swappiness. Swappiness is the priority of using Swap of Linux system. When the amount of free RAM remaining equals the value of Swappiness that is set as a percentage, the Linux server will switch to use. For instance, if our server has only 10% free RAM and Swappiness is set to 10, the server will switch to using Swap. To proceed we check the current swappiness parameter of our Server by running the command below:

**`$ cat /proc/sys/vm/swappiness`**

![current swappiness](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/68437cad-53ed-4c1e-a5f1-3668f870a59d)

+ The output image above shows that Linux switches to using Swap when the amount of physical RAM reaches 30%. We change this parameter to 10 by executing the following command:

**`$ sudo sysctl vm.swappiness=10`**

![change swappiness](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d8f766f9-d74c-4c78-9f75-e5a580aedded)

+ As shown in the image above, we are able to confirm that swappiness has been changed to 10. We also verify that our whole **SWAP** setup is enabled by running the commands below:
  
```
$ cat /proc/swaps
$ free
```

![swap complete](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cf9966f6-f1d8-47e2-9800-ed406070cd07)

+ With the creation and configuration of **SWAP** complete as refelected in the above image, we proceed to install the compatible version of the remi-release dependency with the following command:

**`$ sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm`**

![remi dependency successful](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3ccc7694-f1b2-4858-9790-758c01e67a62)

+ As shown in the image above, the installation was successful and we were able to overcome our **BLOCKER**

+ We proceed to run the remainder of the commands from **iv** without any issues:

```
$ sudo yum module list php
$ sudo yum module reset php
$ sudo yum module enable php:remi-7.4
$ sudo yum install php php-opcache php-gd php-curl php-mysqlnd
$ sudo systemctl start php-fpm
$ sudo systemctl enable php-fpm
$ sudo setsebool -P httpd_execmem 1
```

![installed remaining commands](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b9ddaaa8-8511-4cd7-9691-4b830559c3f4)

<br> <br/>

**v.** After completing the installations, we run the following command to restart Apache:

**`$ sudo systemctl restart httpd`**

**vi.** Next, we create a directory for Wordpress, we enter the created directory and we download WordPress in its compressed format. Then we decompress WordPress and we copy the wp-config-sample.php file to wp-config.php from where WordPress derives its base configuration. Afterwards we copy wordpress directory to **/var/www/html/** We implement these by executing the following set of commands:

```
$ mkdir wordpress
$ cd wordpress
$ sudo wget http://wordpress.org/latest.tar.gz
$ sudo tar xzvf latest.tar.gz
$ sudo rm -rf latest.tar.gz
$ sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
$ sudo cp -R wordpress /var/www/html/
```

![worpress installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4bc24590-6987-4903-9ade-18043f95e37e)

**vi.** We complete the WordPress installation process by configuring SELinux policies with the set of commands below:

```
 $ sudo chown -R apache:apache /var/www/html/wordpress
 $ sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 $ sudo setsebool -P httpd_can_network_connect=1
```

![configure selinux](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/caa6f297-0f2b-471b-ac5f-e267ee09cf3f)

#### <br>Step 2: Install MySQL on EC2 Database Server Instance<br/>

**i.** We begin this step by updating the repository if we have not already done so:

**`$ sudo yum -y update`**

![sudo yum update db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eb1ab85b-8f10-4a06-aac1-3eb5c75953d8)

**ii.** Next, we execute the following command to install MySQL:

**`$ sudo yum install mysql-server`**

![MySQL server installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/096dc9e5-5f62-49ef-985d-7979fd287857)

**iii.** After installation, we check if the service is up and running with the command below:

**`$ sudo systemctl status mysqld`**

![mysql inactive](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2f33e693-a975-4468-89df-9f70d85646d4)

**iv.** The above image shows that MySQL server is currently inactive. So we execute the following commands to restart the service and enable it so that it will be running even after a system reboot:

**`$ sudo systemctl restart mysqld && sudo systemctl enable mysqld`**

![mysql active](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b4393fc1-2819-4fb6-82de-9c5aee6e94f0)

As shown in the above image, after restarting and enabling MySQL we reconfirm the service status and we can see that the service is active and running.

#### <br>Step 3: Configure MySQL Database to Work with WordPress<br/>

We configure MySQL to work with WordPress by first of all accessing the MySQL console and creating the WordPress Database. Next, we create a user and assign the user password. Then we grant the database privileges to the newly created user.  After this, we flush the privileges. The IP address used when creating the user and flushing the user privileges must be the **`<Web-Server-Private-IP-Address>`**. We conclude by viewing the database to confirm our configuration and then we exit. We implement all of these by executing the following:

```
$ sudo mysql
mysql> CREATE DATABASE wordpress;
mysql> CREATE USER `qbuser`@`172.31.22.239` IDENTIFIED BY 'password123';
mysql> GRANT ALL ON wordpress.* TO `qbuser`@`172.31.22.239`;
mysql> FLUSH PRIVILEGES;
mysql> SHOW DATABASES;
mysql> exit
```

![configure mysql database](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/48a6a3ae-73d2-4281-89aa-965f40f503f0)

#### <br>Step 4: Open Port 3306 on EC2 Database Instance<br/>

**i.** In this step, we need to ensure we open MySQL port **3306** on DB Server EC2. For extra security, we will allow access to the DB server **ONLY** from our Web Server’s private IP address, so in the Inbound Rule configuration source will be specified as **/32**. We do this by following the steps below:

**ii.** In the AWS console navigation pane, we choose **Instances**.

![AWS navigation pane](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eb88a0b7-15c4-43e3-8187-e9f61aa26fbe)

**iii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![AWS Instance summary DB server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9c9054ad-12c2-4613-b261-e388b80778e2)

**iv.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**v.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**vi.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **3306** 

+ In the space with the magnifying glass under **Source**, we enter our Web Server’s private IP address **172.31.22.239/32**

+ Click on **Save rules** at the bottom right corner of the page.

![save rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc836d6a-f277-4ded-b4e8-9319a78bae23)


#### <br>Step 5: Configure WordPress to connect to Remote Database<br/>

**i.** Next, on the Web Server, we execute the following command to install MySQL client: 

**`$ sudo yum install mysql`**

![install mysql client](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9b8721f3-0083-4175-9689-a495fd1c293a)

**ii.** And then we test that we can connect from our Web Server to our Database server using MySQL client:

**`$ sudo mysql -u qbuser -p -h 172.31.22.52`**

![connect to mysql from web server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a6b72332-fa6a-4229-a5c3-caed9f461c3f)

As shown in the output image above, we connected to the DB Server successfully. Note that the user we used in connecting is the same user we created in Mysql server on the Database (DB) Server. It should also be noted that the IP address we used in connecting is the **`<DB-Server-Private-IP-address>`**.

**iii.** We verify that we can successfully execute **`mysql> SHOW DATABASES;`** command and see a list of existing databases.

![show databases](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8b562b26-2b09-4060-a667-daf10bb75440)

**iv.** The next thing we need to do is change the requisite permissions and configuration so that Apache can use WordPress.
To implement this, we need to create a configuration file for wordpress in order to point client requests to the WordPress directory.

**`$ sudo vi /etc/httpd/conf.d/wordpress.conf`**

```
<VirtualHost *:80>
ServerAdmin qbuser@172.31.22.52
DocumentRoot /var/www/html/wordpress

<Directory "/var/www/html/wordpress">
Options Indexes FollowSymLinks
AllowOverride all
Require all granted
</Directory>

ErrorLog /var/log/httpd/wordpress_error.log
CustomLog /var/log/httpd/wordpress_access.log common
</VirtualHost>
```
Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

![wordpress configuration file](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2fc12da8-a30e-4c20-8da0-bc142aadd3ad)

**v.** To apply the changes, we restart Apache with the following command:

**`$ sudo systemctl restart httpd`**

![system restart httpd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/64c45637-3c31-4aa4-a5dd-bd1d307afee3)

**vi.** Next, we edit the **wp-confg** file with the command below:

**`$ sudo vi /var/www/html/wordpress/wp-config.php`**

And we add the following configuration:

```
define('DB_NAME', 'wordpress');
define('DB_USER', 'qbuser');
define('DB_PASSWORD', 'password123');
define('DB_HOST', '172.31.22.52');
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
```

Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

![configure wp-config](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d61b08f9-5608-4e1d-8436-43aad6b33893)

As indicated in the above image, we modify the values to correspond to our database name, database user, password and DB host which will be our **`<DB-Server-Private-IP-Address>`**

**vii.**  Red Hat Enterprise Linux usually comes with SELinux enabled. This can be an issue, especially during the installation of web applications. We therefore need to configure the right SELinux context to the **/var/www/html/wordpress** directory.

**`$ sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/wordpress(/.*)?"`**

**viii.** And for the changes to take effect, we execute the following command:

**`$ sudo restorecon -Rv /var/www/html/wordpress`**

![semange](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fad1ac08-3c70-4994-a428-43e21c8366f3)

After executing the two commands as shown in the image above, we then reboot our instance.

#### <br>Step 6: Open Port 80 on EC2 Web Server Instance<br/>

**The next step is to open TCP Port 80 on our Web Server machine. Port 80 is the defualt port used by web browsers to access web pages. So we implement this by adding a rule to our EC2 configuration to allow http traffic via port 80. The steps are listed below:

**i.** Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).

**ii.** In the navigation pane, choose **Instances**.

![AWS navigation pane](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/eb88a0b7-15c4-43e3-8187-e9f61aa26fbe)

**iii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![web server instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e6a2cd33-8550-4998-b08a-2309a9aea800)

**iv.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**v.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**vi.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Type**, choose **HTTP**. 

+ In the space with the magnifying glass under **Source**, we leave it at **Custom** and select **0.0.0.0/0** Selecting the **Source** setting as **0.0.0.0/0** means we can access our server from any IP address. i.e. both locally and from the internet.

+ Click on **Save rules** at the bottom right corner of the page.

![save rules port 80](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3aacbbe5-d7e1-4cfa-8226-50584756e108)

#### <br>Step 7: Complete WordPress Installation on the Browser<br/>

**i.** The last step is to complete the installation from a web browser. We launch our browser and enter our web server’s Public IP address using the following syntax:

**`http://<Web-Server-Public-IP-Address>/`**

**ii.** We enter our web server’s IP address into our browser as shown below:

**`http://13.53.122.139`**

**iii.** The page we see is as shown in the image below, so we select our preferred installation language and we click on the **Continue** button at the buttom of the page. 

![browser output webserver](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3aeef39a-4187-4abd-9712-a5f13a0485ca)

**iv.** In the next step, we fill in our site's details, then we scroll down the page and click on **Install WordPress**.

![wordpress welcome page](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5858b437-dbb0-4f21-ac5f-899690bf442e)

**v.** The next page we see is the installation success page which tells us WordPress has been installed. To log in, we click the **Login** button at the bottom of the page.

![Wordpress successful installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e6a2d7ba-a982-4a42-9ccf-02cab0406ec4)

**vi.** On the login page, we provide the username and password that we provided when filling in our site's details and then we click **Log In**.

![Word press log in](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/32fc91a8-977c-423e-8c04-209045c8330e)

**vii.** Finally, we get to see the WordPress Dashboard as shown in the image below:

![WordPress complete](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5ee0be44-c3f5-4dc7-acf1-091746d93596)

### <br>CONCLUSION<br/>

We have been able to successfully complete this project. The goal of our project was to to build and manage a scalable WordPress website using AWS EC2 and LVM (Logical Volume Management) storage. We began our project implementation by provisioning an EC2 Instance of Red Hat Linux to serve as our Web Server. And then we deployed and configured LVM for our Linux based Web server. This involved creating and attaching 3 EBS volumes, partitioning disks and then creating and configuring physical and logical volumes. Next, we created the relevant directories which we mounted on the logical volumes. We subsequently launched a second RedHat EC2 instance that was given a role as our Database (DB) Server. We then repeated the same steps as for the Web Server, but with some important differences such as creating **db-lv** logical volume instead of **apps-lv** that we created on the web server and mounting it to **/db** directory instead of **/var/www/html/**. The next thing we did was to install WordPress on our Web Server and configure it to use MySQL Database. This required us to install the Apache web server and its dependencies and it was at this point that we encountered some **Blockers**. Fortunately, we were able to resolve the issues and then we proceeded successfully to install MySQL on our EC2 Database Server instance and then create and configure MySQL database to work with WordPress. Next, on our Web Server, we configured WordPress to be able to connect to our remote database on the DB server. We concluded our project by opening the requisite ports on both our Web and Database servers and then we accessed WordPress through the browser with our Web server's public IP address and proceeded to complete the WordPress installation process. Thank you.


