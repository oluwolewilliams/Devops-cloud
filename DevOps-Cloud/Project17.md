# Automate Infrastructure With IaC using Terraform. Part 2

In this project, we shall go deeper in to automating other parts of our infrastructure on AWS. However, before we begin, it is very important to fully understand certain concepts around Networking.

## Understanding Basic Network Concepts

#### IP Address:

An IP address is a unique address that identifies a device on the internet or a local network. IP stands for "Internet Protocol," which is the set of rules governing the format of data sent via the internet or local network.

#### VPCs

A virtual private cloud (VPC) is an on-demand configurable pool of shared resources allocated within a public cloud environment, providing a certain level of isolation between the different organizations (denoted as users hereafter) using the resources.

#### Subnets

A subnet, or subnetwork, is a segmented piece of a larger network. More specifically, subnets are a logical partition of an IP network into multiple, smaller network segments.

#### CIDR Notation

CIDR notation (Classless Inter-Domain Routing) is an alternate method of representing a subnet mask. It is simply a count of the number of network bits (bits that are set to 1) in the subnet mask.

#### IP Routing

IP routing is the process of sending packets from a host on one network to another host on a different remote network.

#### Internet Gateway

An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet


#### NAT Gateway

A NAT gateway is a Network Address Translation (NAT) service. You can use a NAT gateway so that instances in a private subnet can connect to services outside your VPC but external services cannot initiate a connection with those instances.

#### Routing Tables

A routing table, or routing information base (RIB), is a data table stored in a router or a network host that lists the routes to particular network destinations, and in some cases, metrics (distances) associated with those routes. The routing table contains information about the topology of the network immediately around it.

## Continue Infrastructure Automation with Terraform

In continuation from where we stopped in the previous project, we shall continue our implementation using the same network architecture diagram. In this project, we shall proceed to carry out the following:

1. Networking Resources: We create our Networking resources.

2. AWS Identity and Access Management: Identity Access Control configuration automation.

3. Compute Resources: Automate creation of compute resources and associated configuration.

4. Storage and Database Resources: Automate creation of AWS Storage and Database resources.

![152831445-844e3865-0317-4bf4-969a-490a7c1e06ba](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/fc82a9b8-d0b1-4328-83ba-aa569c54dfaa)

### <br>Networking Resources<br/>

Firstly, we need to create our network resources.Since we have created our public subnets in the previous project, we shall start off here by creating our private subnets.

#### Step 1: Create Private Subnets.

We shall create 4 private subnets keeping in mind the following principles:

- Make sure to use variables or `length()` function to determine the number of AZs.

- Use variables and `cidrsubnet()` function to allocate `vpc_cidr` for subnets.

- Keep variables and resources in separate files for better code structure and readability.

- Tag all the resources created so far. Explore how to use `format()` and `count` functions to automatically tag subnets with its respective number.

We create the private subnet by adding the following code snippet into our **`main.tf`** file.

```
# Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

 tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}
```

![create private subnets](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c3d72f4d-40f1-4901-9042-8f0976caa136)

As can be seen above, we have incorporated tagging in a bid to effectively manage our resource. We can now tag all our resources using the same format as shown below.

```
 tags = merge(
    var.tags,
    {
      Name = format("%s-NameofResource-%s", var.name, count.index)
    },
  )
```

This also means that anytime we need to make any change to the tags, we simply do that in one single place. Also, if we look closely at the code snippet, we can see that our key-value pairs are not hard coded. Rather, we have created variables for **`name`** and **`count`** that will be attached to each resource name.

#### Step 2: Create Internet Gateway. 

- We create a file called **`internet_gateway.tf`** and enter the following lines of code:

```
resource "aws_internet_gateway" "ig" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-%s-%s!", var.name, aws_vpc.main.id,"IG")
    }, 
  )
}
```

![create internet gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b9be0a3e-30ce-44d3-8352-80f0a1129127)

In the **`tags`** section of the code above, we have used **`format()`** function to dynamically generate a unique name for this resource. The first part of the **`%s`** takes the value of **`var_name`** as stored in our **variables.tf** file while the second takes the interpolated value **`aws_vpc.main.id`** and the third **`%s`** appends a literal string **`IG`** and finally an exclamation mark is added at the end.

#### Step 3: Create NAT Gateway.

We create a file called **`nat_gateway.tf`** and then enter the following block of code to create one NAT gateway and assign an elastic IP (EIP) address to it:

