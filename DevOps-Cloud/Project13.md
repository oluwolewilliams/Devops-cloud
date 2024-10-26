## ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

[Dynamic assignments](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html#includes-dynamic-re-use) in ansible refer to the use of variables whose values are calculated or determined at runtime. This is in contrast to static assignments, where the values of variables are defined and fixed at the time of writing the playbook. In this project, we will continue configuring our UAT Servers whilst learning and practicing new Ansible concepts and modules. We shall discover the flexibility of Ansible with dynamic assignments (include) and  we shall leverage on Ansible pre-built solutons to expand our automation capabilities.

### Project Dependencies

In order to successfully execute this project, the following prerequisites need to be in place:

1. **`Jenkins-Ansible`** Server from [Project 12.](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project12.md)
   
2. Database Server Instance running Ubuntu Linux Distribution.
  
3. Loadbalancer Instance running Ubuntu Linux Distribution.
   
4. Two UAT Web Server Instances running Red Hat Enterprise Linux Distribution.

### <br>Introduction to Ansible Dynamic Assignments (include) and Community Roles<br/>

Dynamic assignments are useful in scenarios where the values of variables need to be determined based on certain conditions or inputs. From our work in [Project 12](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project12.md) we can surmise that static assignments make use of the **`import`** ansible module. However, on the other hand, the module that enables dynamic assignments is the **`include`** ansible module.

```
import = Static
include = Dynamic
```

When the **`import`** module is used, all statements are pre-processed at the time playbooks are parsed. This means that when the **`site.yml`** playbook is executed, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

On the other hand, when **`include`** module is used, all statements are processed only during execution of the playbook. This means that after the statements are parsed, any changes to the statements encountered during execution will be used.Although it is quite notable that static assignments are mostly used for playbooks (due to the fact that Dynamic assignments are less reliable and can be difficult to debug). Dynamic assignments have however proved useful for environment specific variables as will be introduced in this project. This project shall consist of two parts:

**1.** Making use of Dynamic Assignments.

**2.** Making use of Community Roles.

### <br>Making use of Dynamic Assignments<br/>

#### <br>Step 1: Introducing Dynamic Assignment Into the Structure<br/>

**i.** In our GitHub repository, we execute the following command to create and move into a new branch that we will call **`dynamic-assignments`**.

**`$ git checkout -b dynamic-assignments`**

![git checkout -b](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8da27569-6e77-4652-9439-0f970b4135f4)

**ii.** Next we create a folder and name it **`dynamic-assignments`**

**`$ mkdir dynamic-assignments`**

![mkdir dynamic assignments](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/075643a0-5b53-48ef-a951-00ce266c5fb6)

**iii.** We use the set of commands below to move into the dynamic-assignments folder and create a file named **`env-vars.yml`**

```
$ cd dynamic-assignments

$ touch env-vars.yml
```

![cd touch env-vars](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/49eea68b-0195-497f-8240-176b35354bcc)

**iv.** At this point, our GitHub directory structure is as shown in the image below:

![directory structure 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/af3e864d-ac1f-4074-afb6-a20ed5936b1c)

**v.** Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment. For this reason, we proceed to create a folder **`env-vars`** to keep each environment’s variables file.

**`$ mkdir env-vars`**

![mkdir env-vars](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/43692f09-2d1f-438e-80dc-cacc2d5f2fac)

**vi.** Then, we move into the created **`env-vars`** folder, and for each environment, we create new **`YAML`** files which we will use to set variables.

```
$ cd env-vars

$ touch dev.yml stage.yml uat.yml prod.yml
```

![cd env-vars and create files](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/00de21e8-3e20-4fdc-ab9e-712a0a099c91)

**vi.** Our directory layout now looks as shown in the image below:

![directory structure 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e97bad3a-856d-47fe-9049-50a67396e8d1)

**vii.** Now satisfied with our folder structure, we move ahead and paste the following block of instruction into the **`env-vars.yml`** file.

```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```

![env-vars-yml config](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f251f135-e39a-4ba6-8b02-d9db467fef96)

**viii.** There are three things to note about the configuration above in **vii.**

1. We used **`include_vars`** syntax instead of **`include`**, this is because Ansible developers decided to separate different features of the module. From Ansible version **2.8**, the **`include module`** is deprecated and variants of **`include_*`** must be used. These are:

+ [include_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html#include-role-module)
  
+ [include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html#include-tasks-module)
  
+ [include_vars](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_vars_module.html#include-vars-module)

  
In the same version, variants of **`import`** were also introduced, such as:

+ [import_role](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_role_module.html#import-role-module)
  
+ [import_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html#import-tasks-module)
  
2. We made use of [special variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) **`{{ playbook_dir }}`** and **`{{ inventory_file }}`**. **`{{ playbook_dir }}`** will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. **`{{ inventory_file }}`** on the other hand will dynamically resolve to the name of the inventory file being used, then append **`.yml`** so that it picks up the required file within the **`env-vars`** folder.

3. We are including the variables using a loop. **`with_first_found`** implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

#### <br>Step 2: Update `site.yml` with Dynamic Assignments<br/>

The next step is to update the **`site.yml`** file to make use of the dynamic assignment. (It is however important to note that we cannot test it just yet. Rather, this is us preparing the stage for what is to come.) **`site.yml`** should look like this:

```
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```

![site-yml config](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0e49a73d-e805-4f00-b778-1f5a58ef01d2)

#### <br>Step 3: Update Git with Latest Code in `dynamic-assignments` Branch<br/> 

At this point, we we need to push all the changes we made locally to our remote Github repository.

**i.** We use the following commands to stage, commit and push our branch to GitHub:

```
$ git status

$ git add <selected files>

$ git commit -m "commit message"

$ git push --set-upstream origin dynamic-assignments
```

![git push set upstream](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/753db294-3be8-4b12-8abb-3a911d8148e9)

**ii.** The next thing we do is to create a **Pull Request** in GitHub by following [these steps:](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 

+ From the **`ansible-config-mgt`** repository page, we click on the **"Pull requests"** tab and then in the next page we click on the **"New pull request"** button.

![pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a27f564c-53f7-49c8-9905-aa7e440fc17b)

![new pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/46e0162b-0cb7-4ff4-8514-6fd0c8ab0a7c)

+ This takes us to the **"Compare changes"** page where we choose the **`dynamic-assignments`** branch to set up a comparison with the **`main`** branch.

![compare changes](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/6eddc09c-5460-4d07-a11b-ac6e38daaeb0)

+ Once we set up the comparisons between the **`main`** and the **`dynamic-assignments`** branch, we then proceed to click on the **"Create pull request"** button.

![comparing changes create pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/85e25bdb-6afc-4a40-a644-73389a193d5d)

+ In the next page, we input a pull request message inside the dialogue box and we click on the  **"Create pull request"** button.

![open a pull request](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f8b48d19-4265-445b-a576-615bd6b7b371)

**iii.** Now as shown in the image below, we act as a reviewer and we examine the changes in the **`dynamic-assignments`** branch and check for conflicts with the **`main`** branch.

![merge pull request 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/272752ff-9315-44c9-9987-003e6904c9d1)

**iv.** As we are satisfied and happy with the changes made in **`dynamic-assignments`**, we click on **"Merge pull request"** and then we click on **"Confirm merge"**

![confirm merge](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/cca3dc74-e690-4243-b32f-166f5f994e9f)

**v.** This takes us to the next page which shows that **`dynamic-assignments`** has been successfully merged to **`main`** branch.

![merge successful](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2a2402b5-1279-452c-8d71-6e7983600ca5)

### <br>Making use of Community Roles<br/>

Now it is time to create a role for MySQL database which will perform the functions of installing the MySQL package, creating a database and configuring users. It is important to note that there are tons of roles that have already been developed by other open source engineers out there. These roles are actually production ready, and dynamic to accomodate most of Linux flavours. So rather than re-inventing the wheel, we simply make use of Ansible Galaxy again and we can simply download a ready to use ansible role.

#### <br>Step 1: Download Mysql Ansible Role<br/>

**i.** We proceed to browse available community roles [here](https://galaxy.ansible.com/ui/). We decide to use a [MySQL role developed by](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/mysql/) **`geerlingguy`**.

**ii.** For now, we no longer need webhooks and Jenkins to update our codes on the **`Jenkins-Ansible`** server, so we disable it by clicking on **"Disable Project"** in the project status page in our Jenkins environment.

![disable project](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e8476a18-db68-4167-8304-8bc72699feb3)

**ii.** Next, we head to our Jenkins-Ansible server and then we go to the **`ansible-config-mgt`** directory and we checkout from the **`dynamic-assignments`** branch into the main branch, and then we pull down the latest changes.

```
$ git checkout main

$ git pull
```

![git checkout main git pull](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9e766003-76a0-4292-977c-d2a0103a2f62)

**iii.** Then we execute the following command to create and move into a new branch that we will call **`roles-feature`**.

**`$ git checkout -b roles-feature`**

![git-checkout-b roles-feature](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/bccf8b97-bf72-4af3-a2f1-73b5bb30bfc7)

**iv.** Inside the **`roles`** directory, we create a new MySQL role with ansible-galaxy and then we rename the folder to **`mysql`**:

```
$ ansible-galaxy install geerlingguy.mysql

$ mv geerlingguy.mysql/ mysql
```

![ansible galaxy geerling guy](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/003e1e95-c811-4f29-9c93-ddd11c8180bf)


#### <br>Step 2: Configure Mysql Ansible Role<br/>

**i.** The next thing we do is to go through the **README.md** file, and edit roles configuration to use the correct credentials for MySQL required for the **`tooling`** website. We navigate via VS Code explorer through **`roles > mysql > defaults > main.yml`** and we edit configuration in the **`main.yml`** file as shown in the image below:

![main-yml edit](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5d778f8a-33f3-47d9-a736-f8af44d8ba05)

**ii.** To reference, the database role, we do the following:

+ Go into the **README.md** file and copy the following block of code:

  ```
  ---
  - hosts: database
    roles:
    - role: geerlingguy.mysql
      become: yes
  ```

+ Then we move into the **`static-assignments`** folder and create a file we name **`db.yml`**

 ```
$ cd static-assignments

$ touch db.yml
```

![cd and touch db-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/63ed4dcd-7b8d-4fa3-9804-5357c963279a)

+ Then we enter the file via VS code explorer and we paste in the configuration as shown in the image below:

![reference db-yml role](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3d1151c1-7006-4f20-a829-d835362ce739)

**iii.** We also need to reference the **`db.yml`** role inside **`site.yml`**. So, we navigate via the VS Code file explorer and put the following configuration inside site.yml:

```
-  hosts: db
- name: Installing db
  import_playbook: ../static-assignments/db.yml
```

![db-yml in site-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/804f1ce1-9efa-4893-89d0-96803d6de96b)

**iv.** We proceed to update the inventory **`ansible-config/inventory/dev.yml`** file with the Private IP address of our DB server using the following syntax:

```
[db]
<db-Server-Private-IP-Address> ansible_ssh_user=ubuntu
```

![dev-yml config](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5c242a60-a8ad-41b2-b490-e5edc7f9b84e)


#### <br>Step 3: Commit and Test DB Playbook<br/>

In this step, we shall stage and commit our changes in the **`roles-feature`** branch. then we shall test run the db playbook. We decided to do this with the consideration that it would be cleaner and less cumbersome rather than executing it with the other roles we are going to download and configure later on in this project. It will also be easier to debug and investigate any potential errors our playbook might encounter.

**i.** Using the following set of commands, we add all untracked files to the staging area and then we commit the changes:

```
$ git add .

$ git commit -m "saved all updates to role-features branch"
```

![commit to roles-feature branch](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7d3051cd-6b9c-40d8-afca-d1944498de7f)

**ii.** The next step is to ensure connection to our **`Jenkins-Ansible`** server via **`ssh-agent`** as we did in [Project 11](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project11.md) and then with the commands below, we run the playbook against our **`dev inventory`**.

```
$ cd /home/ubuntu/ansible-config-mgt

$ ansible-playbook -i /inventory/dev.yml playbooks/site.yml
```

#### BLOCKER❗

![blocker error 5](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b6dc5ef9-6b01-42a9-9fc0-2956179ab1d8)

+ We got the error above when we ran the playbook, but after we examined the error message and did some investigations we realised that the hostname in the inventory folder in the **`dev.yml`** file was **`db`** while while in the static-assignments folders in the **`db.yml`** file, we had the hostname as **`database`**. So we fixed this issue and the playbook ran successfully as shown in the images below:

![plybook success 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/570c2107-929b-4c5e-b2fd-63d213875f7b)

![playbook success 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/41124e9d-36aa-4791-9987-3c6294f2714d)

#### <br>Step 4: Load Balancer Roles<br/>

We want to be able to make a choice on which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively: Nginx and Apache.

**i.** We refer to what we did earlier with **`msql`** role and we do the same for both nginx and apache load balancer roles. Inside the **`roles`** directory, we create new roles for both nginx and apache load balancers with ansible-galaxy. And then we rename the folders to **`nginx-lb`** and  **`apache-lb`** respectively.

```
$ sudo ansible-galaxy install geerlingguy.nginx

$ mv geerlingguy.nginx/ nginx-lb

$ sudo ansible-galaxy install geerlingguy.apache

$ mv geerlingguy.apache/ apache-lb
```

![install load balancer roles](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3bb63621-6379-4d83-8576-64d6bfb7d4e1)

**ii.** To make Nginx and Apache function as a loadbalancer, we need to update the **`default/main.yml`** file inside the Nginx and Apache roles with the private IP address of our two UAT webservers as shown in the images below:

![diagram 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/c4a8192c-f78a-4c59-987f-26870634a42b)

![make apache loadbalancer](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/adf0b7d4-3467-4119-ac24-491c9453744e)

**iii.** Since we cannot use both Nginx and Apache as load balancer at the same time, we need to make use of variables to enable either one depending on preference. We proceed to declare the variables in the **`default/main.yml`** file inside the Nginx and Apache roles as follows:

+ We name each variable **`enable_nginx_lb`** and **`enable_apache_lb`** respectively.

+ We set both variable values to false as follows: **`enable_nginx_lb: false`** and **`enable_apache_lb: false`**

+ We declare another variable **`load_balancer_is_required`** and we set its value to **`false`** as well.

![set variables to false](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5302d8d5-e6df-4ccc-b32f-c4d7e21bf9fb)

![set apache variables to false](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9fb75b2a-01c8-4d58-9cb4-c464a43fada0)

**iv.** Next, we update **`static-assignements`** by creating a **`loadbalancers.yml`**:

**`$ touch loadbalancers.yml`**

![load balancers-yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e3c6ffbe-179f-403e-8893-d9175f8dc078)

**v.** And then, we subsequently paste the following block of configuration into the file we just created:

```
- hosts: lb
  roles:
    - { role: nginx-lb, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache-lb, when: enable_apache_lb and load_balancer_is_required }
  become: true
```

![diagram 5](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/104e5699-0d5b-4302-a619-abc4c9ac240f)

**vi.** The next step is to update the entry point for our playbooks, the **`site.yml`** file with the following configuration block:

```
-hosts: lb
- name: Loadbalancers assignment
- import_playbook: ../static-assignments/loadbalancers.yml
when: load_balancer_is_required 
```

![load balancers site yml](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/18e1a902-bea0-443a-acf9-5d1bc217f560)

**vii.** Afterwards, we update our **`inventory/dev.yml`** file with the private IP address of the load balancer server as shown below:

![diagram 7](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f9a8ff3b-4849-4edc-8945-1374fed055fd)

**viii.** To complete our configuration, we need to decide whether it is Nginx or Apache that is used as a loadbalancer at any given time.  We make use of the **`env-vars/dev.yml`** file to define which load balancer to use in the UAT environment by setting the respective environmental variables to **`true`** or **`false`**.

+ By setting the variable below in the **`env-vars/dev.yml`** file, we activate the load balancer and enable **`nginx`**.

```
enable_nginx_lb: true
load_balancer_is_required: true
```
+ By setting the variable below in the **`env-vars/dev.yml`** file, we activate the load balancer and enable **`apache`**.

```
enable_apache_lb: true
load_balancer_is_required: true
```
+ **Tip**: *After testing the variable for the first load balancer, we must stop the load balancer service (e.g **`sudo systemctl stop apache2`** before setting the variable for the second load balancer and subsequently running the playbook against it. This is imperative to avoid errors as two loadbalancers cannot run simultaneously on the same server.*
  
**ix.** Next, we run our playbook by executing the command below:

**`ansible-playbook -i inventory/dev.yml playbooks/site.yml`**

+ The following images show the playbook output when we set the variable to enable **`nginx`** load balancer:

![nginx playbook 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/00174d30-7be1-4128-bfc2-5058ec9763b1)

![nginx playbook 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/96162e62-c5bf-48d6-9da6-285cb0d3ecb8)

+ The following images show the playbook output when we set the variable to enable **`apache`** load balancer:

![apache playbook 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ffd2102f-0cbe-4c9e-b59d-ef3509bf7997)

![apache playbook 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8808a331-3d36-47ea-86e9-488ad7c61dd0)

#### <br>Step 5: Update Git with Latest Code in `roles-feature` Branch<br/> 

At this point, we we need to push all the changes we made locally to our remote Github repository.

**i.** We use the following commands to stage, commit and push our branch to GitHub:

```
$ git status

$ git add <selected files>

$ git commit -m "commit message"

$ git push --set-upstream origin roles-feature
```

![git 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a91f5ccc-e301-45c2-834f-28e043283a80)

![git 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4b0c3004-afee-4c5e-beeb-9d6c1ffd7bea)

![git 3](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/1aaf349b-422a-48ba-ad02-6f3784a2c972)

**ii.** The next thing we do is to create a **Pull Request** in GitHub by following [these steps:](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) 

+ From the **`ansible-config-mgt`** repository page, we click on the **"Pull requests"** tab and then in the next page we click on the **"New pull request"** button.

![pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a27f564c-53f7-49c8-9905-aa7e440fc17b)

![new pull requests](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/46e0162b-0cb7-4ff4-8514-6fd0c8ab0a7c)

+ This takes us to the **"Compare changes"** page where we choose the **`roles-feature`** branch to set up a comparison with the **`main`** branch.

![compare changes 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/3da7a80b-f71a-4571-8396-8c968e12e05c)

+ Once we set up the comparisons between the **`main`** and the **`roles-feature`** branch, we then proceed to click on the **"Create pull request"** button.

![comparing changes create pull request 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/16ad29fb-3e7f-4e2b-8ee1-81647ec89c9e)

+ In the next page, we input a pull request message inside the dialogue box and we click on the  **"Create pull request"** button.

![open a pull request 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2c49be69-a895-4a12-9420-9f1a16498166)

**iii.** Now as shown in the image below, we act as a reviewer and we examine the changes in the **`roles-feature`** branch and check for conflicts with the **`main`** branch.

![merge pull request 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/9469a339-9db0-4b14-bffe-8230f381674e)

**iv.** As we are satisfied and happy with the changes made in **`roles-feature`**, we click on **"Merge pull request"** and then we click on **"Confirm merge"**

![confirm merge 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/70ca9698-9d77-47cd-8cbf-34f983c7e91f)

**v.** This takes us to the next page which shows that **`roles-feature`** has been successfully merged to **`main`** branch.

![merge successful 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5ec17b6f-c0b7-4dcc-a914-9ea3ebba29fb)

### <br>Conclusion<br/>

The output images from **ix.** above show that we have been able to successfully use Ansible Dynamic Assignments and Roles to prepare UAT environment for tooling web solution. The goal of our project was to demonstrate the flexibility of Ansible with dynamic assignments (include) and leverage on Ansible roles to how the automation capabilities demonstrated in Projects [11](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project11.md) and [12](https://github.com/QBDev0ps/DevOps-Cloud-projects/blob/main/Project12.md) can be extended.

We began the project by creating and moving into a new branch **`dynamic-assignments`**. Here, we created the necessary folders and files and then inputted the necessary configuration to set our variables. Importantly, in our configuration, we used **`include_vars`** syntax instead of **`include`**, as the  **`include module`** has been deprecated since Ansible version **2.8**, and variants of **`include_*`** must be used. We made use of special variables that will help Ansible dynamically resolve the name of the inventory file being used and that will also help to determine the location of the running playbook, and from there navigate to other paths on the filesystem. We included the variables using a loop. **`with_first_found`** which implies that, looping through the list of files, the first one found is used. After, this we updated the **`site.yml`** file to make use of the dynamic assignment and then we staged, committed and pushed all the changes we made locally in our branch to our remote Github repository. In our repository, we initiated a pull request and merged the **`dynamic-assignments`** branch to our main branch.

In the next phase of this project, we began working with community roles. We created and moved into **`roles-feature`** branch. We then browsed available community roles in [ansible-galaxy](https://galaxy.ansible.com/ui/). We decided to use a [MySQL role](https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/mysql/) developed by **`geerlingguy`**.The next thing we did was to configure Mysql Ansible Role. We did this by creating **`db.yml`** file inside the **`static-assignments`** folder. After putting in the necessary configuration, we moved ahead to reference the **`db.yml`** role inside **`site.yml`**. With the configuration in place inside **`site.yml`**, we proceeded to update the inventory **`ansible-config/inventory/dev.yml`** file with the Private IP address of our DB server. Next, we staged and committed our changes and then we tested the playbook. We ran into a bit of a blocker at this point, but we were able to correct it by fixing a configuration issue and our playbook ran successfully.

The final phase of this project was to demonstrate the implementation of load balancer roles. We needed to be able to make a choice on which Load Balancer to use, Nginx or Apache, so we created two roles respectively: Nginx and Apache with by installing **`geerlingguy`** ansible-galaxy roles as we did earlier. To make Nginx and Apache function as a loadbalancer, we updated the **`default/main.yml`** file inside the Nginx and Apache roles with the private IP address of our two UAT webservers. And since we cannot use both Nginx and Apache as load balancer at the same time, we had to  declare the variables in the **`default/main.yml`** file (inside the Nginx and Apache roles) to enable either one depending on preference. Next, we updated **`static-assignements`** by creating and configuring the **`loadbalancers.yml`** file, we updated the entry point for our playbooks **`site.yml`** with the necessary configuration and we  we updated our **`inventory/dev.yml`** file with the private IP address of the load balancer server. We completed our configuration by using the **`env-vars/dev.yml`** file to define which load balancer to use in the UAT environment. We did this by switching the respective environmental variables settings for **`nginx`** and **`apache`** to **`true`** or **`false`** depending on which is in use. We tested this out for both load balancers and our playbook ran successfully. To wrap up the project in a nice tidy bow, we we staged, committed and pushed all the changes we made locally in the **`roles-feature`** branch to our remote Github repository. Here we created a pull requestand merged the changes to the main branch. Thank you. 
