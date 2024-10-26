Migrating to the Cloud with Containerization Part 1. Docker and Docker Compose
==============================================================================

Until now we have been using AWS EC2 instances in Amazon Virtual Private Cloud (AWS VPC) to deploy our web solutions. While this approach is straightforward and scalable, we can envisage some potential issues in a scenario where there is need to deploy many small applications (it can be web front-end, web-backend, processing jobs, monitoring, logging solutions, etc.) and some of the applications will require various Operating Systems and runtimes of different versions and conflicting dependencies - in such case we would need to spin up servers for each group of applications with the exact OS/runtime/dependencies requirements. When our use case scales out to tens/hundreds and even thousands of applications (e.g., when we talk of microservice architecture), this approach becomes very tedious and challenging to maintain.

In this project, we shall learn how to solve this problem and begin to practice the technology that revolutionized application distribution and deployment back in 2013! We are talking of Containers and imply Docker. Even though there are other application containerization technologies, Docker is the standard and the default choice for shipping an app in a container!

### <br>Introduction to Docker<br/>

Docker is an open-source platform that enables developers to automate the deployment, scaling, and management of applications within lightweight, portable containers. Containers package an application and its dependencies together, ensuring consistency across multiple environments from development to production.

#### Key Concepts

**i.** **Containers:** These are Lightweight, standalone, and executable packages that include everything needed to run a piece of software, including the code, runtime, system tools, libraries, and settings. Containers ensure consistency across environments, faster deployment, and efficient resource utilization.

**ii.** **Images:** These are immutable templates that create containers. They include the application code, libraries, dependencies, and other filesystem content needed to run the application. They are built from a Dockerfile using the docker build command.

***iii.** Dockerfile: This is a text file containing a series of instructions on how to build a Docker image. It specifies the base image, application code, dependencies, environment variables, and commands to run.

**iv.** **Docker Hub:** This is a cloud-based repository where Docker users can store and share Docker images. It allows users to upload their images and use images created by others.

#### Advantages of Docker

**i.** **Portability:** Applications run consistently on any environment where Docker is installed, eliminating the "works on my machine" problem.

**ii.** **Efficiency:** Containers share the same OS kernel, which reduces overhead and improves performance compared to traditional virtual machines.

**iii.** **Isolation:** Each container runs in its own isolated environment, ensuring that applications do not interfere with each other.

**iv.** **Scalability:** Easily scale applications horizontally by adding more container instances.

**v.** **Rapid Deployment:** Containers can be started quickly, speeding up the development and deployment process.

#### Use Cases for Docker

**i.** **Microservices:** Each microservice can run in its own container, making it easier to manage, scale, and deploy.

**ii.** **Continuous Integration and Continuous Deployment (CI/CD):** Docker simplifies the build, test, and deployment process by providing consistent environments for each stage.

**iii.** **Development Environments:** Developers can create and share reproducible development environments.

**iv.** **Cloud Migration:** Applications packaged in containers can be easily migrated to different cloud providers.

**v.** **Batch Processing:** Containers can be used for running batch jobs and data processing tasks in a controlled and reproducible manner.

### <br>Install Docker and prepare for migration to the Cloud<br/>

