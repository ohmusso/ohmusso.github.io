---
layout: post
title:  AWS WordPress構築2
date:   2023-03-16 22:00:00 +0900
categories: aws
tags: aws wordpress
---

login instance by ssh

``` powershell
ssh -i "~/.ssh/udemy try ssh key.pem" ec2-user@${publicip}
```

``` bash
sudo su
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "hellow world" > /var/www/html/index.html
```

## construct LAMP

### Apache

``` bash
yum install -y httpd
httpd -v
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl is-enabled httpd
## give permissions to apache
### check group where the current user belong to
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod -R g+wrxs /var/www
```

### PHP

wordpress require php.

if you know about available php version, check the following web site.
<https://repost.aws/ja/knowledge-center/ec2-install-extras-library-software>

``` bash
sudo amazon-linux-extras install -y php8.2
```

#### PHP Test

* create phpinfo page

``` bash
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfoinfo.php
```

* access phpinfo page

access to <http://${ec2 ipv4 address}/phpinfo.php>

success if phpinfo page is displayed

* when check is done, delete phpinfo.php

``` bash
rm /var/www/html/phpinfo.php
```

### wordpress

``` bash
wget http://ja.wordpress.org/latest-ja.tar.gz
tar -xzf latest-ja.tar.gz
```

move wordpress to www drectory

``` bash
cp -r wordpress/* /var/www/html/
```

wordpress
 user: udem
 password: yVENBAe4g#!NHr%lou

## Create AWS RDS(MySQL)

### Create SecurityGroup for RDS

#### inbound port

protocol: tcp
port: 3306(mysql)
cidr: SecurityGroup EC2

### Create Subnet for RDS

#### Go to RDS Dashboard

### Create RDS Instance

user: admin
password: ZaUe4YkKDyLh0Axht7wZ
endpoint: wordpress-db.cpjjq1wzubjv.ap-northeast-1.rds.amazonaws.com

## Migration

MySQL in EC2 to RDS(MySQL)

#### Export database

``` bash
mysqldump -u root -p wordpress > export.sql
```

#### Import database

``` bash
mysql -u admin -p -h wordpress-db.cpjjq1wzubjv.ap-northeast-1.rds.amazonaws.com wordpress < export.sql
```

#### Stop MySQL in EC2

``` bash
sudo systemctl stop mysqld
sudo systemctl disable mysqld
```

#### Check migration

##### login RDS

``` bash
mysql -u admin -p -h wordpress-db.cpjjq1wzubjv.ap-northeast-1.rds.amazonaws.com wordpress 
```

##### show database

``` sql
show databases;
use wordpress;
show tables;
exit;
```

#### Reset database config of Wordpress

##### Reset config and Create backup

``` bash
mv /var/www/html/wp-config.php wp-config.php.bk
```

##### Access wordpress (EC2 Instance)

##### set RDS infomation

databese: wordpress
database user name: see ### Create RDS Instance user
database password: see ### Create RDS Instance password
database hostname: see ### Create RDS Instance endpoint

## Create ELB Instance

go to EC2 dashboard

### Create SecurityGroup for ELB

inbound: http anywhere
outbound: http anywhere

Must set outbound.
ELB use http outbound to helthcheck.

### Crate TragetGroup

### Create ALC(Application Load Balancer)

### Test ELB

#### Check Apache access log

##### Show Apache accsess log

``` bash
sudo tail -f /var/log/httpd/access_log
```

##### ssh to EC2 main and Show Apache access log

##### ssh to EC2 sub and Show Apache access log

##### access ELB

Details/DNS name: wordpress-elb-1963843660.ap-northeast-1.elb.amazonaws.com

access log is written when access ELB.

## Fix Internet gateway

User can access to EC2 without going through ELB.
Block user from accessing EC2 directly.

### Modify EC2 SecurityGroup

#### inbound

delete: http anywhre
create: http ELB SecurityGroup

## RDS MultiAZ

not free.

### Modify RDS Instance

#### Availability and Durable

set to 'Create Standby Instance'

## Cleanup

### ELB

### EC2 Instance

### EC2 Elastic IP

### EC2 AMI

### EC2 EBS Snapshot

### RDS Instance

### RDS Subnet

### SecurityGroup

### VPC

### EC2 KeyPair
