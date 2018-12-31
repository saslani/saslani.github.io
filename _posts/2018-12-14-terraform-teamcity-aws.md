---
layout: post
title:  "Host Teamcity on AWS, using Terraform and Ansible"
date:   2018-12-26 16:15:15
categories: Infrastructure as Code
---

## NOTE
- This article is still **WIP** but I noticed some clones on my github already. So I am putting this up, in case anybody needs it sooner :)


I see infrastructure as engineering and not just operations. We made changes, add or delete modules, and not just maintaining them or keeping things running. Hence, I am pro Infrastructure as Code. Terraform is a great tool for Infrastructure engineering. Moreover,
- I like programing rather than click though UI!
- I want to minimize my work if I have to change the host (i.e. move from AWS to Azure, etc)
- I see the graph dependencies
- I want to encourage my team members who are not keen on "DevOpsy" work, to see how easy it is to use Infrastructure as Code


In this tutorial, we'll build a very basic infrastructure to host TeamCity on AWS, using terraform. Some familiarity with AWS, TeamCity and programing knowledge is required, this is not an AWS tutorial. This tutorial provides a very basic hosting, if you need a more sophisticated setup, this is a good starting point for you, then you can add your specific infrastructure requirements. ELBs, high availability, zero downtime, disaster recovery, IP range limitation etc, are out of scope for this tutorial.

#### Prerequisites:
- Good understanding of AWS: We won't be focusing on AWS UI and how to set things up via click-through. Although, I will show you how to navigate AWS to visually see your changes.
- Fair understanding of TeamCity: We will not be going through TeamCity training. We will just confirm that it works properly
- Familiarity with a programing language
- We won't go deep into learning terraform, you can refer to the HashiCorp documents if you want to learn more

