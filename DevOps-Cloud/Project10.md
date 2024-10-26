## DOCUMENTATION FOR DEVOPS TOOLING WEBSITE SOLUTION

In the [previous project](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project9.md), we implemented a WordPress solution that is ready to be filled with content and can be used as a full fledged website or blog. Subsequently, in this project, we will be adding some more value to our solutions by implementing a comprehensive DevOps tooling website solution that will integrate various tools and technologies to create a unified platform that enhances collaboration, automation and efficiency for software development and operations teams.

### <br>Introduction to DevOps Tooling Website Solution<br/>

The goal of this project is to build a tooling website solution which will enable easy access to DevOps tools within the corporate organisation. The solution we will implement will consist of the following components:

**1.** **Infrastructure**: Amazon Web Services (AWS)

**2.** **Web Server**: Red Hat Enterprise Linux 8

**3.** **Database Server**: Ubuntu 20.04 + MySQL

**4.** **Storage Server**: Enterprise Linux 8 + NFS Server

**5.** **Programming Language**: PHP

**6.** **Code Repository**: GitHub

Based on the infrastructure to be used and the goal of this project, it shall consist of three parts:

**1.** Configure Network File Server (NFS) Server with storage infastructure.

**2.** Configure Backend Database Server.

**3.** Configure Web Servers.

As shown in the diagram below, we shall implement a solution where three stateless Web Servers will share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for the Web Servers it would look like a local file system from where they can serve the same files.

![3 TIER Tooling-Website-Infrastructure](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/61a271c9-4efd-45ff-a71e-94e62340faf8)

### <br>Configure Network File Server (NFS) Server with storage infastructure<br/>

To begin our project we need to deploy and configure LVM for our Linux based NFS server. We do this by implementing the following steps:


#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Red Hat Linux that will serve as our NFS Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/04cf9f94-d41c-44ca-a108-346911a7a646)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 8 (HVM).

![application and OS images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2beaaf89-c746-4ed5-a434-f0989bfb3db1)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Create EBS Volumes<br/>

The next course of action is to create 3 EBS volumes of 10GB each in the same Availability Zone as our EC2 Linux NFS Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Click on **"Create Volume"** at the top right hand corner of the **"Volumes"** page.

![Create Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e710712-916e-4543-8b37-b2dc3a357c12)

**iii.** On the **"Volume Settings"** page, we select the **"Volume Type"**, set **"Size"** as 10GB, we select the **"Availability Zone"** of our EC2 Web Server and the we click on **"Create Volume"** at the bottom right corner of the page. 

![Volume Settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc5ac363-e6f6-45af-9e5a-19cf3342a6ed)

**iv.** We repeat **i-iii** above twice to create two more Elastic Block Store (EBS) Volumes.

#### <br>Step 3: Attach EBS Volumes to EC2 NFS Server Instance<br/>

After creating the 3 EBS volumes we proceed to attach them to our EC2 NFS Server instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)

**i.** We open the AWS console and click on **"EC2"**, then we scroll down in the navigation pane and click on **"Volumes"** under **"Elastic Block Store"**.

![volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2befe33d-7849-4677-9252-fd7d4af93b26)

**ii.** Select the Volume you wish to attach under the Volumes list then right click and select **"Attach Volume"**.

![Attach Volume](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/32ec447b-5d02-4328-8582-528573bb3667)

**iii.** In the **"Attach Volume"** page, under **"Instance"**, we select our EC2 Linux NFS Server Instance, under **"Device name"** the name can be changed from **/dev/sdf** through **/dev/sdp** depending on preferences, then we click on **"Attach Volume"** at the bottom right corner of the page.

![Attach Volume 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/36039f97-ae1c-4eb8-85ae-8239b16f5e32)

**iv.** We repeat **i-iii** above twice to attach the remaining two Elastic Block Store (EBS) Volumes to our EC2 Linux NFS Server Instance.

#### <br>Step 4: Connect to the NFS Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have created and attached our EBS volumes, we must next connect to the NFS server via an SSH client. This will enable us to subsequently be able to run commands and begin configuration on our NFS server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Update NFS Server and Inspect Attached Block Devices<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![Sudo yum update -y](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c5f2681a-141f-4dc7-8d8a-ac527184a9e3)

**ii.** Subsequently, we inspect what block devices are attached to the web server with the following command:

**`$ lsblk`**

As can be seen in the image below, our EBS volumes are shown using the **`nvme`** naming convention rather than **`xvdf`**. This is because our block devices are connected through an NVMe port which uses the nvme driver on Linux. It should also be noted that EBS volumes are typically exposed as NVMe block devices on instances built on the Nitro System. The device names are **/dev/nvme0n1**, **/dev/nvme1n1**, and so on. The Nitro System is a collection of hardware and software components built by AWS that enable high performance, high availability, and high security.

