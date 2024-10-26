# Automate Infrastructure With IaC using Terraform. Part 4 - Terraform Cloud

In [Project 18](https://github.com/QuadriBello/DevOps-Cloud/blob/main/Project18.md), we refactored our terraform codes into modules to make our code base less cumbersome and more reusable and then subsequently we introduced the concept of backends and we migrated our `.tf ` state file to an AWS S3 bucket to enable easy collaboration with other members of the DevOps team. In this project we shall incorporate more new concepts into our deployment and ensure that our overall infrastructure as depicted in the image below is working by the time we are done.

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/785904a9-508a-4d2c-be7c-b6f9f1640a4f)

Up till now, we have had to install Terraform ourselves and we have been running Terraform codes from a command line on our local computer. However, in the Cloud world it is quite common to provide a managed version of an open-source software like Terraform. Managed means that we do not have to install, configure and maintain it ourselves - we just have to create an account and use it "as A Service".

### Introduction to Terraform Cloud

[Terraform Cloud](https://www.hashicorp.com/products/terraform) is a managed IaC service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events. It helps to enhance collaboration between developers and DevOps engineers, simplify workflow and improve security overall around the product.

By default, Terraform CLI performs operations on the server when it is invoked, it is perfectly fine if you have a dedicated role who can launch it, but if you have a team who works with Terraform - you need a consistent remote environment with remote workflow and shared state to run Terraform commands. This is where Terraform Cloud comes in. It runs in a consistent and reliable environment, and includes easy access to shared state and secret data, access controls for approving changes to infrastructure, a private registry for sharing Terraform modules, detailed policy controls for governing the contents of Terraform configurations and more. Teams can connect Terraform to version control, share variables, run Terraform in a stable remote environment, and securely store remote state.

Terraform Cloud executes Terraform commands on disposable virtual machines, this remote execution is also called [remote operations](https://developer.hashicorp.com/terraform/cloud-docs/run/remote-operations).

In this project, in addition to terraform cloud, we shall also be utilizing the combined powers of Packer and Ansible to automate infrastructure provisioning in AWS.

* [Packer](https://www.packer.io/) is an open source tool for creating identical machine images for multiple platforms from a single source configuration. Packer is lightweight, runs on every major operating system, and is highly performant, creating machine images for multiple platforms in parallel.

* [Ansible](https://www.ansible.com/) is an open-source automation tool, or platform, used for IT tasks such as configuration management, application deployment, intraservice orchestration, and provisioning.

By combining these tools, we shall automate the provisioning process and ensure consistent, reproducible infrastructure deployments.

### Build AMIs Using Packer

As stated before, we shall use packer in this project to create the AMIs for the launch templates used by the Auto Scaling Group. We have the Packer codes we used for this purpose stored in this [repository](https://github.com/QuadriBello/QB-PBL19-Codebase.git). Packer creates the instances, provision the instances using shell scripts and creates the AMIs. It also deletes the instances after the completion of the AMIs creation process.

**1.** Run the Packer command to build AMIs

To carry out our AMi builds, we run the packer build command as shown below:

**i.** We build the bastion ami using the following command:

**`$ packer build bastion.pkr.hcl`**

![packer build bastion](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/84bf2053-b2e3-45a9-92a6-3fc8dbf328c0)

**ii.** We build the nginx ami using the following command:

**`$ packer build nginx.pkr.hcl`**

![packer build nginx](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6145565c-9683-4c4b-bcc2-372ee36d8eb7)

**iii.** We build the ubuntu ami using the following command:

**`$ packer build ubuntu.pkr.hcl`**

![packer build ubuntu](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/20255b10-02d8-448d-b1d7-2ac7d47698c2)

**iv.** We build the web ami using the following command:

**`$ packer build web.pkr.hcl`**

![packer build web](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b45cbebd-7cdd-4ade-aab1-1080306de1eb)

As shown in the image below we can verify in our AWS console that the AMIs were successfully built.

![ami images created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6c9e6922-f930-4a75-b99d-1e68c09e4be4)

**2.** Update Terraform script with AMI IDs generated from Packer build.

We proceed to update the **`terraform.auto.tfvars`** with the AMI IDs.

![update autotfvars with ami ids](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/77ce78e2-73c1-4519-a282-89d9b8e5cded)

### Migrate `.tf` codes to Terraform Cloud

In this step, we shall be migrating our codes to Terraform Cloud and manage our AWS infrastructure from there:

**1.** Create a Terraform Cloud account

**i.** We navigate to the [terraform cloud homepage](https://app.terraform.io/signup/account), create a new account and verify our email address.

![terraform cloud homepage](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/44d3a030-328b-4722-8833-b44b03232084)

**ii.** Next we select "Create Organization", we choose a name for our organization and create it.

![create organization](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/63cef564-1e90-4bb7-bcb4-570a7062b72c)

**2.** Configure a Workspace

Here, we decide to use **`version control workflow`** as it is the most common and recommended way to run Terraform commands triggered from our git repository.

![create workspace](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3e181484-a542-493b-94bf-1ebf554b8eff)

**i.** We create a new repository in our GitHub and we call it [**`qb-terraform-cloud`**](https://github.com/QuadriBello/qb-terraform-cloud.git), then we push our Terraform codes developed in the previous projects to the repository.

![create new repo and push terraform code](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c894a7d7-2e73-486c-add9-6210575fd819)

![create new repo and push terraform code 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ae0af401-b2b7-4d92-b32b-f7876a4bc620)

![repository for terraform cloud](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9585a6c8-da5c-48a2-bdfb-9f9b91396230)

**ii.** We choose `version control workflow` and we are subsequently prompted to connect our version control system (VCS) which is our GitHub account to our workspace - we follow the prompt and add our newly created repository to the workspace.

![connect to VCS](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ea3977ad-9d0a-4dd1-bd61-e2e15c3c15a6)

![choose repository](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ab1bdaae-4561-4ccc-a6d9-d0df4bfeb2d4)

**iii.** We move on to "Configure settings", provide a description for our workspace and we leave all the rest settings as default, then we click "Create workspace".

![complete create workspace](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/253196d6-db4f-4adb-b7b9-55f09b6309da)

**3.** Configure Variables

Terraform Cloud supports two types of workspace variables: Environment variables and Terraform variables. Either type can be marked as sensitive, which prevents them from being displayed in the Terraform Cloud web UI and makes them write-only.

![workspace variables](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9cbb95b4-3763-4b0e-8882-a63f2534eb2e)

**i.** Terraform variables are the variables we have declared in our configuration. We however do not need to specify these variables in the Terraform Cloud UI since Terraform cloud can also load the default values from our **`.auto.tfvars`** file in the configuration.

![terraform autovars](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9b70982b-bb09-475c-95dd-e26d1b560a80)

**ii.** Environment variables are variables we need to set to be used in the Terraform runtime environment. Here we set two environment variables: **`AWS_ACCESS_KEY_ID`** and **`AWS_SECRET_ACCESS_KEY`**, and we set the values that we used in [Project 16](https://expert-pbl.darey.io/en/latest/project16.html). These credentials will be used to provision our AWS infrastructure by Terraform Cloud.

![aws access key and secret key environment variable](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5160f29e-8be6-44ce-8c6d-769e0d7ea770)

**iii.** After setting these 2 environment variables as shown in the image above, our Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources. However, we make sure to update the code in **`backend.tf`** with the name of our Terraform Cloud organization and workspace.

![backend terraform cloud organisation and workspace names](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bf189825-65de-4b64-85dc-4f8d858e63b3)

**4.** Push changes to remote repository and run Terraform Script

Next, we push the changes we made in the terraform code on our local machine to the GitHub remote repository we created earlier for terraform cloud.

![push changes to remote](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/51612c6b-b8d7-450a-beff-a97071763af4)

**i.** Whenever we push updated code from our local machine to our github repository **`qb-terraform-cloud`**, the version control functionalities of GitHub kicks in and triggers terraform cloud to automatically create a plan as shown below:

![automatically triggered plan](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/437b2313-b4fa-4dc7-b111-6e65403e7d43)

**ii.** We proceed to click on Confim and apply. Then a dialogue comes up which asks us to add a comment to explain the action. We type in a comment and then we click on Confirm plan.

![click on confirm and apply](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/98eab84e-f22a-444b-996f-f498a989124b)

![confirm plan](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/acc85abc-a60a-4f3e-9fdb-6ef1fd4f04a9)

**iii.** Subsequently, terraform cloud starts the apply process and creates our infrastructure in AWS.

![terraform apply starts running](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/aa8c369d-c3e7-46d3-918c-4e87e11e64ac)

![apply finished](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/87a9f401-ec72-436a-bc16-bc4d6f7fb9f2)

**iv.** On completion of the apply process as shown in the image above, we navigate to the AWS console to check on our Target groups.

![health check fail](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b858d779-e801-4957-9a69-5bfa7801a9f0)

Here, as seen in the image above, we notice that the instances in the target groups are unhealthy. This is because we are yet to have the instances properly configured.

**v.** To fix the above issue, since it is the listeners that route traffic to the target groups which contain our instances, we navigate back to our terraform code and we comment out the nginx, web and tooling listeners in the **`alb.tf`** file. We choose to do this since we will be running into a lot of errors if we attempt to configure the instances right away with Ansible.

![comment out listeners](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/eafa7866-22d3-47ea-a8d9-b2377c864d07)

![comment out web and tooling listeners](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/91a621e8-3a9d-48f8-9866-bb3a3ff41da8)

**vi.** To ensure that the autoscaling group does not spin up instances to the load balancer, we also need to comment it out. So we navigate to the **`asg-bastion-nginx.tf`** and **`asg-webserver.tf`** files to implement this.

![comment out autoscaling 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/437fbfcc-db1d-4a3e-8e4a-adf8fba916b7)

![comment out auto scaling  2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3ad9da12-d554-49c2-a169-e94152d339f7)

**vii.** Next, we push the changes to GitHub and then we apply the plan in Terraform Cloud.

![push changes again](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a8c2ed3c-69f5-4137-b9c7-01b3b0a788f4)

![apply listener changes](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/44f1e0fe-c33b-48aa-9300-ac5d2a18c4f2)

![apply finished listener changes](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e6aad91f-0823-4abd-b35f-58dd6181baa6)

**viii.** As we can see in the image above the apply run was successfully completed. We proceed to navigate to our AWS console and as we can see in the image below, there are no more targets registered to the target groups.

![no more registered targets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7a099bc9-6e3f-44d6-a39d-df26cb6ccefa)

**ix.** And also when we check our Loadbalancers we can see that there are no listeners attached.

![no listeners attached](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/587240b6-c9c5-49a3-b4ab-1dbffdf53612)

**5.** Update Ansible script with values from Terraform output

To do this, we do not update the ansible script on our local machine. Rather, we SSH into the Bastion host and clone down the ansible script from our repository then we go ahead to make our changes.

**i.** But before we proceed, we will need to set up SSH Agent on our windowss machine. We proceed to do this with the help of the [official windows openssh key management documentation](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement) and we execute the following commands on Windows Powershell.

```
# By default the ssh-agent service is disabled. Configure it to start automatically.
# Make sure you're running as an Administrator.
Get-Service ssh-agent | Set-Service -StartupType Automatic

# Start the service
Start-Service ssh-agent

# This should return a status of Running
Get-Service ssh-agent

# Now load your key files into ssh-agent
ssh-add <user-key>

# Ensure Key has been added
ssh-add -l
```

![add open ssh](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/da42e48f-bb8d-4eb5-9ab6-d5ddbe597b06)

**ii.** To configure our infrastructure with Ansible, we proceed need to ssh into the bastion instance and then we clone down the Ansible repository in github.

![connect to bastion via ssh](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fa68a89d-bce6-4d5a-b714-0c899ff50f87)

![clone down ansible repo](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7a2e83c1-58e7-43a7-ae59-52e41c594f94)

**iii.** Ansible will also require a connection to our AWS account in order to to automatically obtain the IP address for our dynamic inventory so we proceed to execute **`$ aws configure`** and then we add our **`AWS ACCESS KEY ID`** and **`AWS SECRET ACCESS KEY`**

![run aws configure](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6c67ac5d-3ea7-49fb-8ce3-46c6c4b1c28a)

**iv.** To test the connection and ensure that Ansible is able to fetch the IP addresses, we execute the following command:

**`$ ansible-inventory -i inventory/aws_ec2.yml --graph`**

![run graph command to get ip addresses](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0f74f8d6-746b-4dc1-936b-069bae5df544)

**v.** Next, we will need to carry out the following updates in Ansible:

- Update dns name for ialb in the **`nginx.conf.j2`** file in nginx reverse proxy.

![DNS name copied](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4336a3b9-7734-4e45-a64b-1954565e8397)

![updatedns name for ialb in nginx-con-j2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ff7dec20-dcae-456c-96fa-82495ed04ae6)

- Copy RDS endpoint and update for Wordpress and Tooling. Also update password and username for Wordpress and Tooling.

![rds end point copied](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c698cc2c-fa05-4cfe-b2b3-33e734eba20f)

![wordpress endpoint password and username updated](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0aeda8c9-41ad-48bc-a880-8840d085595c)

![tooling end point password and username updated](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9b1d1f5b-f91e-4e90-8e74-0aef20e7c315)

- Copy and update EFS access point ID and file system ID for Wordpress.

![acesspoint for wordpress1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/63a5be46-ebbd-47e4-be16-3325aee8a3bc)

![acesspoint for wordpress2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/45a38ada-dc02-42a1-971a-3bc355ef0716)

![wordpress acesspoint copied](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/487eea25-81d6-44f1-9cf6-606a36e3ae67)

- Copy and update EFS access point ID and file system ID for Tooling.

![access point for tooling 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/66ae14cf-9c40-4e6d-93c3-12a1906d49e5)

![access point for tooling 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6580537f-317d-419a-acfc-e8834faaabb5)

![tooling accesspoint updated](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a1479212-2bc4-4653-a898-6a6e7b0c17ea)

- Update roles path in **`ansible.cfg`** file.

![update roles path](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/152f07e9-8112-4440-a19e-425080d46de4)

- Export the roles path with the following command:

 **`$ export ANSIBLE_CONFIG=/home/ec2-user/ansible-deploy-pbl-19/roles`**

![export ansible config](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5c663859-33be-43ba-ade6-35194c442efb)

**6.** Run ansible Playbook

**i.** We execute the following command to run our ansible playbook and use ansible to configure our infrastructure

**`$ ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml`**

![ansible playbook success 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b5ad36e1-d876-4e96-b213-3c18c585c61e)

![ansible playbook success 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/eb7e422a-c8fd-4ba6-87c4-0a34b03cd738)

![ansible playbook success 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/87c59f17-f016-45bc-871b-ce6d58dbea2b)

**ii.** Our playbooks ran successfully as seen in the images above. Subsequently, we proceed to the terraform script in our local machine and we uncomment the auto scaling attachment in the **`asg-bastion-nginx.tf`** and **`asg-tooling-wordpress.tf`** files. We also make sure to remove the comments on the listeners in **`alb.tf`**

![uncomment autoscaling](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/67fcdf51-4cdf-4d04-9e56-6eb959f8c942)

![uncomment alb listener](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6fea53fe-628b-43e3-b46a-4733520c6669)

**iii.** Next, we commit and push the changes to our remote GitHub repository.

![uncomment commit](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6c487af9-44e5-4795-adbc-a444bcea90ce)

**iv.** Terraform cloud automatically detects the changes and runs a plan which we apply accordingly.

![plan uncomment](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d977999c-22d7-43f7-b52a-889db7149db5)

![uncomment plan finished](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/34e6b0a5-80ce-4175-a886-3cb9b722f92c)

**v.** We navigate to Route 53 in our AWS console and we copy the URL records for both Wordpress and Tooling websites.

![route 53 URL records](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/818cebe1-7f07-49c3-b20c-a1f487bf68eb)

**vi.** We paste the records in our browser and as seen in the images below, both the Wordpress and Tooling websites are functional and working.

![Word press working](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3f666cd3-bda9-465f-b54f-155b3e0ab543)

![tooling website working](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ba086ba4-85f9-4472-9198-5e5d1acaa001)


### Conclusion 

We have now successfully come to the end of this project. We have been able to combine the use of tools like Packer to build our AMIs, Terraform in combination with Github and Terraform Cloud to deploy our infrastructure and then Ansible as a configuration management tool to configure our resources. Our objective was to implement these tools in automating the deployment of AWS cloud solutions for 2 company websites and we were able to acheive this. In the future, we will look to extend the scope of the project and take a step further by automating the whole deployment process using Jenkins. Thank you.



 



