---
layout: post
title:  "Host Teamcity on AWS, using Terraform and Ansible"
date:   2018-12-26 16:15:15
categories: Infrastructure as Code
---

why terraform?
why teamcity?

Prerequisites:
- Good understanding of AWS: We won't be focusing on AWS UI and how to set things up via click-through
- Fair understanding of teamcity: We will not be going through teamcity training. We will just confirm that it works properly
- Familiarity with a programing language
- We won't go deep into learning terraform, you can refer to the HashiCorp documents if you want to learn more

Getting started:
- download Teamcity. Teamcity 2017.2+ has a free account with 3 build agents for 100 build configurations. So you can start there and purchase the license if you need to.
- install terraform
- get an aws account (if you just want to use Azure, then read the next article)

https://www.jetbrains.com/teamcity/download/#section=aws
https://confluence.jetbrains.com/display/TCD18/Running+TeamCity+Stack+in+AWS
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair
https://github.com/JetBrains/teamcity-s3-artifact-storage-plugin/issues/6
https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filter=active
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html
https://dev-ops-notes.com

create a non-admin user with proper accesses to perform the following. Avoid using your admin user to make changes in the aws.

Small AWS charge. To keep the charge minimum run ```terraform destroy``` at the end of the tutorial, if you wish.
I am using main.tf, variables.tf, and outputs.tf throught this project, you may wish to change the names, or just develop in one single file. All tf files in a directory will be compiled together. So separating them, is my personal preference.

I am using MAC OS Majave, so change the bash commands appropriately, if you are using any other OS

---------------------------------------------------------------------
In this tutorial, we'll build the most basic infrastructure to host Teamcity. That would be create a Virtual Private Cloud, with 2 subnets (1 public for the app and 1 private for the database). A Gateway and Route Table to communicate with the Internet. ----NAT----. We'll build and configure a database in the private subnet. and last but not the least use Ansible to provision our instances and tie all together.


- Create a VPC
  - single subnet
  - routing
- Create a security group
- Launch a public EC2 instance to host the Teamcity Web App
- Launch a private EC2 instance to host the database
- Create database (RDS)
- Use ansible to provision the public EC2
- tie it all together


## Create a VPC with a Single Subnet and EC2 instance to host the Teamcity web

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

**main.tf**
```terraform
provider "aws" {
  region = "us-east-1"
}
```

