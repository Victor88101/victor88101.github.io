---
layout: post
category: Linux
title: 《CentOS6.5 使用Yum搭建Lnmp环境》
tagline: by Vv
tags: Linux 
---

# 安装环境：
CentOS 6.5 x86_64

# 目录：
> 1.安装apache

> 2.安装php

> 3.安装mysql

# 内容：

## 1.安装apache

### 1.1 安装

```
yum install httpd
```
### 1.2 启动

```
service httpd start
```
### 1.3 配置开机启动

```
chkconfig httpd on
```

## 2.安装配置php
因为CentOS自带的yum源php版本较低，这里首先添加第三方yum源。
# 2.1 添加yum源

```
rpm -Uvh http://ftp.iij.ad.jp/pub/linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm

rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```
# 2.2 配置yum源
Remi源默认是没有启用的，我们来启用Remi源，修改 /etc/yum.repos.d/remi.repo 文件，把文件内的 enabled=0 改为 enabled=1 ，注意：改文件内有2个 enabled=0 我们修改 [remi]下面的，不要修改[remi-test]下面的。

使用yum list php 查看一下：


```
php.x86_64                          5.5.38-6.el6.remi                   @remi-php55
```


### 2.3 安装php

```
yum install php php-bcmath php-mbstring  php-gd php-xml php-mysql
```

### 2.4 测试

创建文件```/var/www/html/info.php```,内容如下：

```
<?php
phpinfo();
?>
```
访问：http://127.0.0.1/info.php

## 3.安装mysql

### 3.1 安装

```
yum -y install mysql mysql-server
```

### 3.2 启动mysql

```
service mysqld start
```
### 3.3 设置开机启动

```
chkconfig mysqld on
```


### 3.4 为root账户设置密码


```
mysql_secure_installation
# 回车，根据提示输入Y，输入2次密码，回车，根据提示一路输入Y，最后出现：Thanks for using MySQL!
#  MySql密码设置完成，重新启动 MySQL：
#重启
service mysqld restart
```