![lsblk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e3f43630-958c-4c6d-b4a5-eebf08ab4e1c)

The **`lsblk`** command reveals that **/dev/nvme0n1** is the default storage device attached to our EC2 instance and has four partitions **/dev/nvme0n1p1-4** with **/dev/nvme0n1p4** mounted as the root device. The block devices we created are listed as **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** and as can be seen, they have no mount points because they are not yet mounted.

**iii.** To see all mounts and free space on our Web Server, we run the command below.

**`$ df -h`**

![df -h](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2cfe8778-107c-48b8-a68e-878eed860816)

**iv.** We also proceed to check the **/dev/** directory with the following command: 

**`$ ls /dev/`**

![ls dev](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/079bef77-e12b-49db-b0a5-58c308771c2b)

As can be seeen in the above image, the executed command lists all Linux devices and we can see that our attached block devices  **/dev/nvme1n1**, **/dev/nvme2n1** and **/dev/nvme3n1** are listed.

#### <br>Step 6: Partition Disks and Install lvm2 Package<br/>

**i.** In this step, we proceed to create a single partition on each of the three (3) Disks using the **`gdisk`** utility. We partition **/dev/nvme1n1** by executing the following command: 

**`$ sudo gdisk /dev/nvme1n1`**

As shown in the output image below, we enter **`?`** to list out all the available commands in the gdisk console, we enter the **`p`** command to provide information about available space in hard disk to create a new partition. Subsequently we enter the **`n`** command to create a new partition, we follow through with the prompts and then we enter **`w`** to write the partition table to disk and exit the gdisk console. When the system requests for input with **`Do you want to proceed? (Y/N):`**,  we enter **`y`** to confirm our earlier operation.

![sudo gdisk](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a5b37c83-2f90-4748-a090-abd71d005a33)

We repeat the same process above to create a single partition on **/dev/nvme2n1** and **/dev/nvme3n1**.

**ii.** Afterwards, we run the **`lsblk`** utility to view the newly configured partition on each of the three (3) disks.

![lsblk 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/23676904-26bb-4cb3-84d2-2bd6e51bf49f)

As can be seen in the image above, we have our newly configured partitions on each of the three (3) disks listed as **`nvme1n1p1`**, **`nvme2n1p1`** and **`nvme3n1p1`**

**iii.** The next course of action is to install the **`lvm2`** package using the following command:

**`$ sudo yum install lvm2 -y`**

![install lvm2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/450cf513-5859-42b5-8bbc-636d9b2560ae)

**iv.** Then we run the command below to check for available partitions:

**`$ sudo lvmdiskscan`**

![sudo lvmdiskscan](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e40db3a8-5549-483e-bc44-401aefb6143e)

#### <br>Step 7: Create Physical and Logical Volumes<br/>

**i.** For the next step, we use the **`pvcreate`** utility to mark each of our three (3) partitioned disks as physical volumes (PVs) to be used by LVM:

**`$ sudo pvcreate /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create physical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/71e43f90-ba1b-42e0-9908-b8b3b02b4438)

**ii.** Then afterwards, we verify that the physical volumes (PVs) have been created by executing the command below:

**`$ sudo pvs`**

![sudo pvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/177f5cfa-aff0-4d49-81ba-2cf42db3044c)

**iii.** After this, we proceed to use the **`vgcreate`** utility to add all three (3) physical volumes (PVs) to a volume group (VG) that we will be naming **nfsdata-vg**

**`$ sudo vgcreate nfsdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

![create volume group](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/778a3827-cd9b-45d6-8dca-d3f99490b46d)

**iv.** And then we confirm that our volume group (VG) has been created by successfully executing the following command:

**`$ sudo vgs`**

![sudo vgs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/abded900-be1e-47a7-bb1c-7759b9a2a03c)

**v.** The next step is to create three (3) logical volumes (LVs) **lv-opt**, **lv-apps** and **lv-logs** using the **`lvcreate`** utility. 

```
$ sudo lvcreate -n lv-opt -L 9G nfsdata-vg
$ sudo lvcreate -n lv-apps -L 9G nfsdata-vg
$ sudo lvcreate -n lv-logs -L 9G nfsdata-vg
```

![create logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f50679a1-39b3-42af-b858-8023d16d183b)

**vi.** And then we confirm that our logical volumes have been created by successfully executing the following command:

**`$ sudo lvs`**

![sudo lvs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a51722e4-114c-4993-80c3-370427b0a757)

**vii.** Then we verify our entire setup of Volume Group (VG), Physical Volumes (PV) and Logical Volumes (LV) with the following commands:

**`$ sudo vgdisplay -v`**

