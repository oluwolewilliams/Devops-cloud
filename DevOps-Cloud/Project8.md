## DOCUMENTATION FOR AUTOMATING LOAD BALANCER CONFIGURATION WITH SHELL SCRIPTING

This project demonstrates how to use shell scripting to automate the set up and maintenance of a load balancer to effectively enhance efficiency and reduce manual effort to the barest minimum.

### <br>Introduction to Automating Load Balancer Configuration with Shell Scripting<br/>

Load balancing refers to the method of efficiently distributing network traffic equally across a pool of resources i.e a group of back end servers that support an application. Modern applications must process millions of users simultaneously and return the correct text, videos, images, and other data to each user in a fast and reliable manner.

![what-is-a-load-balancer](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/194dc498-8204-45ec-9c86-b9455879adb5)

A [Shell Script](https://en.wikipedia.org/wiki/Shell_script) is a collection of commands and instructions that are executed sequentially in a shell. Essentially, shell scripting helps us automate repetitive tasks.

In this project, we will be automating the entire process of deploying two back end webservers with a load balancer distributing traffic across both servers. But rather than implementing these tasks manually we will be executing a shell script that will carry out the tasks automatically therefore demonstrating how we can speed up the deployment of services and reduce the chance of making errors in our day to day activities.

### <br>Automating the Deployment and Configuration of Web Servers<br/>

To begin our project we need to deploy and configure two Apache Web Servers. To do this, we will provision an EC2 instance running Ubuntu 22.04 then we will execute a script that will update our EC2 instance and afterwards, the script will install apache web server on it. Subsequently, the script will configure apache to listen on Port 8000 and finally update the default page of the web servers to display their public IP address. Our script will pass one argument which will be the Public IP adress of the EC2 instance that is being provisioned. To run our script, we will implement the following steps:

#### <br>Step 1: Provisioning EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Ubuntu Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our web server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/07d96d27-7f7a-4bca-898a-c3dd8370e19b)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Ubuntu Linux Server 22.04 LTS (HVM).

![Application and OS Image](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ae844641-2121-49de-99e3-67c8621c4027)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Open Port 8000<br/>

We will be running our Web Server on Port 8000 while the Load Balancer will be running on Port 80. We will therefore need to open Port 8000 to allow traffic from anywhere. To implement this, we need to add a rule to the Security Group of our Web Server:

**i.** In the AWS  console navigation pane, we choose **Instances**.

![Instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75d2208f-d030-4f44-9667-23521332607f)

**ii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/13c46f47-1bf7-4563-909a-7d2eb30ac749)

**iii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**v.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **8000** 

+ In the space with the magnifying glass under **Source**, choose **Anywhere**.

+ Click on **Save rules** at the bottom right corner of the page.

![save rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c4bb985a-04dc-4463-998a-07b047e49207)

#### <br>Step 3: Connect to the Webserver via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have opened the necessary port, we must next connect to the web server via an SSH client. This will enable us to subsequently be able to run commands on the terminal of our web server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [PuTTY](https://putty.org/) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our SSH client by following [these instructions:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-linux-inst-ssh.html)

+  To initiate connection to the webserver, we click on Instance ID. Then at the top of the page, we click on connect.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/13c46f47-1bf7-4563-909a-7d2eb30ac749)

+  Next, we click on **"SSH Client"** tab and we copy the ssh command under **"Example:"** as shown in the image below:

![connect to instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cdcd7d88-efb6-475a-beea-836895b4e8e9)

+  Next, we open a terminal on our ssh client in our local machine, then we change directory **`cd`** to the downloads folder and then we paste and execute the ssh command we copied in the previous step.

  ![ssh to connect instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6592a5ce-f740-4b40-8328-0e8eb9a56562)

+  We click on **Enter** and type **Yes** when prompted. This connects us to a terminal on our EC2 instance.

 ![instance connected](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2cb0cd87-3c12-420e-a126-4b0ebdd57f66)

 #### <br>Step 4: Prepare and Execute Script<br/>

To prepare and execute our script, we shall carry out the actions below:

**i.** We proceed to create and open a file called _**install.sh**_ file by entering the following command:

**`$ sudo vi install.sh`**

**ii.** We switch the Vi editor to insert mode by pressing **`i`** and then we copy and paste in the following script:

```
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2
```

![sudo install sh script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/52f05c63-14f1-4fa3-ab92-3321af3ee828)

**iii.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**iv.** We change permissions to make the file executable by running the command below:

**`$ sudo chmod +x user-input.sh`**

**v.** As stated before, running our shell script requires an argument which is the Public IP adress of the launched EC2 instance. We run our Shell Script by executing the following command:

  **`$ sudo ./install.sh 13.51.194.67`**

 ![run script web server 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/209af177-e9d9-49aa-ae45-d271cfcc5b84)

