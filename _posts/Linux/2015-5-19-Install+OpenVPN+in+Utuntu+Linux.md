---
layout: post
category: Linux
title: 《Ubuntu 12.04安装、配置OpenVPN》
tagline: by Vv
tags: Linux 
---



# 环境 #
***

    Linux版本：Ubuntu 12.04.5
    Windows版本：Win7 64bit
    OpenVpn服务器版本：OpenVPN 2.2.1 x86_64
    OpenVpn客户端版本：OpenVPN GUI V5 64bit

# 目录 #
***

    一、安装配置OpenVPN服务
    二、安装配置OpenVPN客户端
	三、配置OPenVPN使用账号、密码登陆
	四、配置允许客户端访问OpenVPN服务器内网

# 内容 #
***
<br/>

## 一、安装配置OpenVPN服务 ##

### 1.使用apt-get安装OpenVPN ###

    root@iZ28ynpxejlZ:~# apt-get install openvpn

### 2.创建rsa认证文件目录 ###

    root@iZ28ynpxejlZ:/# mkdir /etc/openvpn/easy-rsa/
    root@iZ28ynpxejlZ:/# cd /etc/openvpn/easy-rsa/
    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# cp -r /usr/share/doc/openvpn/examples/easy-rsa/2.0/* ./
    
### 3.生成认证文件 ###

    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# source vars 
    NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys
    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# ./clean-all 
    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# ./build-ca
    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# ./build-key-server ItbleOpenVPN
    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# ./build-key client1
	root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# ./build-key client2
    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# ./build-dh 
    
之后可以在/etc/openvpn/easy-rsa/keys/目录下看到生成的文件：

    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# ls keys/
    01.pem  client1.crt  client2.key index.txt.old serial.old
    02.pem  client1.csr  dh1024.pem  ItbleOpenVPN.crt
    03.pem  client1.key  index.txt   ItbleOpenVPN.csr
    ca.crt  client2.crt  index.txt.attr  ItbleOpenVPN.key
    ca.key  client2.csr  index.txt.attr.old  serial

### 4.修改服务器配置文件 ###

    root@iZ28ynpxejlZ:/etc/openvpn/easy-rsa# cd /etc/openvpn
    root@iZ28ynpxejlZ:/etc/openvpn# cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz ./
	root@iZ28ynpxejlZ:/etc/openvpn# gzip -d server.conf.gz 
	root@iZ28ynpxejlZ:/etc/openvpn# vi server.conf 

这里只把修改的地方列出来：

    # Which local IP address should OpenVPN
    # listen on? (optional)
	# 这个地方是服务器的地址
    local 114.211.211.211      

    # Any X509 key management system can be used.
    # OpenVPN can also use a PKCS #12 formatted key file
    # (see "pkcs12" directive in man page).
	# 这三个文件是前面生成的认证文件的位置
    ca /etc/openvpn/easy-rsa/keys/ca.crt
    cert /etc/openvpn/easy-rsa/keys/ItbleOpenVPNServer.crt
    key /etc/openvpn/easy-rsa/keys/ItbleOpenVPNServer.key  # This file should be kept secret
    
    # Diffie hellman parameters.
    # Generate your own with:
    #   openssl dhparam -out dh2048.pem 2048
	# 这个是前面生成的dh2048.pem文件的位置
    dh /etc/openvpn/easy-rsa/keys/dh2048.pem
    
    
    # Configure server mode and supply a VPN subnet
    # for OpenVPN to draw client addresses from.
    # The server will take 10.8.0.1 for itself,
    # the rest will be made available to clients.
    # Each client will be able to reach the server
    # on 10.8.0.1. Comment this line out if you are
    # ethernet bridging. See the man page for more info.
	# 这个是给客户端分配ip地址的范围
    server 10.8.0.0 255.255.255.0
    
    # Maintain a record of client <-> virtual IP address
    # associations in this file.  If OpenVPN goes down or
    # is restarted, reconnecting clients can be assigned
    # the same virtual IP address from the pool that was
    # previously assigned.
	# 服务器根据这个文件里面“客户端——ip”的对应关系，给客户端分配ip地址
    ifconfig-pool-persist ipp.txt
     
    # Uncomment this directive to allow different
    # clients to be able to "see" each other.
    # By default, clients will only see the server.
    # To force clients to only see the server, you
    # will also need to appropriately firewall the
    # server's TUN/TAP interface.
	# 设置客户端之间可以相互访问
    client-to-client


