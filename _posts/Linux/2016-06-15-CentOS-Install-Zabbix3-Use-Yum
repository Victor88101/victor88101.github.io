---
layout: post
category: Linux
title: 《CentOS6.5 使用Yum安装Zabbix3.0》
tagline: by Vv
tags: Linux 
---

# 安装环境
CentOS 6.5 x86_64

# 目录
> 1.lamp环境搭建
> 
> 2.zabbix服务器端安装
>
> 3.zabbix-web安装
>
> 4.zabbix-agent安装

# 说明
zabbix主要有zabbix、zabbix_agent、zabbix_get、zabbix_proxy、zabbix_sender、zabbix_server、zabbix-web、zabbix_java_gateway等几个功能模块。
其中服务器端需要zabbix、zabbix_server、zabbix_get，web管理段需要zabbix_web,agent端需要zabbix_anent、zabbix_sender、zabbix_java_gateway(可选)


# 内容：

## 1.lamp环境搭建

### 1.1安装apache

#### 1.1.1 安装

```
yum install httpd
```
#### 1.1.2 启动

```
service httpd start
```
#### 1.1.3 配置开机启动

```
chkconfig httpd on
```

### 1.2 安装配置php
因为CentOS自带的yum源php版本较低，这里首先添加第三方yum源。
# 1.2.1 添加yum源

```
rpm -Uvh http://ftp.iij.ad.jp/pub/linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm

rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```
#### 1.2.2 配置yum源
Remi源默认是没有启用的，我们来启用Remi源，修改 /etc/yum.repos.d/remi.repo 文件，把文件内的 enabled=0 改为 enabled=1 ，注意：改文件内有2个 enabled=0 我们修改 [remi]下面的，不要修改[remi-test]下面的。

使用yum list php 查看一下：


```
php.x86_64                          5.5.38-6.el6.remi                   @remi-php55
```


#### 1.2.3 安装php

```
yum install php php-bcmath php-mbstring  php-gd php-xml php-mysql
```
#### 1.2.4 配置php
修改/etc/php.ini,内容如下：
```
post_max_size=16M
max_execution_time=300
max_input_time=300
date.timezone=Asia/Shanghai
always_populate_raw_post_data=-1
```
#### 1.2.5 测试

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

## 2.zabbix服务器端安装

### 2.1 安装yum源 


```
rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/6/x86_64/zabbix-release-3.0-1.el6.noarch.rpm
```

### 2.2 安装zabbix所需模块
这里的主机既是服务器端也是客户端，所以需要安装以下模块：
```
yum install zabbix-server zabbix-server-mysql zabbix-agent zabbix-get zabbix-sender
```
### 2.3 配置数据库

#### 2.3.1 创建数据库及数据库用户

使用root用户登录数据库，执行以下命令：

```
CREATE DATABASE zabbix DEFAULT CHARACTER SET utf8 COLLATE utf8_bin;

GRANT ALL ON zabbix.* TO 'zabbix'@'%' IDENTIFIED BY 'zabbix';

flush privileges;
```

#### 2.3.2 导入数据

```
zcat /usr/share/doc/zabbix-server-mysql-3.0.7/create.sql.gz |mysql -h10.192.87.101 -uroot -p zabbix
```
这里使用的mysql安装在另外服务器上，如果使用本地mysql可以去掉“-h10.192.87.101”。

### 2.4 修改zabbix-server配置

```
vi /etc/zabbix/zabbix_server.conf
```
修改内容如下：

```
DBHost=10.192.87.101
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
```
重新启动zabbix-server
```
service zabbix-server restart
```

## 3.zabbix-web安装
### 3.1 安装
yum源配置同上。
```
yum install zabbix-web zabbix-web-mysql 
```
把web文件拷贝到/var/www/html/目录：
```
cp -r /usr/share/zabbix /var/www/html/
```
把/etc/zabbix目录所属改为apache启动账户
```
chown -R apache:apache /etc/zabbix
```

### 3.2 配置php

修改/etc/php.ini,内容如下：
```
post_max_size=16M
max_execution_time=300
max_input_time=300
date.timezone=Asia/Shanghai
always_populate_raw_post_data=-1
```
配置完成之后需要重启apache

```
service httpd restart
```


### 3.3 使用zabbix-web管理zabbix-server
通过浏览器打开http://****/zabbix,
## 4.zabbix-agent安装
### 4.1 安装
```
yum install zabbix-agent zabbix-sender
```

### 4.2 启动
```
service zabbix-agentd start
```
### 4.3 设置开机启动

```
chkconfig zabbix-agentd on
```
