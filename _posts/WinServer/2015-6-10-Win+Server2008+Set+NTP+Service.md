---
layout: post
category: WinServer
title: 《Windows Server 2008搭建时间同步服务器及客户端使用》
tagline: by Vv
tags: WinServer
---

> 今天朋友找我帮忙设置一台WinServer2008服务器的时间同步，设置成每小时同步一次。就在网上搜了下，顺带着找到了如何搭建时间同步服务器方法，这里做个记录。

# 环境 #

***

	Windows版本：WinServer2008 64bit

# 目录 #

***
    一、开启NTP服务
    二、配置客户端时间同步

# 内容 #
***
<br/>

## 一、开启NTP服务 ##

### 1.开启NTP服务器模式 ###

因为默认情况下，WinServer 2008服务器是作为NTP客户端工作的，所以必须修改注册表，以使系统作为NTP服务器运行。
按`win+r`，输入`regedit`，打开注册表，找到`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpServer\Enabled`键，把值设定为`1`。

### 2.设定时钟 ###

找到`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\Config\AnnounceFlags`键，把值设定为`5`。
设定为`5`就是强制主机将它自身宣布为可靠的时间源，从而使用内置的互补金属氧化物半导体 (CMOS) 时钟。如果要采用外面的时间服务器就用默认的`a`值即可。

### 3.重启服务 ###

在命令行中输入`net stop w32time && net start w32time`，重启时间服务。

>如果说防火墙打开了的话，要配置允许123端口的UDP连接。

## 二、配置客户端时间同步 ##

客户端配置很简单，这里有个两种方式。

### 1.通过日期和时间修改功能修改 ###

点击桌面右下角显示时间的地方，单击`更改日期和时间设置`，会弹出`日期与时间`修改的对话框。单击`Internet 时间`选项卡，单击`更改设置`按钮，在输入框里输入NTP服务器的IP地址，单击`更新`就行了。默认是每周自动更新一次。想要修改更新频率，就需要采用第二种方法。

### 2.通过修改注册表修改 ###

#### 2.1 设定NTP服务器地址 ####

修改`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpClient\SpecialPollTimeRemaining`键值为`NTP服务器地址,0`。

#### 2.2 设置自动更新频率 ####

修改`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpClient\SpecialPollInterval`键值，修改为十进制`3600`。（单位为秒，我这里设置的是1小时，更新一次）
