# Cloud-Task-3

# About the Project :
 
The motive is to create a fully secured Web Portal for our company. For this, I have created a setup for Wordpress Software with a dedicated Databse server that will only be accessible by the Wordpress & not by the outside world.

# What is Wordpress ?

WordPress (WP, WordPress.org) is a free and open-source content management system (CMS) written in PHP and paired with a MySQL or MariaDB database. Features include a plugin architecture and a template system, referred to within WordPress as Themes. WordPress was originally created as a blog-publishing system but has evolved to support other types of web content including more traditional mailing lists and forums, media galleries, membership sites, learning management systems (LMS) and online stores. WordPress is used by more than 60 million websites, including 33.6% of the top 10 million websites as of April 2019,[6][7] WordPress is one of the most popular content management system solutions in use. WordPress has also been used for other application domains such as pervasive display systems (PDS).


# Project Description :

Steps:

1- Write a Infrastructure as code using terraform, which automatically create a VPC.

2- In that VPC we have to create 2 subnets:
a) public subnet [ Accessible for Public World! ] 
b) private subnet [ Restricted for Public World! ]

3- Create a public facing internet gateway for connecting our VPC/Network to the internet world and attach this gateway to our VPC.

4- Create a routing table for Internet gateway so that instance can connect to outside world, update and associate it with public subnet.

5- Launch an ec2 instance which has Wordpress already setup, having the security group allowing port 80 so that our client can connect to our wordpress site. Also attach the key to instance to facilitate further login.

6- Launch an ec2 instance which has MYSQL already setup with security group allowing port 3306 in private subnet so that our wordpress VM can connect with the same. Also attach the key with the same.

Note: Wordpress instance has to be part of public subnet so that our client can connect with our site. MYSQL instance has to be part of private subnet so that outside world can't connect to it. Don't forgot to add auto ip assign and auto dns name assignment option should also be enabled.



# Steps :-

**Step - 1:**  First of all, configure your AWS profile in your local system using cmd. Fill your details & press Enter.


                  aws configure --profile Sparsh
                  AWS Access Key ID [****************WO3Z]:
                  AWS Secret Access Key [****************b/hJ]:
                  Default region name [ap-south-1]:
                  Default output format [None]:
                  
                 
**Step - 2:** Next, we need to create a VPC. Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways. You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications.

The terrfaorm code to create a VPC is as follows :-


                resource "aws_vpc" "sparsh_vpc" {
                cidr_block = "192.168.0.0/16"
                instance_tenancy = "default"
                enable_dns_hostnames = true
                tags = {
                  Name = "sparsh_vpc"
                }
              }
              
              
              
**Step - 3:** Now, we need to create two subnets in this VPC :


a) public subnet [ Accessible for Public World! ] 

b) private subnet [ Restricted for Public World! ]


Subnet is “part of the network”, in other words, part of entire availability zone. Each subnet must reside entirely within one Availability Zone and cannot span zones.
The terraform code to create both the Private & the Public Subnet is as follows :


                resource "aws_subnet" "sparsh_public_subnet" {
                vpc_id = "${aws_vpc.sparsh_vpc.id}"
                cidr_block = "192.168.0.0/24"
                availability_zone = "ap-south-1a"
                map_public_ip_on_launch = "true"
                tags = {
                  Name = "sparsh_public_subnet"
                }
              }
              
              
              
              resource "aws_subnet" "sparsh_private_subnet" {
                vpc_id = "${aws_vpc.sparsh_vpc.id}"
                cidr_block = "192.168.0.0/24"
                availability_zone = "ap-south-1a"
                tags = {
                  Name = "sparsh_private_subnet"
                }
              }
              
              
 
**Step - 4:** Next, we create a Public facing Internet Gateway. An internet gateway is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet. 

The terraform code to create the Gateway is as follows :

                resource "aws_internet_gateway" "sparsh_gw" {
                vpc_id = "${aws_vpc.sparsh_vpc.id}"
                tags = {
                  Name = "sparsh_gw"
                }
              }
              
              
