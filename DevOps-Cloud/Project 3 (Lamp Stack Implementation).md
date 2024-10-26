# DOCUMENTATION FOR LAMP STACK IMPLEMENTATION PROJECT

## This project shows the implementation of the LAMP Stack. It covers essential aspects of the LAMP stack implementation such as the process of setting up a Linux Environment, configuring the Apache Web server, managing MySQL databases, and writing PHP code for server side functionality.

### Pre Installations and Dependencies

In order to execute this project successfully we need to first of all complete the following prerequisites:

1. Setup an AWS Account : [Create a free tier AWS Account](https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all) by following these [steps](https://repost.aws/knowledge-center/create-and-activate-aws-account).

2. Spin up an EC2 Instance with Ubuntu Server: Launch an EC2 instance by following [these steps.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) __IMPORTANT: From the Amazon Machine Image (AMI Image) tab, be sure to select the free tier eligible version of Ubuntu Linux Server 20.04 LTS (HVM) rather than the HVM version of Amazon Linux 2 Server as directed in the link.__

3. Download and Install an SSH client: Download and install [PuTTY](https://putty.org/) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

4. Establish connection with your EC2 instance: Connect to your EC2 instance via your SSH client by following [these instructions.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)

**After completing the necessary prerequisites, we proceed to implement the following steps to fully implement our LAMP Stack and deploy it in AWS Cloud:**


## STEP 1: Installing Apache and Updating the Firewall

The apache web server is the most popular and widely used web server in the world. It is fast, reliable, secure, and can be customized to meet the needs of different environments with the use of modules and extensions. However, before we install apache we need to firstly update our ubuntu web server by running the command below: 

**`$ sudo apt update`**

![sudo apt update](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/84b1b3b6-bb08-454b-adf9-3df0ad2ae3f0)

After completing the update process we can now proceed to install the apache web server, To do this, we execute the following command:

**`$ sudo apt install apache2`**

![apache installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/05c63919-b424-479c-b075-377091ff5118)

To verify that the apache2 service is running on our ubuntu server, we enter the following command:

**`$ sudo systemctl status apache2`**

![apache status](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9defa108-ca6d-495a-80ba-d837eda3fcd6)

As can be seen in the status image above, apache is fully active and running.

The next step is to open TCP Port 80 on our machine. Port 80 is the defualt port used by web browsers to access web pages. So we implement this by adding a rule to our EC2 configuration to allow http traffic via port 80. The steps are listed below:

1. Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).

2. From the top navigation bar, select a Region for the security group. Security groups are specific to a Region, so you should select the same Region in which you created your instance.

3. In the navigation pane, choose **Instances**.

4. Select your instance and, in bottom half of the screen, choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

5. For the security group to which you'll add the new rule, choose the security group ID link to open the security group.

6. On the **Inbound rules** tab, choose **Edit inbound rules**.

7. On the **Edit inbound rules** page, do the following:

a. Choose **Add rule**.

b. For **Type**, choose **HTTP**.

c. Under **Source**, leave it at **Custom** and select **0.0.0.0/0** in the space with the magnifying glass.

d. Choose **Save rules** at the bottom right corner of the page

![edit inbound rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9b290578-acd5-48ea-a297-a589ac127e6b)

Selecting the **Source** setting as **0.0.0.0/0** means we can access our server from any IP address. i.e. both locally and from the internet.

To verify that the apache2 webpage is accessible locally from our ubuntu machine, we run the following command:

**`$ curl http://localhost:80`**   or  **`$ curl http://127.0.0.1:80`**

![verify apache access](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3e1685db-4835-4ec1-b961-546ac37976d9)

In the output image above, we can see the [HTML](https://en.wikipedia.org/wiki/HTML) code of our Apache Service web page displayed on our terminal.

Next we need to test to ensure that our web server is responding to requests from the internet. To implement this, we open a web browser and in the adress bar, we enter the url using this syntax **`http://<PUBLIC-IP-ADDRESS>:80`**. We can retrieve our public IP address by entering the following command:

**`$ curl -s http://169.254.169.254/latest/meta-data/public-ipv4`**

![public IP](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3cddd5d8-4011-4a1b-ba9c-e9acfff2d02d)

Now that we have the public IP address, we proceed to enter the following url in our browser:

**`http://16.170.229.201:80`**

![apache ubuntu web page](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cb19a8a7-d441-4a77-8147-4ff11db1eb37)

As shown in the output image above, we can see the apache ubuntu default web page which tells us that our web server is now properly installed and accessible through our firewall.


## STEP 2: Installing MySQL

After completing the setup of our Web Server, we need to install a Database Management System to be able to store and manage data for our webpage. MySQL is an open-source Relational Database Management System (RDBMS) that enables users to store, manage, and retrieve structured data efficiently. It is widely used for various applications, from small-scale projects to large-scale websites and enterprise-level solutions. To run this installation, we enter the following command:

**`$ sudo apt install mysql-server -y`**

![mysql installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7d431123-ba91-4309-8bf8-189778e6062c)

After the installation is complete, we log into the MySQL console by executing the command below:

**`$ sudo mysql`**

![log into mysql](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cf8bdd85-5bd0-4917-9d7a-5bfde21ce9ea)

Next, we need to run a script that is preinstalled with MySQL to help secure access to our database system. However, before running the script, we need to set a password (which will be defined as **`PassWord.1`**) for the root user. We implement this by entering the command below:

**`mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`**

Then we exit with the following command:

**`mysql> exit`**

![alter sql user password and exit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/62d41aaf-5e05-42ad-a3e2-b1b2f06e5cba)

Next we run the MySQL interactive script by executing the following command:

**`$ sudo mysql_secure_installation`**

As shown in the output image below, we use the **`VALIDATE PASSWORD PLUGIN`** to set up a strong password for MySQl root user and we follow all the prompts to choose our preferences.

![mysql script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/971685c5-5b76-48fe-a1d9-106749e7ec04)

Next, with the following command, we confirm our abilty to log into the MySQl console with our newly created root password:

**`$ sudo mysql -p`**

This prompts us for the root password and as we can see in the output image below, we are able to log in. 

![test mysql password](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8a4f751f-c30e-4967-9805-29271d2d310c)

As shown in the above image, to exit the MySQL console, we simply execute the command below:

**`mysql> exit`**


## STEP 3: Installing PHP

After installing Apache Web Server and MySQL Database Management System. It is now time for our PHP installation. PHP is the component of our LAMP Stack setup that allows webpages run dynamic processes to enable content to be displayed on the end user's browser. Apart from the **`php`** package, we will need to install two additional packages namely:

a. **`php-mysql`**: This enables communication between PHP and MySQL based databases.

b. **`libapache2-mod-php`**: This enables Apache to hanle PHP files.

To proceed, we install these three packages all together at the same time by entering the following command:

**`$ sudo apt install php libapache2-mod-php php-mysql`**

As seen in the ouput image below the sytem installs PHP along with the other two modules.

![php installation](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2d38b817-bdf8-493f-a124-8be618e7304c)

We can execute the command below to confirm our version of PHP:

**`php -v`**

![check php version](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e495cfc3-f3da-4632-b2f2-778b3184fde2)

At this point, we have successfully installed all four components of the LAMP Stack:

☑️ Ubuntu Linux Operating System

☑️ Apache HTTP Server

☑️ MySQL Database Management System

☑️ PHP


## STEP 4: Creating a Virtual Host for your Website using Apache

Apache virtual host enables us to have multiple websites on a single machine. To test our LAMP setup with a PHP script, we will need to create a virtual host to hold our website's files and folders. To do this we need to firstly set up a domain we will be calling **`projectlamp`**.

To create the directory for **`projectlamp`**, we execute the following command:

**`$ sudo mkdir /var/www/projectlamp`**

Next we execute the command below to assign ownership of the created directory by using the `$USER environment variable which references your current system's user:

**`$ sudo chown -R $USER:$USER /var/www/projectlamp`**

Then, using the vi editor, we create and open a new configuration file in Apache’s `sites-available` directory by executing the following command:

**`$ sudo vi /etc/apache2/sites-available/projectlamp.conf`** 

The above command creates a blank file. Then we press `i` on the keybord to enter insert mode and we paste the following configuration:

```
<VirtualHost *:80>
   ServerName projectlamp
   ServerAlias www.projectlamp
   ServerAdmin webmaster@localhost
   DocumentRoot /var/www/projectlamp
   ErrorLog ${APACHE_LOG_DIR}/error.log
   CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

![sudo vi configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1311a6a9-7508-4c9d-8237-a995eeb6da0a)

<br />To save and close the file, we use the folowing steps:<br />
a. Press `esc` on the keyboard.<br />
b. Type `:` <br />
c. Type `wq`<br />
d. Press `Enter`<br />

To show the new file in the sites available directory, we execute the following command:

**`$ sudo ls /etc/apache2/sites-available`**

The output of the above command is as shown in the image below:

![sudo ls sites available](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/62b27627-377e-4adf-a8c8-83c3279fc887)

As it stands, we have implemented a VirtualHost configuration that directs Apache to to use /var/www/projectlamp as its web root directory when serving projectlamp.

We can now execute **a2ensite** command to enable the new virtual host:

**`$ sudo a2ensite projectlamp`**

Then we use **a2dessite** command to disable apache's default website:

**`$ sudo a2dissite 000-default`**

Next, we run the command below to ensure that there are no syntax errors in the configuration file:

**`$ sudo apache2ctl configtest`**

And afterwards, using the command below, we reload Apache so that the changes can take effect:

**`$ sudo systemctl reload apache2`**

Next, to test that our Virtual Host works as expected, we create an index.html file in the /var/www/projectlamp directory:

```
sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html
```
Then the next step is to go to our browser to open our website URL via our public IP address (syntax is: **http://<Public-IP-Address>:80**) or via our server's DNS name (syntax is: **http://<Public-DNS-Name>:80**) In our own use case, we enter the following url in our browser:

**`http://16.170.229.201:80`**

The ouput from the browser is as shown in the image below:

![website working](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/22cb1241-cf8a-40d7-8329-fe8f05798969)


## STEP 5: Enable PHP on the Website

With the default **DirectoryIndex** settings in Apache, the `index.html` file will always take precedence over the `index.php` file. We however need to modify this behaviour so that `index.php` can become the default landing page. To implement this, we will need to use **`vim`** (it is the same thing as **`vi`**) to edit the **/etc/apache2/mods-enabled/dir.conf** file and change the order in which the **index.php** file is listed within the **DirectoryIndex** directive:

**`$ sudo vim /etc/apache2/mods-enabled/dir.conf`**

Next, we execute the change as shown below:

```
<IfModule mod_dir.c>
        #Change this:
        #DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm
        #To this:
        DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```

**(Please note that lines 1-3 in the code above will not be recognised as they have been commented out with the `#` sign)**

![vim index php](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d7bdfe70-9093-4d8d-a552-5083b7dc663b)

After saving and closing the file we use the command below to reload the Apache service to enable the changes take effect:

**`$ sudo systemctl reload apache2`**

The next step is to create a PHP test script to confirm that PHP is correctly installed and configured on the server. To execute this, we create a file named `index.php` in the projectlamp web root folder.

`$ vim /var/www/projectlamp/index.php`

This will open a blank file. We then add the following PHP code inside the file:

```
<?php
phpinfo();
```

Afterwards, we save and close the file and then we reload the web page. The output seen is as shown in the image below:

![php webpage](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ef419b6b-a034-4f03-b237-4e192a75741d)

The page in the image above contains relevant and sensitive information about the configurations of our PHP environment and our Ubuntu Server. So after going through the details on the page, we opt to remove it with the **`rm`** command:

**`$ sudo rm /var/www/projectlamp/index.php`**


## This brings us to the conclusion of this project. Thank you!