![sudo vgdisplay](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ffbdda15-513c-4bb9-8dc7-b4c9afc56b4a)

**`$ sudo lsblk`**

![sudo lsblk set up verification](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/04b9bd0e-7781-4f52-80b3-b68887058302)

**viii.** To complete the process, we use **`mkfs.xfs`** to format the logical volumes (LVs) with **xfs** filesystem.

**`$ sudo mkfs -t xfs /dev/nfsdata-vg/lv-opt && sudo mkfs -t xfs /dev/nfsdata-vg/lv-apps && sudo mkfs -t xfs /dev/nfsdata-vg/lv-logs`**

![format logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5ab3f85b-9e84-4386-b336-246300626b71)

#### <br>Step 8: Create Directories and Mount on Logical Volumes<br/>

In this step, we need to create the directory to hold our application files and then another directory to store backup of log data and then a directory we will use for Jenkins in the future. After creating the directories we will respectively mount them  on the created logical volumes **lv-apps**, **lv-logs** and **lv-opt**.

**i.** We use the following command to create the **/mnt/apps** directory to store our application files:

**`$ sudo mkdir -p /mnt/apps`**

**ii.** We use the command below to create the **/mnt/logs** directory to store backup of log data:

**`$ sudo mkdir -p /mnt/logs`**

**iii.** We use the following command to create the **/mnt/opt** directory to to be used by Jenkins Server in a future project:

**`$ sudo mkdir -p /mnt/opt`**

**iv.** Then we execute the following commands to mount the logical volumes on to the directories we just created:

```
$ sudo mount /dev/nfsdata-vg/lv-apps /mnt/apps
$ sudo mount /dev/nfsdata-vg/lv-logs /mnt/logs
$ sudo mount /dev/nfsdata-vg/lv-opt /mnt/opt
```

![mount logical volumes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/203269d2-5dd7-4e5f-ad2e-894e2a7e9647)

**vii.** The next step is to use the universally unique identifier (UUID) of the device to update the **/etc/fstab** file so that the mount configuration will persist after the restart of the server. We check the UUID of the device by entering the command below:

**`$ sudo blkid`**

![sudo blkid](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ffa0853d-6ebe-409f-b3d1-f3ee5bbf4202)

**viii.** We copy the UUID as shown in the above image and we open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**ix.** We paste in the copied UUID whilst removing the leading and ending quotes and update the **/etc/fstab** file as shown in the image below:

![mounts for nfs server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6f1c76a0-27ce-42de-a613-35d6bcfd50f7)

**x.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**xi.** We test our mount configuration with the following command:

**`$ sudo mount -a`**

**xii.** Then we reload the daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**xiii.** To complete our configuration process, we verify our entire setup by executing the following command:

**`$ df -h`**

![df -h final output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5650411d-f55e-4a4f-8e8c-827c19e1ca5d)

The output must look like what we have in the image above.

#### <br>Step 9: Install NFS Server<br/>

**i.** First of all, we update the server machine if we have not already done so:

**`$ sudo yum -y update`**

![Sudo yum update -y](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c5f2681a-141f-4dc7-8d8a-ac527184a9e3)

**ii.** The next course of action is to install the NFS server using the nfs-utils package as shown in the command below:

**`$ sudo yum install -y nfs-utils`**

![install NFS server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/605821d1-650a-4ca0-9df2-a547f414208c)

**iii.** After the installation is complete, we start the NFS service with the following command:

**`$ sudo systemctl start nfs-server.service`**

**iv.** Next, we execute the command below to configure NFS to always start on system reboot:

**`$ sudo systemctl enable nfs-server.service`**

![enabls nfs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2baa542e-058d-49df-9def-207cb9f2251b)

**v.** And then to conclude, we check the status of the NFS service to verify that it is active and enabled:

 **`$ sudo systemctl status nfs-server.service`**

 ![status NFS service](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7ca6c7e3-b832-4913-846f-250ff332472f)

#### <br>Step 10: Export the Mounts for Webservers' Subnet CIDR to Connect as Clients<br/>

We will keep things simple in this project by installing all three webservers inside the same subnet, but in production set ups in real world scenarios, we would have had to seperate each tier in its own subnet to mainatain higher levels of security. To check the subnet cidr, we do the following:

**i.** We open the AWS console and click on **"EC2"**, then we click on **"Instances"**. 

![instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8806a332-0fb1-446e-af06-6f67362facd5)

**ii.** Then we scroll to our instance and click on the link under **"Instance ID"** to reveal the **"Instance summary"** page.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a3c03fd0-c60a-4d15-a94e-65247fa08f6a)

**iii.** Next, we scroll down and under the **"Networking"** tab, we click on the subnet link under **"Subnet ID"**.

![subnet ID](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9efb3a04-c873-4c59-87f1-b740fafd66b3)

