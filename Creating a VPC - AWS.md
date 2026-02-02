#aws_default_vpc 

There are many ways to create a VPC. In this runbook, we will explore the process of manually creating a demo VPC in the console, and breaking it down, followed by what is needed to create a VPC in [[Terraform]].  

While the manual process seemingly long, the process is made infinitely quicker using Terraform (and other [[Infrastructure as Code - IaC]] tools)  to remove the manual steps required for approved infrastructure requests. Let's start with creating a VPC

Say that you have been tasked with deploying basic infrastructure on AWS to host a proof of concept environment. You will need both public and private subnets and will also need to span multiple [[Availability Zones (AZs)]] in order to test failover and disaster recovery scenarios. The apps will be internet facing and will also need to access the internet in order to get security and OS updates.

Here are the tasks at hand:

- Task 1: Create a new VPC in your account in the mx-central-1 region
- Task 2: Create public and private subnets in three different Availability Zones
- Task 3: Deploy an Internet Gateway and attach it to the VPC
- Task 4: Provision a NAT Gateway (a single instance will do) for outbound connectivity
- Task 5: Ensure that route tables are configured to properly route traffic based on the requirements
- Task 6: Delete the VPC resources
- Task 7: Prepare files and credentials for using Terraform to deploy cloud resources
- Task 8: Set credentials for Terraform deployment
- Task 9: Deploy the AWS infrastructure using Terraform
- Task 10: Delete the AWS resources using Terraform to clean up our AWS environment

The first 6 tasks represent the manual steps, while the last 4 represent your needed Terraform to get the objectives accomplished.

![[Pasted image 20260202095822.png]]

---
### Manual Steps

Tasks 1 through 6 can be done in the VPC and More menu. For the sake of understanding the intricacies of each step, we will break them down individually and go through t

#### Tasks 1 through 6 using the VPC and More
1. Log into the AWS Console > VPC > Your VPCs
	1. Confirm that you are in the region that you would like to create your VPC in by looking at the top right of your screen. If not, then change the region to the correct one. In this case we are building in Mexico (mx-central-1)
2. Resources to create - VPC and more
3. Name tag - make sure to select auto-generate; give the default resources name you'd like
	1. memar
4. IPv4 CIDR - provide CIDR range
	1. 10.22.0.0/16
5. Number of [[Availability Zones (AZs)]] - 3
6. Number of public subnets - 3
7. Number of private subnets - 3
8. NAT Gateway (this costs $ just FYI) - regional (allows access to the internet from any AZ within a VPC through a single exit point)
9. VPC endpoint - S3 gateway
10. Create VPC

#### Tasks 1 through 6 doing each step individually
#### Task 1 - create VPC
1. Log into the AWS Console > VPC > Your VPCs
2. Resources to create - VPC only
3. Name tag - memar
4. IPv4 CIDR range - 10.22.0.0/16
5. Create VPC

#### Task 2 - create public and private subnets in 3 different [[Availability Zones (AZs)]]
1. VPC > Subnets > Create subnet
2. Select a VPC - memar
3. Subnet name - private-subnet-1
4. Availability zone - mx-central-1a
5. IPv4 VPC CIDR block - 10.22.0.0/16
6. IPv4 subnet CIDR block - 10.22.101.0/24
7. Create subnet
8. Head back to VPC > Subnets > Create subnet
9. Repeat the above steps to create additional subnets including 2 more private subnets, 3 public subnets making sure to fulfill the following requirements

| Subnet Name      | Availability Zone | CIDR Block     |
| ---------------- | ----------------- | -------------- |
| private-subnet-2 | mx-central-1b     | 10.22.102.0/24 |
| private-subnet-3 | mx-central-1c     | 10.22.103.0/24 |
| public-subnet-1  | mx-central-1a     | 10.22.1.0/24   |
| public-subnet-2  | mx-central-1b     | 10.22.2.0/24   |
| public-subnet-3  | mx-central-1c     | 10.22.2.0/24   |

