---
layout: post
category: Mysql
title: 《修改Mysql字符集》
tagline: by Vv
tags: Mysql
---

# 环境 #
***
	Linux版本：CentOS 6.5
	Mysql版本：Mysql 5.5

# 目录 #
***
	1.查看
	2.修改

# 内容 #

***

修改字符集的方法有很多，我这里只列出我常用的也是比较靠谱的一种。

## 一、查看 ##
	
登录MySQL，使用命令`SHOW VARIABLES LIKE 'character%';`查看当前字符集，
	
	[root@Serv225 ~]# /usr/local/mysql/bin/mysql -h 127.0.0.1 -u root
	mysql> SHOW VARIABLES LIKE 'character%';
显示如下：

    +--------------------------+----------------------------+
    | Variable_name | Value |
    +--------------------------+----------------------------+
    | character_set_client | utf8 |
    | character_set_connection | utf8 |
    | character_set_database | latin1 |
    | character_set_filesystem | binary |
    | character_set_results | utf8 |
    | character_set_server | latin1 |
    | character_set_system | utf8 |
    | character_sets_dir | /usr/local/mysql/charsets/ |
    +--------------------------+----------------------------+

`character_set_database`和`character_set_server`的默认字符集还是`latin1`。

## 二、修改 ##

编辑mysql的`my.cnf`文件，修改其中的字符集键值（注意配置的字段细节）

1、在`[client]`字段里加入`default-character-set=utf8`，如下：

    [client]
    port = 3306
    socket = /var/lib/mysql/mysql.sock
    default-character-set=utf8

2、在`[mysqld]`字段里加入`character-set-server=utf8`，如下：

    [mysqld]
    port = 3306
    socket = /var/lib/mysql/mysql.sock
    character-set-server=utf8

3、在`[mysql]`字段里加入`default-character-set=utf8`，如下：

    [mysql]
    no-auto-rehash
    default-character-set=utf8

修改完成后，使用`service mysql restart`命令重启mysql服务就可以生效了。

> 注意：[mysqld]字段与[mysql]字段是有区别的。

重启后，再使用`SHOW VARIABLES LIKE 'character%';`命令查看，发现数据库编码全已改成utf8。



    +--------------------------+----------------------------+
    | Variable_name | Value |
    +--------------------------+----------------------------+
    | character_set_client | utf8 |
    | character_set_connection | utf8 |
    | character_set_database | utf8 |
    | character_set_filesystem | binary |
    | character_set_results | utf8 |
    | character_set_server | utf8 |
    | character_set_system | utf8 |
    | character_sets_dir | /usr/local/mysql/charsets/ |
    +--------------------------+----------------------------+


4、如果上面的都修改了还乱码，那剩下问题就一定在connection连接层上。解决方法是在发送查询前执行一下下面这句（直接写在SQL文件的最前面）：
	
	SET NAMES 'utf8';

它相当于下面的三句指令：

    SET character_set_client = utf8;
    SET character_set_results = utf8;
    SET character_set_connection = utf8;