```
resource "aws_eip" "nat_eip" {
  vpc        = true
  depends_on = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-EIP", var.name)
    },
  )
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = element(aws_subnet.public.*.id, 0)
  depends_on    = [aws_internet_gateway.ig]

  tags = merge(
    var.tags,
    {
      Name = format("%s-Nat", var.name)
    },
  )
}
```

![create NAT gateway](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e705db18-c298-4391-983d-85233bc5c0e2)

The NAT gateway has a dependency on the exitence of an elastic IP address so as seen in the code block above, we create an elastic IP first and then we create the NAT gateway. We can also see the use of **`depends_on`** to indicate that the Internet Gateway resource must be available before the resources in this step can be created.

#### Step 4: Create AWS Routes.

We create a file called **`route_tables.tf`** and then we use the following code blocks to create routes for both public and private subnets.

```
# create private route table
resource "aws_route_table" "private-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Private-Route-Table-%s", var.name, var.environment)
    },
  )
}

# associate all private subnets to the private route table
resource "aws_route_table_association" "private-subnets-assoc" {
  count          = length(aws_subnet.private[*].id)
  subnet_id      = element(aws_subnet.private[*].id, count.index)
  route_table_id = aws_route_table.private-rtb.id
}

# create route table for the public subnets
resource "aws_route_table" "public-rtb" {
  vpc_id = aws_vpc.main.id

  tags = merge(
    var.tags,
    {
      Name = format("%s-Public-Route-Table-%s", var.name, var.environment)
    },
  )
}

# create route for the public route table and attach the internet gateway
resource "aws_route" "public-rtb-route" {
  route_table_id         = aws_route_table.public-rtb.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.ig.id
}

# associate all public subnets to the public route table
resource "aws_route_table_association" "public-subnets-assoc" {
  count          = length(aws_subnet.public[*].id)
  subnet_id      = element(aws_subnet.public[*].id, count.index)
  route_table_id = aws_route_table.public-rtb.id
}
```

![create route tables](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/83e7825a-34bd-40ef-b9e3-127da1d7d4a6)

It is of note that we have introduced a new variable **`var.environment`** which we have defined accordingly in our **`variables.tf`** file as shown below.

![variables specify](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/818ced82-70db-48e2-b5f0-45153cdd6af9)

Now we proceed to run the commands below.

```
$ terraform plan

$ terraform apply
```

![terraform plan](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/89814135-f19b-4ae0-8e7e-975f7962e157)

![terraform apply](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e8325f04-e78d-40f0-882f-606ad57c6c01)

After executing the command, we can see that the following resources have been created:

- [x] - Our main vpc 
- [x] - 2 Public subnets 
- [x] - 4 Private subnets 
- [x] - 1 Internet Gateway
- [x] - 1 NAT Gateway
- [x] - 1 EIP
- [x] - 2 Route tables

![network resources created](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8ac1739d-d9cb-4d53-a3f9-e4fc59f8c8e2)


### <br>AWS Identity and Access Management<br/>

Now that we are done with the Networking part of our AWS set-up, we proceed to move on to identity and Access Control configuration automation using Terraform.

#### IAM and Roles