**Step - 5:** Next, we create a Routing Table & associate it with the Public Subnet. A routing table, or routing information base (RIB), is an electronic file or database-type object that is stored in a router or a networked computer, holding the routes (and in some cases, metrics associated with those routes) to particular network destinations. This information contains the topology of the network close to it. The terrafrom code for the same is as follows :

                resource "aws_route_table" "sparsh_rt" {
                vpc_id = "${aws_vpc.sparsh_vpc.id}"

                route {
                  cidr_block = "0.0.0.0/0"
                  gateway_id = "${aws_internet_gateway.sparsh_gw.id}"
                }

                tags = {
                  Name = "sparsh_rt"
                }
              }


              resource "aws_route_table_association" "sparsh_rta" {
                subnet_id = "${aws_subnet.sparsh_public_subnet.id}"
                route_table_id = "${aws_route_table.sparsh_rt.id}"
              }
              
              
              
**Step - 6:** Now, we create our security group which will be used while launching Wordpress. This security group has the permissions for outside connectivity.
A security group acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic. Inbound rules control the incoming traffic to your instance, and outbound rules control the outgoing traffic from your instance. Security groups are associated with network interfaces.


               resource "aws_security_group" "sparsh_sg" {

                name        = "sparsh_sg"
                vpc_id      = "${aws_vpc.sparsh_vpc.id}"


                ingress {
                
                  description = "allow_http"
                  from_port   = 80
                  to_port     = 80
                  protocol    = "tcp"
                  cidr_blocks = [ "0.0.0.0/0"]

                }


                   ingress {
                 
                   description = "allow_ssh"
                   from_port   = 22
                   to_port     = 22
                   protocol    = "tcp"
                   cidr_blocks = ["0.0.0.0/0"]
                 }
                 
                 ingress {
                 
                   description = "allow_icmp"
                   from_port   = 0
                   to_port     = 0
                   protocol    = "tcp"
                   cidr_blocks = ["0.0.0.0/0"]
                 }
                 
                 ingress {
                 
                   description = "allow_mysql"
                   from_port   = 3306
                   to_port     = 3306
                   protocol    = "tcp"
                   cidr_blocks = ["0.0.0.0/0"]
                 }

                 egress {
                   from_port   = 0
                   to_port     = 0
                   protocol    = "-1"
                   cidr_blocks = ["0.0.0.0/0"]
                 }
                 
                 
                  tags = {

                  Name = "sparsh_sg"
                }
              }


**Step : 7** Now, we create another security group which will be used to launch MYSQL. This security group will keep the MYSQL accessible only through the wordpress and not through outside world.



                resource "aws_security_group" "sparsh_sg_private" {

                name        = "sparsh_sg_private"
                vpc_id      = "${aws_vpc.sparsh_vpc.id}"
              
                ingress {
                
                 description = "allow_mysql"
                 from_port   = 3306
                 to_port     = 3306
                 protocol    = "tcp"
                 security_groups = [aws_security_group.sparsh_sg.id]
                 
               }


               ingress {
               
                 description = "allow_icmp"
                 from_port   = -1
                 to_port     = -1
                 protocol    = "icmp"
                 security_groups = [aws_security_group.sparsh_sg.id]
                 
               }

               egress {
               
                from_port   = 0
                to_port     = 0
                protocol    = "-1"
                cidr_blocks = ["0.0.0.0/0"]
                ipv6_cidr_blocks =  ["::/0"]
              }
              
              
              tags = {

                  Name = "sparsh_sg_private"
                }
              }
              
              
              
  **Step - 8:** Now, we are ready to go. We launch our Wordpress and MYSQL instances using all the resources that we have created above. 
  
  **Wordpress* -

            resource "aws_instance" "wordpress" {
            
            ami           = "ami-ff82f990"
            instance_type = "t2.micro"
            key_name      =  "sparsh_key"
            subnet_id     = "${aws_subnet.sparsh_subnet.id}"
            security_groups = ["${aws_security_group.sparsh_sg.id}"]
            associate_public_ip_address = true
            availability_zone = "ap-south-1a"


            tags = {
              Name = "sparsh_wordpress"
              }
            } 


 **MYSQL** -
 
                        resource "aws_instance" "sql" {
                        ami             =  "ami-08706cb5f68222d09"
                        instance_type   =  "t2.micro"
                        key_name        =  "sparsh_key"
                        subnet_id     = "${aws_subnet.sparsh_private_subnet.id}"
                        availability_zone = "ap-south-1a"
                        security_groups = ["${aws_security_group.sparsh_sg_private.id}"]
                        
                        tags = {
                         Name = "sparsh_sql"
                         }
                       } 
