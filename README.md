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
