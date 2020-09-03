![alt text](https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcSqbDdtw15lXcffklE79HPByzmbarOFIUeoQQ&usqp=CAU)
# AWS_TASK4-

## Task Description-

Perform task-3 with an additional feature to be added that is NAT Gateway to provide the internet access to instances running in the private subnet.

Performing the following steps:
1.  Write an Infrastructure as code using terraform, which automatically create a VPC.
2.  In that VPC we have to create 2 subnets:
    1.   public  subnet [ Accessible for Public World! ] 
    2.   private subnet [ Restricted for Public World! ]
3. Create a public facing internet gateway for connect our VPC/Network to the internet world and attach this gateway to our VPC.
4. Create  a routing table for Internet gateway so that instance can connect to outside world, update and associate it with public subnet.
5.  Create a NAT gateway for connect our VPC/Network to the internet world  and attach this gateway to our VPC in the public network
6.  Update the routing table of the private subnet, so that to access the internet it uses the nat gateway created in the public subnet
7.  Launch an ec2 instance which has Wordpress setup already having the security group allowing  port 80 sothat our client can connect to our wordpress site. Also attach the key to instance for further login into it.
8.  Launch an ec2 instance which has MYSQL setup already with security group allowing  port 3306 in private subnet so that our wordpress vm can connect with the same. Also attach the key with the same.




### Prerequisite-
1. AWS Console Account
2. AWS CLI
3. Terraform Installed
4. Putty or Mobaxterm

## Lets Begin!!!
#### Step 1:
##### Defining the provider on which we have to create infrastructure.
    provider "aws" {
    region = “ap-south-1”
    profile = “default”
    }

#### Step 2:
##### Creating a VPC with CIDR Block 192.168.0.0/16 and enable DNS Hostnames to assign dns names to instances.
    resource “aws_vpc” “myvpc”{
     cidr_block = “192.168.0.0/16”
     instance_tenancy = “default”
     enable_dns_hostnames = true
    tags = {
     Name = “abhishekvpc”
     }
    }

#### Step 3:
##### Creating public and private subnet.
##### Public subnet-
    resource "aws_subnet" "public" {
     vpc_id     = aws_vpc.vpc.id
     cidr_block = "192.168.10.0/24"
     availability_zone = "ap-south-1b"
     map_public_ip_on_launch = "true"
    tags = {
      Name = "public-subnet"
     }
    }
##### Private subnet-
    resource "aws_subnet" "private" {
     vpc_id     = aws_vpc.vpc.id
     cidr_block = "192.168.20.0/24"
     availability_zone = "ap-south-1a"
    tags = {
      Name = "private-subnet"
     }
    }

#### Step 4:
#####  Creating internet gateway to connect our VPC to internet.
    resource "aws_internet_gateway" "gateway" {
     vpc_id = aws_vpc.myvpc.id
    tags = {
      Name = "abhi_gateway"
     }
    }

#### Step 5:
##### Write a Terraform code to Create a routing table for Internet gateway so that instance can connect to outside world, update and associate it with public subnet.
    resource "aws_route_table" "route" {
      vpc_id = aws_vpc.vpc.id
           route {
              cidr_block = "0.0.0.0/0"
              gateway_id = aws_internet_gateway.gateway.id
          }
    tags = {
                 Name = "gatewayroute"
          }
       }
    resource "aws_route_table_association" "public"
     {
      subnet_id   = aws_subnet.public.id
      route_table_id = aws_route_table.route.id
     }
#### Step 6: 
##### Connecting Routing Table to Public Subnet

    resource "aws_route_table_association" "subnet_assosiate" {
      subnet_id      = aws_subnet.public.id
      route_table_id = aws_route_table.route.id
    }
#### Step 7: 
##### Creating an Elastic IP for the AWS VPC.

    resource “aws_eip” “abhi_eip”{
     vpc = true 
    }
#### Step 8:
##### Creating a NAT gateway and allocate the elastic ip to it and put it in public subnet.

    resource “aws_nat_gateway” “abhinat1”{
     allocation_id = aws_eip.abhi_eip.id
     subnet_id = aws_subnet.public.id
    
    tags = {
     Name = “abhinat1”
 
    }
    
    }
#### Step 9:
##### Creating a Route Table with CIDR block 0.0.0.0/0 to connect any ip and attaching it to NAT Gateway.

    resource "aws_route_table" "route_2"{
      vpc_id = aws_vpc.myvpc.id
    
    route {
    
        cidr_block = "0.0.0.0/0"
        nat_gateway_id = aws_nat_gateway.aninat1.id
      }
    
    depends_on = [
        aws_vpc.myvpc
      ]
    
    tags = {
        Name = "route_2"
      }
    }
