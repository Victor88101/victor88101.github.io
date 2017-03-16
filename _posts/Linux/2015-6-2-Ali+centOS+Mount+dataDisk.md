---
layout: post
category: Linux
title: 《阿里云CentOS服务器挂载数据盘》
tagline: by Vv
tags: Linux
---

# 环境 #
***
	Linux版本：CentOS 6.5

# 目录 #
***
	1.查看为挂载数据盘
	2.创建分区、格式化数据盘
	3.挂载数据盘
	4.相关知识

# 内容 #
***
## 1.查看未挂载数据盘 ##

使用df命令查看所有已挂载的磁盘

	[root@Serv105 ~]# df -h
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/xvda1       20G  1.5G   18G   8% /
	tmpfs           498M     0  498M   0% /dev/shm

可以看到里面并没有数据盘。
这就需要使用fdisk命令可以查看所有的磁盘

    [root@Serv105 ~]# fdisk -l
    
    Disk /dev/xvda: 21.5 GB, 21474836480 bytes
    255 heads, 63 sectors/track, 2610 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00078f9c
    
    Device Boot  Start End  Blocks   Id  System
    /dev/xvda1   *   1261120970496   83  Linux
    
    Disk /dev/xvdb: 53.7 GB, 53687091200 bytes
    255 heads, 63 sectors/track, 6527 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
 
我们可以看到`/dev/xvdb: 53.7 GB`这块磁盘。我们所要做的就是把这块磁盘挂载到`/data/`目录。

## 2.在数据盘上创建分区并格式化 ##

创建分区使用`fdisk`命令：

    [root@Serv105 ~]# fdisk /dev/xvdb

    Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
    Building a new DOS disklabel with disk identifier 0xe9145c2b.
    Changes will remain in memory only, until you decide to write them.
    After that, of course, the previous content won't be recoverable.
    
    Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

	WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

	Command (m for help): n	
	Command action
	   e   extended
	   p   primary partition (1-4)
	p	#输入p创建主分区
	Partition number (1-4): 
	Value out of range.
    Partition number (1-4): 1
    First cylinder (1-6527, default 1): 
    Using default value 1
    Last cylinder, +cylinders or +size{K,M,G} (1-6527, default 6527): 
    Using default value 6527

    Command (m for help): wq #输入wq保存退出
    The partition table has been altered!
    
    Calling ioctl() to re-read partition table.
    Syncing disks.

我们这里使用整个磁盘创建一个分区，这里在
完成之后，我们再用`fdisk -l`命令查看：

    [root@Serv105 ~]# fdisk -l
    
    Disk /dev/xvda: 21.5 GB, 21474836480 bytes
    255 heads, 63 sectors/track, 2610 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00078f9c

    Device Boot      Start         End      Blocks   Id  System
	/dev/xvda1   *           1        2611    20970496   83  Linux

    Disk /dev/xvdb: 53.7 GB, 53687091200 bytes
    255 heads, 63 sectors/track, 6527 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0xe9145c2b

    Device Boot      Start         End      Blocks   Id  System
	/dev/xvdb1               1        6527    52428096   83  Linux

在最下面，可以看到在`/dev/xvdb`上已经创建了一个分区`/dev/xvdb1`。下面我们还需要使用`mkfs.ext3`命令对`/dev/xvdb1`进行格式化。

    [root@Serv105 ~]# mkfs.ext3 /dev/xvdb1

    mke2fs 1.41.12 (17-May-2010)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    3276800 inodes, 13107024 blocks
    655351 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=4294967296
    400 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
        4096000, 7962624, 11239424

    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done
    
    This filesystem will be automatically checked every 20 mounts or
    180 days, whichever comes first.  Use tune2fs -c or -i to override.

## 3.挂载数据盘 ##

下面我们需要修改`/etc/fstab`文件进行数据盘的挂载。`/etc/fstab`文件里面是文件系统的一些信息。系统在启动时，通过读取整个文件，自动挂载磁盘。

	[root@Serv105 ~]# echo '/dev/xvdb1    /data    ext3   defaults   0 0' >> /etc/fstab
	[root@Serv105 ~]# mkdir /data
	[root@Serv105 ~]# mount -a

使用`df`命令查看：

	[root@Serv105 ~]# df -h
	Filesystem      Size  Used Avail Use% Mounted on
	/dev/xvda1       20G  1.5G   18G   8% /
	tmpfs           498M     0  498M   0% /dev/shm
	/dev/xvdb1       50G  180M   47G   1% /data

可以看到，已经挂载成功。