### 5.启动OpenVPN服务 ###

    root@iZ28ynpxejlZ:/etc/openvpn# service openvpn start
		* Starting virtual private network daemon(s)...
    	 *   Autostarting VPN 'server'


## 二、配置OpenVPN客户端 ##

### 1.Win7下配置 ###

#### 1.1 安装 ####

可以从这里下载：[openvpn-install-2.3.6-I601-x86_64.zip](http://bd.oyksoft.com/oyksoft/openvpn-install-2.3.6-I601-x86_64.zip)
然后，解压，一路Next。

> 需要注意的地方是，要使用管理员权限运行以及winXP兼容安装，就是在安装文件上右键，属性，兼容性，选中“以兼容模式运行这个程序（WinXP）”和“以管理员身份运行此程序”。

#### 1.2 下载认证文件 ####

把之前在OpenVPN服务器上生成的认证文件(`ca.crt、client1.crt、client1.key`)下载到openvpn客户端的配置文件目录下,我这里是：

    D:\Program Files\OpenVPN\config\

### 1.3 修改配置文件 ###

把`D:\Program Files\OpenVPN\sample-config\client.ovpn`文件，拷贝到
D:\Program Files\OpenVPN\config\下面。
修改如下：
    
    # The hostname/IP and port of the server.
    # You can have multiple remote entries
    # to load balance between the servers.
	# 所要连接的OpenVPN服务器地址
    remote 114.211.211.211 1194

    # SSL/TLS parms.
    # See the server config file for more
    # description.  It's best to use
    # a separate .crt/.key file pair
    # for each client.  A single ca
    # file can be used for all clients.
	# 设置所需认证文件
    ca ca.crt
    cert client1.crt
    key client1.key

这里修改了两个地方，一是OpenVPN服务器地址；另外一个是认证文件的名称。
好了，现在可以进行连接。

### 2.Ubuntu下配置 ###

#### 2.1 安装 ####

安装方式和服务器一样，这里就不再重复。

#### 2.2 下载认证文件 ####

把之前在OpenVPN服务器上生成的认证文件(`ca.crt、client1.crt、client1.key`)下载到/etc/openvpn/目录下。

#### 2.3 修改配置文件 ####

把`/usr/share/doc/openvpn/examples/sample-config-files/client.conf` 拷贝到`/etc/openvpn/`目录下，并修改。

    root@iZ28ynpxejlZ:~# cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/
	root@iZ28ynpxejlZ:~# vi /etc/openvpn/client.conf

修改内容同Win7.

#### 2.4 运行 ####

    root@iZ28ynpxejlZ:~# openvpn /etc/openvpn/client.conf

## 三、配置OPenVPN使用账号、密码登陆 ##

### 1.服务器配置 ###

这里是在前面配置的基础之上进行的修改。

#### 1.1 修改配置文件 ####

编辑`/etc/openvpn/server.cnf`

    root@iZ28ynpxejlZ:/etc/openvpn# vi server.conf 

添加如下内容：

    # 设置不需要cert文件认证
    client-cert-not-required
	# 设置通过用户名密码登陆
    username-as-common-name
	# 设置临时目录
    tmp-dir "/etc/openvpn/tmp/"
	# 设置登陆认证插件
    plugin /etc/openvpn/openvpn-auth-pam.so /etc/pam.d/openvpn

在`/etc/pam.d/`目录下创建文件`openvpn`

    root@iZ28ynpxejlZ:/etc/openvpn# vi /etc/pam.d/openvpn

内容如下：

	auth    required        pam_unix.so    shadow    nodelay
	auth    requisite       pam_succeed_if.so uid >= 500 quiet
	auth    requisite       pam_succeed_if.so user ingroup vpnusers quiet
	auth    required        pam_tally2.so deny=4 even_deny_root unlock_time=1200
	account required        pam_unix.so

> 说明：
> 
> 1.这里设置的`vpnusers`，就是指允许`vpnusers`用户组内的用户作为OpenVPN的登陆用户；
> 
> 2.这里的`deny=4`是指4次密码错误之后，用户就会被锁定，锁定的时间就是这里`unlock_time=1200`，即1200秒。`even_deny_root`这个是指限制范围包括`root`用户。
> 
> 3.查看和清除登录错误次数记录的命令：
> 
> 查看：`pam_tally2 --user`
> 
> 清除：`pam_tally2 --user username --reset`
 
#### 1.2 拷贝所需文件、创建临时目录 ####

    root@iZ28ynpxejlZ:/etc/openvpn# cp /usr/lib/openvpn/openvpn-auth-pam.so ./
    root@iZ28ynpxejlZ:/etc/openvpn# mkdir /etc/openvpn/tmp/
    root@iZ28ynpxejlZ:/etc/openvpn# chmod 777 /etc/openvpn/tmp/

#### 1.3 添加登录openvpn的用户 ####

    root@iZ28ynpxejlZ:/etc/openvpn# groupadd vpnusers
    root@iZ28ynpxejlZ:/etc/openvpn# useradd vpnuser1 -g vpnusers -s /sbin/nologin
    root@iZ28ynpxejlZ:/etc/openvpn# passwd vpnuser1
    Enter new UNIX password: 
    Retype new UNIX password: 
    passwd: password updated successfully

这里的`useradd vpnuser1 -g vpnusers -s /sbin/nologin`这条命令是指，添加一个`vpnusers`用户组的用户`vpnuser1`，同时设置不允许登陆系统。这样，`vpnuser1`这个用户只能作为OpenVPN专用用户进行登陆，而不能登录系统。

重启openvpn服务。

### 2.客户端配置 ###

客户端的配置比较简单，这里只需要修改client.cnf文件就可以了。

#### 2.1 修改配置文件 ####

这里把`cert、key`注释掉，然后添加`auth-user-pass
`，修改后如下：

    # SSL/TLS parms.
    # See the server config file for more
    # description.  It's best to use
    # a separate .crt/.key file pair
    # for each client.  A single ca
    # file can be used for all clients.
    ca ca.crt
    #cert client.crt
    #key client.key
	auth-user-pass

这里需要注意的是，`ca ca.crt`这个地方不能注释掉。同样，和这个相关的`D:\Program Files\OpenVPN\config\ca.crt`文件也要存在。
好了，重新连接，输入用户名、密码，ok。

注：这里有另外一个方法，不用设置`ca.crt`文件，只需要添加`<ca></ca>`标签，然后，把`ca.crt`里面的内容放到里面。完成之后的内容为：

    # SSL/TLS parms.
    # See the server config file for more
    # description.  It's best to use
    # a separate .crt/.key file pair
    # for each client.  A single ca
    # file can be used for all clients.
    # ca ca.crt
    # cert client.crt
    # key client.key
    <ca>
    -----BEGIN CERTIFICATE-----
    ****省略*****
    -----END CERTIFICATE-----
    </ca>

	auth-user-pass

## 四、配置允许客户端访问OpenVPN服务器内网 ##

配置允许客户端访问OpenVPN服务器内网只需要在服务器端进行配置。

### 1.修改配置`/etc/openvpn/server.cnf`文件 ###

修改如下配置：

    # Push routes to the client to allow it
    # to reach other private subnets behind
    # the server.  Remember that these
    # private subnets will also need
    # to know to route the OpenVPN client
    # address pool (10.8.0.0/255.255.255.0)
    # back to the OpenVPN server.
    # 这里是服务器所要转发的网络地址，我这里设置的是服务器所在内网的网络号
    push "route 10.164.27.0 255.255.255.0"

### 2.添加路由转发规则 ###

#### 2.1 开启路由转发功能 ####

	root@iZ28ynpxejlZ:~# echo "1">/proc/sys/net/ipv4/ip_forward    

#### 2.2 添加转发规则 ####

	root@iZ28ynpxejlZ:~# iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j SNAT --to-source 10.164.27.123
	root@iZ28ynpxejlZ:~# iptables-save

这里的`10.8.0.0/24`为前面设置的客户端IP地址范围，`10.164.27.123`为服务器内网IP地址。
之后，重启服务器就可以了。