**iv** In the **"Subnets"** page we proceed to locate the subnet cidr under **"IPv4 CIDR"**

![subnet cidr](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1524fd70-92fd-479c-952f-ca5eab3950c4)

**v.** Next, we execute the following set of commands to set up and apply permissions that will allow our Web Servers to read, write, and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

![change permissions](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6815b6db-a72c-4b08-8896-d93b002fa419)

**vi.** Afterwards, we edit the **/etc/exports** file to configure access to NFS for clients within the same subnet. We open **/etc/exports** with the following command:

**`$ sudo vi /etc/exports`**

**vii.** We paste in the following configuration whilst including our subnet cidr:

```
/mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
```

![sudo vi etc exports](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e7d2d0c8-29d2-4858-974f-46f96606b3b8)

**viii.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**ix.** To export all file system paths we have specified in the /etc/exports file, we enter the following command:

**`$ sudo exportfs -arv`**

![sudo exportfs arv](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fa1f3364-f234-4cd6-af82-c00ca56279db)

#### <br>Step 11: Check Port used by NFS and open it using Security Groups<br/>

In order for NFS server to be accessible from our client, we must check which port it is using and open it in the Inbound rules settings for security groups. We must also open the following ports: **TCP 111**, **UDP 111**, **UDP 2049** in addition to the NFS port.

**i.** To check the port which is in use by NFS, we run the following command:

**`$ rpcinfo -p | grep nfs`**

![nfs port](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/11cafc51-a37d-49c8-8294-abcd6767d09b)

**ii.** Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).

**iii.** In the navigation pane, choose **Instances**.

![instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8806a332-0fb1-446e-af06-6f67362facd5)

**iv.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a3c03fd0-c60a-4d15-a94e-65247fa08f6a)

**v.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**vi.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**vii.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Type**, choose **NFS**. 

+ In the space with the magnifying glass under **Source**, we leave it at **Custom** and select **0.0.0.0/0** Selecting the **Source** setting as **0.0.0.0/0** means we can access our server from any IP address. i.e. both locally and from the internet.

+  Add custom rules for **TCP 111**, **UDP 111**, **UDP 2049**.

+ Click on **Save rules** at the bottom right corner of the page.

![save rules ports](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a77d527a-c6af-4b1b-9082-bcad7b5d9c20)

### <br>Configure Backend Database Server<br/>

After completing configuration for the NFS Sever, we need to deploy and configure our Database Server. We do this with the following steps:

#### <br>Step 1: Provision and connect to EC2 Instance<br/>

**i.** We launch an EC2 instance of Ubuntu Linux that we will name as **`DB server`** by following [these steps](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) __IMPORTANT: From the Amazon Machine Image (AMI Image) tab, be sure to select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM) rather than the HVM version of Amazon Linux 2 Server as directed in the link.__

![Deploy DB server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2798d398-3c0f-4faf-8a42-ba74ada879b2)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

**iii.** It is especially important and good practice to keep software up to date for security purposes. Packages in a Linux distribution are updated frequently to fix bugs, add features, and protect against security exploits. So after launching our server we run the following command to update it:

**`$ sudo apt update -y`**

![sudo apt update -y db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6ab7d7d1-4123-4163-8e2a-79428a3918af)

### <br>Step 2: Install MySQL RDBMS on **`DB server`** <br/>

 We install MySQL _**Server**_ Software on our "Server" by entering the following command:

**`$ sudo apt install mysql-server -y`**

![sudo apt install mysql server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7ce01f6f-1095-4b49-9a92-c7b0bc1ffd65)

### <br>Step 3: Configure MySQL Database Server`** <br/>

**i.** We start by creating a database that will be called **"tooling"**:

```
# Log into MySQL Console
$ sudo mysql

# Create Database
mysql> CREATE DATABASE tooling;
```

![create database](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4436b0bb-8de9-42d6-91bd-8bab543f726a)

**ii.** Next, using the subnet cidr of our web servers, we create a database user that we name **"webaccess"** and we set the user password:

**`mysql> CREATE USER 'webaccess'@'172.31.16.0/20' IDENTIFIED BY 'password';`**

![create user db](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fc819a40-fb63-4937-be7f-074311fe8df3)

**iii.** After creating the user, we need to grant user **"webaccess"** full privileges on the **"tooling"** and then we flush privileges to free up cached server memory.

```
# Grant Privileges to created user on all Databases
mysql> GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.16.0/20';

# Flush Privileges
mysql> FLUSH PRIVILEGES;
```

![grant flush privileges](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0cc5bad7-20d0-4ff1-858f-cd910c15472a)

**iv.** Next, we use the following command to confirm that the created database is in the list of databases:

**`mysql> SHOW DATABASES;`**