Then the next step is to go to our browser to open our webpage URL via our public IP address (syntax is: **`http://<Public-IP-Address>:8000`**). In our own use case, we enter the following url in our browser:

**`http://13.51.194.67:8000`**
  
  As observed in the image below, our script runs successfully and we are able to see the desired output.

  ![browser output webserver 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9276636d-471c-4c81-bdc0-600bddd26e12)

#### <br>Step 5: Install and Configure Second Web Server<br/>

Now that we have completed the installation and configuration of our first Web Server, we repeat **Steps 1-4** above to deploy and configure our second Apache Web Server.

The image below shows that we were able to get the desired output in our browser after running our script on the second EC2 instance.

![browser output web server 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/21d43870-ca56-4d3c-a837-3d3818a3003f)

### <br>Automating the Deployment and Configuration of Nginx Load Balancer<br/>

Having successfully deployed and configured our two backend web servers, we now move on to the set up of our Nginx Load Balancer. However, as a prerequisite, we need to provision an EC2 instance running Ubuntu Linux 22.04 after which we will add a rule to the Security group to open port 80 and allow access to anywhere. Then finally, we will initiate a connection to the Load Balancer via the terminal so that we can execute the script that will install Nginx and configure it as a load balancer for our two backend servers. To run our shell script, we will implement the following:

**i.** To begin, we provision a new EC2 instance of Ubuntu 22.04. Then we ensure Port 80 on this server is opened to accept traffic from anywhere. We follow **Step 1** and **Step 2** above to implement this.

![load balancer instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fa94b953-8e2c-4243-bcde-ac5285db2343)

**ii.** Next we connect to our newly provisioned server via an SSH client. We follow **Step 3** above to carry this out.

![ssh load balancer](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a399f1a2-1f43-46b5-90bd-d56b9c795391)

**iii.** On the terminal, we create and open a file called _**nginx.sh**_ by entering the following command:

**`$ sudo vi nginx.sh`**

**iv.** We switch the Vi editor to insert mode by pressing **`i`** and then we copy and paste in the following script:

```

#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserser 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx
```

![load balancer shell script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1d8f1a62-0526-421f-918d-e1d52b10b690)

**v.** Afterwards, on our keyboard, we press **`esc`**, type **`:wq!`** to save and quit immediately and press **`enter`** to confirm exit.

**vi.** We change permissions to make the file executable by running the command below:

**`$ sudo chmod +x nginx.sh`**

**vii.** Our shell script takes three arguments. The first argument is the Public IP address of the EC2 instance where our Nginx Load Balancer is installed. The second argument is the URL of our first backend Web Server while the third argument is the URL of our second backend Web Server. We run our Shell Script by executing the following command:

  **`$  ./nginx.sh 13.51.85.125 13.51.194.67:8000 13.48.193.97:8000`**

  ![execute nginx load balancer shell script](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fa153f09-7966-4e35-b856-402b4a8607df)

Then the final step is to go to our browser to paste in the public IP address of our Nginx loadbalancer (syntax is: **`http://<Public-IP-Address>:80`**). In our own use case, we enter the following URL in our browser:

**`http://13.51.85.125:80`**

As shown in the output images below, the pages we see are the same pages served by both of our webservers. When you reload the page the load balancer serves content from our webservers one after the other.

![load balancer browser output 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4edb2dc4-d268-40eb-a55b-f054fff06070)

![load balancer browser output 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/52fd74dc-cb9b-40ec-9b6b-3d9d96990811)

With the power of scripting, we have been able to automate a simple Load Balancer set up. When you reload the browser page the load balancer serves content from our webservers one after the other. This is due to the **Load Balancing Algorithm** which distributes requests sequentially to each server in our server pool to ensure efficient utilization of resources and improve overall system performance, reliability and availability.

  ### <br>CONCLUSION<br/>

We have been able to successfully complete this project. The goal of our project was to automate a simple Load Balancer configuration with Shell Scripting. We began our project implementation by provisioning two EC2 instances of Ubuntu Linux Server 20.04 LTS (HVM). The next thing we did was to add a rule to the Security Group of both of our Web Servers to open Port 8000 to allow traffic from anywhere. We subsequently connected to our servers via an SSH client and we ran our shell script to install and configure Apache Web Server to serve a page showing its Public IP Address. We then provisioned a fresh EC2 instance of Ubuntu 22.04 and we ensured Port 80 on this server was opened to accept traffic from anywhere. Next, we proceeded to run another shell sccript to install Nginx and configure it as a Load Balancer to serve content from both of our backend Apache Web Servers. We concluded the project by pasting our Load Balancer Public IP Adresss into our browser and we were sucessfully served pages from both of our Apache web servers one after the other. Thank you.
