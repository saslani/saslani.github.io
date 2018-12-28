---
layout: post
title:  "Host Teamcity on AWS, using Terraform and Ansible"
date:   2018-12-26 16:15:15
categories: Infrastructure as Code
---

I chose terraform as a provider because:
- I like programing rather than click though UI!
- I want to minimize my work if I have to change the host (i.e. move from AWS to Azure, etc)
- I see the graph dependencies
- I want to encourage my team members who are not keen on "DevOpsy" work, to see how easy it is to use Infrastructure as Code


In this tutorial, we'll build a very basic infrastructure to host TeamCity on AWS, using terraform. Some familiarity with AWS, TeamCity and programing knowledge is required, this is not an AWS tutorial.

#### Prerequisites:
- Good understanding of AWS: We won't be focusing on AWS UI and how to set things up via click-through. Although, I will show you how to navigate AWS to visually see your changes.
- Fair understanding of TeamCity: We will not be going through TeamCity training. We will just confirm that it works properly
- Familiarity with a programing language
- We won't go deep into learning terraform, you can refer to the HashiCorp documents if you want to learn more

#### Getting started:
- [TeamCity](https://www.jetbrains.com/teamcity/download/#section=aws) 2017.2+ has a free account with 3 build agents for 100 build configurations. So you can start there and purchase the license if you need to.
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
- Small AWS charge. To keep the charge minimum run **terraform destroy at the end of the tutorial, if you wish.
- I am using main.tf, variables.tf, and outputs.tf throught this project, you may wish to change the names, or just develop in one single file. All tf files in a directory will be compiled together. So separating them, is my personal preference.
- I am using MAC OS Majave, so change the bash commands appropriately, if you are using any other OS


#### What are we building?

- Launch a single public instance
  + Create a VPC
  + Single public subnet
  + Routing
  + Create a security group
  + Launch a public EC2 instance to host the TeamCity app
- Launch a private instance to host our DB
  + Add new security group for the server instance
  + Add NAT
  + Add a private subnet
  + Modify routing
  + Launch a private EC2 instance to host the database
- Add Database and backup
  - Create database (RDS)
  - Create S3 bucket for database backup
- Provisioning
  + Use ansible to provision the public EC2
- Tie it all together
  + Create an agent and run a build!


## Create a VPC with a Single Subnet and EC2 instance to host the TeamCity web

Once you installed terraform, create an empty folder where you want to write your code, and cd into it:

```bash
$ mkdir teamcity
$ cd teamcity
```

If you have created your amazon account within the past couple of years, it's likely that it comes with a default VPC. We are going to create an isolated Virtual Private Cloud and subnets and use them to launch EC2 instances.

In the root directory, create the main.tf. **tf** is Terraform convention. We will use this main.tf as a driver that contains other modules to build pieces we need.

```bash
$ touch main.tf
```

Now let's write some terraform code. The biggest advantage of infrastructure as code, as we discussed earlier is that it's scalable and easy to change provider. In this tutorial we're going to use AWS, thus, our provider is aws. 

My account is in **us-east-1**. Check yours to pick the right region. 

_main.tf_
```terraform
provider "aws" {
  region = "us-east-1"
}
```

Next, we want to add [**aws_vpc**](https://www.terraform.io/docs/providers/aws/r/vpc.html) resource to create our VPC. We call it **vpc**. This is like naming a variable so you can access it later throughout your project.

For [cidr_block](https://whatismyipaddress.com/cidr), we're using the **10.0.0.0** as opposed to **192.168* because it's more common, 192.168 is mainly associated with your personal IP. 

enable_dns_hostnames by default is false. We want to enable DNS hostnames in the VPC, so set it to true.

Add tags, Name, to see the name in the VPC list, specially if you have more than 1 VPC, so that you can easily distinguish them.

_main.tf_
```terraform
recource "aws_vpc" "vpc" {
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

**Export into the shell**
```bash
$ export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_KEY"
$ export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY"
```

and if you want to use a **.tfvars**, add the terraform file to your root directory:

```bash
$ touch terraform.tfvars
```

_terraform.tfvars_
```terraform
aws_access_key = "YOUR_ACCESS_KEY"
aws_secret_key = "YOUR_SECRET_KEY"
```

Run ```$ terraform refresh``` and you see the following error:

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
```terraform
variable "aws_access_key" {
  description = "AWS access key"
}

variable "aws_secret_key" {
  description = "AWS secret key"
}
```

Now run ```$ terraform refresh``` all should be good! Next, use ```$ terraform plan``` to see the plan that's going to be executed and verify that's what your want to do. In our case, we want to **create** a **vpc**:

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

Looks great, run ```$ terrafrom apply``` to execute the plan. You will be prompted to verify if that's what you want to do, enter **yes**.

Great!!!! Now if you navigate to VPC on your AWS account (AWS > Services > VPC). You see that the VPC is created! This is step 1 in your architecture diagram:

![](/assets/images/infrastructure_as_code/terraform_teamcity_aws/create_vpc.png "Teamcity VPC")


Now that we have the VPC created, let's add a single public subnet. But before we do so, let's a small refactoring. Some of you may think, it's not need yet, you may go ahead and use a single file. Personally, I like to add a little structure to my projects, knowing this is going to grow.

In terraform, modules, are similar to classes in OO programing. I want to move the job of creating [VPC](https://www.terraform.io/docs/providers/aws/d/vpc.html), [subnets](https://www.terraform.io/docs/providers/aws/d/subnet.html), and [routing](https://www.terraform.io/docs/providers/aws/d/route_table.html) to it's own module. 

```bash
$ mkdir vpc
$ cd vpc
$ touch subnets.tf
```

_vpc > subnets.tf_
```terraform
resource "aws_vpc" "vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags {
    Name = "Teamcity VPC"
  }
}
```

In the root directory change the _main.tf_ to look like:
```terraform
provider "aws" {
  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
  region     = "${var.region}"
}

module "vpc" {
  source = "vpc"
}
```

Notice that I have but the region in a variable so I can use it later. If you wish to do so, add the region to the _variable.tf_ in the root directory, where you added aws secret key and access key:
```bash
.
.

variable "region" {
  default = "us-east-1" #replace with your region
}
```

Now your teamcity folder structure should look something like this:

```bash
├── main.tf
├── terraform.tfstate
├── terraform.tfstate.backup
├── terraform.tfvars
├── variables.tf
└── vpc
    ├── subnets.tf
```

Before we go father, let's go ahead and add this to git. Add the following **.gitignore** to the root directory:

**Important: make sure you add the tfvars that's holding your AWS credentials to the gitignore file**

```bash
$ touch .gitignore
```

```
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


```bash
$ git init
$ git add
$ git commit "initial checking: Create VPC in AWS"
```

Now we're ready to add a single subnet. Navigate to _vpc > subnets.tf_ inside the vpc folder. We want to add an [aws_subnet](https://www.terraform.io/docs/providers/aws/d/subnet.html) resource and name it public. I am passing a variable _availability_zone_ since I don't have a preference. If you do, you can hardcode this to be "us-east-1a" for example. 

```terraform
.
.

resource "aws_subnet" "public" {
  availability_zone = "${element(var.availability_zones, count.index)}"
  cidr_block        = "${var.public_cidr_block}"
  vpc_id            = "${aws_vpc.vpc.id}"

  tags {
    Name = "Public Teamcity Subnet"
  }
}
```

_vpc > variables.tf_
```terraform
.
.

variable "public_cidr_block" {
  default = "10.0.0.0/24"
}
```

Change the _main.tf_
```terraform
.
.

data "aws_availability_zones" "zones" {}

module "vpc" {
  source             = "vpc"
  availability_zones = ["${data.aws_availability_zones.zones.names}"]
}
```

Now we need to add _vpc > variables.tf_ for the vpc module to declare what parameters we are going to send. If you hard coded the availability zone(s), skip this section.
```bash
$ touch vpc/variables.tf
```

```terraform
variable "availability_zones" {
  description = "List of availability zones over which to distribute subnets"
  type        = "list"
}
```

Looks good... 

 ```bash 
 #Since we just added a module, we need to run init first
 $ terraform init 
 .
 .
 # verify the changes: + create and + module.vpc.aws_subnet.public
 $ terraform plan
 .
 .
 # yes to the changes
 $ terraform apply
 ```

 Log into AWS again or refresh your VPC page. From the left sub-menu navigate to subnets. You will see a new subnet created: "Public Teamcity Subnet". Select the newly created subnet to see the details. i.e. your availability zone and IP.
 CLick on the Route Table tab, you will see one row: 

 Destination: 10.0.0.0/16  \|  Target: local

 Let's add a router to our VPC. Create a routing.tf in the vpc folder:

 ```bash
 $ touch vpc/routing.tf
 ```

 In this file, we need to create an aws_internet_gateway (we name it vpc_igw). Then create an [aws_route_table](https://www.terraform.io/docs/providers/aws/d/route_table.html) (vpc_public). Last but not the least, an association resource, [aws_route_table_association](https://www.terraform.io/docs/providers/aws/r/route_table_association.html) (vpc_public).

_vpc > routing.tf_
```terraform
resource "aws_internet_gateway" "vpc_igw" {
  vpc_id = "${aws_vpc.vpc.id}"

  tags {
    Name = "Teamcity Gateway"
  }
}

resource "aws_route_table" "vpc_public" {
  vpc_id = "${aws_vpc.vpc.id}"

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.vpc_igw.id}"
  }

  tags {
    Name = "Teamcity Public Subnet Route Table"
  }
}

resource "aws_route_table_association" "vpc_public" {
  subnet_id      = "${aws_subnet.public.id}"
  route_table_id = "${aws_route_table.vpc_public.id}"
}
```

Let's apply our changes:

```bash
$ terraform apply
```

From the AWS console, navigate to VPC > Subnets, or just refresh the page. Select the subnet you had created earlier and click on Route Table. Notice the new 0.0.0.0/0 igw-xyz has been created. 0.0.0.0 is to accept request form any IP.

Now our diagram looks more like:

![](/assets/images/infrastructure_as_code/terraform_teamcity_aws/vpc_subnet_routing_table.png "Teamcity VPC, Subnet, Custom Route Table")

So far, so good. Next we need to add a security group, to increase security of our VPC. In the root directory, create a folder sg and inside it main.tf and variable.tf:

```bash
$ mkdir sg
$ touch sg/main.tf
$ touch sg/variable.tf
```

Inside the sg > main.tf, we are going to create an aws_security_group (teamcity_web_sg) and add ingress (right to enter a property) and egress (right to exit a property) rules:

_vpc > main.tf_
```terraform
resource "aws_security_group" "teamcity_web_sg" {
  name        = "Teamcity_sg"
  description = "Allow Teamcity SSH and HTTP inbound connection"
  vpc_id      = "${var.vpc_id}"

  # Allow SSH
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Allow HTTP for teamcity on port 8111
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
    Name = "Teamcity Web Security Group"
  }
}
```

_vpc > variables.tf_
```terraform
variable "vpc_id" {
  type        = "string"
  description = "VPC ID in which to deploy RDS"
}
```

Call sg module from the _main.tf_ in the root directory to call the newly create module:

```terraform
.
.

module "sg" {
  source = "sg"
  vpc_id = "${module.vpc.vpc_id}"
}
```

Note: _source_ refers to the location of the module, you may wish a git repo (for example if a different team owns that module) instead of creating your own.

Since we created a new module we need to initialize again:

```bash
$ terraform init
```

You will see an error where vpc_id is unknown.:

```bash
[13:07:03][teamcity_setup]$ terraform init
Initializing modules...
- module.vpc
- module.sg

Error: module 'sg': "vpc_id" is not a valid output for module "vpc"
```

To fix this error, add _vpc > outputs.tf_ for the vpc module, to output the vpc id that we created:

```bash
$ touch vpc/outputs.tf
```

_vpc > outputs.tf_
```terraform
output "vpc_id" {
  value = "${aws_vpc.vpc.id}"
}
```

Now run the init again and execute the plan (verify the plan, if you wish, before executing it):

```terraform
$ terraform init
$ terraform apply
```

Form your AWS console, navigate to VPC > Internet Gateways. You will see the new gateway that you have created (Teamcity Gateway). Click on the gateway, notice the Attached VPC ID is the one VPC id that you created earlier in the tutorial. To see the security group, click on the Security Group from the left nav.

Now that we have the VPC and security group created. Let's go ahead and build our first EC2 instance, that's a public instance to, later, host the teamcity web.

Let's create en **ec2** module, with two files: _main.tf_ and _variables.tf_
```bash
$ mkdir ec2
$ touch ec2/main.tf
$ touch ec2/variables.tf
```

Inside the main.tf, we need to add an [aws_instance](https://www.terraform.io/docs/providers/aws/d/instance.html) (teamcity) resource. For the ami, I am using a verified debian image that I trust, you may wish to use ubuntu. We need to pass in the security group and the public subnet id that we created. I am also adding a lifecycle to create an instance before destroying it, this is to make sure I have an instance running. For the purpose of this simple tutorial, we will not be using ELBs, you may wish to do so if you want a highly available instance.

_ec2 > main.tf_
```terraform
resource "aws_instance" "teamcity" {
  ami                         = "${var.ami}"
  instance_type               = "m3.medium"
  vpc_security_group_ids      = ["${var.web_security_groups_id}"]
  subnet_id                   = "${var.subnet_id}"
  associate_public_ip_address = true

  tags = {
    Name = "Teamcity"
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

_ec2 > variable.tf_
```terraform
variable "ami" {
  description = "trusted debian ami"
  default     = "default value"
}

variable "subnet_id" {
  default = "default value"
}

variable "web_security_groups_id" {
  default = "default value"
}
```

From the root _main.tf_ call the ec2 module:
```terraform
.
.

module "ec2" {
  source = "ec2"
  ami    = "${var.debian_ami}"
  web_security_groups_id = "${module.sg.web_security_groups_id}"
  subnet_id          = "${module.vpc.public_subnet}"
}
````

Notice that we need the security group id that we created in the sg module and the public subnet id that we created in the vpc module. So let's go ahead and output those so that we can access them when calling the ec2 module.

_vpc > outputs.tf_
```terraform
.
.

output "public_subnet" {
  value = "${aws_subnet.public.id}"
}
```

```bash
$ touch sg/outputs.tf
```

_sg > outputs.tf_
```terraform
output "web_security_groups_id" {
  value = "${aws_security_group.teamcity_web_sg.id}"
}
```

I am passing the debiam_ami as a variable, so I need to add that to the _variables.tf_ in the root directory. Skip the following if you are hard-coding the ami image name.

_variables.tf_
```
.
.
variable "debian_ami" {
  default = "ami-0bd9223868b4778d7"
}
```

Now let's apply our changes:
```bash
$ terraform init
$ terraform apply
```

Congrats! You just used terraform to create your first instance! Now let's ssh into it. I have created a key_pair to use when I ssh into the instance. You may wish to use your id_rsa or create a new one (teamcity) and add it to ssh-agent. You can create your key or generate one from AWS. Here are both ways:

1. Manually create one yourself:
```bash 
$ ssh-keygen -t rsa -C "teamcity" -P '' -f ~/.ssh/teamcity; chmod 400 ~/.ssh/teamcity.

#start the agent in the background
$ eval "$(ssh-agent -s)"
Agent pid 59566

#Add teamcity private key to the ssh-agent and passphrase in the keychain
$ ssh-add -K ~/.ssh/teamcity
```

Use pbcopy to copy your "PUBLIC" key and add it to aws. You can also copy the public key to your desktop and use the import feature from AWS > EC2 > key Pairs

```bash
$ pbcopy < ~/.ssh/teamcity.pub
```
Navigate to AWS > EC2 > Key Pair. Select "Import Key Pair" and import or paste your key.

Now let's attach that key pair to our instance.


2. Go to AWS console > EC2 > Key Pairs
  2.1. Click on create and give it a name (teamcity), this is going to download the .pem (in your Download folder, perhaps)
  2.2. Copy the .pem in your .ssh, if you wish.
  3.3. chmod 400 ~/.ssh/teamcity.pem



Continue....




Pass in the key name to the ec2 module. You can hard-code the key name or pass it as a variable:

_main.tf_
```terraform
module "ec2" {
  .
  .
  key_name = "${var.key_name}"
}
```

_variable.tf_
```terraform
.
.

variable "key_name" {
  default = "teamcity"
}
```

_ec2 > main.tf_
```terraform
resource "aws_instance" "teamcity" {
  .
  .

  key_name = "${var.key_name}"
  .
  .
}
```

_ec2 > variables.tf_
```terraform
.
.

variable "key_name" {
  default = "default value"
}
```

Apply the changes:

```bash
$ terraform apply

# Notice the following:
# -/+ destroy and then create replacement
# -/+ module.ec2.aws_instance.teamcity (new resource required)
```

Note: If you look at the AWS console > EC2 > Instances, you will notice that the new instance is being initialized and the old one is terminated. Don't worry about removing the terminated instance as AWS will take care of, automatically, it later.

Wait till the instance is up and healthy before ssh into it. 

Next, in the ec2 module, I have created an output variable called "teamcity_public_ip" so I can easily see the public IP of the instance I want to ssh into.

```bash 
$ touch ec2/outputs.tf
```

_ec2 < outputs.tf_
```terraform
output "teamcity_public_ip" {
  value = "${aws_instance.teamcity.public_ip}"
}
```

 You can find your IP from AWS console, however, the point is to not having constantly use the console: Navigate to AWS > console > EC2 > Instances.
- Choose the instance (Teamcity) and copy the "Public DNS".
- You may also right click on the instance and select "connect". In the modal, you will find the example of how to can ssh into the container. 

**Notes**
- example uses .pem, you don't have to.
- I used debian, so I can ssh into the instance using admin@

Copy the Public DNS and try ssh'ing onto your instance:
```bash
ssh -i ~/.ssh/teamcity admin@ec2-18-208-217-36.compute-1.amazonaws.com
```

While we're at it, I added an outputs.tf to the root directory to show the teamcity_web_ssh_command, because I am lazy.

_outputs.tf_
```terraform
output "teamcity_web_ssh_command" {
  value = "${format("ssh -i ~/.ssh/teamcity admin@ec2-%s.compute-1.amazonaws.com", "${replace("${module.ec2.teamcity_public_ip}", ".", "-")}")}"
}
```

Use terraform refresh to see the teamcity_web_ssh_command output you just created:
```bash
$ terraform refresh

# teamcity_web_ssh_command = ssh -i ~/.ssh/teamcity admin@ec2-x-y-z-d.compute-1.amazonaws.com
```

Since we used Debian, we need to install apache2 in order to access our server via browser. But let's skip this since we're going to host TeamCity.

Run the following to see the teamcity_web_ssh_command, then ssh into the machine:
```bash
$ terraform output teamcity_web_ssh_command
$ ssh -i ~/.ssh/teamcity admin@ec2-x-y-z-d.compute-1.amazonaws.com
```


## Add a Database (RDS)
RDS needs multiple zones and at least 2 private subnets. Before we add a private subnet, however, we need to create a NAT. NAT needs an Elastic IP to work. So first, let's go ahead and make those changes. We will be using [aws_eip](https://www.terraform.io/docs/providers/aws/r/eip.html), [aws_nat_gateway](https://www.terraform.io/docs/providers/aws/d/nat_gateway.html), ....

```bash
$ touch vpc > nat.tf
```

_vpc > nat.tf_
```terraform
resource "aws_eip" "nat_gw_eip" {
  vpc = true

  tags {
    Name = "Teamcity NAT"
  }
}

resource "aws_nat_gateway" "gw" {
  allocation_id = "${aws_eip.nat_gw_eip.id}"
  subnet_id     = "${aws_subnet.public.id}"

  tags {
    Name = "Teamcity NAT Gateway"
  }
}
```

Now we're reading to create a route table and association for the NAT.

_vpc > routing.tf_
```terraform
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

Now we're ready to create our private subnet
_vpc > subnets.tf_
```terraform
resource "aws_subnet" "private" {
  availability_zone = "${element(var.availability_zones, count.index)}"
  cidr_block        = "${element(var.private_cidr_block, count.index)}"
  count             = "${length(var.private_cidr_block)}"
  vpc_id            = "${aws_vpc.vpc.id}"

  tags {
    Name = "${format("TeamCity Private Subnet %d", count.index + 1)}"
  }
}
```

_vpc > variables.tf_
```terraform
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

_vpc > outputs.tf_
```terraform
.
.

output "private_subnet" {
  value = ["${aws_subnet.private.*.id}"]
}
```

_main.tf_
```terraform
.
.

module "vpc" {
  .
  .
  length = 2
}
```

Note: If you get an error for a duplicate IP, run the **terraform destroy** and the apply the changes again.
Apply the changes
```bash
$ terraform apply
```

Now that we have the private subnets, let's create our RDS. Let's start with adding a security group for RDS.

_sg > main.tf_
```terraform
.
.

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

_sg > variables.tf_
```terraform
output "rds_security_groups_id" {
  value = "${aws_security_group.rds_sg.id}"
}
```

```bash
$ mkdir rds
$ touch rds/main.tf
$ touch rds/variables.tf
$ touch rds/outputs.tf
```

We will be using [aws_db_instance](https://www.terraform.io/docs/providers/aws/d/db_instance.html) resource.
_rds > main.tf_
```terraform
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

_rds > variables.tf_
```terraform
variable "vpc_id" {
  type        = "string"
  description = "VPC ID in which to deploy RDS"
}
```

_rds > outputs.tf_
```terraform
output "web_security_groups_id" {
  value = "${aws_security_group.teamcity_web_sg.id}"
}

output "rds_security_groups_id" {
  value = "${aws_security_group.rds_sg.id}"
}
```

And now let's call the module
_main.tf_
```terraform
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

_variables.tf_
```terraform
variable "db_name" {
  default = "teamcity-rds"
}

variable "db_password" {
  type = "string"
}

variable "db_username" {
  default = "teamcityuser"
}
```

_terraform.tfvars_
```terraform
db_password = "terraform123"
```


This would take a while, so go ahead for coffee and stretching!
```bash
$ terraform init
$ terraform plan
$ terraform apply
```

To see your RDS, choose RDS from AWS console :)

While we're at it, let's create an S3 bucket for the backup
This is going to be very short and sweet!

```bash
$ mkdir s3
$ touch s3/main.tf
$ touch s3/variables.tf
$ touch s3/outputs.tf
```

_s3 > main.tf_
```terraform
resource "aws_s3_bucket" "backup_bucket" {
  bucket = "${var.name}"
  acl    = "private"

  tags {
    Name = "${var.description}"
  }
}
```

_s3 > variables.tf_
```terraform
variable "name" {
  description = "Name of the S3 Bucket"
}

variable "description" {
  description = "Description of the S3 Bucket (for tagging)"
}
```

_s3 > outputs.tf_
```terraform
output "arn" {
  value = "${aws_s3_bucket.backup_bucket.arn}"
}
```

_main.tf_
```terraform
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
So far we have created a VPC, a public subnet and an EC2 instance to host TeamCity. We build 2 private subnets and created an RDS in the subnet. In addition, an S3 bucket for the backup. At this point, my folder structure looks like this:

```bash
.
├── ec2
│   ├── main.tf
│   ├── outputs.tf
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
    ├── nat.tf
    ├── outputs.tf
    ├── routing.tf
    ├── subnets.tf
    └── variables.tf
```

**Commit the changes, if you haven't done yet.**

So far, this is now our infrastructure document looks like
### TBC: Architecture Diagram


## Provisioning

Now let's write some ansible to do some heavy lifting and configure teamcity for us on the instance we just built


```bash
```


............MANUAL STEPS
We can install Teamcity manually or write an ansible for it. If you prefer to set up manually, ssh into the machine and follow the steps bellow. Otherwise, skip this section and continue on **Add a Private Subnet and EC2 Instance to contain the Database for Teamcity** to build a private subnet, add an EC2 instance, create RDS and then use ansible to provision teamcity:

```bash
# Install docker:
$ sudo apt-get install apt-transport-https dirmngr
# answer Y

#Add Docker package depository to your /etc/apt/sources.list sources list:
$ echo 'deb https://apt.dockerproject.org/repo debian-stretch main' | sudo tee --append /etc/apt/sources.list

# Obtain docker's repository signature and updated package index:
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys F76221572C52609D
$ sudo apt-get update

#Install docker
$ sudo apt-get install docker-engine
# answer Y

### Install teamcity
$ sudo docker system prune -a
# answer y

$ sudo docker pull jetbrains/teamcity-server

$ sudo mkdir /home/teamcity
$ sudo mkdir /var/log/teamcity

$ sudo docker run -it --name teamcity-server-instance \
-v /home/teamcity:/data/teamcity_server/datadir \
-v /var/log/teamcity:/opt/teamcity/logs \
-p 8111:8111 \
jetbrains/teamcity-server
```

## Tie it all together
Very teamcity is up: `http://ec2-x.y.z.d.compute-1.amazonaws.com:8111`
Follow the instruction to install teamcity, using the buildin HSQL (this could take a while)
You can use the Teamcity [docs](https://confluence.jetbrains.com/display/TCD18/Configure+and+Run+Your+First+Build) to configure an agent







--------------
Please let me know what you think about this tutorial, if I have missed any steps :)



## References
- https://confluence.jetbrains.com/display/TCD18/Running+TeamCity+Stack+in+AWS
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair
- https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html
- https://dev-ops-notes.com


