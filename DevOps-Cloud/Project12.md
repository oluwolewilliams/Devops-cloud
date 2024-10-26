## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)
 
In this project, we will continue working with **`ansible-config-mgt`** repository and make some improvements on our code. We will refactor our Ansible code, create assignments and learn how to use the import functionality. Imports enables the ability to effectively re-use previously created playbooks in a new playbook. In essence, it allows us to organise our tasks and reuse them when necessary. 

### <br>Introduction to Code Refactoring<br/>
 
The goal of this project is to demonstrate how Ansible refactoring works and its usefulness. [Refactoring](https://en.wikipedia.org/wiki/Code_refactoring) is a general term in computer programming. It means making changes to the source code without changing the expected behaviour of the software. The main idea of refactoring is to enhance code readability, increase maintainability and extensibility, reduce complexity and add proper comments without affecting the logic. In this project, we will be making some slight changes to the code but the overall state of the infrastructure shall remain the same. This project shall consist of three parts:

**1.** Refactor Ansible Code.

**2.** Configure UAT Webservers with Roles.

**3.** Reference Webserver Role.

### <br>Refactor Ansible Code <br/>

In the first part of our project, we shall refactor Ansible code by importing other playbooks into **`site.yml`**. However, before we begin, we need to make some changes to our Jenkins Job. 

#### <br>Step 1: Jenkins Job Enhancement<br/>

With the way our Jekins job is currently configured, every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. To enhance it, we shall be introducing a new Jenkins project/job and we will require **`Copy Artifact`** plugin.

**i.** In our **`Jenkins-Ansible`** server, we use the comand below to create a new directory called **`ansible-config-artifact`** – which we will be using to store all artifacts after each build.

**` $ sudo mkdir /home/ubuntu/ansible-config-artifact`**

**ii.** Then with the following command, we change permissions for this directory, so that Jenkins would be able to save files there:

**` $ chmod -R 0777 /home/ubuntu/ansible-config-artifact`**

![mkdir-chmod](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9b18781a-c240-4219-a69c-7650aef46f9e)

**iii.** Next, we will need to install the **`copy artifact`** plugin without restarting Jenkins:

+ From the main Jenkins Environment, we click on **"Manage Jenkins"**.

 ![manage jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/490e07d7-4916-493e-bb16-f541914e99ff)

+ On the System Configuration page, we click on **"Plugins"**.

 ![plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9f23b0ef-46e5-43ae-a876-ffb499a7896d)

+ On the Plugins page, we click on Available plugins, then we type **`copy artifact`** in the search box.

 ![copy artifact plugin](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/853dd4bf-213f-4ced-afc2-551766f50411)

+ From here, we can see the **`copy artifact`** plugin we wish to install, so we check the **"Install"** checkbox beside it and we click on the **"Install"** button. After doing this we will be able to see the download progress page.

![download progress](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/092efd2f-0771-4987-ad3a-de3c189c547f)

**iv.** Next, we create a new Freestyle project (as we did in [Project 11](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project11.md)) and we name it **`save_artifacts`**:

+ From the Jenkins web console, we click on **"New item"**

![jenkins new item](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6dd5ad81-1d92-4908-813f-eecce8717871)

+ In the next page under **"Enter an item name"** we type in **`save_artifacts`**, then we select **"Freestyle project"** and we click on **"Ok"** at the bottom of the page.

![save-artifacts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a6d3b084-9bcf-4cba-a2cd-c99d4680b78a)

**v.** This project will be triggered by the completion of our existing **`ansible`** project. So therefore we configure it accordingly:

+ In the Jenkins configuration page, under **"General"** we click on check box for **"Discard old builds"** and under **"Max # of builds to keep"** we enter our desired number.

![discard old builds](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e356d83f-2a65-43ff-a46c-0525726eafad)

+ Next under **"Build Triggers"**, we click on the check box beside **"Build after other projects are built"** and under the **"Projects to watch"** dialogue box we type in **`ansible`**.

![build trigger](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4342fb71-c919-4146-9d71-f6f62837b38a)

**vi.** The main idea of the **`save_artifacts`** project is to save artifacts into the **`/home/ubuntu/ansible-config-artifact`** directory. To achieve this, we proceed as follows:

+ Under **"Build Steps"** we click on the **"Add build step"** drop down button and we select **"Copy artifacts from another project"**.

 ![build steps](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8e593233-ab61-4346-9968-8aaa52f0f646)

+ Under **"Project name"**, we specify **`ansible`** as the source project, under **"Artifacts to copy"**, we input __**__ to copy all artifacts and then we put in  **`/home/ubuntu/ansible-config-artifact`** as the **"Target directory"**.

 ![copy artifacts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1e887922-96f9-4ed9-b8be-334a8b01dbaa)

+ Then we click on **"Apply"** and **"Save"** at the bottom of the page.

![apply and save](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ab54ff0a-83b4-41cd-b599-f11164d6e3a6)

**vii.** The next thing we do is to test our set up by making some changes in the **README.MD** file inside our **`ansible-config-mgt`** repository (right inside the **`main`** branch). If both Jenkins jobs have completed one after another – we shall see our files inside the **`/home/ubuntu/ansible-config-artifact`** directory and it will be updated with every commit to our **`main`** branch.

![console output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b56eac24-c25a-4fa9-862d-624168008446)

As seen in the above image, the **`save_artifacts`** job was successfully triggered and completed. To confirm that our files are inside the **`/home/ubuntu/ansible-config-artifact`** directory, we execute the following command:

**` $ ls /home/ubuntu/ansible-config-artifact`**

![confirmation ansible artifacts](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a1a1831f-57de-491d-997a-c0519e88061e)

#### <br>Step 2: Refactor Ansible code by importing other playbooks into `site.yml`<br/>

**i.** Before starting to refactor the codes, we need to pull down the latest code from the **`main`** branch, and create a new branch, which we will name **`refactor`**:

+ From the VS code GUI in the **`main`** branch of our ansible-config-mgt repository, we click on the **"Views and More Actions"** button then we select **"pull"** to pull down the latest code from the **`main`** branch.

 ![pull latest code](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/19605205-1fa8-4317-9248-f13ace4be134)

+ From the VS Code environment we go to the bottom of the page and we click on **`main`**.

+ ![create new branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/659f1c32-4fa4-4d80-a97f-1c53bb1d70a1)

+ Afterwards, we enter the new branch name **`refactor`** and we press **Enter**.

![branch name](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ef38ba25-19e3-4420-bda2-63182009bd1a)

+ Then we subsequently click on the **"Publish Branch"** button.

![publish branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b5deffd8-4b4d-4451-bdc7-77cfde70d2f9)


**ii.** Next, we enter the command below to checkout the newly created branch **`refactor`** to our local machine and start refactoring our code:

**`$ git checkout refactor`**

![git checkout refactor](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d77bf426-8555-43e4-9fc6-342f6758f0b5)

**iii.** Then within the playbooks folder, we create a new file and name it **`site.yml`** – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, **`site.yml`** will become a parent to all other playbooks that will be developed. This is including **`common.yml`** that we created previously.

```
$ cd playbooks/

$ touch site.yml
```

![touch site yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f2cfb881-26a8-470d-9fec-5a622e0cee29)

**iv.** The next thing we do is to create a new folder in the root of the repository and name it **`static-assignments`**. This is where all other children playbooks will be stored. This is merely for easy organization of our work and it is therefore not an Ansible specific concept as we can always choose to organise our work in different ways.

**`$ mkdir static-assignments`**

![mkdir static assignments](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2ecd13b7-9150-4e55-b668-34ab228cff33)

**v.** Next we move **`common.yml`** file into the newly created **`static-assignments`** folder:

**` $ sudo mv playbooks/common.yml static-assignments/`**

![move playbooks to static assignments](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5e9a7a21-80b3-4284-9999-9563b6ba6449)

**vi.** Then from inside the **`site.yml`** file, we import the **`common.yml`** playbook.

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```

![import playbooks in site yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5f6f9b82-fd90-4011-a9d9-6a5fff435574)

**vii.** At this point, our folder structure is as shown in the image below:

![folder structure](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a828d7d7-a213-4c6b-aa99-f393804de278)

#### <br>Step 3: Run `ansible-playbook` command against the `dev` Environment<br/>

Since we need to apply some tasks to our **`dev`** servers and **`wireshark`** is already installed – we proceed to create another playbook under **`static-assignments`** and name it **`common-del.yml`**. However, in this playbook, we configure the deletion of wireshark utility.

**i.** After moving into the **`static-assignments`** directory, we create a new playbook with the following command:

**`$ touch common-del.yml`**

![common-del-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d122b8c2-b15c-4830-9cd7-9ae87f73bfb7)

**ii.** Next, we open up the newly created file in VS Code and we paste in the following configuration:

```
---
- name: update web and nfs servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB and db servers
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```

![edit common-del-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1f61c947-78eb-478a-9c56-a474df41b7da)

**iii.** After completing our configuration, we update **`site.yml`** with **`- import_playbook: ../static-assignments/common-del.yml`** instead of **`common.yml`** as shown below:

```
---
- hosts: all
- import_playbook: ../static-assignments/common-del.yml
```

![edit sites-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5c0c23e7-49a0-45de-9592-3c512a530141)

**iv.** Then we proceed to run the playbook against the **`dev.yml`** servers in our inventory file:

```
$ cd /home/ubuntu/ansible-config-mgt/

$ ansible-playbook -i inventory/dev.yml playbooks/site.yml
```

![run playbook](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/dd078e3b-c656-4303-a55a-3b3a64102997)

**v.** After successfully running our playbook, we confirm that **`wireshark`** has indeed been deleted from all the servers by executing the following commands:

```
$ ssh <user@public-IP-address>

$ which wireshark

$ wireshark --version
```

![which wireshark](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce2f6924-52d0-404d-8db7-917a7781daa6)

### <br>Configure UAT Webservers with Roles <br/>

Writing tasks to configure Web Servers in the same playbook we used earlier would make our configuration cumbersome, so instead we decide to make use of a dedicated role which ensures that our configuration would be reusable.

#### <br>Step 1: Provision Two (2) EC2 Instances of RHEL 8<br/>

Our first step is to launch 2 fresh Red Hat Enterprise Linux 8, EC2 instances that we will be using as our **`UAT`** servers, and which would be named accordingly as – **`Web1-UAT`** and **`Web2-UAT`**.

**i.** We begin by spinning up an EC2 Instance of Red Hat Linux. We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

 **ii.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**iii.** Under **Name and tags**, we provide a unique name for our server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e77b17db-21b0-40d6-ae57-030cac0e40b1)
  
**iv.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Red Hat Enterprise Linux 8 (HVM).

![application and OS images](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2beaaf89-c746-4ed5-a434-f0989bfb3db1)

**v.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![keypair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9a924391-71e8-4bea-babf-c3dacb6f78b1)

**vi.** Under the Network settings tab, we click on **"Edit"** then under **"Subnet Info"** we click on the dropdown and select the same subnet and availability zone that is in use by our NFS Server.

![network settings subnet](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce8dc829-2da0-444e-b3c7-7bf969be6bf5)
  
**vii.** And then, we click on **"Launch Instance"**

![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

**viii.** At the end of this process, we have our two UAT instances up and running.

![web uat instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a98ee202-8407-49de-bd03-fdb35d513b4e)

#### <br>Step 2: Create Roles Directory<br/> 

To create a role, we must create a directory called **`roles/`**, relative to the playbook file or in the **`/etc/ansible/`** directory. 

**i.** To create this folder structure, we decide to use the **`ansible-galaxy`** utility inside the **`ansible-config-mgt/roles`** directory. We execute by using the following commands:

```
$ mkdir roles

$ cd roles

$ ansible-galaxy init webserver
```

![init ansible galaxy](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1c8eb3e1-734b-4b73-8075-7ccc9f0d6be0)

**ii.** At this point, the folder structure looks exactly as shown in the image below.

![roles directory structure 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65eb1cd9-cbd6-48a3-b623-5477f7ed34b7)

**iii.** Next, we remove the unnecessary folders **`tests`** and **`vars`** and their contents. 

```
$ rm -r tests

$ rm -r vars
```

![remove unnecessary files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8d117f4b-8a3c-4883-8279-2d392c431212)

**iv.** We do not have the templates folder in our directory structure so we manually create it:

**`$ mkdir templates`**

![mkdir templates](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cb13ba69-a68d-473b-8315-e6a74a97950c)

**v.** After cleaning up our directory, the **`roles`** folder structure looks exactly as shown below:

![directory structure 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2cc6dc1c-fd46-4ad7-bac9-9f38064d3dae)

#### <br>Step 3: Update Configuration for UAT Webservers<br/>

**i.** We proceed to update the inventory **`ansible-config/inventory/uat.yml`** file with the Private IP addresses of the 2 UAT Webservers using the following syntax:

```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user=ec2-user
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user=ec2-user
```

![edit uat-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c68884d3-d8e5-495b-8bc8-9acfb0da5c4b)

**ii.** Next, we navigate with the VS Code file explorer to the **`/etc/ansible/ansible.cfg`** file and uncomment the **`roles_path`** string and provide a full path to our roles directory **`roles_path = /home/ubuntu/ansible-config/roles`**, to enable Ansible know where to find configured roles.

#### BLOCKER❗

+ We could not find the **`ansible/ansible.cfg`** directory in **`/etc`** so we decided to run the following command to get information about our ansible version and the location of its configuration file.

**`$ ansible --version`**

![ansible --version no config file](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cc2fa71a-1cfd-4500-9afd-aacac81e5168)

+ As can be seen in the above image, there is no configuration file present in our ansible set up. So we decide to execute the command below to generate a fully commented-out ansible.cfg file in the current working directory as specified in the [official ansible documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings):

**`$ ansible-config init --disabled -t all > ansible.cfg`**

![ansible init error](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7a6f4959-754b-492f-97ec-647749775af9)

+ As shown in the output image above, we get an error that the **`init`** command is not supported for our current ansible version. So we proceed to update the ansible repo and install a more recent ansible version that supports the **`init`** command:

```
$ sudo add-apt-repository --yes --update ppa:ansible/ansible

$ sudo apt install ansible-core
```

![update ansible repo](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/85c07a88-66e7-4319-97a3-ceafef8e96fe)

![install ansible core](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d8df087a-6d49-4a3a-ab35-ca5d741154a0)

+ After completing the installation, we proceed to run the following command again:

**`$ ansible-config init --disabled -t all > ansible.cfg`**

![ansible init successful](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f5405d52-14eb-4d2d-97da-d18f7eab91d4)

+ As seen in the output image above, we are successful this time. Next, we navigate to our current working directory via VS Code Explorer and now we can see and access the **`ansible.cfg`** file. 

![ansible-cfg file](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/41c0ccb9-13ad-40e8-b262-66192febdf48)

+ After overcoming our **BLOCKER** we navigate with the VS Code file explorer to the **`/etc/ansible/ansible.cfg`** file and uncomment the **`roles_path`** string and provide a full path to our roles directory **`roles_path = /home/ubuntu/ansible-config/roles`**, to enable Ansible know where to find configured roles.

![roles-path](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8d68cc8f-f28d-47ba-9eaf-263fff911fd9)

**iii.** The next step is to start adding some logic to the webserver role. We move into the **`tasks`** directory, and within the **`main.yml`** file, we proceed to write configuration tasks to do the following:

+ Install and configure Apache (**`httpd`** service)

+ Clone Tooling website from GitHub **`https://github.com/<your-name>/tooling.git`**.
  
+ Ensure the tooling website code is deployed to **`/var/www/html`** on each of 2 UAT Web servers.
  
+ Make sure **`httpd`** service is started.

**iv.** After completing the configuration, our **`main.yml` file consists of the following tasks:

```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/QBDev0ps/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

![main-yml configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0f6dc885-5432-41f8-9d02-43aab7299e05)

### <br>Reference Webserver Role <br/>

#### <br>Step 1: Reference Web Server Role in `static-assignments` <br/>

**i.** We start by navigating to the **`static-assignments`** folder, and within this directory, we execute the command below to create a new assignment for **uat-webservers** which we name as **`uat-webservers.yml`**. 

**`$ touch uat-webservers.yml`**

![create uat-webservers-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/faa08ed6-bed2-4ab7-821d-74f0b05213c9)

**ii.** The next step is to open up the **`uat-webservers.yml`** file via the VS Code file explorer and then we paste in the following configuration to reference the webserver role:

```
---
- hosts: uat-webservers
  roles:
     - webserver
```

![uat-webservers-yml-configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7abdfd35-970a-417a-ad22-416a6c591818)

**iii.** Considering that the entry point to our ansible configuration is the **`site.yml`** file, we therefore need to reference our **`uat-webservers.yml`** role inside **`site.yml`**. So, we navigate via the VS Code file explorer and put the following configuration inside **`site.yml`**:

```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

![site yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/227bcd76-645d-4c4c-b6e4-278a41bc5431)

#### <br>Step 2: Commit and Test <br/>

In this step, we shall commit our changes, create a Pull Request and then merge the changes in the refactor branch to the main branch. Afterwards we will make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to our Jenkins-Ansible server into **`/home/ubuntu/ansible-config/`** directory.

**i.** Using the following set of commands, we add all untracked files to the staging area and then we commit the changes:

```
$ git add .

$ git commit -m "saved all updates to refactor branch"
```

![commit changes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/73a838ce-92b8-435a-bd49-471ea730f418)

**ii.** Next we push the refactor branch to our github repository using the following command:

**`$ git push --set-upstream origin refactor`**

![git push refactor](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ff3cf491-b762-495c-b240-c7407a3cf6c3)

**iii.** The next thing we do is to create a **Pull Request** in GitHub by following [these steps:](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 

+ From the **`ansible-config-mgt`** repository page, we click on the **"Pull requests"** tab and then in the next page we click on the **"New pull request"** button.

![pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a27f564c-53f7-49c8-9905-aa7e440fc17b)

![new pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/46e0162b-0cb7-4ff4-8514-6fd0c8ab0a7c)

+ This takes us to the **"Compare changes"** page where we choose the **`refactor`** branch to set up a comparison with the **`main`** branch.

![compare changes refactor](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/408ec46e-bd47-4c5b-999e-af4bd96f26f1)

+ Once we set up the comparisons between the **`main`** and the **`refactor`** branch, we then proceed to click on the **"Create pull request"** button.

![create pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8b9baea3-2856-4787-ba57-1c0e9a2dce4c)

+ In the next page, we input a pull request message inside the dialogue box and we click on the  **"Create pull request"** button.

![pull request 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/52c38f08-6b60-4a92-b637-01adda2298a6)

**iv.** Now as shown in the image below, we act as a reviewer and we examine the changes in the **`refactor`** branch and check for conflicts with the **`main`** branch.

![merge pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c6f96909-f948-42a4-8734-736ba64383ec)

**v.** As we are satisfied and happy with the changes made in **`refactor`**, we click on **"Merge pull request"** and then we click on **"Confirm merge"**

![confirm merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/39b7e91f-52bb-4c81-b01f-1f34467ca43d)

**vi.** This takes us to the next page which shows that **`refactor`** has been successfully merged to **`main`** branch.

![successful merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e88ae9b7-3b67-4201-a146-6b250d6dd9ac)

**vii.** As can be seen in the console output images below, our existing webhook configuration in Github and Jenkins triggered the **`ansible`** build job which upon successful completion consequently triggered the **`save_artifact`** build job. Both jobs ran successfully and copied all the build artifacts/files to our **`Jenkins-Ansible`** server into the **`/home/ubuntu/ansible-config/`** and **`/home/ubuntu/ansible-config-artifact/`** directories respectively.

![ansible jenkins console output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1fd00eef-be52-4ea0-a86d-905749a099e2)

![save-artifact console output](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a1994a93-72ab-43d7-b94d-1777a445afa7)

**viii.** The next step is to connect to our **`Jenkins-Ansible`** server via **`ssh-agent`** as we did in [Project 11](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project11.md) and then with the commands below, we run the playbook against our **`uat inventory`**.

```
$ cd /home/ubuntu/ansible-config-mgt

$ ansible-playbook -i /inventory/uat.yml playbooks/site.yml
```

#### BLOCKER❗

![ansible blocker](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca0229e6-390b-4062-8710-6935eb9cadf8)

+ We got the error above when we ran the playbook, but after we examined the error message and did some investigations we realised it was a permision issue. Our current user did not have the adequate rights to **`/inventory/uat.yml`**. So to fix this, we executed the following command:

**`sudo chmod 744 inventory/uat.yml`**

+ And then we ran our playbook again:

**`$ ansible-playbook -i /inventory/uat.yml playbooks/site.yml`**

![playbook successful 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0154d7cb-c1e6-4f8f-833a-ec688b2156f1)

![playbook successful 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/78d77ff5-4c44-4f35-8322-49a5b77c7d69)

**ix.** As can be seen in the images above, we resolved the **BLOCKER** and our playbook ran successfully. 

**x.** To confirm our playbook executed the tasks properly and complete the project, we go to our browser and try to reach the UAT web servers using the syntax below:

**`http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php`**

![uat 1 successful](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/fe1830a0-77b4-4f01-81bd-82007c9b6935)

![uat 2 successful](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/87f650f1-576c-49ba-8b8c-15e32004f7f6)

### <br>Conclusion<br/>

The web browser images above show that we have been able to successfully deploy and configure two UAT Web Servers using Ansible **`imports`** and **`roles`** and our setup looks somewhat like what we have in the image below.

![final architecture](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce5a4765-fdfc-446c-95e8-391188b16134)

The goal of this project was to demonstrate how Ansible refactoring works to  enhance code readability, increase maintainability and extensibility. We began the project by enhancing our existing Jenkins Job **`ansible`** with the introduction of a new Jenkins project/job **`save_artifacts`** which required the use of the **`Copy Artifact`** plugin. After this, we pulled down the latest code from the **`main`** branch, and create a new branch **`refactor`** where we began making changes and refactoring ansible code. Essentially, we created a new file **`site.yml`** to not only be seen as a parent to all other playbooks, but also used as an entry point into the entire infrastructure configuration and we included and referenced our other children playbooks from here. To ensure our work stays well organized, we also created a folder **`static-assignments`** in the root of our repository to be used as storage for all our children playbooks. 

The next phase of the project involved using **`roles`** to configure two UAT web servers. We launched 2 fresh Red Hat Enterprise Linux 8, EC2 instances to be used as the **`UAT`** servers, and then in our **`Jenkins-Ansible`** server, we created the **`roles/`** directory and used the **`ansible-galaxy`** utility to generate the appropriate folder structure. The next step was to update the relevant inventory file with the Private IP addresses of the 2 UAT Webservers. Then we attempted to update the ansible configuration file to enable Ansible know where to find configured roles but we ran into a **BLOCKER** here. The **`ansible.cfg`** file was absent from our setup so we had to manually generate it and make the necessary configuration changes.

Afterwards, we started adding some logic to the webserver role and we wrote configuration tasks to install and configure Apache (**`httpd`** service), clone Tooling website from GitHub **`https://github.com/<your-name>/tooling.git`**, ensure the tooling website code is deployed to **`/var/www/html`** on each of 2 UAT Web servers and make sure **`httpd`** service is started. Then we referenced our Web Server Role in the **`static-assignments`** folder. Next, we committed our changes, created a Pull Request and then merged the changes from the refactor branch to the main branch. Afterwards we made sure sure webhook triggered two consequent Jenkins jobs that ran successfully and copied all the files to our Jenkins-Ansible server into **`/home/ubuntu/ansible-config/`** directory.The next step was to connect to our **`Jenkins-Ansible`** server via **`ssh-agent`** as we did in [Project 11](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project11.md) and then run the playbook against our UAT inventory file where we configured the Private IP addresses of the 2 UAT Webservers. We ran into another blocker here as ansible was unable to gain access to our inventory directory but we were able to rectify this by fixing the permissions of the directory. From here onwards, our playbook ran successfully and via our browser we were able to access the tooling website deployed on our two UAT servers. Thank you.
