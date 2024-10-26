## Automate Infrastructure With IaC using Terraform. Part 3 - Refactoring

Improving cloud infrastructure automtion is esential for enhancing operational efficiency, reducing manual effort and achieving greater agility in managing and scaling resources in cloud environments.

In the past two previous projects we have developed AWS Infrastructure code using Terraform and tried to run it from our local workstation.
Now it is time to introduce some more advanced concepts and enhance our code.

In this project we shall explore, alternative Terraform [modules](https://developer.hashicorp.com/terraform/language/modules) and [backends](https://www.terraform.io/docs/language/settings/backends/index.html).

### <br>Introduction to Terraform Modules<br/>

Simply put, terraform modules serve as best practices to structure our **`.tf`** files. At this time, we can appreciate the inherent difficulties in navigating through all the Terraform code blocks if they are all written in a single long **`.tf`** file. It is imperative for DevOps engineers to produce reusable and comprehensive IaC code structure, and one of the tool that Terraform provides out of the box is [**Modules**](https://www.terraform.io/docs/language/modules/index.html).

Modules are the main way to package and reuse resource configurations with Terraform. They serve as containers that allow us to logically group Terraform codes for similar resources in the same domain (e.g., Compute, Networking, AMI, etc.). A module consists of a collection of **`.tf`** and/or **`.tf.json`** files kept together in a directory. There are primarily two types of modules depending on how they are written (root and child modules). One **root module** can call other **child modules** and insert their configurations when applying Terraform configuration. This concept makes our code structure neater, and it allows different team members to work on different parts of configuration at the same time. 

We can also create and publish our modules to [Terraform Registry](https://registry.terraform.io/browse/modules) for others to use and vice-versa use someone's modules in our projects.

We can refer to existing **child modules** from our **root module** by specifying them as a source, as shown below:

```
module "network" {
  source = "./modules/network"
}
```

Note that the path to 'network' module is set as relative to our working directory.

Alternatively, we can also directly access resource outputs from the modules, like this:

```
resource "aws_elb" "example" {
  # ...

  instances = module.servers.instance_ids
}
```

In the example above, we will have to have module 'servers' to have an output file to expose variables for this resource.

### <br>Refactoring our project using Terraform Modules<br/>

In continuation of [Project 17](https://github.com/QuadriBello/DevOps-Cloud/blob/main/Project17.md), we shall make use of modules to refactor our entire code. This will help simplify our codebase. In the Project 17 [repository](https://github.com/darey-devops/PBL-project-17.git), we had a single list of long file for creating all of our resources, but this solution is far from optimal as it makes our codebase vey hard to read and understand thereby making potential future changes very cumbersome to implement.

To proceed, we shall break down all our Terraform codes to have all resources in their respective modules. We shall combine resources of a similar type into directories within a 'modules' directory. 

We begin our refactoring by creating a directory **`PBL-Project18-Terraform-Modular-Architecture`** which will be the root-module. Inside the root-module, we create a directory named **`modules`**:

```
$ mkdir PBL-Project18-Terraform-Modular-Architecture 

$ cd PBL-Project18-Terraform-Modular-Architecture

$ mkdir modules
```

Next, we navigate to the modules directory, and we create the directories that will hold the diiferent resources just as shown below:

```
- modules
  - ALB
  - EFS
  - RDS
  - Autoscaling
  - compute
  - VPC
  - security
```

![modules](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/cf38315d-66ee-445e-b8a7-58af563b610b)


We ensure that each module contains the following files:

```
- main.tf (or %resource_name%.tf) file(s) with resources blocks
- outputs.tf (optional, if you need to refer outputs from any of these resources in your root module)
- variables.tf (as we learned before - it is a good practice not to hard code the values and use variables)
```

For our **`root modules`**, we create the following files outside of the **`modules`** sub directory but still within our root directory:

```
main.tf

provider.tf

terraform.tfvars

vars.tf file.
```

![root modules](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a916ac2f-4c5c-4193-b675-8fe250adf154)

Then we proceed to copy and refactor the code containing the resources that were created in [Project 17](https://github.com/QuadriBello/DevOps-Cloud/blob/main/Project17.md) and we paste them into each of their relevant modules.

The modularized code for this project can be found in this [repository](https://github.com/QuadriBello/PBL-Project18-Terraform-Modular-Architecture.git).

### <br>Completing the Terraform configuration<br/>

After completing the refactoring/modularization process, the resultant configuration structure in our working directory now looks as shown below:

```
└── PBL
    ├── modules
    |   ├── ALB
    |     ├── ... (module .tf files, e.g., main.tf, outputs.tf, variables.tf)     
    |   ├── EFS
    |     ├── ... (module .tf files) 
    |   ├── RDS
    |     ├── ... (module .tf files) 
    |   ├── autoscaling
    |     ├── ... (module .tf files) 
    |   ├── compute
    |     ├── ... (module .tf files) 
    |   ├── network
    |     ├── ... (module .tf files)
    |   ├── security
    |     ├── ... (module .tf files)
    ├── main.tf
    ├── providers.tf
    ├── terraform.tfvars
    └── variables.tf
```

![terraform module infrastructure](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/afb76ee4-7f89-4296-ad3e-1ac33790074e)

Now, the code is much more well-structured and can be easily read, edited and reused by our DevOps team members.

Next, we proceed to run the following commands:

```
$ terraform init

$ terraform fmt

$ terraform validate
```

![terraform init](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/660a6c78-04c3-4e41-94fe-a64e43d4b294)

After confirmation that our configuration is valid, we execute the following commands:

```
$ terraform plan

$ terraform apply --auto-approve
```

![terraform plan 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ce06bca9-7dbe-4a20-a09f-605ce2414687)

![terraform plan 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b2b50f80-efa2-4faa-a422-f8315c820299)

![terraform apply complete](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/99155ec7-3d39-428d-b2ac-de3c0f57d2b2)

<br>we can see from the images below that our resources were deployed successfully.<br/>

![terraform plan 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ce06bca9-7dbe-4a20-a09f-605ce2414687)

![terraform plan 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b2b50f80-efa2-4faa-a422-f8315c820299)

![terraform apply complete](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/99155ec7-3d39-428d-b2ac-de3c0f57d2b2)

<br>With our project now complete, we can see in the images below that our resources were deployed successfully.<br/>

![VPC created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c59c58b5-a626-4419-9ddc-c258fa908b0d)

![private and public subnets created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ce2b10c5-2297-4c13-bf71-aa61e2f87af6)

![internet gateway created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3d3d17bf-0995-45ad-bf61-9f461de1d481)

![NAT gateway created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a585f708-2b5a-42ab-8bee-30a0f0142594)

![elastic IP created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/694e70d0-7717-41b6-be6d-bfdf0611d56e)

![route tables created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/eae4a121-90ad-4785-8339-001e8906deda)

![ec2 instance role and policy created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9aa04f48-e275-4954-aa44-61da5505ccda)

![security groups created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/40fccab3-82fa-4ed1-a7d8-43739fb09d25)

![certificate created from ACM](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/12708e7a-3ff7-423b-bcf7-f8cfb8d0be89)

![ialb and ealb created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a868fcc3-906a-4c63-a3c2-f5da65ae592f)

![Auto scaling groups created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/11f33e0d-1791-4b30-90d3-879d91f39ca7)

![launch templates created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/4b6c0b7e-7623-4e40-af44-349418244909)

![EC2 instances created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/00bc14da-2e92-4ed9-8c35-c2d5423c6ce1)

![EFS created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c7ba3471-6941-4e9f-a33a-2b32485f4d60)

![RDS created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f3d1aab2-4bcb-48dd-8b17-f7ca731d1f7c)

### <br>Introduction to Backend on [S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)<br/>

Each Terraform configuration can specify a backend, which defines where and how operations are performed, where state snapshots are stored, etc.
Our state file is basically where terraform stores all the state of the infrastructure in `json` format.

So far, we have been using the default backend, which is the **`local backend`** - it requires no configuration, and the state file is stored locally. This mode can be suitable for learning purposes, but it is not a robust solution, so it is better to store it in some more reliable and durable storage.

The second problem with storing this file locally is that, in a team of multiple DevOps engineers, other engineers will not have access to a state file stored locally on our computer.

To solve this, we will need to configure a backend where the state file can be accessed remotely by other DevOps team members. There are plenty of different standard backends supported by Terraform that we can choose from but since we are already using AWS - we shall choose an [S3 bucket as a backend](https://www.terraform.io/docs/language/settings/backends/s3.html).

Another useful option that is supported by S3 backend is [State Locking](https://www.terraform.io/docs/language/state/locking.html) - it is used to lock our current state for all operations that could write and change the state. This prevents others from acquiring the lock and potentially corrupting our state. State Locking feature for S3 backend is optional and requires another AWS service - [DynamoDB](https://aws.amazon.com/dynamodb/).

To proceed, we shall need to re-initialize Terraform to use S3 backend and to do this, we will be carrying out the following:

- Add S3 and DynamoDB resource blocks before deleting the local state file
  
- Update terraform block to introduce backend and locking
  
- Re-initialize terraform

- Delete the local `tfstate` file and check the one in S3 bucket
  
- Add `outputs`
  
- `terraform apply`

**Step 1:** We add the following code to the _main.tf file in the root module and we replace the name of the S3 bucket we created in [Project-16](https://expert-pbl.darey.io/en/latest/project16.html).

```
resource "aws_s3_bucket" "terraform_state" {
  bucket = "dev-terraform-bucket"
  # Enable versioning so we can see the full revision history of our state files
  versioning {
    enabled = true
  }
  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}
```

![terraform s3 bucket](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/33472a08-b6d9-4d2f-b070-ce8b9a86ade6)

Terraform stores Passwords and secret keys processed by resources in the state files. Hence, we should enable encryption with **`server_side_encryption_configuration`** in the above code.

**Step 2:** Next, we will create a DynamoDB table to handle locks and perform consistency checks. In previous projects, locks were handled with a local file as shown in **`terraform.tfstate.lock.info`**. Since we now have a team mindset, causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore, with a cloud storage database like DynamoDB, anyone running Terraform against the same infrastructure can use a central location to control a situation where Terraform is running at the same time from multiple different people.

Dynamo DB resource for locking and consistency checking:

```
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

![terraform dynamodb](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/064f970e-8d09-4039-9063-b3e6f0964177)

Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, we proceed to run **`terraform apply`** to provision resources.

**Step 3:** To configure S3 Backend, we create a file **`backend.tf`** in the root directory and we add the following code snippet:

```
terraform {
  backend "s3" {
    bucket         = "dev-terraform-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "eu-central-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

![backendtf](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/20e48642-a7d8-4f03-a3fa-02e91dd9f043)

At this stage, we need to re-initialize the backend. We run **`terraform init`** and confirm the change to the backend by typing **`yes`**.

![terraform init s3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/0c88bc48-54c1-4440-b382-9bd2d19e18ec)

**Step 4:** In this step, we verify the changes. 

- We navigate to our AWS console and we can confirm that **`.tfstatefile`** is now inside the S3 bucket.

![terraform tfstate](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7a8e9062-0a15-48f9-b3d5-1c594d84b06a)

- The DynamoDB table which we created has an entry which includes state file status.

![state file status](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bfbc3c12-2e0a-4063-a50e-d08afd776061)

- We navigate to the DynamoDB table inside AWS and leave the page open in our browser.  Then we execute **`terraform plan`** and while that is running, we refresh the browser and see how the lock is being handled.

![terraform plan lock](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/331ea4b5-6d43-45a8-949e-e9f90a1e3ca5)

After `terraform plan` completes, we refresh the DynamoDB table and we can see that the lock id has now vanished. 

![lock id vanished](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/bea8a755-bdfd-4ae2-8171-8bb4d1a09c1b)

### <br>Conclusion<br/>

We have now come to the end of this project. We have successfully refactored our codebase using child modules and root modules. This will ensure seamless and easy reusability by the time we carry out some more configuration on our files in the next project. We also implemented terraform backend using AWS S3 buckets. By doing this, our DevOps team members can now have easy access to the state file since it is now being stored with a remote, robust and reliable solution. Subsequently, we used DynamoDB to store our state lock files. This is really important when working in a team where multiple developers are trying to update the same Terraform state file as it means a central location can now be used to perform consistency checks and also prevent Terraform state file(**`terraform.tfstate`**) from accidental updates by putting a lock on the file so that the current update can be finished before processing the new change.

We shall now proceed to bring down the deployed infrastructure by running **`$ terraform destroy`**.

However, before we run this command, we will need to comment out the configuration in **`backend.tf`** to prevent terraform from deleting our S3 bucket configuration files and causing errors. Once we comment out the code, we run the following command to restore the former state of the **`terraform.tfstate file`**. 

**`$ terraform init -migrate-state`**

![comment out backend](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/950d4c36-b083-4039-a4c8-fea9ab5dfaee)

Then we execute the command below and we destroy the infrastructure.

**`$ terraform destroy`**

![terraform destroy](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6fd403aa-30c7-428f-92bf-6e42b7d3e93a)


