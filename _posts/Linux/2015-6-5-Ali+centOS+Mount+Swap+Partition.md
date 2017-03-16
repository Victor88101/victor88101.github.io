---
layout: post
category: Linux
title: 《阿里云CentOS服务器挂载Swap分区》
tagline: by Vv
tags: Linux
---

# 环境 #
***
	Linux版本：CentOS 6.5

# 目录 #
***
	1.使用`dd`命令创建一个4G的整块文件
	2.把`/var/swap`格式化为swap分区
	3.使用`swapon`命令开启交换分区
	4.设置开机自动挂载Swap分区
	5.设置Swap分区使用规则

# 内容 #
***

在使用阿里云CentOS服务器的时候，发现Swap分区一直为0：

	top - 17:55:41 up 2 days,  3:05,  2 users,  load average: 0.61, 0.36, 0.26
	Tasks:  79 total,   2 running,  77 sleeping,   0 stopped,   0 zombie
	Cpu(s):  2.0%us,  3.8%sy,  0.0%ni, 34.8%id, 58.4%wa,  0.3%hi,  0.7%si,  0.0%st
	Mem:   1920740k total,  1381600k used,   539140k free,     7044k buffers
	Swap:        0k total,        0k used,        0k free,   269528k cached

当时还没怎么注意，后来发现不太对。原来，默认阿里云服务器并没有划分Swap分区，只能自己添加了。
添加Swap分区通常有两种方法：
1.使用未划分的磁盘空间，创建一个分区，然后格式化为swap格式，之后挂载使用。
2.使用`dd`命令创建一个整块文件，然后格式化为swap格式，作为swap分区使用，之后挂载使用。当然这种方法创建的swap分区性能比第一种方法差一些。

我这里因为整个磁盘在系统初始化的的时候都已经被全部使用，所以只能采取第二种方法。

## 1.使用`dd`命令创建一个4G的整块文件 ##

使用`df`命令查看所有已挂载的磁盘

	[root@Serv105 ~]# df -h
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/xvda1       20G  1.5G   18G   8% /
	tmpfs           498M     0  498M   0% /dev/shm

可以看到里面并没有Swap分区。同时，通过`free -m`命令看到swap为0：

	[root@Serv105 ~]$ free -m
           		  total       used       free     shared    buffers     cached
	Mem:          1875       1632        243          0         29        435
	-/+ buffers/cache:       1167        708
	Swap:         0          0       0


使用`dd`命令初始化一个Swap文件：

    [root@Serv105 ~]$ dd if=/dev/zero of=/var/swap bs=1M count=4096
    4096+0 records in
    4096+0 records out
    4294967296 bytes (4.3 GB) copied, 92.0653 s, 46.7 MB/s

## 2.把`/var/swap`格式化为swap分区 ##

	[root@Serv105 ~]$ mkswap /var/swap
	mkswap: /var/swap: warning: don't erase bootbits sectors on whole disk. Use -f to force.
	Setting up swapspace version 1, size = 4194300 KiB
	no label, UUID=7c0630eb-7c7c-4517-a566-5b2d060942c0

## 3.使用`swapon`命令开启交换分区 ##

	[root@Serv105 ~]$ swapon /var/swap

现在使用`free -m`命令查看，就可以看到Swap已经可以用了。

	[root@Serv105 ~]$ free -m
    			  total       used       free     shared    buffers     cached
	Mem:          1875       1808         67          0          6       1389
	-/+ buffers/cache:        412       1463
	Swap:         4095          0       4095

## 4.设置开机自动挂载Swap分区 ##

前面使用`swapon`开启交换分区的操作，在系统重启之后就会失效。所以，我们这里需要通过修改`/etc/fstab`文件，设置成开机自动挂载Swap分区。

	[root@Serv105 ~]$ echo '/dev/swap    swap    ext3   defaults   0 0' >> /etc/fstab

## 5.设置Swap分区使用规则 ##

Swap分区虽然已经启用，但是`used`一直为0。这是因为Swap分区的启用是有一定规则的。我们可以查看`/proc/sys/vm/swappiness`文件。


	[root@Serv105 ~]# cat /proc/sys/vm/swappiness
	0

这个值的意思就是：当内存使用`100%-0%=100%`的时候，采用Swap分区。当然，这个`0`的意思并不是绝对的当内存用完了到时候，才使用Swap，只是说尽可能不使用Swap。阿里云服务器默认这个值为0，是因为，采用Swap会频繁读取硬盘，加大IO负担。所以让程序运行尽可能的使用内存而不是Swap。当然，当内存吃紧的时候，还是要用的。这个值通常设置为`40%-60%`。

	[root@Serv105 ~]# echo "40">/proc/sys/vm/swappiness

这种修改方式只会临时有效，当系统重启之后，就会失效。想要彻底有效需要修改`/etc/sysctl.conf`配置文件，里面有一参数`vm.swappiness = 0`，把它修改为需要的值。

	[root@Serv105 ~]# vi /etc/sysctl.conf
	[root@Serv105 ~]# sysctl

这是因为系统启动的时候，会先读取`/etc/sysctl.conf`里面的参数`vm.swappiness`。通过这个参数来设置`/proc/sys/vm/swappiness`的值。

	[root@Serv225 ~]# free -m
             	  total       used       free     shared    buffers     cached
	Mem:          1875       1685        190          0        113        194
	-/+ buffers/cache:       1377        498
	Swap:         4095          5       4090

现在查看，Swap已经被使用了。
