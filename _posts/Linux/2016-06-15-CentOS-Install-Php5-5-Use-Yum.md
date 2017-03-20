---
layout: post
category: Linux
title: 《CentOS6.5 Yum 安装Php5.5》
tagline: by Vv
tags: Linux 
---

> CentOS 6.5 Yum源里的Php版本较低，这里通过添加epel和remi的Yum源来安装Php5.5
# 1.添加yum源

```
rpm -Uvh http://ftp.iij.ad.jp/pub/linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm

rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
```
# 2.配置
Remi源默认是没有启用的，我们来启用Remi源，修改 /etc/yum.repos.d/remi.repo 文件，把文件内的 enabled=0 改为 enabled=1 ，注意：改文件内有2个 enabled=0 我们修改 [remi]下面的，不要修改[remi-test]下面的。

# 3.安装php
使用yum list php 查看一下：


```
php.x86_64                          5.5.38-6.el6.remi                   @remi-php55
```

可以看到已经是5.5。使用yum 安装：

```
yum install php php-bcmath php-mbstring php-fpm php-gd php-xml php-mysql
```

