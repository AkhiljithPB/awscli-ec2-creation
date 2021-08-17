# Launching an EC2 instance with apache webserver and additional volume for document root using AWS CLI.

## Description
###### Here, we will look into the process of launching an EC2 instance and install apache webserver. Also, create, attach additional volume for ducument root that is through AWS-Command Line Interface.

## Features

- Security group to allow ports 22,80,443.
- Key pair for instance.
- Additional EBS volume for website's document_root.
- apache webserver
- Sample website template.

## Step 1: Configure IAM user on your local machine

Here, I have created an IAM user with adminstrator access and I'm configuring the user on my local machine to manage AWS account throuh aws_cli.


```sh
C:\Users\user1>aws configure
AWS Access Key ID [****************ZCUR]: **************
AWS Secret Access Key [****************2TiW]: ******************
Default region name [ap-south-1]: ap-south-1  <<<<<<Region
Default output format [json]: json
```
## Step 2: Creating a new security group

The below "create-security-group" command will create a new security group with name "cli-sg". "create-tags" helps to add tag to the new security group.
```sh
>aws ec2 create-tags --resources sg-0c6df17de485cfc80 --tags Key=Name,Value=cli-sg
{
    "GroupId": "sg-0c6df17de485cfc80"
}
>aws ec2 create-tags --resources sg-0c6df17de485cfc80 --tags Key=Name,Value=cli-sg
```
Enabling 22,80,443 ports on the new security group using group-name
```sh
aws ec2 authorize-security-group-ingress --group-name cli-sg --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name cli-sg --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name cli-sg --protocol tcp --port 443 --cidr 0.0.0.0/0
```
Finally you can use the "describe-security-groups" command to cross-check the security group informations.
```sh
aws  ec2  describe-security-groups --group-name cli-sg
```
## Step 2: Adding a fresh keypair for my new instance.
here, the "create-key-pair" command used to create a new key pair and the private key will be saved to aws-cli.pem.
```sh
>aws ec2 create-key-pair --key-name aws-cli --query "KeyMaterial" --output text > aws-cli.pem
```
To describe the new Keypair,
```sh
>aws ec2 describe-key-pairs --key-name aws-cli
```
## Step 3: Creating a new instance for my webserver.

I'm using the latest Amazon Linux 2 AMI for my instance and the AMI ID can be queried using the below command,
```sh
aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region ap-south-1
```
After copying the AMI ID I'm proceeding with creating instance using the AMI ID.
```sh
aws ec2 run-instances --image-id ami-04db49c0fb2215364 --security-group-ids sg-0c6df17de485cfc80 --instance-type t2.micro --key-name aws-cli
```
Created new instance and adding a Name tag to identify my instance.
```sh
aws ec2 create-tags --resources i-05068a1c414f16b83 --tags Key=Name,Value=webserver-cli
```
To describe your instances,
```sh
aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro" --query "Reservations[].Instances[].InstanceId"
```
Also, created an additional EBS volume of 1GB for the document root of my website and attached it into my newly created instance.
```sh
>aws ec2 create-volume --volume-type gp2 --size 1 --availability-zone ap-south-1b
>aws ec2 attach-volume --volume-id vol-0f5377dce4593f60a --instance-id i-05068a1c414f16b83 --device /dev/sdf
```
## Step 4: Install and start httpd on the server and mount the attached additional volume.

Here, I have installed httpd service and mounted my new volume as my document root for my website /var/www/html.
```sh
[root@ip-172-31-7-88 ~]# yum install httpd -y ; systemctl start httpd.service; systemctl enable httpd.service
[root@ip-172-31-7-88 ~]# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  482M     0  482M   0% /dev
tmpfs          tmpfs     492M     0  492M   0% /dev/shm
tmpfs          tmpfs     492M  420K  492M   1% /run
tmpfs          tmpfs     492M     0  492M   0% /sys/fs/cgroup
/dev/xvda1     xfs       8.0G  1.6G  6.5G  20% /
tmpfs          tmpfs      99M     0   99M   0% /run/user/1000
/dev/xvdf1     ext4      992M  2.6M  923M   1% /var/www/html <<<doc_root<<<
[root@ip-172-31-7-88 ~]#
```
## Step 4: Hosting a sample website.

I have downloaded a sample html template for my website and also added an entry on /etc/hosts for my site blog.tkhouse.tech to point to my instance IP for testing the working of the website.
```sh
[root@ip-172-31-7-88 html]# pwd
/var/www/html
[root@ip-172-31-7-88 html]# ll
total 1360
drwxr-xr-x 6 apache apache    4096 Dec 16  2018 2100_artist
-rw-r--r-- 1 apache apache 1354194 Dec 16  2018 2100_artist.zip
-rw-r--r-- 1 apache apache     453 Oct 19  2018 ABOUT THIS TEMPLATE.txt
drwxr-xr-x 2 apache apache    4096 Apr 20  2018 css
drwxr-xr-x 2 apache apache    4096 Mar  5  2018 fonts
drwxr-xr-x 2 apache apache    4096 Mar  5  2018 images
-rw-r--r-- 1 apache apache   11098 Dec 16  2018 index.html
drwxr-xr-x 2 apache apache    4096 Mar  5  2018 js
[root@ip-172-31-7-88 html]#
```
### website is loading.
![alt text](https://github.com/AkhiljithPB/awscli-ec2-creation/blob/3b706dc9a70c90fa68757e02d3d3e8ac8e1c7742/website.png?raw=true)