#### Task 3 - Deploy an Internet Gateway and attach it to the VPC 
1. VPC > Internet Gateways > Create internet gateway
2. Name - demo-igw
3. Create internet gateway
4. In created igw , select Actions > Attach to VPC > select VPC created in step 1
5. Attach internet gateway

#### Task 4 - Provision a NAT gateway for outbound connectivity
1. VPC > NAT gateways > Create NAT gateway
2. Name - demo-nat-gateway
3. Availability mode - regional
4. VPC > select VPC created in task 1
5. Connectivity - public
6. Method of elastic IP allocation - automatic
7. Create NAT gateway

#### Task 5 - Ensure that route tables are configured to properly route traffic based on the requirements
1. VPC > Route tables > Create route table
2. Name - public-rtb
3. VPC - select VPC created in task 1
4. Create route table
5. Head to VPC > route tables > select public-rtb
6. In the bottom go to Subnet associations > Edit subnet associations
7. Select the 3 PUBLIC subnets available from the list of available subnets (same 3 pubilc ones created in Task 2)
8. Click Save associations button
9. Repeat this step for the private-rtb and select the 3 PRIVATE subnets that were created in task 2
10. Now that the subnets have been associated with the proper route table, we need to add the routes to ensure network traffic is routed correctly 
11. From the VPC > Route Tables > select the public-rtb again. In the bottom pane, select the Routes tab and click Edit Routes
12. Click Add route > Enter 0.0.0.0/0 in the Destination text box (to define our new route destination). 
13. Click the text box for Target, select Internet Gateway, and select the Internet Gateway that was created in Task 3. Click Save changes to save the route configuration
14. Repeat this step to add a route to the private-rtb. 
	1. Destination : 0.0.0.0/0; Target : NAT Gateway

#### Task 6 - the breakdown (deleting VPC resources)
1.  Deletion order : NAT gateway > VPC (and attached resources)
2. VPC > NAT gateway > Select demo-nat-gateway > Actions > Delete NAT gateway 
3. VPC > Your VPCs > Select memar VPC > Actions > Delete VPC

### Terraform steps

#### Task 7 - Prepare files and credentials for using Terraform to deploy cloud resources
1. Open new folder with main.tf and variables.tf files

Variables.tf file should look like this
````sh
variable "aws_region" {
  type    = string
  default = "mx-central-1"
}

variable "vpc_name" {
  type    = string
  default = "memar_vpc"
}

variable "vpc_cidr" {
  type    = string
  default = "10.22.0.0/16"
}

variable "private_subnets" {
  default = {
    "private_subnet_1" = 1
    "private_subnet_2" = 2
    "private_subnet_3" = 3
  }
}

variable "public_subnets" {
  default = {
    "public_subnet_1" = 1
    "public_subnet_2" = 2
    "public_subnet_3" = 3
  }
}
````

Main.tf file should look like this
````sh
# Configure the AWS Provider
provider "aws" {
  region = "mx-central-1"
}

#Retrieve the list of AZs in the current AWS region
data "aws_availability_zones" "available" {}
data "aws_region" "current" {}

#Define the VPC
resource "aws_vpc" "vpc" {
  cidr_block = var.vpc_cidr

  tags = {
    Name        = var.vpc_name
    Environment = "demo_environment"
    Terraform   = "true"
  }
}

#Deploy the private subnets
resource "aws_subnet" "private_subnets" {
  for_each          = var.private_subnets
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, each.value)
  availability_zone = tolist(data.aws_availability_zones.available.names)[each.value]

  tags = {
    Name      = each.key
    Terraform = "true"
  }
}

#Deploy the public subnets
resource "aws_subnet" "public_subnets" {
  for_each                = var.public_subnets
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, each.value + 100)
  availability_zone       = tolist(data.aws_availability_zones.available.names)[each.value]
  map_public_ip_on_launch = true

  tags = {
    Name      = each.key
    Terraform = "true"
  }
}

