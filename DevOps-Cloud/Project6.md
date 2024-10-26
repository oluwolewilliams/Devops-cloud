## DOCUMENTATION FOR IMPLEMENTING CLIENT SERVER ARCHITECTURE USING MySQL DATABASE MANAGEMENT SYSTEM (DBMS)

This project demonstrates the implementation of a basic client-server setup using MySQL RDBMS. Client-Server refers to a model of computing where the client requests a service and the server provides it. It is an architecture in which two or more computers (i.e. seperate entities of **Client** which is the computer sending the request and **Server** which is the computer responding to the request) communicate through the internet over a network to send and receive requests between one another. 

In our set up, we shall establish connection over a Local Network between our Client Machine and our MySQL Server Database Engine. And we shall ascertain that we can perform SQL queries remotely on the SQL Database Server via our Client.

We shall proceed by utilizing the following steps:


### <br>Step 1: Create and Configure two Linux Based Virtual Servers<br/>

**i.** We launch two EC2 instances of Ubuntu Linux by following [these steps](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) and we respectively name them as **`mysql server`** and **`mysql client`**.  __IMPORTANT: From the Amazon Machine Image (AMI Image) tab, be sure to select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM) rather than the HVM version of Amazon Linux 2 Server as directed in the link.__

**ii.** It is especially important and good practice to keep software up to date for security purposes. Packages in a Linux distribution are updated frequently to fix bugs, add features, and protect against security exploits. So after launching our servers we run the following command on both servers to update them:

**`$ sudo apt update`**

![sudo apt update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/041248b2-4d11-482b-814f-26d31fcac19b)


### <br>Step 2: Install MySQL *Server* Software on **`mysql server`** <br/>

 We install MySQL _**Server**_ Software on our "Server" by entering the following command:

**`$ sudo apt install mysql-server -y`**

![mysql-server installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/21156ca6-0d05-4df8-82c2-6cde82ccf4dc)


### <br>Step 3: Install MySQL *Client* Software on **`mysql client`** <br/>

And then we install MySQL _**Client**_ Software on our "Client" by entering the following command:

**`$ sudo apt install mysql-client -y`**

![mysql-client installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/072942eb-73e3-4575-8a2a-8d065e2d4ec5)



### <br>Step 4: Configure **`mysql server`** and Create a New User<br/>

Next, we shall configure our MySQL RDBMS. To enable access from **`mysql client`**, we will need to create a dedicated user on **`mysql server`**. But first of all, we shall set a password for the root user and then we will proceed to run a script that is preinstalled with MySQL to help prevent security exploits and secure access to our database system by modifying some less secure default settings. We do this by executing the following:

```
# Log into MySQL Console
$ sudo mysql

# Set Password for the Root User
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Password8';

# Exit MySQL Console
mysql> exit
```

![set root mysql password and exit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3a5af8c1-32b6-42b9-9517-838f0b109d25)

Next we run the MySQL interactive script by executing the following command:

**`$ sudo mysql_secure_installation`**

As shown in the output image below, we use the **`VALIDATE PASSWORD PLUGIN`** to set up a strong password for MySQL root user and we follow all the prompts to choose our preferences.

![mysql script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0890ddb7-fbbc-4871-8786-bdf50d81e61e)

Next we create a new user (and we can also decide to create a new database) by executing the following:

```
# Log into MySQL Console
$ sudo mysql -p

# Create a New User
 mysql> CREATE USER 'new_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Password8@';

# Create a New database
mysql> CREATE DATABASE project6_db;
```

Then the next thing to do is to execute the following to grant requisite privileges to **`new_user`** and then flush privileges to free up cached server memory.

```
# Grant Privileges to created user on all Databases
mysql> GRANT ALL PRIVILEGES ON *.* TO 'new_user'@'%' WITH GRANT OPTION;

# Flush Privileges
mysql> FLUSH PRIVILEGES;

# Exit MySQL Console
mysql> exit
```

The output image for the above executed commands is shown below:

![mysql server configuration_create user-db-privileges](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/941aa61e-85a9-422c-86fb-94cdba9f36d8)

Finally, we use the following to confirm our abilty to log into the MySQl console with our newly created user:

```
#Log in with newly created user
$ sudo mysql -u new_user -p

# Exit MySQL Console
mysql> exit
```

![confirm new user](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0cc95c6b-a265-4321-87f6-7d1f18c0cd41)


### <br>Step 5: Edit Inbound rules on **`mysql server`** to enable access to **`mysql client`** traffic. <br/>

By default, both of our running EC2 virtual servers are located in the same local virtual network, so they can communicate with each other via local IP addresses. We use the  local IP address of MySQL _**Server**_ to connect from  MySQL _**Client**_ . But for this connection to occur, we will need to create a new entry in **"Inbound rules"** in MySQL _**Server**_ Security Groups. We execute this by doing the following:

**i.** In the AWS  console navigation pane, we choose **Instances**.

**ii.** We Select our **`mysql server`** instance and in bottom half of the screen, choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

**iii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

**v.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **3306** which is the default TCP port for MySQL _**Server**_.

+ In the space with the magnifying glass under **Source**, we type in the Private IP Address of **`mysql client`**. This gives an added layer of security by ensuring that no other IP addresses can reach **`mysql server`** except **`mysql client`**.

+ Choose **Save rules** at the bottom right corner of the page

![edit inbound rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f6d4240d-229e-4388-8cd7-1cd4957f12a9)


### <br>Step 6: Configure **`mysql server`** to allow connections from remote hosts <br/>

To implement this we enter the following command to open MySQL Config file:

**`$ sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`**

Then as shown in the image below, we press **`i`** to enter insert mode and change **"bind-address"** from **‘127.0.0.1’** to **‘0.0.0.0’**.

![configure mysql config file to allow connection from remote hosts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/08e62559-c942-491f-9cf5-712b178ab820)

Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

To complete this process, we need to restart the MySQl service with the following command:

**`$sudo systemctl restart mysql`**


### <br>Step 7: Connect Remotely from **`mysql client`** to **`mysql server`** Database Engine <br/>

We perform this function from **`mysql client`**  without using **SSH**. Rather, we use the **MySQL** utility along with the private IP Address of **`mysql server`** by executing the following command:

**`$ sudo mysql -h 172.31.25.87 -u new_user -p`**


### <br>Step 8: Confirm Successful Connection <br/>

To confirm that we have successfuly connected to the remote **`mysql server`**, we run the following SQL query:

**`mysql> Show databases;`**

If we look closely at the output image below, we can see the database we created **`project6_db`** listed among other databases. This image shows that we were able to deploy a fully functional MySQL client-server set up.

![confirm connection](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a2e66f77-b904-4da4-8f2a-eea479cc655f)


### <br>CONCLUSION<br/>

We have been able to successfully complete this project. We started our project implementation by creating and configuring two linux based virtual servers. Then we installed MySQL _**Server**_ Software on the first machine designated as the MySQL Database Server and MySQL _**Client**_ Software on the second machine designated as the machine that will send connection requests. The next thing we did was to configure our **`mysql server`** by setting a root password, running a security script, creating a new user, granting the new user the requisite rights and creating a new database. After this we edited inbound rules on **`mysql server`** to enable connection from **`mysql client`**. In addition, we configured **`mysql server`** to allow connections from remote hosts. We subsequently initiated a remote connection from **`mysql client`** to **`mysql server`** Database Engine and we were able to successfully confirm the connection by running an SQL query.