![show databases](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3cd0b679-a18e-4cc2-94d8-66513b76e805)

**v.** Finally, we confirm our abilty to remotely log into the MySQl console with our newly created user **"webaccess"**:

```
#Log in with newly created user from a remote server
$ sudo mysql -h 172.31.17.182 -u webaccess -p

# Exit MySQL Console
mysql> exit
```

### <br>Configure Web Servers<br/>

In this step, we will be launching three web servers. We need to ensure that the web servers can serve the same content from shared storage solutions, which in our case are the MySQL Database and NFS Server.

For storing shared files that our Web Servers will use, we will utilize NFS and mount previously created logical Volume **`lv-apps`** to the **/var/www** folder where Apache stores files to be served to the users.

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved. We proceed by implementing the following steps:

#### <br>Step 1: Configure NFS Client<br/>

**i.** We begin by spinning up an EC2 Instance of Red Hat Linux. We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

 + We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

 + Under **Name and tags**, we provide a unique name for our server.

![Name and tags web server 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/345f3147-6b01-4f1c-8fde-0cbf3accb81f)
  
+ From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 8 (HVM).

![application and OS images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2beaaf89-c746-4ed5-a434-f0989bfb3db1)

+ Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)

+ Under the Network settings tab, we click on **"Edit"** then under **"Subnet Info"** we click on the dropdown and select the same subnet and availability zone that is in use by our NFS Server.

![network settings subnet](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce8dc829-2da0-444e-b3c7-7bf969be6bf5)
  
+ And then, we click on **"Launch Instance"**