#Create route tables for public and private subnets
resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block     = "0.0.0.0/0"
    gateway_id     = aws_internet_gateway.internet_gateway.id
    #nat_gateway_id = aws_nat_gateway.nat_gateway.id
  }
  tags = {
    Name      = "demo_public_rtb"
    Terraform = "true"
  }
}

resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block     = "0.0.0.0/0"
    # gateway_id     = aws_internet_gateway.internet_gateway.id
    nat_gateway_id = aws_nat_gateway.nat_gateway.id
  }
  tags = {
    Name      = "demo_private_rtb"
    Terraform = "true"
  }
}

#Create route table associations
resource "aws_route_table_association" "public" {
  depends_on     = [aws_subnet.public_subnets]
  route_table_id = aws_route_table.public_route_table.id
  for_each       = aws_subnet.public_subnets
  subnet_id      = each.value.id
}

resource "aws_route_table_association" "private" {
  depends_on     = [aws_subnet.private_subnets]
  route_table_id = aws_route_table.private_route_table.id
  for_each       = aws_subnet.private_subnets
  subnet_id      = each.value.id
}

#Create Internet Gateway
resource "aws_internet_gateway" "internet_gateway" {
  vpc_id = aws_vpc.vpc.id
  tags = {
    Name = "demo_igw"
  }
}

#Create EIP for NAT Gateway
resource "aws_eip" "nat_gateway_eip" {
  domain     = "vpc"
  depends_on = [aws_internet_gateway.internet_gateway]
  tags = {
    Name = "demo_igw_eip"
  }
}

#Create NAT Gateway
resource "aws_nat_gateway" "nat_gateway" {
  depends_on    = [aws_subnet.public_subnets]
  allocation_id = aws_eip.nat_gateway_eip.id
  subnet_id     = aws_subnet.public_subnets["public_subnet_1"].id
  tags = {
    Name = "demo_nat_gateway"
  }
}
````


#### Task 8 - Set credentials for Terraform deployment

This should already be done, but if you haven't already done so you can head over to the AWS console then head to IAM > Users > select your user > generate an access key and secret key from a user with administrative privileges. 

In Git Bash type in the following:
````sh
aws configure --profile YOUR_PREFERRED_PROFILE_NAME
````

When prompted, enter your AWS Access Key ID, Secret Access Key, default region, and default output format. 

In PowerShell, you need to do the following
````sh
PS C:\> $Env:AWS_ACCESS_KEY_ID="<YOUR ACCESS KEY>"
PS C:\> $Env:AWS_SECRET_ACCESS_KEY="<YOUR SECRET KEY>"
````


#### Task 9 - Deploy the AWS Infrastructure using Terraform

Never forget: the Terraform i(v)PAD

First step will be enterring the following in your shell session
````sh
terraform init
````

This should give you an output that has the backend initializing and leads to "Terraform has successfully initialized"

At this stage run:
````sh
terraform validate
````

Once initialized, you can set a plan that will take your end goal (your Terraform code) and lays out what is needed in order to make that plan a reality

````sh
terraform plan
````

This will give you an output describing what is being planned and leading to something along the lines of "Plan: 18 to add, 0 to change, 0 to destroy"

If you like what you see, you can go ahead to the next step:
````sh
terraform apply -auto-approve
````
Note: should you want to manually approve the changes that are put applied, you can omit the -auto-approve flag. 

Once this has run you should see something along the lines of "Apply complete! Resources: 18 added, 0 changed, 0 destroyed." Everything should be showing up in your console as well (easy way to double check that all was done as expected)

#### Task 10 - Breakdown (delete) the AWS resources to clean up the AWS environment

The code to do this is quite simple:
````sh
terraform destroy -auto-approve
````


That's it. The end. FIN

---

