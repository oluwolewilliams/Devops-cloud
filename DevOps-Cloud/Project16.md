Automate Infrastructure With IaC using Terraform Part 1
============================================

After we have built AWS infrastructure for 2 websites manually, it is time to automate the process using Terraform. 

Let us start building the same set up with the power of Infrastructure as Code (IaC)

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a07fce80-4415-4e3c-961e-ff5e1af744cf)

### Introducion to Infrastructure as Code with Terraform

Infrastructure as code is writing your infrastructure (servers, databases, networks etc.) to version controlled scripts or code. It is your entire environment and infrastructure represented and maintained in a configuration code base. You can use one command to build your whole infrastructure and you can also use one command to tear it all down.

Terraform is an open source infrastructure as code tool created by Hashicorp. It is agentless and it can communicate via providers with other tools such as AWS, Kubernetes, Jenkins, Docker etc. Other benefits of terraform include:

**i.** Terraform is multicloud. You can deploy on multiple cloud environments with a single configuration file.

**ii.** Terraform is stateful. It allows you to track your resource changes throughout your deployment in a state file which is the single source of truth for your environment. Terraform uses this state file to determine the changes that you're making to manage your infrastructure to make sure that it matches with your configuration.

**iii.** Terraform is version controlled. All your engineers and developers can work on a single publicly available code base.

**iv.** Terraform is declarative. You declare what the end state of your infrastructure should be like and terraform will get you there.

**v.** There is no need for manual click ups. There won't be any need to manually deploy infrastructure.

**vi.** Terraform is great for disaster recovery. It helps to save a lot of money as there might be no need to spend huge amounts on disatser recovery if you can spin up and spin down all your infrastructure in seconds.

Terraform helps to minimize user errors.

### The secrets of writing quality Terraform code

The secret recipe of a successful Terraform projects consists of:

- Your understanding of your goal (desired AWS infrastructure end state)
  
- Your knowledge of the `IaC` technology used (in this case - Terraform)
  