We want to pass an [IAM](https://docs.aws.amazon.com/iam/index.html) [Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) to our EC2 instances to give them access to some specific resources. To do this, we will need to create **`AssumeRole`** then next, we create an IAM policy for the role and afterwards, we attach the policy to the role.

#### Step 1:  Create [`AssumeRole`](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html).

Assume Role uses Security Token Service (STS) API that returns a set of temporary security credentials that you can use to access AWS resources that you may not normally have access to. These temporary credentials consist of an access key ID, a secret access key, and a security token. Typically,  **`AssumeRole`** is used within your account or for cross-account access.

We proceed by creating a new file **`roles.tf`** and adding the following code:

```
resource "aws_iam_role" "ec2_instance_role" {
  name = "ec2_instance_role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Sid    = ""
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      },
    ]
  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume role"
    },
  )
}
```

![create assume role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/e8416b46-3044-4c31-8246-bff846c540d2)

In the above code block, we are creating **`AssumeRole`** with **`AssumeRole policy`**. It grants to an entity, (in our case an EC2 instance) permissions to assume the role.

#### Step 2: Create an [IAM policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) for this role.

This is where we need to define a required policy (i.e., permissions) according to our requirements. Here, we define a policy allowing an IAM role to perform an action **`describe`** applied to EC2 instances:

```
resource "aws_iam_policy" "policy" {
  name        = "ec2_instance_policy"
  description = "A test policy"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:Describe*",
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]

  })

  tags = merge(
    var.tags,
    {
      Name = "aws assume policy"
    },
  )

}
```

![create iam policy for role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b8d3a9be-166b-445c-9be0-06fd5fc50795)

#### Step 3: Attach the `Policy` to the `IAM Role`. 
    
This is where, we will be attaching the policy which we created above, to the role we created in the first step:

```
resource "aws_iam_role_policy_attachment" "test-attach" {
  role       = aws_iam_role.ec2_instance_role.name
  policy_arn = aws_iam_policy.policy.arn
}
```

![attach policy to IAM role](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3b548c38-83ee-4cf4-8194-eddc02127be9)

#### Step 4: Create an [`Instance Profile`](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html) and interpolate the `IAM Role`.

```
resource "aws_iam_instance_profile" "ip" {
  name = "aws_instance_profile_test"
  role = aws_iam_role.ec2_instance_role.name
}
```

![create instance profile](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/68af537f-58e7-4246-b83d-4ea96d528042)

### <br>Compute Resources<br/>

At this stage, we need to create our compute resources and all other associated resources and configuration. These include Security Groups, Target Group (for Nginx, Wordpress and Tooling), certificate from AWS certificate manager, an External Application Load Balancer and Internal Application Load Balancer, launch template (for Bastion, Tooling, Nginx and Wordpress), and an Auto Scaling Group (ASG) (for Bastion, Tooling, Nginx and Wordpress).

#### Step 1: Create Security Groups.

We are going to create all the security groups in a single file, then we are going to refrence this security group within each resources that needs it.

To proceed, we create a new file **`security.tf`** and paste in the following commands to create security groups for the Internal and External load balancer, the bastion server, Nginx server, the tooling and wordpress webserver and the data layer:

```
# security group for alb, to allow acess from anywhere for HTTP and HTTPS traffic
resource "aws_security_group" "ext-alb-sg" {
  name        = "ext-alb-sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow TLS inbound traffic"

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "ext-alb-sg"
    },
  )

}


# security group for bastion, to allow access into the bastion host from your IP
resource "aws_security_group" "bastion_sg" {
  name        = "bastion_sg"
  vpc_id      = aws_vpc.main.id
  description = "Allow incoming HTTP connections."

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "Bastion-SG"
    },
  )
}

#security group for nginx reverse proxy, to allow access only from the external load balancer and bastion instance
resource "aws_security_group" "nginx-sg" {
  name   = "nginx-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "nginx-SG"
    },
  )
}

resource "aws_security_group_rule" "inbound-nginx-http" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.ext-alb-sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

resource "aws_security_group_rule" "inbound-bastion-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.nginx-sg.id
}

# security group for ialb, to have acces only from nginx reverser proxy server
resource "aws_security_group" "int-alb-sg" {
  name   = "int-alb-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "int-alb-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-ialb-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.nginx-sg.id
  security_group_id        = aws_security_group.int-alb-sg.id
}

# security group for webservers, to have access only from the internal load balancer and bastion instance
resource "aws_security_group" "webserver-sg" {
  name   = "webserver-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "webserver-sg"
    },
  )

}

resource "aws_security_group_rule" "inbound-web-https" {
  type                     = "ingress"
  from_port                = 443
  to_port                  = 443
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.int-alb-sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

resource "aws_security_group_rule" "inbound-web-ssh" {
  type                     = "ingress"
  from_port                = 22
  to_port                  = 22
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.webserver-sg.id
}

# security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port
resource "aws_security_group" "datalayer-sg" {
  name   = "datalayer-sg"
  vpc_id = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = "datalayer-sg"
    },
  )
}

resource "aws_security_group_rule" "inbound-nfs-port" {
  type                     = "ingress"
  from_port                = 2049
  to_port                  = 2049
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-bastion" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.bastion_sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}

resource "aws_security_group_rule" "inbound-mysql-webserver" {
  type                     = "ingress"
  from_port                = 3306
  to_port                  = 3306
  protocol                 = "tcp"
  source_security_group_id = aws_security_group.webserver-sg.id
  security_group_id        = aws_security_group.datalayer-sg.id
}
```

Security group for alb, to allow acess from anywhere for HTTP and HTTPS traffic.

![security group for alb](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/49535659-6b78-4eb9-9861-c6148b3f5566)

Security group for bastion, to allow access into the bastion host from your IP.

![security group for bastion](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/d8339829-e967-4bbf-8118-eb0d8a26c397)

Security group for nginx reverse proxy, to allow access only from the external load balancer and bastion instance.

![security group for nginx reverse proxy](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/8b55e4ba-b1b9-41c7-91c3-f4178983e346)

Security group for ialb, to have access only from nginx reverse proxy server.

![security group for ialb](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b633d9ce-c08b-420b-b84b-65f0c7c033a8)

Security group for webservers, to have access only from the internal load balancer and bastion instance.

![security group for webservers](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/563bc1bf-dd46-4a09-a2cb-5fdd082b53b5)

Security group for datalayer to alow traffic from websever on nfs and mysql port and bastiopn host on mysql port.

![security group for data layer](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f7230468-0054-4e47-ab5d-081ed7a43358)

#### Step 2: Create Certificate From Amazon Certificate Manager.

We create a new file **`cert.tf`** and then we paste in the following code to create and validate a certificate AWS:

```
# The entire section creates a certificate, public zone, and validates the certificate using DNS method.

# Create the certificate using a wildcard for all the domains created in qbdevops.co.uk
resource "aws_acm_certificate" "qbdevops" {
  domain_name       = "*.qbdevops.co.uk"
  validation_method = "DNS"
}

# calling the hosted zone
data "aws_route53_zone" "qbdevops" {
  name         = "qbdevops.co.uk"
  private_zone = false
}

# selecting validation method
resource "aws_route53_record" "qbdevops" {
  for_each = {
    for dvo in aws_acm_certificate.qbdevops.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
    }
  }

  allow_overwrite = true
  name            = each.value.name
  records         = [each.value.record]
  ttl             = 60
  type            = each.value.type
  zone_id         = data.aws_route53_zone.qbdevops.zone_id
}

# validate the certificate through DNS method
resource "aws_acm_certificate_validation" "qbdevops" {
  certificate_arn         = aws_acm_certificate.qbdevops.arn
  validation_record_fqdns = [for record in aws_route53_record.qbdevops : record.fqdn]
}

# create records for tooling
resource "aws_route53_record" "tooling" {
  zone_id = data.aws_route53_zone.qbdevops.zone_id
  name    = "tooling.qbdevops.co.uk"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}


# create records for wordpress
resource "aws_route53_record" "wordpress" {
  zone_id = data.aws_route53_zone.qbdevops.zone_id
  name    = "wordpress.qbdevops.co.uk"
  type    = "A"

  alias {
    name                   = aws_lb.ext-alb.dns_name
    zone_id                = aws_lb.ext-alb.zone_id
    evaluate_target_health = true
  }
}
```

![create certificate](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c509f643-6089-4b7e-a5ef-b5f418b29c23)

#### Step 3: Create External Application Load Balancer.

In this step, we shall create an external (Internet facing) [Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/application-load-balancer-getting-started.html) which serves to balance traffic for the Nginx servers.

We proceed by creating a file **`alb.tf`** and then we paste in the following code to create the external(internet-facing) load balancer, after which we create the target group and then, we create the lsitener rule:

```
# create an ALB to balance the traffic between the Instances

resource "aws_lb" "ext-alb" {
  name     = "ext-alb"
  internal = false
  security_groups = [
    aws_security_group.ext-alb-sg.id,
  ]

  subnets = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "TCS-ext-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
```

![create external alb](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/39b7d600-2156-4967-9907-e7e89030c63c)

<br>To inform our ALB where to  route the traffic we need to create a [`Target Group`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) to point to its targets:<br/>

```
resource "aws_lb_target_group" "nginx-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }
  name        = "nginx-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}
```

![create target group for external](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/b682d45e-0730-4316-88bf-6fffc6940125)

<br>Then we will need to create a [`Listner`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) for this target group:<br/>

```
resource "aws_lb_listener" "nginx-listner" {
  load_balancer_arn = aws_lb.ext-alb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.qbdevops.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.nginx-tgt.arn
  }
}
```

![create listener for external](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/17679de6-25f1-4fa0-b9ac-e33b182a6dbe)

Next, we add the following outputs to **`output.tf`** to print the output values on screen:.

```
output "alb_dns_name" {
  value = aws_lb.ext-alb.dns_name
}

output "alb_target_group_arn" {
  value = aws_lb_target_group.nginx-tgt.arn
}
```

![output](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/17fa5623-d445-44ef-9c55-9f04ea077013)

#### Step 4: Create Internal Application Load Balancer.

Next, we create an Internal (Internal) [Application Load Balancer (ALB)](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-internal-load-balancers.html) to balance traffic between the webservers.

For the Internal Load balancer we will follow the same concepts as with the external load balancer and we add the following code snippets inside the **`alb.tf`** file:

```
# ----------------------------
#Internal Load Balancers for webservers
#---------------------------------

resource "aws_lb" "ialb" {
  name     = "ialb"
  internal = true
  security_groups = [
    aws_security_group.int-alb-sg.id,
  ]

  subnets = [
    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  tags = merge(
    var.tags,
    {
      Name = "TCS-int-alb"
    },
  )

  ip_address_type    = "ipv4"
  load_balancer_type = "application"
}
```

![create internal alb](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/61b3705b-b649-4781-a6d2-ea1575c13b28)

<br>To inform our ALB where to route the traffic, we need to create a [`Target Group`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html) to point to its targets:<br/>

```
# --- target group  for wordpress -------

resource "aws_lb_target_group" "wordpress-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "wordpress-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}

# --- target group for tooling -------

resource "aws_lb_target_group" "tooling-tgt" {
  health_check {
    interval            = 10
    path                = "/healthstatus"
    protocol            = "HTTPS"
    timeout             = 5
    healthy_threshold   = 5
    unhealthy_threshold = 2
  }

  name        = "tooling-tgt"
  port        = 443
  protocol    = "HTTPS"
  target_type = "instance"
  vpc_id      = aws_vpc.main.id
}
```

![create target group for internal](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7c9116fb-9555-41fc-96c4-785ef4da6d22)

<br>Then, we will need to create a [`Listner`](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) for this target Group:<br/>

```
# For this aspect a single listener was created for the wordpress which is default,
# A rule was created to route traffic to tooling when the host header changes


resource "aws_lb_listener" "web-listener" {
  load_balancer_arn = aws_lb.ialb.arn
  port              = 443
  protocol          = "HTTPS"
  certificate_arn   = aws_acm_certificate_validation.qbdevops.certificate_arn


  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.wordpress-tgt.arn
  }
}

# listener rule for tooling target

resource "aws_lb_listener_rule" "tooling-listener" {
  listener_arn = aws_lb_listener.web-listener.arn
  priority     = 99

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.tooling-tgt.arn
  }

  condition {
    host_header {
      values = ["tooling.qbdevops.co.uk"]
    }
  }
}
```

![create listener for internal](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c2055fe0-50fe-4556-af46-e346fa632df0)

####  Step 5: Create An Auto Scaling Group (ASG).

Now, we need to configure our ASG to be able to scale the EC2s in and out, depending on the application traffic.

Before we start configuring an ASG, we need to create the launch template and the the AMI needed. For now we are going to use a random AMI from AWS, then in project 19, we will use [Packer](https://www.packer.io/intro)to create our ami.

 Based on the architecture we need for Auto Scaling groups for bastion, nginx, wordpress and tooling, we will create two files; **`asg-bastion-nginx.tf`** will contain Launch Template and Austoscaling group for Bastion and Nginx, while **`asg-wordpress-tooling.tf`** will contain Launch Template and Austoscaling group for wordpress and tooling.

To proceed, we create a file **`asg-bastion-nginx.tf`** wand we paste in the following code which creates notification for all the auto scaling group:

```
#### creating sns topic for all the auto scaling groups
resource "aws_sns_topic" "abdul-sns" {
  name = "Default_CloudWatch_Alarms_Topic"
}

# creating notification for all the auto scaling groups

resource "aws_autoscaling_notification" "abdul_notifications" {
  group_names = [
    aws_autoscaling_group.bastion-asg.name,
    aws_autoscaling_group.nginx-asg.name,
    aws_autoscaling_group.wordpress-asg.name,
    aws_autoscaling_group.tooling-asg.name,
  ]
  notifications = [
    "autoscaling:EC2_INSTANCE_LAUNCH",
    "autoscaling:EC2_INSTANCE_TERMINATE",
    "autoscaling:EC2_INSTANCE_LAUNCH_ERROR",
    "autoscaling:EC2_INSTANCE_TERMINATE_ERROR",
  ]

  topic_arn = aws_sns_topic.abdul-sns.arn
}

resource "random_shuffle" "az_list" {
  input = data.aws_availability_zones.available.names
}

resource "aws_launch_template" "bastion-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.bastion_sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "$(random_shuffle.az_list.result)"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "bastion-launch-template"
      },
    )
  }

  user_data = filebase64("${path.module}/bastion.sh")
}

# ---- Autoscaling for bastion  hosts

resource "aws_autoscaling_group" "bastion-asg" {
  name                      = "bastion-asg"
  max_size                  = 2
  min_size                  = 2
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 2

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.bastion-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "TCS-bastion"
    propagate_at_launch = true
  }

}

# launch template for nginx

resource "aws_launch_template" "nginx-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.nginx-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "random_shuffle.az_list.result"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "nginx-launch-template"
      },
    )
  }

  user_data = filebase64("${path.module}/nginx.sh")
}

# ------ Autoscaling group for reverse proxy nginx ---------

resource "aws_autoscaling_group" "nginx-asg" {
  name                      = "nginx-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [
    aws_subnet.public[0].id,
    aws_subnet.public[1].id
  ]

  launch_template {
    id      = aws_launch_template.nginx-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "TCS-nginx"
    propagate_at_launch = true
  }

}

# attaching autoscaling group of nginx to external load balancer
resource "aws_autoscaling_attachment" "asg_attachment_nginx" {
  autoscaling_group_name = aws_autoscaling_group.nginx-asg.id
  alb_target_group_arn   = aws_lb_target_group.nginx-tgt.arn
}
```

![asg bastion 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/10067e4c-ea94-44e6-9823-c72aa86cbdce)

![asg bastion 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ab906fec-164d-4949-b37c-79564a2251c3)

![asg bastion 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/7ec52ebd-2d2f-4610-ad02-b8c93857b2b0)

<br>Next, we create a new file **`asg-wordpress-tooling.tf`** and we enter the following code which creates launch templates and Auto Scaling Group for both the wordpress and tooling webserver:<br/>

```
# launch template for wordpress

resource "aws_launch_template" "wordpress-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "$(random_shuffle.az_list.result)"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "wordpress-launch-template"
      },
    )

  }

  user_data = filebase64("${path.module}/wordpress.sh")
}

# ---- Autoscaling for wordpress application

resource "aws_autoscaling_group" "wordpress-asg" {
  name                      = "wordpress-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1
  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.wordpress-launch-template.id
    version = "$Latest"
  }
  tag {
    key                 = "Name"
    value               = "TCS-wordpress"
    propagate_at_launch = true
  }
}

# attaching autoscaling group of  wordpress application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_wordpress" {
  autoscaling_group_name = aws_autoscaling_group.wordpress-asg.id
  alb_target_group_arn   = aws_lb_target_group.wordpress-tgt.arn
}

# launch template for toooling
resource "aws_launch_template" "tooling-launch-template" {
  image_id               = var.ami
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.webserver-sg.id]

  iam_instance_profile {
    name = aws_iam_instance_profile.ip.id
  }

  key_name = var.keypair

  placement {
    availability_zone = "$(random_shuffle.az_list.result)"
  }

  lifecycle {
    create_before_destroy = true
  }

  tag_specifications {
    resource_type = "instance"

    tags = merge(
      var.tags,
      {
        Name = "tooling-launch-template"
      },
    )

  }

  user_data = filebase64("${path.module}/tooling.sh")
}

# ---- Autoscaling for tooling -----

resource "aws_autoscaling_group" "tooling-asg" {
  name                      = "tooling-asg"
  max_size                  = 2
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "ELB"
  desired_capacity          = 1

  vpc_zone_identifier = [

    aws_subnet.private[0].id,
    aws_subnet.private[1].id
  ]

  launch_template {
    id      = aws_launch_template.tooling-launch-template.id
    version = "$Latest"
  }

  tag {
    key                 = "Name"
    value               = "TCS-tooling"
    propagate_at_launch = true
  }
}


# attaching autoscaling group of  tooling application to internal loadbalancer
resource "aws_autoscaling_attachment" "asg_attachment_tooling" {
  autoscaling_group_name = aws_autoscaling_group.tooling-asg.id
  alb_target_group_arn   = aws_lb_target_group.tooling-tgt.arn
}
```

![asg wordpress 1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/f9271018-e90e-4909-ba56-8975986856bb)

![asg wordpress 2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/c4d76d74-ddd1-4399-acf1-68580c72acae)

![asg wordpress 3](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/321aefc0-1cdb-4e14-9124-b1252a657248)

![asg wordpress 4](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/9c6170ce-0177-4d2f-bd94-821227ab6503)

<br>Subsequently, we create four files that will be used as user data for launching the four different servers:<br/>

 **i.** **For the bastion server `bastion.sh`**

```
#!/bin/bash 
yum install -y mysql 
yum install -y git tmux 
yum install -y ansible
```

![bastionsh](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/1902e39f-4606-4081-a1bb-d84525337d5f)

**ii.** **For the Nginx server `nginx.sh`**

```
#!/bin/bash

cd ~
sudo yum install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo git clone https://github.com/Tonybesto/TCS-Project-Configuration.git
sudo cp RCR-Project-Configuration/reverseProxy.conf /etc/nginx/
sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
sudo touch nginx.conf
sudo sed -n 'w nginx.conf' reverseProxy.conf
sudo systemctl restart nginx
sudo rm -rf reverseProxy.conf
```

![nginxsh](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/07ea1198-b012-4f81-8f8a-e1cd4a6257c4)

**iii.** **For the Wordpress server `wordpress.sh`**

```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-001f1cb7432271a8b fs-0a487970ca2f2d267:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/tcs-database.cgk2jcnauxqt.us-east-2.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/TCSadmin/g" wp-config.php 
sed -i "s/password_here/1234567890/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
```

![wordpresssh](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/09f29d3e-4f61-4a2b-b4a8-1c0660a939fd)

**iv.** **For Tooling Webserver `tooling.sh`**

```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-0044c4bc467c71528 fs-0a487970ca2f2d267:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/Tonybesto/tooling.git
mkdir /var/www/html
cp -R /tooling/html/*  /var/www/html/
cd /tooling
mysql -h rcr-dbmysql.crvnhmpyxtuf.us-east-1.rds.amazonaws.com -u admin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('172.31.32.49', 'webaccess', 'password', 'tooling');/$db = mysqli_connect('tcs-database.cgk2jcnauxqt.us-east-2.rds.amazonaws.com', 'TCSadmin', '1234567890', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
```

![toolingsh](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ace05f55-b103-4dd9-b4cc-08740af78927)

### <br>Storage and Database Resources<br/>

Now we are at the stage of our project where we will be creating AWS storage and database resources namely Elastic File System (EFS) and [MySQL Relational Database System (RDS)](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html).

#### Step 1: Create Elastic File System (EFS).

In order to create an EFS we need to create a [KMS key](https://aws.amazon.com/kms/getting-started/).

AWS Key Management Service (KMS) makes it easy for us to create and manage cryptographic keys and control their use across a wide range of AWS services and also in our applications.

To proceed, we create the file, **`efs.tf`** and we add in the following code snippet: 

```
# create key from key management system
resource "aws_kms_key" "ACS-kms" {
  description = "KMS key "
  policy      = <<EOF
  {
  "Version": "2012-10-17",
  "Id": "kms-key-policy",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::${var.account_no}:user/segun" },
      "Action": "kms:*",
      "Resource": "*"
    }
  ]
}
EOF
}

# create key alias
resource "aws_kms_alias" "alias" {
  name          = "alias/kms"
  target_key_id = aws_kms_key.ACS-kms.key_id
}

```

<br>Next, we create EFS and it's mount targets by adding the following code block to **`efs.tf`**:<br/>

```
# create Elastic file system
resource "aws_efs_file_system" "ACS-efs" {
  encrypted  = true
  kms_key_id = aws_kms_key.ACS-kms.arn

  tags = merge(
    var.tags,
    {
      Name = "ACS-efs"
    },
  )
}



# set first mount target for the EFS 
resource "aws_efs_mount_target" "subnet-1" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = aws_subnet.private[2].id
  security_groups = [aws_security_group.datalayer-sg.id]
}


# set second mount target for the EFS 
resource "aws_efs_mount_target" "subnet-2" {
  file_system_id  = aws_efs_file_system.ACS-efs.id
  subnet_id       = aws_subnet.private[3].id
  security_groups = [aws_security_group.datalayer-sg.id]
}


# create access point for wordpress
resource "aws_efs_access_point" "wordpress" {
  file_system_id = aws_efs_file_system.ACS-efs.id

  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {
    path = "/wordpress"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }

}


# create access point for tooling
resource "aws_efs_access_point" "tooling" {
  file_system_id = aws_efs_file_system.ACS-efs.id
  posix_user {
    gid = 0
    uid = 0
  }

  root_directory {

    path = "/tooling"

    creation_info {
      owner_gid   = 0
      owner_uid   = 0
      permissions = 0755
    }

  }
}
```

![efs1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/5095641e-dc79-43a7-b1ca-0bc61bde2037)

![efs2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/3338619b-9dfa-4aeb-9baa-5a196d91e455)

#### Step 2: Create MySQL RDS.

To create the RDS, we create the **`rds.tf`** file and then we add the following snippet of code:

```
# This section will create the subnet group for the RDS instance using the private subnet
resource "aws_db_subnet_group" "ACS-rds" {
  name       = "acs-rds"
  subnet_ids = [aws_subnet.private[2].id, aws_subnet.private[3].id]

 tags = merge(
    var.tags,
    {
      Name = "ACS-rds"
    },
  )
}


# create the RDS instance with the subnets group
resource "aws_db_instance" "ACS-rds" {
  allocated_storage      = 20
  storage_type           = "gp2"
  engine                 = "mysql"
  engine_version         = "5.7"
  instance_class         = "db.t2.micro"
  name                   = "abduldb"
  username               = var.master-username
  password               = var.master-password
  parameter_group_name   = "default.mysql5.7"
  db_subnet_group_name   = aws_db_subnet_group.ACS-rds.name
  skip_final_snapshot    = true
  vpc_security_group_ids = [aws_security_group.datalayer-sg.id]
  multi_az               = "true"
}
```

![rds](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/79d1a8f7-19b0-4ade-beb7-8d20d2a0af51)

### <br>Conclusion<br/> 

We have now come to the tail end of this project and we pretty much have all the infrastructure elements ready to be deployed automatically.

However, it is imperative to note that we have referenced some variables in our resources and we have declared them in the **`variables.tf`** file. Therefore, our **`variables.tf`** now looks as shown below:

```
variable "region" {
  type = string
  description = "The region to deploy resources"
}

variable "vpc_cidr" {
  type = string
  description = "The VPC cidr"
}

variable "enable_dns_support" {
  type = bool
}

variable "enable_dns_hostnames" {
  dtype = bool
}

variable "enable_classiclink" {
  type = bool
}

variable "enable_classiclink_dns_support" {
  type = bool
}

variable "preferred_number_of_public_subnets" {
  type        = number
  description = "Number of public subnets"
}

variable "preferred_number_of_private_subnets" {
  type        = number
  description = "Number of private subnets"
}

variable "name" {
  type    = string
  default = "ACS"

}

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}


variable "ami" {
  type        = string
  description = "AMI ID for the launch template"
}


variable "keypair" {
  type        = string
  description = "key pair for the instances"
}

variable "account_no" {
  type        = number
  description = "the account number"
}


variable "master-username" {
  type        = string
  description = "RDS admin username"
}

variable "master-password" {
  type        = string
  description = "RDS master password"
}
```

![variablestf1](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/85472e88-132e-4b69-b523-34c12352115b)

![variablestf2](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ade37651-7cc9-4422-94b4-28a6d5aaf626)

<br>Subsequently, we have also updated our **`terraform.tfvars`** file where we have declared the values for the variables in our **`varibles.tf`** file:<br/>

```
region = "us-east-1"

vpc_cidr = "172.16.0.0/16"

enable_dns_support = "true"

enable_dns_hostnames = "true"

enable_classiclink = "false"

enable_classiclink_dns_support = "false"

preferred_number_of_public_subnets = "2"

preferred_number_of_private_subnets = "4"

environment = "dev"

ami = "ami-0b0af3577fe5e3532"

keypair = "qb-ex"

# Ensure to change this to your acccount number
account_no = "123456789"


db-username = "abdul"


db-password = "devopspbl"


tags = {
  Enviroment      = "production" 
  Owner-Email     = "moyor_bello@yahoo.co.uk"
  Managed-By      = "Terraform"
  Billing-Account = "1234567890"
}
```

![tfvars](https://github.com/QuadriBello/DevOps-Cloud/assets/140855364/ca5ab8b7-b213-4252-ad26-44b307e99407)

<br>We now proceed to run the following commands to deploy our resources:<br/>

```
$ terraform plan

$ terraform apply --auto-approve
```

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