![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

**iii.** After connecting to our server we must first update all installed packages and their dependencies before commencing configuration. We do this by executing the following command: 

**`$ sudo yum update -y`**

![web server update ](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/860e8f0f-527e-443a-981a-67085f7ed561)

**iv.** Next we execute the command below to install NFS Client:

**`$ sudo yum install -y nfs-utils nfs4-acl-tools`**

![NFS client installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c0ce39b0-78ad-406d-9913-6f36278a6b05)

**v.** Our next course of action is to create the **/var/www/** directory, mount it and target the NFS server's export for apps. _**The IP address in the command below must be replaced with the Private IP Address of the NFS Server**_ 

```
$ sudo mkdir /var/www
$ sudo mount -t nfs -o rw,nosuid 172.31.23.65:/mnt/apps /var/www
```

![create and mount var www](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/19028c30-6fa4-4767-8738-5068f5edb568)

**vi.** Subsequently, we verify that NFS was mounted successfully by running the command below:

**`$ sudo df -h`**

![verify NFS mount sudo df -h](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b1de4e2e-8483-46e7-bb53-fde0352a79cb)

**vii.** The next step is to edit the **/etc/fstab** file to ensure the changes will persist after the reboot of the server. We open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**viii.** Then we copy and paste in the following line of configuration: 

**`<NFS-Server-Private-IP>:/mnt/apps /var/www nfs defaults 0 0`**

![configure etc fstab](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f6131a98-cbcf-447c-a3fa-799c2325ad90)

**ix.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit. Then we reload the Daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**x.** Next, we install [Remi Repository](http://www.servermom.org/how-to-enable-remi-repo-on-centos-7-6-and-5/2790/), Apache and PHP by executing the following set of commands:

**`$ sudo yum install httpd -y`**

![sudo yum install httpd](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7c22305e-3465-43ce-9fa7-2a1714bbbe92)

**`$ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`**

![epel release installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2b92189c-9aec-4651-ab92-ce7b5661bd7f)

**`$ sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`**

![remi release installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1c529046-4bb0-410c-9b03-7d12aed6955f)

**`$ sudo dnf module reset php`**

![sudo dnf module reset](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/56371aa3-7ba9-4479-a16d-de72d99ae5d0)

**`$ sudo dnf module enable php:remi-7.4`**

![sudo dnf module enable](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/184533e4-7163-4627-b206-f58a10e14c63)

**`$ sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`**

![sudo dnf php opcache](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2a60b15d-b6f6-4bc1-acc1-e093f31c39f6)

```
$ sudo systemctl start php-fpm

$ sudo systemctl enable php-fpm

$ sudo setsebool -P httpd_execmem 1
```

![sudo systemctl start and enable php](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/565941d6-6338-4ee8-bc56-28e5fe1b5f1d)

**xi.** We repeat steps **i.** to **x.** for an additional two (2) Web Servers.

**xii.** Next, we have to verify that Apache files and directories are available on the Web Server in **/var/www** and also on the NFS server in **/mnt/apps**. To do this, we create a new file **`test.txt`** from one Web Server and check if the same file is accessible from other Web Servers. If we see the same files, then it means NFS is mounted correctly. We proceed by executing the following command on **WebServer1**:

**`$ sudo touch /var/www/test.txt`**

**xiii.** And on the other Web Servers we enter the command below to check if the same file is accessible from their **/var/www** directory:

**`$ sudo ls /var/www`**

![view test 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5af742ff-6f25-4f6c-b811-2b7537dc9051)

![view test 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2fda5a51-cbda-4729-9890-65acc8193984)

As can be seen in the above output images, we are able to access the **`test.txt`** file we created in **WebServer1** from **WebServer2** and **Webserver3**.

**xiv.** Now we need to locate the log folder for Apache on the Web Server and mount it to the NFS server's export for logs. _**The IP address in the command below must be replaced with the Private IP Address of the NFS Server**_

**`$ sudo mount -t nfs -o rw,nosuid 172.31.23.65:/mnt/logs /var/log/httpd`**

![mount var log](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/144e2ecf-66a6-493b-8e94-e84572aa0e12)

**xv.** Subsequently, we verify that the mount was successful by running the command below:

**`$ sudo df -h`**

![df -h mount var log](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d3543fc5-e5c3-4fab-9d5e-155d7f7871c3)

**xvi.** The next step is to edit the **/etc/fstab** file to ensure the changes will persist after the reboot of the server. We open the **/etc/fstab** file with the following command:

**`$ sudo vi /etc/fstab`**

**xvii.** Then we copy and paste in the following line of configuration: 

**`<NFS-Server-Private-IP>:/mnt/logs /var/log/httpd nfs defaults 0 0`**

![etc fstab var log configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4105b668-d8ae-4bd7-943a-a8e1daa87fc7)

**xviii.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit. Then we reload the Daemon with the command below:

**`$ sudo systemctl daemon-reload`**

**xix.** We repeat steps **xiii.** to **xvii.** for our other two (2) Web Servers.

#### <br>Step 2: Deploy a Tooling Application to Web Servers into Shared NFS Folder<br/>

**i.** We begin by forking the tooling source code from [Darey.io Github Account](https://github.com/darey-io/tooling) to our own Github account. To do this, we follow the steps described [here](https://www.youtube.com/watch?v=f5grYMXbAV0):

+  We click on the fork icon at the top of the [Darey.io Github Account](https://github.com/darey-io/tooling)

  ![fork tooling github](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/67ed3f7a-09d8-4a92-907c-af6b6b34dedd)
  
+  Then on the **"Create a new fork"** page we click on **"Create Fork"**

 ![Create fork](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c03921ab-820b-4530-8604-92de7cf81cdf)

**ii.** The next step is to deploy the tooling website code to our Web server and ensure that the html file from the repository is deployed into **/var/www/html**. To however do this, we need to first of all install git on the web server:

**`$ sudo yum install git -y`**

![sudo yum install git](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d285c1fd-9a98-4362-a658-c93c7e6bef45)

**iii.** After completing the git installation, we proceed to clone the repository from GitHub to the web server:

+ On the Github **tooling** page, we click on **Code** and then under **HTTPS** we copy the URL.

  ![clone git repo](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/887f9eb7-8207-4817-96c7-48605935f144)

+  In our Web Server terminal we execute the **`git clone`** command along with the copied repository URL:

  **`$ git clone https://github.com/QBDev0ps/tooling.git`**

  ![git clone command](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/40a3ca48-ec52-4045-aa05-5e089b187a3d)

**iv.** Next, we change to the tooling directory and copy the files from the html folder in the repository to the **/var/www/html** directory:

```
$ cd tooling
$ sudo cp -r html/* /var/www/html/
```

![copy tooling](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fbcdad12-1c49-4bfe-93ee-7fb700685f93)

**v.** Subsequently we execute the following command to verify that the files were successfully copied:

**`$ sudo ls /var/www/html/`**

![verify copy tooling](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/35a2ce1b-46ce-47d3-b64c-e0584b064cc6)

#### <br>Step 3: Open TCP Port 80 on EC2 Web Server Instance<br/>

We need to open Port 80 as this is the port number assigned to commonly used internet communication protocol, Hypertext Transfer Protocol (HTTP). It is the default network port used to send and receive web pages.

**i.** In the AWS  console navigation pane, we choose **Instances**.

![instances web server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2b06060c-15e8-4bab-99a4-4771f320b10d)

**ii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary web server](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/be53ce38-03d8-41f4-b5af-226e10db9d5b)

**iii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**v.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ Under **Type**, click on the drop down and select **HTTP**. **Port Range** automatically defaults to **80** when this is done.

+ In the space with the magnifying glass under **Source**, choose **Anywhere**.

+ Click on **Save rules** at the bottom right corner of the page.

![save rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4062c8ba-edfb-453a-8ea3-c5b48c250bde)

**vi.** Afterwards we the try to confirm apache is active and running with the following commands:

```
$ sudo systemctl restart httpd

$ sudo systemctl status httpd
```

![httpd error](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2edc13e0-88eb-4dee-93b4-1a27d1affa10)

But as can be seen in the output image above, our attempt failed as the httpd service is unable to write to the log directory. 

**vii.** To fix the above issue, we proceed to give apache permissions to the **`/var/www/html`** directory with the following command:

**`$ sudo chown -R apache:apache /var/www/html`**

+ And then we disable SELinux policies with the following command:

**`$ sudo setenforce 0`**

![change apache rights](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/25f6138f-2e69-41fa-97ab-a3a575919285)

+ To make this change permanent, open the **`/etc/sysconfig/selinux`** config file with the command below and set **`SELINUX=disabled`**

**`$ sudo vi /etc/sysconfig/selinux`**

![selinux disabled](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/44269f0b-1132-44a0-be0f-eacabfeba41b)

+ Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit. Then we restart httpd and confirm that it is active and running:

```
$ sudo systemctl restart httpd

$ sudo systemctl status httpd
```

![httpd active and running](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8642dfcf-81f6-4b44-857c-105b6dcda9ab)

#### <br>Step 4: Update the Website’s Configuration to Connect to the Database<br/>

**i.** This step requires us to update the website’s configuration file **`/var/www/html/functions.php`** to connect to the database. We do this by running the following command: 

**`$ sudo vi /var/www/html/functions.php`**

![connect to database](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f1f5f389-88d0-4ffb-a6f4-4b827afcaef4)

**ii.** As shown in the image above, we update the config file with the Private IP address of our Database server, the username for our database, the password and the name of the database. Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit. 

**iii.** Next we need to apply **`tooling-db.sql`** script (which can be found in the repository we cloned)  to our database. But before we do this we need to install MySQL Client with the following command:

**`$ sudo yum install mysql`**

![install mysql client](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4d019161-314b-47c2-b84c-1c2216659624)

#### BLOCKER 1❗

+ We encountered the error shown in the output image below when we tried to access our database remotely.

![blocker](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a558b2f8-8096-4a47-806f-3304a74d5166)
  
**iv.** To fix the blocker and ensure that we are able to access our database remotely we need to go to the Database Server and edit **/etc/mysql/mysql.conf.d/mysqld.cnf** by changing **bind-address = 127.0.0.1** to **0.0.0.0**

**`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`**

![bind address edit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/56436326-2a49-499d-bca4-b2170b41e49a)

**v.** And afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit. Then we apply the **`tooling-db.sql`** script to our database using the commands below:

```
$ cd tooling
$ sudo mysql -h 172.31.17.182 -u webaccess -p tooling < tooling-db.sql
```

![apply tooling script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/235931cc-0540-463b-bc7f-a618d2250f19)

The script creates a table **users** in database **tooling** and creates an entry for admin. As can be seen in the above image, we successfully executed the command without any errors which indicates that we successfully initated a remote connection to the database. 

#### <br>Step 5: Create in MySQL a new admin user<br/>

**i.** To complete our configuration we need to create in MySQL, a new admin user with the username: **myuser** and password: **password**. To implement this, we connect to MySQL from the Web Server using the **"webaccess"** user and the private IP Address of the Database server:

**`$ sudo mysql -h 172.31.17.182 -u webaccess -p`**

![sql connection successful](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9b47e7cc-502a-4f64-a76a-ac5b74dbdd59)

**ii.** We run the following query to create a new admin user:

```
mysql> INSERT INTO tooling.users (id, username, password, email, user_type, status) VALUES (2, 'myuser', '5f4dcc3b5aa765d61d8327deb882cf99', 'user@mail.com', 'admin', '1');
```

![insert new admin user](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/78cb0f8c-d08b-47be-af34-9621b47899d2)

**iii.** Next we view our users in the database for additional confirmations:

**`mysql> SELECT * FROM tooling.users;`**

![show database users](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f52b1afa-766f-4388-aa0b-51adad81364f)

The output image above shows that we have in the **tooling.users** table, the new entry **myuser** as well as the **admin** user we added with **`tooling-db.sql`** script.

**iv.** Then we proceed to open the website in our browser using the following syntax:

**`http://<WebServerPublicIP>/index.php`**

#### BLOCKER 2❗

+ We encountered the error shown in the output image below when we tried to access the index page of our tooling website.

 ![Screenshot 2023-10-13 115648 browser error](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/433f7545-3300-4fc4-806f-490d4cb5c4b8)

+ As seen in the browser output above, rather than serving the webpage of our website, the code of the functions configuration that was being called from **functions.php** and the HTML code in our **index.php** file, were being displayed onto the browser screen.

+ To fix this, we decided to inspect the **functions.php** file with the following command:

  **`$ sudo vi /var/www/html/functions.php`**

+ There, we noticed the opening **`<?php`** tag was missing so we replaced it as shown in the image below:

![blocker 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/581d2eb2-877d-4567-837c-f0b5ded7ac5d)

+ And then afterwards on our keyboard, we pressed on **`esc`**, typed **`:wq!`** to save and quit immediately and pressed **`enter`** to confirm exit.

+ To open the tooling website we used the syntax as stated in **iv.** above and we entered the following:

  **`http://13.51.170.40/index.php`**

  ![browser access](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1bfbdbbd-266a-40b4-bd6b-5d759c935069)

+ As shown in the image above, the webpage was rendered correctly thus solving our **Blocker**. Next we attempt to log in.

#### BLOCKER 3❗

+ We encountered an issue when we tried to log into the tooling website. As shown below, the tooling website required us to enter a password to be four (4) characters only. This posed a problem for us. Because our password was in a long hash string as was seen when we viewed our users in the **tooling** database (in **Step 5.** **iii.**).

  ![blocker 3](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/647bea69-3fe6-42e2-b4c2-714d5ad7fe43)

+ To fix this issue we are required to do a slight configuration change in the **functions.php** file. We enter the following command:

**`$ sudo vi /var/www/html/functions.php`**

+ And then we look for **`$password = md5($password);`** which is a bit of code that is responsible for hashing passwords before they are saved in the database. We comment this code out with **`//`** as shown in the image below:

![md password](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8f5b8a60-1a94-4523-955b-80d0f57be1c3)

+ And afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

+ Next, we add a new admin user in MySQL with the query below:

```
mysql> INSERT INTO tooling.users (id, username, password, email, user_type, status) VALUES (3, 'myuser', 'pass', 'user@mail.com', 'admin', '1');
```
![add new user query](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2f729a50-c6af-4053-84d6-d34b455c03f4)

+ And then we view our users in the database for confirmations:

**`mysql> SELECT * FROM tooling.users;`**

+ As can be seen in the output image below the password for **User id 3** (the new user we just added) is not hashed. This means our 'slight' change worked.

![create new user](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7ba49585-1c3c-47ee-806c-edf0db6694a6)

**v.** We log in to our tooling website with our four character password as shown below:

![enter password](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/78cf7180-d141-43f2-ad7d-b82cfab7c6a7)

**vi.** The log in was successful, we are now able to access our tooling applications from our three webservers via our NFS mount.

![project complete](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/34e19f17-689b-4c01-9d0a-d731b69d7141)

### <br>CONCLUSION<br/>

We have been able to successfully complete this project. The goal of our project was to implement a tooling website solution that will integrate various DevOps tools and technologies to enhance collaboration, automation and efficiency for DevOps teams. To do this we had to implement a solution where three stateless Web Servers will share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. 

We began our project implementation by deploying and configuring a Network File Server (NFS) Server with storage infastructure. This involved provisioning an EC2 Instance of Red Hat Linux to serve as our NFS Server after which we deployed and configured LVM on the server. This process required creating and attaching 3 EBS volumes, partitioning disks and then creating and configuring physical and logical volumes. Next, we created the relevant directories which we mounted on the logical volumes. And then we installed the NFS Server software.To complete this process, we exported the mounts for Webservers' Subnet CIDR to connect as clients and then opened the ports used by NFS in security groups.

We subsequently launched a second EC2 instance (but this time with Ubuntu Linux) that was given a role as our Database (DB) Server. We then installed and configured MySQL Relational Database Management System (RDBMS). The configuration required us to create a database and then a database user using the subnet cidr of our web servers. We also granted the newly created user full privileges on the database we created.

The final step required us to install three web servers and ensure that the web servers could serve the same content from shared storage solutions, which in our case are the MySQL Database and NFS Server. We spun up up three EC2 Instances of Red Hat Linux under the same subnet in use by the NFS Server. Next, we installed and configured NFS Client on them. The configuration involved  creating the **/var/www/** directory, mounting it and targeting the NFS server's export for apps. Then we had to install Apache, PHP and their requisite dependencies. After this, we deployed the tooling application to our web servers via the shared NFS folder by forking the tooling source code from Darey.io Github account to our own Github account, installing Git on the webserver, cloning the repository and copying from the html folder in the repository to the **/var/www/html** directory. After ensuring Port 80 on the web server is open, we needed to update the website’s configuration to connect to the database and then in MySQL, create a new admin user that would be used to access the tooling website. Here we faced some **Blockers** but fortunately, we were able to resolve them and we concluded our project by successfully connecting to the database, creating the new admin user, logging in to the tooling website and being able to access it with the Public IP Address of all 3 of our web servers. Thank you.