First we need to install [Docker Engine](https://docs.docker.com/engine/install/) which is a client-server application that contains:

- A server with a long-running daemon process dockerd.
- APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon.
- A command-line interface (CLI) client docker.

To install Docker on an Ubuntu instance we take the following steps:

**i.** Set up the repository: 
  
  ```
  sudo apt update -y
  sudo apt upgrade -y
  sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
  ```

![update server](https://github.com/user-attachments/assets/b6b6d019-9c6c-4a60-bfe9-5a6b00fd9afe)

![upgrade server](https://github.com/user-attachments/assets/39e89611-07f4-4de3-81f8-ad014294f931)

![apt install](https://github.com/user-attachments/assets/2e45b264-cc33-4f6d-97f8-b20835d091df)

**ii.** Add Docker’s official GPG key: 

```
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

![add official gpg key](https://github.com/user-attachments/assets/e66a3ccc-c5fc-4e53-9a76-1cde89e19061)

**iii.** Set up Docker repository:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

![add docker repo](https://github.com/user-attachments/assets/69b271cd-f133-42b2-bae2-6ae8e3069c9f)

**iv.** Install Docker Engine:

```
sudo apt update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

![update server 2](https://github.com/user-attachments/assets/05f2c9ff-292a-424e-9887-f263baf2f1d4)

![install docker](https://github.com/user-attachments/assets/d05237fa-a5b7-4b3a-9a1f-e918a7364355)
 
**v.** Verify that Docker Engine is installed correctly by running the hello-world image.

![docker run hello world](https://github.com/user-attachments/assets/06654f7d-b219-43fc-924d-85a6b10c3188)

In this project we will be using the Tooling web application we used in our previous projects, which is a PHP-based web solution backed by a MySQL database. We proceed by migrating the Tooling Web Application from a VM-based solution into a containerized one.

### MySQL in container

We start assembling our application from the Database layer - we will use a pre-built MySQL database container, configure it, and make sure it is ready to receive requests from our PHP application.

#### Step 1: Pull MYSQL Docker image from [Docker Hub Registry](https://hub.docker.com)

**i.** We begin by pulling the appropriate [Docker image for MySQL](https://hub.docker.com/_/mysql):

**`sudo docker pull mysql/mysql-server:latest`**

![docker pull](https://github.com/user-attachments/assets/f29ae0d8-5bd6-47d7-99ea-6139dbb571e4)

**ii.** Then we subsequently enter the following command to list the images and check that we have downloaded them successfully:

**`sudo docker images`**

![docker images](https://github.com/user-attachments/assets/74c45485-2632-489e-ad95-db203c7f4221)

#### Step 2: Deploy the MySQL Container to your Docker Engine

**i.** Once we have the image, we move on to deploying a new MySQL container with:

**`sudo docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest`**

**Notes**
- Replace `<container_name>` with the name of your choice. If you do not provide a name, Docker will generate a random one
- The -d option instructs Docker to run the container as a service in the background
- Replace `<my-secret-pw>` with your chosen password
- In the command above, we used the latest version tag. This tag may differ according to the image you downloaded

![docker run container](https://github.com/user-attachments/assets/3c8834df-75c6-4ee8-9ee1-c5b4eaa92004)

**ii.** Then, we check to see if the MySQL container is running: Assuming the container name specified is **`mysql-server`**

**`sudo docker ps -a`**

```
CONTAINER ID   IMAGE                                COMMAND                  CREATED          STATUS                             PORTS                       NAMES
7141da183562   mysql/mysql-server:latest            "/entrypoint.sh mysq…"   12 seconds ago   Up 11 seconds (health: starting)   3306/tcp, 33060-33061/tcp   mysql-server
```

![container image](https://github.com/user-attachments/assets/625a3aaf-2b33-4b48-9f16-f3354d696fea)

Now  as seen in the image above, we see the newly created container listed in the output. It includes container details, one being the status of this virtual environment. The status changes from `health: starting` to `healthy`, once the setup is complete.

#### Step 3: Connecting to the MySQL Docker Container

We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.

**Approach 1**

Connecting directly to the container running the MySQL server:

**`docker exec -it <container_name> mysql -uroot -p`**

Then we provide the root password when prompted. With that, we have connected the MySQL client to the server.

Finally, we change the server root password to protect our database.

![connect directly to container](https://github.com/user-attachments/assets/c2027ab0-b770-4637-9140-c448f4295841)

**Approach 2**

**Note**

Before proceeding with this approach, we will need to stop and remove the previous mysql docker container.

```
sudo docker ps -a

sudo docker stop <container name> 

sudo docker rm <container name>
```

![remove old container](https://github.com/user-attachments/assets/66c98afc-249d-469c-b092-f5ac7d27d028)

To begin with our second approach, we create a network:

**`docker network create --subnet=172.18.0.0/24 tooling_app_network`**

Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers we run. By default, the network we created above is of **`DRIVER`** **`Bridge`** So, also, it is the default network. You can verify this by running the **`docker network ls`** command.  But there are use cases where this is necessary. For example, if there is a requirement to control the **`cidr`** range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the **`--subnet`**

![create docker network](https://github.com/user-attachments/assets/3d77aa8a-fd37-4558-b852-9d3a94ed06a0)


For clarity's sake, we will create a network with a subnet dedicated for our project and use it for both MySQL and the application so that they can connect.

We run the MySQL Server container using the created network.

First, let us create an environment variable to store the root password:

**`export MYSQL_PW=<root-secret-password>`**

Then, we pull the image and run the container, all in one command like below:

```
docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest 
```

![pull image and run container](https://github.com/user-attachments/assets/21c58148-9d5d-48e0-b6e5-1b1cde7193a1)

**Notes**

Flags used:

- **`-d`** runs the container in detached mode
- **`--network`** connects a container to a network
- **`-h`** specifies a hostname

If the image is not found locally, it will be downloaded from the registry.

Subsequently, we verify the container is running:

**`docker ps -a`**

```
CONTAINER ID   IMAGE                                COMMAND                  CREATED          STATUS                             PORTS                       NAMES
7141da183562   mysql/mysql-server:latest            "/entrypoint.sh mysq…"   12 seconds ago   Up 11 seconds (health: starting)   3306/tcp, 33060-33061/tcp   mysql-server
```

![confirm container is running](https://github.com/user-attachments/assets/5f856ce0-1a27-40d3-bcec-3aba7dca2fa9)

As you already know, it is best practice not to connect to the MySQL server remotely using the root user. Therefore, we will create an **`SQL`** script that will create a user we can use to connect remotely.

We create a file and name it **`create_user.sql`** and add the below code in the file:

```
CREATE USER '<user>'@'%' IDENTIFIED BY '<client-secret-password>';
GRANT ALL PRIVILEGES ON * . * TO '<user>'@'%';
```

Next, we run the script:

**`docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < ./create_user.sql`**

![create sql user and run script](https://github.com/user-attachments/assets/1a86046f-1b1e-4c26-8c71-4b1186bd4a2f)

##### Connecting to the MySQL server from a second container running the MySQL client utility

The good thing about this approach is that we do not have to install any client tool on our local machine, and we do not need to connect directly to the container running the MySQL server.

We run the MySQL Client Container:

```
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u <user-created-from-the-SQL-script> -p
```

![connect to mysql server](https://github.com/user-attachments/assets/db7ad184-8156-4e4a-afb7-2e640bc8de98)

**Notes**

Flags used:

- **`--name`** gives the container a name
- **`-it`** runs in interactive mode and Allocate a pseudo-TTY
- **`--rm`** automatically removes the container when it exits
- **`--network`** connects a container to a network
- **`-h`** a MySQL flag specifying the MySQL server Container hostname
- **`-u`** user created from the SQL script
- **`-p`** password specified for the user created from the SQL script
 
#### Prepare database schema 

Now we need to prepare a database schema so that the Tooling application can connect to it.

**i.** Clone the Tooling-app repository from [here](https://github.com/darey-devops/tooling)

**`git clone https://github.com/darey-devops/tooling.git`**

![git clone](https://github.com/user-attachments/assets/34cd4257-f198-45b8-8eec-55f9371e92d6)

**ii.** On your terminal, export the location of the SQL file

**`export tooling_db_schema=~/tooling/html/tooling_db_schema.sql`**

![export tooling db schema](https://github.com/user-attachments/assets/e032a5ad-c957-43df-b8f5-c3f36931fd52)

We can find the **`tooling_db_schema.sql`** in the **`html`** folder of cloned repo.

**iii.** Use the SQL script to create the database and prepare the schema. With the **`docker exec`** command, you can execute a command in a running container.

**`docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema`**

![create database and prepare schema](https://github.com/user-attachments/assets/35fa3f1d-b6ef-4d1c-aeba-9b704a29e976)

**iv.** Update the **`db_conn.php`** file with connection details to the database.

**`sudo vi db_conn.php`**

```
$servername = "mysqlserverhost";
$username = "<user>";
$password = "<client-secret-password>";
$dbname = "toolingdb";
```

![update db conn php](https://github.com/user-attachments/assets/fb66dd1b-0387-4565-aa79-dcdad2f7c578)

**v.** Also update the **`.env`** file with connection details to the database The .env file is located in the html tooling/html/.env folder but not visible in terminal.

**`sudo vi .env`**

```
MYSQL_IP=mysqlserverhost
MYSQL_USER=<username>
MYSQL_PASS=<client-secret-password>
MYSQL_DBNAME=toolingdb
```

![edit env](https://github.com/user-attachments/assets/3b896384-0c1d-463d-8da6-2ff886be9988)

**vi.** Run the Tooling App

Containerization of an application starts with creation of a file with a special name - **`Dockerfile`** (without any extensions). This can be considered as a 'recipe' or 'instruction' that tells Docker how to pack your application into a container. In this project, we will build our container from a pre-created **`Dockerfile`**, but as a DevOps engineer, we must also be able to write Dockerfiles. 

We proceed to watch [this video](https://www.youtube.com/watch?v=hnxI-K10auY) to get an idea how to create your `Dockerfile` and build a container from it.

And on [this page](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/), we find the official Docker best practices for writing Dockerfiles.

Next, we move on to containerize our Tooling application; here is the plan:

- Ensure we have checked out your Tooling repo to your machine with Docker engine
- First, we need to build the Docker image the tooling app will use. The Tooling repo we cloned above has a `Dockerfile` for this purpose. we need to explore it and make sure we understand the code inside it.
- We run `docker build` command
- We launch the container with `docker run`
- We try to access our application via port exposed from a container

Now we proceed:

We ensure we are inside the folder that has the `Dockerfile` and then we build our container with the following command:

**`docker build -t tooling:0.0.1 .`**

![docker build](https://github.com/user-attachments/assets/895dae32-8b66-46e8-9f53-d5c4126cab69)

In the above command, we specify a parameter **`-t`**, so that the image can be tagged **`tooling"0.0.1`** - Also, notice the **`.`** at the end. This is important as it tells Docker to locate the **`Dockerfile`** in the current directory we are running the command. Otherwise, we would need to specify the absolute path to the **`Dockerfile`**.

**vii.** Run the container:

**`docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1`**

![docker run final](https://github.com/user-attachments/assets/d15421b8-131e-4654-bb69-f7d5e36d0f1f)

**Notes**

Flags used:

- We need to specify the **`--network`** flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
- The **`-p`** flag is used to map the container port with the host port. Within the container, **`apache`** is the webserver running and, by default, it listens on port 80. This can be confirmed with the **`CMD ["start-apache"]`** section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

#### Blocker 

![blocker](https://github.com/user-attachments/assets/4ef71ffe-4f92-4ad2-983d-5aef9e7c5193)

To resolve the issue the following steps were followed:

###### Step 1: Install a Text-Based Web Browser

We installed `lynx` which is a commonly used text-based web browsers.

```
sudo apt update
sudo apt install lynx
```

###### Step 2: Create a Symbolic Link to `www-browser`

Once we installed `lynx` or `w3m`, we created a symbolic link to `www-browser`.

```
sudo ln -s /usr/bin/lynx /usr/bin/www-browser
```

###### Step 3: Verify the Installation

We ran the following command to verify that the symbolic link is working:

```
www-browser -dump http://localhost:80/server-status
```

###### Step 4: Adjust the `APACHE_LYNX` Variable

We proceeded to adjust the `APACHE_LYNX` variable in the `/etc/apache2/envvars` file to point to the correct path of the text-based browser.

- Open the `/etc/apache2/envvars` file in a text editor:

```
sudo nano /etc/apache2/envvars
```

- Add or modify the line to point to the correct browser:

```
export APACHE_LYNX='/usr/bin/lynx'
```

- Save the file and exit the text editor.

- Restart Apache to apply the changes:

```
sudo systemctl restart apache2
```

By following these steps, we were able to resolve the issue with the `www-browser` command not being found and properly fetch the Apache server status page.


If everything works, you can open the browser and type `http://localhost:8085`

You will see the login page.

![website success 1](https://github.com/user-attachments/assets/ab704455-e0be-40e0-b729-33e1dab63277)

The default email is `test@gmail.com`, the password is `12345` or you can check users' credentials stored in the `toolingdb.user` table.

![website successs 2](https://github.com/user-attachments/assets/a853589b-0161-446e-ac79-b6dc7f99bd96)