Next, we want to add [**aws_vpc**](https://www.terraform.io/docs/providers/aws/r/vpc.html) resource to create our VPC. We call it **vpc**. This is like naming a variable so you can access it later throughout your project.

For [cidr_block](https://whatismyipaddress.com/cidr), we're using the **10.0.0.0** as opposed to **192.168* because it's more common, 192.168 is mainly associated with your personal IP. 

enable_dns_hostnames by default is false. We want to enable DNS hostnames in the VPC, so set it to true.

Add tags, Name, to see the name in the VPC list, specially if you have more than 1 VPC, so that you can easily distinguish them.

**main.tf**
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

**terraform.tfvars**
```terraform
aws_access_key = "YOUR_ACCESS_KEY"
aws_secret_key = "YOUR_SECRET_KEY"
```

Run ```$ terraform refresh``` and you see the following error:

```bash
Error: provider config 'aws': unknown variable referenced: 'aws_access_key'; define it with a 'variable' block
Error: provider config 'aws': unknown variable referenced: 'aws_secret_key'; define it with a 'variable' block
```

In the root directory create a **variables.tf** file. For the rest of this tutorial, we'll use this file to declare what variables we need for the modules we are going to build.

```bash
$ touch variables.tf
```

Terraform variables are declared in a variable block. You can declare a type, by default it's string, description and a default value

**variables.tf**
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

**vpc > subnets.tf**
```terraform
resource "aws_vpc" "vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags {
    Name = "Teamcity VPC"
  }
}
```

In the root directory change the **main.tf** to look like:
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

Notice that I have but the region in a variable so I can use it later. If you wish to do so, add the region to the **variable.tf** in the root directory, where you added aws secret key and access key:
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

Now we're ready to add a single subnet. Navigate to **vpc > subnets.tf** inside the vpc folder. We want to add an [aws_subnet](https://www.terraform.io/docs/providers/aws/d/subnet.html) resource and name it public. I am passing a variable __availability_zone__ since I don't have a preference. If you do, you can hardcode this to be "us-east-1a" for example. 

```terraform
.
.

resource "aws_subnet" "public" {
  availability_zone = "${element(var.availability_zones, count.index)}"
  cidr_block        = "10.0.0.0/24"
  vpc_id            = "${aws_vpc.vpc.id}"

  tags {
    Name = "Public Teamcity Subnet"
  }
}
```

Change the **main.tf**
```terraform
.
.

data "aws_availability_zones" "zones" {}

module "vpc" {
  source             = "vpc"
  availability_zones = ["${data.aws_availability_zones.zones.names}"]
}
```

Now we need to add **vpc > variables.tf** for the vpc module to declare what parameters we are going to send. If you hard coded the availability zone(s), skip this section.
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

**vpc > routing.tf**
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

**vpc > main.tf**
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

**vpc > variables.tf**
```terraform
variable "vpc_id" {
  type        = "string"
  description = "VPC ID in which to deploy RDS"
}
```

Call sg module from the **main.tf** in the root directory to call the newly create module:

```terraform
.
.

module "sg" {
  source = "sg"
  vpc_id = "${module.vpc.vpc_id}"
}
```

Note: __source__ refers to the location of the module, you may wish a git repo (for example if a different team owns that module) instead of creating your own.

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

To fix this error, add **vpc > outputs.tf** for the vpc module, to output the vpc id that we created:

```bash
$ touch vpc/outputs.tf
```

**vpc > outputs.tf**
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

Let's create en **ec2** module, with two files: **main.tf** and **variables.tf**
```bash
$ mkdir ec2
$ touch ec2/main.tf
$ touch ec2/variables.tf
```

Inside the main.tf, we need to add an [aws_instance](https://www.terraform.io/docs/providers/aws/d/instance.html) (teamcity) resource. For the ami, I am using a verified debian image that I trust, you may wish to use ubuntu. We need to pass in the security group and the public subnet id that we created. I am also adding a lifecycle to create an instance before destroying it, this is to make sure I have an instance running. For the purpose of this simple tutorial, we will not be using ELBs, you may wish to do so if you want a highly available instance.

**ec2 > main.tf**
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

**ec2 > variable.tf**
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

From the root **main.tf** call the ec2 module:
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

**vpc > outputs.tf**
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

**sg > outputs.tf**
```terraform
output "web_security_groups_id" {
  value = "${aws_security_group.teamcity_web_sg.id}"
}
```

I am passing the debiam_ami as a variable, so I need to add that to the **variables.tf** in the root directory. Skip the follwing if you are hard-coding the ami image name.

**variables.tf**
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

Congrats! You just used terraform to create your first instance! Now let's ssh into it. I have created a key_pair to use when I ssh into the instance. You may wish to use your id_rsa or create a new one (teamcity_ul) and add it to ssh-agent

```bash 
$ ssh-keygen -t rsa -C "teamcity_ul" -P '' -f ~/.ssh/teamcity_ul; chmod 400 ~/.ssh/teamcity_ul.

#start the agent in the background
$ eval "$(ssh-agent -s)"
Agent pid 59566

#Add teamcity_ul private key to the ssh-agent and passphrase in the keychain
$ ssh-add -K ~/.ssh/teamcity_ul
```

Use pbcopy to copy your "PUBLIC" key and add it to aws. You can also copy the public key to your desktop and use the import feature from AWS > EC2 > key Pairs

```bash
$ pbcopy < ~/.ssh/teamcity_ul.pub
```
Navigate to AWS > EC2 > Key Pair. Select "Import Key Pair" and import or paste your key.

Now let's attach that key pair to our instance.

Pass in the key name to the ec2 module. You can hard-code the key name or pass it as a variable:

**main.tf**
```terraform
module "ec2" {
  .
  .
  key_name = "${var.key_name}"
}
```

**variable.tf**
```terraform
.
.

variable "key_name" {
  default = "teamcity_ul"
}
```

**ec2 > main.tf**
```terraform
resource "aws_instance" "teamcity" {
  .
  .

  key_name = "${var.key_name}"
  .
  .
}
```

**ec2 > variables.tf**
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

**ec2 < outputs.tf**
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
ssh -i ~/.ssh/teamcity_ul admin@ec2-18-208-217-36.compute-1.amazonaws.com
```

While we're at it, I added an outputs.tf to the root directory to show the teamcity_ssh_command, because I am lazy.

**outputs.tf**
```terraform
output "teamcity_ssh_command" {
  value = "${format("ssh -i ~/.ssh/teamcity_ul admin@ec2-%s.compute-1.amazonaws.com", "${replace("${module.ec2.teamcity_public_ip}", ".", "-")}")}"
}
```

Use terraform refresh to see the teamcity_ssh_command output you just created:
```bash
$ terraform refresh

# teamcity_ssh_command = ssh -i ~/.ssh/teamcity_ul admin@ec2-x-y-z-d.compute-1.amazonaws.com
```

Since we used Debian, we need to install apache2 in order to access our server via browser. Let's get started by ssh-ing to the machine:

```bash
$ ssh -i ~/.ssh/teamcity_ul admin@ec2-x-y-z-d.compute-1.amazonaws.com
$ sudo apt-get update
$ sudo apt-get install yum
$ sudo apt-get install apache2 
$ sudo service apache2 restart 
```

Give it a minute, now use your browser to access the machine: `https://x-y-z-d:443`


## Host Teamcity with bash using the built-in HSQL db.

Run the following to see the teamcity_ssh_command, then ssh into the machine:
```bash
$ terraform output teamcity_ssh_command
$ ssh -i ~/.ssh/teamcity_ul admin@ec2-x-y-z-d.compute-1.amazonaws.com
```

We can install Teamcity manually or write an ansible for it. If you prefer to set up manually, ssh into the machine and follow the steps bellow. Otherwise, skip this section and continue on **Add a Private Subnet and EC2 Instance to contain the Database for Teamcity** to build a private subnet, add an EC2 instance, create RDS and then use ansible to provision teamcity:

```bash
# Install docker:
$ sudo apt-get install apt-transport-https dirmngr

#Add Docker package depository to your /etc/apt/sources.list sources list:
$ echo 'deb https://apt.dockerproject.org/repo debian-stretch main' | sudo tee --append /etc/apt/sources.list

#Obtain docker's repository signature and updated package index:
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys F76221572C52609D
$ sudo apt-get update

#Install docker
$ sudo apt-get install docker-engine

### Install teamcity
$ sudo docker system prune -a
$ sudo docker pull jetbrains/teamcity-server

$ sudo docker run -it --name teamcity-server-instance \
-v /home/teamcity:/data/teamcity_server/datadir \
-v /var/log/teamcity:/opt/teamcity/logs \
-p 8111:8111 \
jetbrains/teamcity-server
```

Very teamcity is up: `http://x.y.z.d:8111`
Follow the instruction to install teamcity, using the buildin HSQL (this could take a while)
You can use the Teamcity [docs](https://confluence.jetbrains.com/display/TCD18/Configure+and+Run+Your+First+Build) to configure an agent




## Add a Private Subnet and EC2 Instance to contain the Database for Teamcity

First, let's create another security group for the server:

**sg > main.tf**
```terraform
.
.

resource "aws_security_group" "teamcity_server_sg" {
  name        = "Teamcity_server_ssh_sg"
  description = "Allow Teamcity SSH inbound connection"
  vpc_id      = "${var.vpc_id}"

  # Allow SSH
  ingress {
    from_port   = 22
    to_port     = 22
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
    Name = "Teamcity SSH Security Group"
  }
}
```

Add an output for the newly created security group id:

**sg > outputs.tf**
```terraform
.
.

output "server_security_groups_id" {
  value = "${aws_security_group.teamcity_server_sg.id}"
}

```

Now, let's create a NET-eded subnet, using [aws_subnet](https://www.terraform.io/docs/providers/aws/d/subnet.html) resource (private)

**vpc > subnets.tf**
```terraform
.
.
resource "aws_subnet" "private" {
  availability_zone = "${element(var.availability_zones, count.index)}"
  cidr_block        = "10.0.2.0/24"
  vpc_id            = "${aws_vpc.vpc.id}"

  tags {
    Name = "Private Teamcity Subnet"
  }
}
```

Now, let's create a [NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html), using [aws_eip](https://www.terraform.io/docs/providers/aws/r/eip.html) and [aws_nat_gateway](https://www.terraform.io/docs/providers/aws/d/nat_gateway.html) resources. Elastic IP is required to create an NAT Gateway:

```bash
$ touch vpc/nat.tf
```

**vpc > nat.tf**
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
}
```

Let's go ahead and create a Route Table for the NAT-ed Subnet, since we know we're going to need it, [using aws_route_table](https://www.terraform.io/docs/providers/aws/d/route_table.html) and [aws_route_table_association](https://www.terraform.io/docs/providers/aws/r/route_table_association.html) resources :)

**vpc > routing.tf**
```terraform
.
.

resource "aws_route_table" "vpc_private" {
  vpc_id = "${aws_vpc.vpc.id}"

  tags {
    Name = "Teamcity private subnet's route table"
  }
}

resource "aws_route_table_association" "vpc_private" {
  subnet_id      = "${aws_subnet.private.id}"
  route_table_id = "${aws_route_table.vpc_private.id}"
}
```

This is a great place to verify our chances:
```bash
$ terraform plan
# + module.vpc.aws_eip.nat_gw_eip
# + module.vpc.aws_nat_gateway.gw
# + module.vpc.aws_route_table.vpc_private
# + module.vpc.aws_route_table_association.vpc_private
# + module.vpc.aws_subnet.private

$ terraform apply
```

Verify the changes in AWS console: 
- VPC > Security Groups
- VPC > Network ACLs
- VPC > Subnets
- VPC > Route Tables
- VPC > Elastic IPs
- VPC > NAT Gateways

Let's go ahead and create an EC2 instance in this private subnet. Our newly created instance will be able to communicate within the private subnet but it's not visible from Internet

**ec2 > main.tf**
```terraform
.
.

# change the subnet_id to public_subnate_id in the teamcity instance
resource "aws_instance" "teamcity" {
  .
  .
  subnet_id = "${var.public_subnet_id}"
  .
  .

}

resource "aws_instance" "teamcity_server" {
  ami                         = "${var.ami}"
  instance_type               = "m3.medium"
  key_name                    = "${var.key_name}"
  vpc_security_group_ids      = ["${var.server_security_groups_id}"]
  subnet_id                   = "${var.private_subnet_id}"
  associate_public_ip_address = false

  tags = {
    Name = "Teamcity server"
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

**ec2 > variables.tf**
```terraform
.
.
# Change the subnet_id to public_subnet_id
variable "public_subnet_id" {
  default = "default value"
}

variable "private_subnet_id" {
  default = "default value"
}

variable "server_security_groups_id" {
  default     = "default value"
}

```

**ec2 > outputs.tf**
```terraform
output "teamcity_private_ip" {
  value = "${aws_instance.teamcity.public_ip}"
}
```

output the private subnet id: **vpc > outputs.tf**
```terraform
output "private_subnet" {
  value = "${aws_subnet.private.id}"
}
```


pass in the private subnet id and rename the subnet_id to public_subnet_id
**main.tf**
```terraform
.
.

module "ec2" {
  .
  .
  public_subnet_id   = "${module.vpc.public_subnet}"
  private_subnet_id  = "${module.vpc.private_subnet}"
  server_security_groups_id = "${module.sg.server_security_groups_id}"
  .
  .
}
```

From AWS console, view the newly created EC2 instance that's in the private subnet. Notice that it doesn't have a public IP.

Add the following to your **outputs.tf**
```terraform
output "server_ssh_command" {
  value = "${format("ssh -i ~/.ssh/teamcity_ul admin@%s", module.ec2.teamcity_private_ip)}"
}
```

SSH into the new instance
```bash
$ terraform refresh
# server_ssh_command = ssh -i ~/.ssh/teamcity_ul admin@x.y.z.d
```

Summary:
So far we have created a VPC, a public and private subnets and 2 EC2 instances, one to host the Teamcity web and one that would contain the database. At this point, my folder structure looks like
```bash
.
├── ec2
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── main.tf
├── outputs.tf
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
### TBC

### Time to create an RDS!

























--------------
Please let me know what you think about this tutorial, if I have missed any steps :)