#### Getting started:
- [TeamCity](https://www.jetbrains.com/teamcity/download/) 2017.2+ has a free account with 3 build agents for 100 build configurations. So you can start there and purchase the license if you need to.
- [Install terraform](https://learn.hashicorp.com/terraform/getting-started/install.html)
  + brew install terraform
- Get an aws account (if you just want to use Azure, then read the next article)
  + create a [non-admin user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console) with proper accesses to perform the following. Avoid using your admin user to make changes in the AWS. Here are my Permissions, adjust based on your needs:
    * AmazonRDSFullAccess
    * AmazonEC2FullAccess
    * AmazonS3FullAccess
    * AmazonDynamoDBFullAccess
    * AmazonVPCFullAccesss
  + If you want to add Route53 or a different IAM group, or auto scaling, not required for this tutorial:
    * CloudWatchFullAccess
    * IAMFullAccess
    * AmazonRoute53FullAccess

**NOTE:** 
- Small AWS charge. To keep the charge minimum run **terraform destroy** at the end of the tutorial, if you wish.
- I am using _main.tf_, _variables.tf_, and _outputs.tf_ consistently throughout this project, you may wish to change the names, or just develop in one single file. All tf files in a directory will be compiled together. So separating them, is my personal preference.
- I am using MAC OS Majave, so change the bash commands appropriately, if you are using any other OS


#### What are we building?

- Create a VPC
- Building a private subnet for RDS
  + Create a public subnet for NAT
  + Add NAT
  + Create a public subnet
  + Create routing
  + Create a security group for the public Subnet
  + Create a security group for the private Subnet
  + Create private subnets
  + Create routing 
- Add RDS
  + Create the database
  + Create S3 backup
- Single public subnet for a public EC2 to host the TeamCity
  + Create a public subnet
  + Create security group
  + Modify the routing
- Launch a public EC2 instance to host the TeamCity (using docker image) and connect to RDS PostgreSQL


Let's get started!!

## Create a VPC
If you have created your amazon account within the past couple of years, it's likely that it comes with a default VPC. We are going to create an isolated Virtual Private Cloud and subnets and use them to host TeamCity.

Once you installed terraform, create an empty folder where you want to write your code, and cd into it:

```bash
$ mkdir teamcity
$ cd teamcity
```

In the root directory, create the main.tf. **tf** is terraform convention. We will use this _main.tf_ as a driver that contains other modules to build pieces we need.

```bash
$ touch main.tf
```

Now let's write some terraform code. The biggest advantage of infrastructure as code, as we discussed earlier is that it's scalable and easy to change provider. In this tutorial we're going to use AWS, thus, our provider is aws. 

I am going to use **us-east-1**. You can use any region you like.

_main.tf_
```
provider "aws" {
  region = "us-east-1"
}
```

Next, we want to add [**aws_vpc**](https://www.terraform.io/docs/providers/aws/r/vpc.html) resource to create our VPC. We call it **vpc**. This is like naming a variable so you can access it later throughout your project.

For [cidr_block](https://whatismyipaddress.com/cidr), we're using the **10.0.0.0** as opposed to **192.168** because it's more common, 192.168 is mainly associated with your personal IP. 

enable_dns_hostnames by default is false. We want to enable DNS hostnames in the VPC, so set it to true.

Add tags, Name, to see the name in the VPC list, specially if you have more than 1 VPC, so that you can easily distinguish them.

_main.tf_
```
resource "aws_vpc" "vpc" {
  cidr_block            = "10.0.0.0/16"
  enable_dns_hostnames  = true

  tags {
    Name = "Teamcity VPC"
  }
}
```

Now you have the most basic code to see terraform in action. In your console, I use iTerm, initialize terraform:

```bash
$ terraform init
```

If you look at your root directory, you see a .terraform folder. This is to keep the current status of your terraform commands and keep track of what's been created or changed.

To see the plan before you execute it, run the following command:

```bash
$ terraform plan
```

You will see the following error because you haven't told terraform how to access your amazon account!

```bash
Error: Error refreshing state: 1 error(s) occurred:

* provider.aws: No valid credential sources found for AWS Provider.
  Please see https://terraform.io/docs/providers/aws/index.html for more information on
  providing credentials for the AWS Provider
```

You can do this in couple different ways. You can export the credentials as environment variables into the terminal shell and you won't be prompted as long as you use that shell. Or you can save them into a ***.tfvars** file and add it to the provider.

**Export into the shell** (if you choose this skip to ****)
```bash
$ export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY"
$ export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY"
```

and if you want to use a **.tfvars**, add the terraform file to your root directory:

```bash
$ touch terraform.tfvars
```

_terraform.tfvars_
```
aws_access_key = "YOUR_ACCESS_KEY"
aws_secret_key = "YOUR_SECRET_KEY"
```

_main.tf_
```
provider "aws" {
  .

  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
}
```

Run 

```bash 
$ terraform plan
``` 

Now you see a different error: :/

```bash
Error: provider config 'aws': unknown variable referenced: 'aws_access_key'; define it with a 'variable' block
Error: provider config 'aws': unknown variable referenced: 'aws_secret_key'; define it with a 'variable' block
```

In the root directory create a _variables.tf_ file. For the rest of this tutorial, we'll use this file to declare what variables we need for the modules we are going to build.

```bash
$ touch variables.tf
```

Terraform variables are declared in a variable block. You can declare a type, by default it's string, description and a default value

_variables.tf_
```
variable "aws_access_key" {
  description = "AWS access key"
}

variable "aws_secret_key" {
  description = "AWS secret key"
}
```

Try again:
```bash
$ terraform plan
``` 

to see the plan that's going to be executed and verify that's what your want to do. In our case, we want to **create** a **vpc**:

```bash
data.aws_availability_zones.zones: Refreshing state...

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + module.vpc.aws_vpc.vpc
      id:                               <computed>
      arn:                              <computed>
      assign_generated_ipv6_cidr_block: "false"
      cidr_block:                       "10.0.0.0/16"
      default_network_acl_id:           <computed>
      default_route_table_id:           <computed>
      default_security_group_id:        <computed>
      dhcp_options_id:                  <computed>
      enable_classiclink:               <computed>
      enable_classiclink_dns_support:   <computed>
      enable_dns_hostnames:             "true"
      enable_dns_support:               "true"
      instance_tenancy:                 "default"
      ipv6_association_id:              <computed>
      ipv6_cidr_block:                  <computed>
      main_route_table_id:              <computed>
      owner_id:                         <computed>
      tags.%:                           "1"
      tags.Name:                        "Teamcity VPC"


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------
```

Looks great, run the following command to execute the plan. You will be prompted to verify if that's what you want to do, enter **yes**.
```bash
$ terraform apply
``` 


Great!!!! Now if you navigate to VPC on your AWS account (AWS > Services > VPC). You see that the VPC is created! This is step 1 in your architecture diagram:

![](/assets/images/infrastructure_as_code/terraform_teamcity_aws/create_vpc.png "Teamcity VPC")


## Building a private subnet for RDS


In this section we're going to add more code to our terraform project. Before we do so, let's do a small refactoring to keep things organized going forward. Some of you may think, it's not need yet unless I call the same block of code 3 times..., you may go ahead and use a single file. Personally, I like to add a little structure to my projects, knowing this is going to grow.

In terraform, modules, are similar to classes in OO programing. I want to move the job of creating [VPC](https://www.terraform.io/docs/providers/aws/d/vpc.html), [subnets](https://www.terraform.io/docs/providers/aws/d/subnet.html), [NAT](https://www.terraform.io/docs/providers/aws/d/nat_gateway.html), and [routing](https://www.terraform.io/docs/providers/aws/d/route_table.html) to it's own module. 

```bash
$ mkdir vpc
$ touch vpc/main.tf
```

Now let's move the "aws_vpc" from _main.tf_ to _vpc/main.tf_

_vpc/main.tf_
```
resource "aws_vpc" "vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags {
    Name = "Teamcity VPC"
  }
}
```

Now change the _main.tf_ to call module vpc. The source's value is the path to the vpc module. You can also include a git url, if a different team is in charge of that module.

_main.tf_
```
.
.

module "vpc" {
  source = "vpc"
}
```

Because we just added a folder (module vpc), we need to run terraform init to let terraform know about this change:
```bash
$ terraform init
```

Moving forward...

RDS needs multiple zones and at least 2 private subnets. Before we add a private subnet, however, we need to create a NAT. NAT needs an Elastic IP to work. NAT, needs to live in the public subnet :D  ...

So first, let's go ahead and make those changes. We will be using [aws_eip](https://www.terraform.io/docs/providers/aws/r/eip.html), [aws_nat_gateway](https://www.terraform.io/docs/providers/aws/d/nat_gateway.html)

```bash
$ touch vpc/nat.tf
```

_vpc/nat.tf_
```
resource "aws_eip" "nat_gw_eip" {
  vpc = true

  tags {
    Name = "TeamCity NAT"
  }
}

resource "aws_nat_gateway" "gw" {
  allocation_id = "${aws_eip.nat_gw_eip.id}"
  subnet_id     = "${aws_subnet.public.id}"

  tags {
    Name = "TeamCity NAT Gateway"
  }
}
```

Now we're ready to add a single public subnet. 

```bash
$ touch vpc/subnets.tf
```

Navigate to _vpc/subnets.tf_ inside the vpc folder. We want to add an [aws_subnet](https://www.terraform.io/docs/providers/aws/d/subnet.html) resource and name it public. I am passing a variable _availability_zone_ since I don't have a preference. If you do, you can hard-code this to be "us-east-1a" for example. 

```
.
.

resource "aws_subnet" "public" {
  availability_zone = "${element(var.availability_zones, count.index)}"
  cidr_block        = "${var.public_cidr_block}"
  vpc_id            = "${aws_vpc.vpc.id}"

  tags {
    Name = "Public TeamCity Subnet"
  }
}
```

Now we're ready to create a route table and association for the NAT.

```bash
$ touch vpc/routing.tf
```

In this file, we need to create an [aws_internet_gateway](https://www.terraform.io/docs/providers/aws/d/internet_gateway.html) (we name it vpc_igw). Then create an [aws_route_table](https://www.terraform.io/docs/providers/aws/d/route_table.html) (vpc_public). Last but not the least, an association resource, [aws_route_table_association](https://www.terraform.io/docs/providers/aws/r/route_table_association.html) (vpc_public).

_vpc/routing.tf_
```
resource "aws_internet_gateway" "vpc_igw" {
  vpc_id = "${aws_vpc.vpc.id}"

  tags {
    Name = "TeamCity Gateway"
  }
}

resource "aws_route_table" "vpc_public" {
  vpc_id = "${aws_vpc.vpc.id}"

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.vpc_igw.id}"
  }

  tags {
    Name = "TeamCity Public Subnet Route Table"
  }
}

resource "aws_route_table_association" "vpc_public" {
  subnet_id      = "${aws_subnet.public.id}"
  route_table_id = "${aws_route_table.vpc_public.id}"
}
```

Everything looks good, since we are already calling th vpc module from main.tf, let's go ahead and see the plan so far:

```bash
$ terraform plan
```

Looks like we need to pass in some variables

```bash
Error: resource 'aws_subnet.public' config: unknown variable referenced: 'availability_zones'; define it with a 'variable' block
Error: resource 'aws_subnet.public' config: unknown variable referenced: 'public_cidr_block'; define it with a 'variable' block
```

```bash
$ touch vpc/variables.tf
```

Let's create a default cidr_block inside the _vpc/variables.tf_ but pass in the availability_zones from _main.tf_.

_vpc/variables.tf_
```
variable "availability_zones" {
  description = "List of availability zones over which to distribute subnets"
  type        = "list"
}

variable "public_cidr_block" {
  default = "10.0.0.0/24"
}
```

and change the _main.tf_, where we call the vpc module:
```
.
.

data "aws_availability_zones" "zones" {}

module "vpc" {
  .
  availability_zones = ["${data.aws_availability_zones.zones.names}"]
}
```

Before we apply our changes, let's create a security group for this public VPC as well:
```bash
$ mkdir sg && touch sg/main.tf
```

Inside the sg/main.tf, we are going to create an [aws_security_group](https://www.terraform.io/docs/providers/aws/d/internet_gateway.html) (teamcity_web_sg) and add ingress (right to enter a property) and egress (right to exit a property) rules:

_sp/main.tf_
```
resource "aws_security_group" "teamcity_web_sg" {
  name        = "TeamCity_sg"
  description = "Allow TeamCity SSH & HTTP inbound connection"
  vpc_id      = "${var.vpc_id}"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8111
    to_port     = 8111
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = "0"
    to_port     = "0"
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags {
    Name = "TeamCity Web Security Group"
  }
}
```

We are using the vpc_id to create the routing, let's go ahead and pass that:

```bash
$ touch sg/variables.tf
```

_sg/variables.tf_
```
```

Last, but not the least, we need to call this module from _main.tf_
```
.
.

module "sg" {
  source = "sg"
  vpc_id = "${module.vpc.vpc_id}"
}
```

Looks like we have a solid code to build our public subnet. Since we added sg module, we need to init again:
```bash
$ terraform init
```

And there's error:
```bash
Initializing modules...
- module.vpc
- module.sg
  Getting source "sg"

Error: module 'sg': "vpc_id" is not a valid output for module "vpc"
```

This is because we're passing vpc_id, but we never defined it anywhere:
```bash
$ touch vpc/outputs.tf
```

_vpc/outputs.tf_
```
output "vpc_id" {
  value = "${aws_vpc.vpc.id}"
}
```

Let's try again:
```bash
$ terraform init
$ terraform plan
# Notice all the new resources that are in green
# Plan: 7 to add, 0 to change, 0 to destroy.

$ terraform apply
```

If you want to see all the changes you made, navigate to:
- AWS > VPCs
- Subnets
- Route Tables
- Internet Gateways
- Elastic IPs
- Nat Gateways
- Network ACLs
- Security Groups

That's a lot of changes! This is a good place to commit our changes to git, if you haven't done it yet. This is my _.ignore_ file:
```bash
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate
*.tfstate.*

# secret files
*.tfvars

# Crash log files
crash.log

# Ignore override files as they are usually used to override resources locally and so
# are not checked in
override.tf
override.tf.json
*_override.tf
*_override.tf.json
```

Now, we're ready to create our private subnet and security group for the RDS:

_vpc/subnets.tf_
```
.
.

resource "aws_subnet" "private" {
  availability_zone = "${element(var.availability_zones, count.index)}"
  cidr_block        = "${element(var.private_cidr_block, count.index)}"
  count             = "${length(var.private_cidr_block)}"
  vpc_id            = "${aws_vpc.vpc.id}"

  tags {
    Name = "${format("TeamCity Private Subnet %d", count.index + 1)}"
  }
}

resource "aws_db_subnet_group" "rds" {
  name        = "teamcity-subnet-group"
  description = "TeamCity RDS Subnet Group"
  subnet_ids  = ["${aws_subnet.private.*.id}"]

  tags {
    Name = "TeamCity RDS Subnet Group"
  }
}
```

_vpc/routing.tf_
```
.
.

resource "aws_route_table" "vpc_private" {
  vpc_id = "${aws_vpc.vpc.id}"

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_nat_gateway.gw.id}"
  }

  tags {
    Name = "TeamCity Private Subnet's Route Table"
  }
}

resource "aws_route_table_association" "vpc_private" {
  count          = "${var.length}"
  subnet_id      = "${element(aws_subnet.private.*.id, count.index)}"
  route_table_id = "${aws_route_table.vpc_private.id}"
}
```

_vpc/variables.tf_
```
.
.

variable "length" {
  default = 1
}

variable "private_cidr_block" {
  type    = "list"
  default = ["10.0.1.0/24", "10.0.2.0/24"]
}
```

and the RDS security group:
```
resource "aws_security_group" "rds_sg" {
  name        = "TeamCity_rds_sg"
  description = "TeamCity RDS Security Group"
  vpc_id      = "${var.vpc_id}"

  # TODO: Should this change to only allow ssh from other IPs?
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = "0"
    to_port     = "0"
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags {
    Name = "TeamCity RDS Security Group"
  }
}
```

Apply the changes:
```
$ terraform apply
```

Congrats! You have everything you need to build your RDS now! Don't forget to commit your changes often!
So far, this is how my folder structure looks like:
```bash
.
├── main.tf
├── sg
│   ├── main.tf
│   └── variables.tf
├── terraform.tfstate
├── terraform.tfstate.backup
├── terraform.tfvars
├── variables.tf
└── vpc
    ├── main.tf
    ├── nat.tf
    ├── outputs.tf
    ├── routing.tf
    ├── subnets.tf
    └── variables.tf
```

and this is our infrastructure document:

### TBC: Infrastructure Diagram

## Add RDS

Let's add our rds module first:
```bash
$ mkdir rds && touch rds/main.tf && touch rds/variables.tf
```

We will be using [aws_db_instance](https://www.terraform.io/docs/providers/aws/d/db_instance.html) resource.
_rds/main.tf_
```
resource "aws_db_instance" "database" {
  identifier                = "${var.instance_identifier}"
  final_snapshot_identifier = "${var.instance_identifier}"
  skip_final_snapshot       = true
  allocated_storage         = "60"
  storage_type              = "gp2"
  engine                    = "postgres"
  instance_class            = "db.m3.medium"
  name                      = "${var.db_name}"
  username                  = "${var.db_username}"
  password                  = "${var.db_password}"
  port                      = 5432
  publicly_accessible       = false
  vpc_security_group_ids    = ["${var.vpc_security_group_ids}"]
  db_subnet_group_name      = "${var.db_subnet_group_name}"
  multi_az                  = true
  backup_retention_period   = 7
  backup_window             = "08:17-08:47"
  maintenance_window        = "sat:09:30-sat:22:00"

  tags {
    Name = "TeamCity RDS"
  }
}
```

_rds/variables.tf_
```
variable "db_password" {
  default = "default value"
}

variable "db_subnet_group_name" {
  default = "default value"
}

variable "db_name" {
  default = "default value"
}

variable "db_username" {
  default = "default value"
}

variable "dns_name" {
  default = "default value"
}

variable "instance_identifier" {
  default = "default value"
}

variable "private_subnet_id" {
  default = "default value"
}

variable "service_name" {
  default = "default value"
}

variable "vpc_id" {
  default = "default value"
}

variable "vpc_security_group_ids" {
  default = "default value"
}
```

Call the rds module form _main.tf_
```
.
.

module "rds" {
  source                 = "rds"
  db_name                = "${var.db_name}"
  db_password            = "${var.db_password}"
  db_username            = "${var.db_username}"
  db_subnet_group_name   = "${module.vpc.db_subnet_group_name}"
  dns_name               = "pg"
  instance_identifier    = "${var.db_name}"
  vpc_id                 = "${module.vpc.vpc_id}"
  private_subnet_id      = ["${module.vpc.private_subnet}"]
  service_name           = "TeamCity"
  vpc_security_group_ids = "${module.sg.rds_security_groups_id}"
}
```

If you run the init, you will see that we're missing a few variables. Let's go ahead and get rid of those errors. First is db_password. We are going to add the password to the terraform.tfvars. This file is not getting checked and it's a safe place to store our secrets.

_terraform.tfvars_
```
db_password = "YOUR_DB_PASSWORD"
```

_variables.tf_
``` terraform
.
.

variable "db_name" {
  default = "teamcity"
}

variable "db_password" {
  type = "string"
}

variable "db_username" {
  default = "teamcityuser"
}
```

We also need to add a few outputs:
```bash
$ touch sg/outputs.tf
```

_sg/outputs.tf_
```
output "rds_security_groups_id" {
  value = "${aws_security_group.rds_sg.id}"
}
```

_vpc/outputs/tf_
```
output "private_subnet" {
  value = ["${aws_subnet.private.*.id}"]
}

output "db_subnet_group_name" {
  value = "${aws_db_subnet_group.rds.name}"
}
```

Now we're ready:
```
$ terraform refresh
$ terraform apply
```

This is going to take a while... so go head for a coffee break and stretching. See you in about 15-20 min!
.
.
.

Nice! Navigate to AWS > RDS to see your newly created RDS!

While we're at it, let's create an S3 bucket for the backup
This is going to be very short and sweet!

```bash
$ mkdir s3 && touch s3/main.tf && touch s3/variables.tf && touch s3/outputs.tf
```

_s3/main.tf_
```
resource "aws_s3_bucket" "backup_bucket" {
  bucket = "${var.name}"
  acl    = "private"

  tags {
    Name = "${var.description}"
  }
}
```

_s3/variables.tf_
```
variable "name" {
  description = "Name of the S3 Bucket"
}

variable "description" {
  description = "Description of the S3 Bucket (for tagging)"
}
```

_s3/outputs.tf_
```
output "arn" {
  value = "${aws_s3_bucket.backup_bucket.arn}"
}
```

_main.tf_
```
.
.

#S3 bucket name has to be unique
module "backup_bucket" {
  source      = "s3"
  name        = "teamcity-backups-84121243"
  description = "TeamCity Backups"
}
```

Apply changes
```bash
$ terraform init
$ terraform apply
```

Looks good!

**Summary:**
So far we have created a VPC, a public subnet, NAT, 2 private subnets, public, private and rds security groups, last but not the least, we created an RDS in the private subnet and an S3 bucket for the backup. At this point, my folder structure looks like this:

```bash
.
├── main.tf
├── rds
│   ├── main.tf
│   └── variables.tf
├── s3
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── sg
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── terraform.tfstate
├── terraform.tfstate.backup
├── terraform.tfvars
├── variables.tf
└── vpc
    ├── main.tf
    ├── nat.tf
    ├── outputs.tf
    ├── routing.tf
    ├── subnets.tf
    └── variables.tf

4 directories, 19 files
```

And this is our infrastructure diagram:
### TBC: Infrastructure Diagram



## Launch a public EC2 instance to host the TeamCity
Now we're ready to build our EC2 instance. We are going to launch this in the public subnet. Next we're going to use TeamCity Docker image and run it in our instance and configure it to use the RDS we just build. 

I am going to use a debian trusted image that I have used before, you may choose a debian, ubuntu, etc. Just keep in mind that they might have different username to use for ssh. For example, debian uses admin.

But before we create the ec2 module, AWS requires a Key Pair to create an instance. We can do this in two ways:

1. If you want to use your existing key, then you can just add it to AWS > EC2 > Key Pairs > Import Key Pair.
2. Navigate to AWS > EC2 > Key Pairs, then select "Create Key Pair". Give it a name (teamcity) and create. This is going to save a .pem key in your default download location. Move the key to a secure location (.ssh for example).


```bash
$ mkdir ec2 && touch ec2/main.tf && touch ec2/variables.tf
```

_ec2/main.tf_
```
resource "aws_instance" "teamcity" {
  ami                         = "${var.ami}"
  instance_type               = "m3.medium"
  key_name                    = "${var.key_name}"
  subnet_id                   = "${var.public_subnet_id}"
  user_data                   = "${data.template_file.teamcity_userdata.rendered}"
  vpc_security_group_ids      = ["${var.vpc_security_group_ids}"]
  associate_public_ip_address = true

  tags = {
    Name = "TeamCity"
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

I want to use the user_data to configure the instance immediately using a template, using resource. Do add this to _ec2/main.tf_


```
.
.

data "template_file" "teamcity_userdata" {
  template = "${file("${path.module}/scripts/setup.sh")}"

  vars {
    db_url      = "${var.db_url}"
    db_port     = "${var.db_port}"
    db_name     = "${var.db_name}"
    db_username = "${var.db_username}"
    db_password = "${var.db_password}"
  }
}
```

_db_setup.sh_ will contain all the steps that I want to run in order to configure my instance. You can also ssh into the machine and run these manually. More oever, I am installing "tree" and "touch" because I use them often, you can customize it however you want.

```bash
$ mkdir ec2/scripts && touch ec2/scripts/setup.sh
```

_ec2/scripts/setup.sh_
```sh
#!/bin/bash
sudo apt-get update
sudo apt-get install tree
sudo apt-get install touch
sudo mkdir -p /opt/teamcity
sudo mkdir -p /opt/teamcity/var/logs
sudo mkdir -p /opt/teamcity/lib/jdbc
sudo mkdir -p /opt/teamcity/config
sudo apt-get install -y apt-transport-https dirmngr
echo 'deb https://apt.dockerproject.org/repo debian-stretch main' | sudo tee --append /etc/apt/sources.list
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys F76221572C52609D
sudo apt-get update
sudo apt-get install -y docker-engine --allow-unauthenticated
sudo docker system prune -af
sudo docker rm $(docker ps -aq)
sudo docker pull jetbrains/teamcity-server
DEBIAN_FRONTEND=noninteractive sudo apt-get update -y
DEBIAN_FRONTEND=noninteractive sudo apt-get install -y wget
wget https://jdbc.postgresql.org/download/postgresql-9.4.1209.jre6.jar 
sudo mv postgresql-9.4.1209.jre6.jar /opt/teamcity/lib/jdbc/
sudo tee /opt/teamcity/config/database.properties <<EOF
connectionUrl=jdbc:postgresql://${db_url}:${db_port}/${db_name}
connectionProperties.user=${db_username}
connectionProperties.password=${db_password}
EOF
sudo sudo apt-get install -y postgresql
sudo docker run --rm -d --name teamcity-server-instance -v /opt/teamcity:/data/teamcity_server/datadir -v /opt/teamcity/var/logs:/opt/teamcity/logs -p 8111:8111 jetbrains/teamcity-server
```

Let's go ahead and create our variables.

_ec2/variables.tf_
```
variable "ami" {
  description = "trusted debian ami"
  default     = "default value"
}

variable "db_url" {
  default = "default value"
}

variable "db_username" {
  default = "default value"
}

variable "db_password" {
  default = "default value"
}

variable "db_port" {
  default = "default value"
}

variable "db_name" {
  default = "default value"
}

variable "key_name" {
  default = "default value"
}

variable "public_subnet_id" {
  default = "default value"
}

variable "vpc_security_group_ids" {
  default = "default value"
}
```

It's time to call out ec2 module.

_main.tf_
```
.
.

module "ec2" {
  source                 = "ec2"
  ami                    = "${var.debian_ami}"
  db_username            = "${var.db_username}"
  db_password            = "${var.db_password}"
  db_name                = "${var.db_name}"
  db_port                = "${module.rds.db_port}"
  db_url                 = "${module.rds.database_address}"
  key_name               = "${var.key_name}"
  public_subnet_id       = "${module.vpc.public_subnet}"
  vpc_security_group_ids = "${module.sg.web_security_groups_id}"
}
```

I kept my debian_ami in the _variables.tf_ because I am going to use that image in couple other places later, you can hard-code it if you want to.

**Notice:** key_name should be the name of the key you added in AWS

_variables.tf_
```
.
.

variable "key_name" {
  default = "teamcity"
}

variable "debian_ami" {
  default = "ami-0bd9223868b4778d7"
}
```

Let's add the rest of the variables and outputs we need.

```bash
$ touch rds/outputs.tf
```

_rds/outputs.tf_
```
output "database_address" {
  value = "${replace(aws_db_instance.database.endpoint, ":5432", "")}"
}

output "db_port" {
  value = "5432"
}
```

_vpc/outputs.tf_
```
.
.

output "public_subnet" {
  value = "${aws_subnet.public.id}"
}
```

_sg/outouts.tf_
```
output "web_security_groups_id" {
  value = "${aws_security_group.teamcity_web_sg.id}"
}
```

Last but not the least, let's output the ssh command so we can easily access our instance.

```bash
$ touch outputs.tf
```

_outputs.tf_
```
output "teamcity_web_ssh_command" {
  value = "${format("ssh -i ~/.ssh/teamcity.pem admin@ec2-%s.compute-1.amazonaws.com", "${replace("${module.ec2.teamcity_web_ip}", ".", "-")}")}"
}
```

Now we need to add an output for ec2 to give us the teamcity_web_ip.

```bash
$ touch ec2/outputs.tf
```

_ec2/outputs.tf_
```
output "teamcity_web_ip" {
  value = "${aws_instance.teamcity.public_ip}"
}
```

It's the moment of truth!!!

```bash
$ terraform init
$ terraform plan
$ terraform apply
```

You should see a similar output to the following:
```bash
Apply complete! Resources: 1 added, 1 changed, 0 destroyed.

Outputs:

teamcity_web_ssh_command = ssh -i ~/.ssh/teamcity.pem admin@ex.y.z.d.compute-1.amazonaws.com
```

# BEAUTIFUL!

Navigate to AWS > EC2. You know have 1 running instance. Wait for the initialization to be done.
When the status check is done and server is up you can access your TeamCity via browser: ex.y.z.d.compute-1.amazonaws.com:8111


Congratulations!

I hope this was helpful. Please let me know your thoughts. 
You can find the code on github: https://github.com/saslani/terraform_teamcity_aws
This is a very simple setup. If you would like to add features, please make a branch. We will keep all new features in the branch for those you need it.

At this point, this is how my folder structure looks like:

```bash
.
├── ec2
│   ├── main.tf
│   ├── outputs.tf
│   ├── scripts
│   │   └── setup.sh
│   └── variables.tf
├── main.tf
├── outputs.tf
├── rds
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── s3
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── sg
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── terraform.tfstate
├── terraform.tfstate.backup
├── terraform.tfvars
├── variables.tf
└── vpc
    ├── main.tf
    ├── nat.tf
    ├── outputs.tf
    ├── routing.tf
    ├── subnets.tf
    └── variables.tf

6 directories, 25 files
```

And this is our final infrastructure diagram

### TBC: Infrastructure Diagram