- Your ability to effectively use up to date Terraform documentation [here](https://www.terraform.io/docs/configuration)

As we go along completing this project, we will get familiar with [Terraform-specific terminology](https://www.terraform.io/docs/glossary.html), such as: 

- [Attribute](https://www.terraform.io/docs/glossary.html#attribute)
- [Resource](https://www.terraform.io/docs/glossary.html#resource)
- [Interpolations](https://www.terraform.io/docs/glossary.html#interpolation)
- [Argument](https://www.terraform.io/docs/glossary.html#argument)
- [Providers](https://www.terraform.io/docs/providers/index.html)
- [Provisioners](https://www.terraform.io/docs/language/resources/provisioners/index.html)
- [Input Variables](https://www.terraform.io/docs/glossary.html#variables)
- [Output Variables](https://www.terraform.io/docs/glossary.html#output-values)
- [Module](https://www.terraform.io/docs/glossary.html#module)
- [Data Source](https://www.terraform.io/docs/glossary.html#data-source)
- [Local Values](https://www.terraform.io/docs/configuration-0-11/locals.html)
- [Backend](https://www.terraform.io/docs/glossary.html#backend)  

We shall make sure to understand them and know when to use each of them. 

Another important concept is [`data type`](https://en.wikipedia.org/wiki/Data_type). This is a general programing concept, it refers to how data is represented in a programming language and defines how a compiler or interpreter can use the data. Common data types are:

- Integer
- Float
- String
- Boolean, etc.

#### Best practices

* Ensure that every resource is tagged using multiple key-value pairs. We will see this in action as we go along.
* Try to write reusable code, avoid hard coding values wherever possible. (For learning purpose, we will start by hard coding, but gradually refactor our work to follow best practices).

### Project Prerequisites 

These are important prerequisites before we begin writing Terraform code.

**i.** Create an IAM user, name it `terraform` (*ensure that the user has only programatic access to your AWS account*) and grant this user `AdministratorAccess` permissions.

**ii.** Copy the secret access key and access key ID. Save them in a notepad temporarily.

**iii.** Configure programmatic access from your workstation to connect to AWS using the access keys copied above and a [Python SDK (boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). Workstation must have [Python 3.6](https://www.python.org/downloads/) or higher.

For Windows, use `gitbash`, For Mac, simply open a `terminal`. Read [here](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html) to configure the Python SDK properly. 

For easier authentication configuration - use [AWS CLI](https://aws.amazon.com/cli/) with `aws configure` command.

**iv.** Create an [S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html) to store Terraform state file. It can be named like `<yourname>-dev-terraform-bucket` (*Note: S3 bucket names must be unique unique within a region partition, you can read about S3 bucken naming [in this article](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)*). We will use this bucket from Project-17 onwards.

![terraform s3 bucket](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1aa020a6-155e-4901-87bc-ea87c58674e5)

**v.** When we have configured authentication and installed `boto3`, we make sure we can programmatically access our AWS account by running following commands in `>python`:

```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```

We shall see our previously created S3 bucket name - `<yourname>-dev-terraform-bucket`

![access to s3 bucket confirmed aws cli setup confirmed](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/51f10785-24a0-48e9-b745-4cf5ccbb13b0)

### Base Infrastructure Automation

We shall now to make use of terraform to set up our infrastructure.

#### VPC | Subnets | Security Groups

We proceed by creating a directory structure. We open Visual Studio Code and do the following:

**i.** Create a folder called `PBL`.
  
**ii.** Create a file in the folder, name it `main.tf`.

After doing this, the setup looks as shown in the image below.

![maintf](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/245fd673-534a-472a-ac8e-0d412bb7527a)

#### Provider and VPC resource section

We set up Terraform CLI as per [this instruction](https://learn.hashicorp.com/tutorials/terraform/install-cli).

**i.** Add `AWS` as a provider, and a resource to create a VPC in the `main.tf` file.
  
**ii.** Provider block informs Terraform that we intend to build infrastructure within AWS.
  
**iii.** Resource block will create a VPC.

```
provider "aws" {
  region = "eu-central-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```

**Note:** We can change the configuration above to create our VPC in another region that is closer to us. The same applies to all configuration snippets that will follow.

![create vpc](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/722f83b1-8238-4bbe-8551-7906c2692f9b)

**iii.** The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by `providers` and `provisioners`. At this stage, we only have `provider` in our **`main.tf`** file. So, Terraform will just download plugin for AWS provider.
  
**iv.** Lets accomplish this with **`$ terraform init`** command as seen in the below image.

![terraform init](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5c6173d3-1a9a-4db6-b908-795b6c35daef)

***Observation***: 

- We notice that a new directory has been created: **`.terraform\...`**. This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that we must execute **`$ terraform init`** again, to download them.

**v.** Moving on, let us create the only resource we just defined. **`aws_vpc`**. But before we do that, we should check to see what terraform intends to create before we tell it to go ahead and create it.

* We run **`$ terraform plan`**
  
* Then, since we are happy with changes planned, we execute **`$ terraform apply`**

![terraform plan apply](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/748473c7-a589-48e4-9958-80c16187db10)

***Observations***: 

- A new file is created `terraform.tfstate` This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.
   
- When we observed closely, we realised that another file gets created during planning and apply. But this file gets deleted immediately. `terraform.tfstate.lock.info` This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same - it allows to avoid duplicates and conflicts.
   
Its content is usually like this.

```
    {
        "ID":"e5e5ad0e-9cc5-7af1-3547-77bb3ee0958b",
        "Operation":"OperationTypePlan","Info":"",
        "Who":"dare@Dare","Version":"0.13.4",
        "Created":"2020-10-28T19:19:28.261312Z",
        "Path":"terraform.tfstate"
    }
```

It is a `json` format file that stores information about a user: user's `ID`, what operation he/she is doing, timestamp, and location of the `state` file.

#### Subnets resource section

According to our architectural design, we require 6 subnets:

- 2 public
  
- 2 private for webservers
  
- 2 private for data layer

**i.** We proceed to create the first 2 public subnets. 

We add below configuration to the `main.tf` file:

```
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1b"
}
```

![2 subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/00d6be07-155b-4cd9-bcec-4c50ed35a7d5)

- We are creating 2 subnets, therefore declaring 2 resource blocks - one for each of the subnets.
  
- We are using the `vpc_id` argument to interpolate the value of the VPC `id` by setting it to `aws_vpc.main.id`. This way, Terraform knows inside which VPC to create the subnet.

**ii.** We run **`$ terraform plan`** and **`$ terraform apply`** 

![terraform plan subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f7fa6c6c-69b7-483c-b91a-ac751e61a927)

![terraform apply subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/6b59fc1d-22d6-4303-927c-e356f7644a8e)

***Observations***: 

- Hard coded values: Both the `availability_zone` and `cidr_block` arguments are hard coded. We should always endeavour to make our work dynamic.
  
- Multiple Resource Blocks: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a `count` argument.

**iii.** Now we improve our code by refactoring it.

*First, we destroy the current infrastructure. Since we are still in development, this is totally fine. Otherwise, **DO NOT DESTROY** an infrastructure that has been deployed to production.*

To destroy whatever has been created we run **`$ terraform destroy`** command, and type **`yes`** after evaluating the plan.

![terraform destroy](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7a551ec8-a2ea-4e93-bfec-f4e5eab97ae7)

#### <ins>Fixing The Problems By Code Refactoring<ins>

**i.** **Fixing Hard Coded Values**: We will introduce variables, and remove hard coding.

  - Starting with the provider block, we declare a variable named `region`, give it a default value, and update the provider section by referring to the declared variable.

    ```
        variable "region" {
            default = "eu-central-1"
        }

        provider "aws" {
            region = var.region
        }

    ```
    
  - We do the same to `cidr` value in the `vpc` block, and all the other arguments.
  
    ```
        variable "region" {
            default = "eu-central-1"
        }

        variable "vpc_cidr" {
            default = "172.16.0.0/16"
        }

        variable "enable_dns_support" {
            default = "true"
        }

        variable "enable_dns_hostnames" {
            default ="true" 
        }

        variable "enable_classiclink" {
            default = "false"
        }

        variable "enable_classiclink_dns_support" {
            default = "false"
        }

        provider "aws" {
        region = var.region
        }

        # Create VPC
        resource "aws_vpc" "main" {
        cidr_block                     = var.vpc_cidr
        enable_dns_support             = var.enable_dns_support 
        enable_dns_hostnames           = var.enable_dns_support
        enable_classiclink             = var.enable_classiclink
        enable_classiclink_dns_support = var.enable_classiclink

        }
    ```

![fixing hard coded values](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/941c5a8f-8b13-4fe5-b8e6-b08f23442662)

**ii.** **Fixing multiple resource blocks**: This is where things become a little tricky. It's not complex, we are just going to introduce some interesting concepts. <ins>Loops & Data sources</ins>
  
Terraform has a functionality that allows us to pull data which exposes information to us. For example, every region has Availability Zones (AZ). Different regions have from 2 to 4 Availability Zones. With over 20 geographic regions and over 70 AZs served by AWS, it is impossible to keep up with the latest information by hard coding the names of AZs. Hence, we will explore the use of Terraform's **Data Sources** to fetch information outside of Terraform. In this case, from **AWS**.

We proceed to fetch Availability zones from AWS, and replace the hard coded value in the subnet's `availability_zone` section.

```
        # Get list of availability zones
        data "aws_availability_zones" "available" {
        state = "available"
        }
```

To make use of this new `data` resource, we will need to introduce a `count` argument in the subnet block: Something like this.

```
    # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }

```

Let us quickly understand what is going on here:

- The `count` tells us that we need 2 subnets. Therefore, Terraform will invoke a loop to create 2 subnets.
  
- The `data` resource will return a list object that contains a list of AZs. Internally, Terraform will receive the data like this 

```
  ["eu-central-1a", "eu-central-1b"]
```

Each of them is an index, the first one is index `0`, while the other is index `1`. If the data returned had more than 2 records, then the index numbers would continue to increment. 

Therefore, each time Terraform goes into a loop to create a subnet, it must be created in the retrieved AZ from the list. Each loop will need the index number to determine what AZ the subnet will be created. That is why we have `data.aws_availability_zones.available.names[count.index]` as the value for `availability_zone`. When the first loop runs, the first index will be `0`, therefore the AZ will be `eu-central-1a`. The pattern will repeat for the second loop.

But we still have a problem. If we run Terraform with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because we still have `cidr_block` hard coded. The same `cidr_block` cannot be created twice within the same VPC. So, we have a little more work to do.

**iii.**  **Making `cidr_block` dynamic**: We will introduce a function **`cidrsubnet()`** to make this happen. It accepts 3 parameters. Let us use it first by updating the configuration, then we will explore its internals.

```
    # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```

A closer look at `cidrsubnet` - this function works like an algorithm to dynamically create a subnet CIDR per AZ. Regardless of the number of subnets created, it takes care of the cidr value per subnet. 

Its parameters are `cidrsubnet(prefix, newbits, netnum)`

- The `prefix` parameter must be given in CIDR notation, same as for VPC.
  
- The `newbits` parameter is the number of additional bits with which to extend the prefix. For example, if given a prefix ending with /16 and a newbits value of 4, the resulting subnet address will have length /20.
  
- The `netnum` parameter is a whole number that can be represented as a binary integer with no more than `newbits` binary digits, which will be used to populate the additional bits added to the prefix

We can experiment how this works by entering the **`terraform console`** and then we keep changing the figures to see the output.

- On the terminal, we run **`$ terraform console`**
  
- type `cidrsubnet("172.16.0.0/16", 4, 0)`
  
- Hit enter
  
- See the output
  
- Keep changing the numbers and see what happens.
  
- To get out of the console, type `exit`

![terraform console](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/a4e96a22-c593-4dde-bbc4-12307c26a5b0)

**iv.** **Removing hard coded `count` value**: If we cannot hard code a value we want, then we will need a way to dynamically provide the value based on some input. Since the `data` resource returns all the AZs within a region, it makes sense to count the number of AZs returned and pass that number to the `count` argument.

To do this, we can introuduce `length()` function, which basically determines the length of a given list, map, or string.

Since `data.aws_availability_zones.available.names` returns a list like `["eu-central-1a", "eu-central-1b", "eu-central-1c"]` we can pass it into a `length` function and get number of the AZs.

**`length(["eu-central-1a", "eu-central-1b", "eu-central-1c"])`**

We open up **`$ terraform console`** and try it.

![length function](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1066e22a-0e50-4b1c-9914-adb4bda61393)

Now we can simply update the public subnet block like this:

```    
# Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = length(data.aws_availability_zones.available.names)
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
```

***Observations***: 

- What we have now, is sufficient to create the subnet resource required. But if when we observe closely, it is not satisfying our business requirement of just `2` subnets. The `length` function will return number 3 to the `count` argument, but what we actually need is `2`.

Now, we proceed to fix this. 

- We declare a variable to store the desired number of public subnets, and set the default value
  
```
  variable "preferred_number_of_public_subnets" {
      default = 2
}
```

- Next, we update the `count` argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the `length` function. This is presentedas shown below.

```
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}
```

To explain the `count` section in the above code block:

- The first part `var.preferred_number_of_public_subnets == null` checks if the value of the variable is set to `null` or has some value defined.
  
- The second part `?` and `length(data.aws_availability_zones.available.names)` means, if the first part is true, then use this. In other words, if preferred number of public subnets is `null` (*Or not known*) then set the value to the data returned by `length` function.
  
- The third part `:` and  `var.preferred_number_of_public_subnets` means, if the first condition is false, i.e preferred number of public subnets is `not null` then set the value to whatever is definied in `var.preferred_number_of_public_subnets`.

Now our entire configuration looks as shown below:

```
# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = 2
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}


# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}

```

![entire configuration](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3fb531ea-6feb-4183-ad51-04f183bd5082)

### <br>Introducing variables.tf and terraform.tfvars<br/>

Instead of havng a long lisf of variables in `main.tf` file, we can actually make our code a lot more readable and better structured by moving out some parts of the configuration content to other files.

- We will put all variable declarations in a separate file.
  
- And we will provide non default values to each of them.

**i.** We create a new file and name it `variables.tf`.

**ii.** We copy all the variable declarations into the new file.

**iii.** We create another file, and name it `terraform.tfvars`.

**iv.** We set values for each of the variables.

So our setup now will be as follows:

#### Main.tf

This is a file that contains our main Terraform configuration.

```
# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}
```

![maintf2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3a63b3ce-eb86-4bb1-ae80-ccbca903546e)

#### variables.tf 

This is a file where we can define variables for our Terraform configuration. The file is used for the declaration of variables, name, type, description, as well as the optional default value for the variable and additional meta data.

```
variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = null
}
```

![variablestf](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9f3d01c9-c4ca-426b-a4b8-8444a74ba21f)

#### terraform.tfvars

This is a file where we actually assign values to the variables. It is used for giving the actual variable values during execution. It allows you to customize a specific execution.

```
region = "eu-central-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

enable_classiclink = "false" 

enable_classiclink_dns_support = "false" 

preferred_number_of_public_subnets = 2
```

![terraformtfvars](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f270c9f7-a112-4b59-a1dd-e73664f71e42)

### <br>Conclusion<br/>

**i.** At this stage, our file structure in the PBL folder should look as shown below:

```
└── PBL
    ├── main.tf
    ├── terraform.tfstate
    ├── terraform.tfstate.backup
    ├── terraform.tfvars
    └── variables.tf
```

To confirm this, we run the following command:

**`$ tree.com //a //f`**

![tree](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/eca28317-842a-4f27-a3b7-cb4d5aee941a)

**ii.** Next, we proceed to run the following commands to ensure everything works.

```
$ terraform plan

$ terraform apply --auto-approve
```

![terraform plan2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c49e28f6-f80a-4e7f-bdf9-c188e0d1cb4f)

![terraform apply auto approve](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3c6714eb-3f06-4fe1-9ea7-57c72713a861)

**iii.** Subsequently, we navigate to our AWS account to confirm that our infrastructure (VPC and Subnets) has indeed been created.

![VPC created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/008e75b7-8774-4393-b0f9-004cb309692d)

![subnets created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1cf9f5cd-b379-47e8-92ec-1897bd686f90)

**iv.** On a final note, we execute **`$ terraform destroy --auto-approve`** to delete the infrastructure.

![terraform destroy2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/681c8c47-be59-475c-9de7-d53dafbdb475)

![terraform destroy 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c5ae5476-e14a-468a-8d14-62098b5703a7)

We have now successfully utilized the power of IaC and learnt how to create and delete AWS Network Infrastructure programmatically with Terraform. 

