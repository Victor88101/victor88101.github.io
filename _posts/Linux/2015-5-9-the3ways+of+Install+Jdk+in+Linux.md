---
layout: post
category: Linux
title: 《LInux安装jdk的三种方法》
tagline: by Vv
tags: Linux
---



#环境
***
<br/>

    Linux版本：CentOS 6.5、Ubuntu 12.04.5
    JDK版本：JDK 1.7

#目录
***
<br/>

    方法一：手动解压JDK的压缩包，然后设置环境变量
    方法二：用yum安装JDK
    方法三：用rpm安装JDK
    方法四：Ubuntu 上使用apt-get安装JDK



#内容
***
<br/>

##方法一：手动解压JDK的压缩包，然后设置环境变量

###1.在/usr/目录下创建java目录 

    [root@localhost ~]# mkdir/usr/java
    [root@localhost ~]# cd /usr/java

###2.下载，然后解压

    [root@localhost java]# curl -O http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.tar.gz 
    [root@localhost java]# tar -zxvf jdk-7u79-linux-x64.tar.gz

###3.设置环境变量

    [root@localhost java]# vi /etc/profile

添加如下内容：

    #set java environment
    JAVA_HOME=/usr/java/jdk1.7.0_79
    JRE_HOME=/usr/java/jdk1.7.0_79/jre
    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    export JAVA_HOME JRE_HOME CLASS_PATH PATH

让修改生效：

    [root@localhost java]# source /etc/profile

###4.验证

    [root@localhost java]# java -version
    java version "1.7.0_79"
    Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
    Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

##方法二：用yum安装JDK(CentOS)

###1.查看yum库中都有哪些jdk版本

    [root@localhost ~]# yum search java|grep jdk
    ldapjdk-javadoc.x86_64 : Javadoc for ldapjdk
    java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
    java-1.6.0-openjdk-demo.x86_64 : OpenJDK Demos
    java-1.6.0-openjdk-devel.x86_64 : OpenJDK Development Environment
    java-1.6.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
    java-1.6.0-openjdk-src.x86_64 : OpenJDK Source Bundle
    java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
    java-1.7.0-openjdk-demo.x86_64 : OpenJDK Demos
    java-1.7.0-openjdk-devel.x86_64 : OpenJDK Development Environment
    java-1.7.0-openjdk-javadoc.noarch : OpenJDK API Documentation
    java-1.7.0-openjdk-src.x86_64 : OpenJDK Source Bundle
    java-1.8.0-openjdk.x86_64 : OpenJDK Runtime Environment
    java-1.8.0-openjdk-demo.x86_64 : OpenJDK Demos
    java-1.8.0-openjdk-devel.x86_64 : OpenJDK Development Environment
    java-1.8.0-openjdk-headless.x86_64 : OpenJDK Runtime Environment
    java-1.8.0-openjdk-javadoc.noarch : OpenJDK API Documentation
    java-1.8.0-openjdk-src.x86_64 : OpenJDK Source Bundle
    ldapjdk.x86_64 : The Mozilla LDAP Java SDK


###2.选择版本，进行安装

我们这里安装1.7版本

    [root@localhost ~]# yum install java-1.7.0-openjdk

安装完之后，默认的安装目录是在: /usr/lib/jvm/java-1.7.0-openjdk-1.7.0.75.x86_64

3.设置环境变量

    [root@localhost ~]# vi /etc/profile

添加如下内容：

    #set java environment
    JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.75.x86_64
    JRE_HOME=$JAVA_HOME/jre
    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    export JAVA_HOME JRE_HOME CLASS_PATH PATH

让修改生效：
    
    [root@localhost java]# source /etc/profile

3.验证
   同上。

注：因为采用yum安装jdk，系统考虑到多版本的问题，会用alternatives进行版本控制。开始，相应版本的jdk安装在/usr/lib/jvm/之后，会在alternatives中注册，在/etc/alternatives目录下会产生一些链接到/usr/lib/jvm/中刚安装好的jdk版本。
在/usr/bin下面会有链接到/etc/alternatives的相应的文件。比如，/usr/bin下面会有一个链接文件java的映射关系如下：

    /usr/bin/java->/etc/alternatives/java
    /etc/alternatives/java-> /usr/lib/jvm/jre-1.7.0-openjdk.x86_64/bin/java

这样，java命令不用设置环境变量，就可以正常运行了。但如果对于tomcat或其他软件来说还是要设置环境变量。
同样，如果安装新的版本jdk，就会重新链接到最新安装的jdk版本。当然，也可以使用alternatives修改所要使用的版本。
具体方法可参照：《使用Linux的alternatives管理多版本的软件》

## 方法三：用rpm安装JDK ##

###1.下载 ###

    [hadoop@localhost ~]$ curl -O http://download.oracle.com/otn-pub/java/jdk/7u79-b15/jdk-7u79-linux-x64.rpm

### 2.使用rpm命令安装 ###

    [root@localhost  ~]# rpm -ivh jdk-7u79-linux-x64.rpm
    Preparing...   ########################################### [100%]
      1:jdk###########################################[100%]
    Unpacking JAR files...
       rt.jar...
       jsse.jar...
       charsets.jar...
       tools.jar...
       localedata.jar...
       jfxrt.jar...

### 3.设置环境变量 ###

    [root@localhost java]# vi /etc/profile

添加如下内容：

    #set java environment
    JAVA_HOME=/usr/java/jdk1.7.0_79
    JRE_HOME=/usr/java/jdk1.7.0_79/jre
    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    export JAVA_HOME JRE_HOME CLASS_PATH PATH

让修改生效：

    [root@localhost java]# source /etc/profile

### 4.验证 ###

    [root@localhost java]# java -version
    java version "1.7.0_79"
    Java(TM) SE Runtime Environment (build 1.7.0_79-b15)
    Java HotSpot(TM) 64-Bit Server VM (build 24.79-b02, mixed mode)

和yum安装类似，不用设置环境变量就可以，运行java命令。rpm安装方式默认会把jdk安装到/usr/java/jdk1.7.0_79，然后通过三层链接，链接到/usr/bin,具体链接如下：

    [root@localhost ~]# cd /bin
    [root@localhost bin]# ll|grep java
    lrwxrwxrwx. 1 root root 25 Mar 28 11:24 jar ->/usr/java/default/bin/jar
    lrwxrwxrwx. 1 root root 26 Mar 28 11:24 java -> /usr/java/default/bin/java
    lrwxrwxrwx. 1 root root 27 Mar 28 11:24 javac ->/usr/java/default/bin/javac
    lrwxrwxrwx. 1 root root 29 Mar 28 11:24 javadoc ->/usr/java/default/bin/javadoc
    lrwxrwxrwx. 1 root root 28 Mar 28 11:24 javaws ->/usr/java/default/bin/javaws
    lrwxrwxrwx. 1 root root 30 Mar 28 11:24 jcontrol ->/usr/java/default/bin/jcontrol
    [root@localhost bin]# cd /usr/java/
    [root@localhost java]# ll
    total 4
    lrwxrwxrwx. 1 root root   16 Mar 28 11:24 default-> /usr/java/latest
    drwxr-xr-x. 8 root root 4096 Mar 28 11:24 jdk1.7.0_79
    lrwxrwxrwx. 1 root root   21 Mar 28 11:24 latest -> /usr/java/jdk1.7.0_79
   
## 方法四：Ubuntu 上使用apt-get安装JDK ##

### 1.查看apt库都有哪些jdk版本 ###

    root@Itble:~# apt-cache search java|grep jdk
    default-jdk - Standard Java or Java compatible Development Kit
    default-jdk-doc - Standard Java or Java compatible Development Kit (documentation)
    gcj-4.6-jdk - gcj and classpath development tools for Java(TM)
    gcj-jdk - gcj and classpath development tools for Java(TM)
    openjdk-6-dbg - Java runtime based on OpenJDK (debugging symbols)
    openjdk-6-demo - Java runtime based on OpenJDK (demos and examples)
    openjdk-6-doc - OpenJDK Development Kit (JDK) documentation
    openjdk-6-jdk - OpenJDK Development Kit (JDK)
    openjdk-6-jre-lib - OpenJDK Java runtime (architecture independent libraries)
    openjdk-6-source - OpenJDK Development Kit (JDK) source files
    openjdk-7-dbg - Java runtime based on OpenJDK (debugging symbols)
    openjdk-7-demo - Java runtime based on OpenJDK (demos and examples)
    openjdk-7-doc - OpenJDK Development Kit (JDK) documentation
    openjdk-7-jdk - OpenJDK Development Kit (JDK)
    openjdk-7-source - OpenJDK Development Kit (JDK) source files
    uwsgi-plugin-jvm-openjdk-6 - Java plugin for uWSGI (OpenJDK 6)
    uwsgi-plugin-jwsgi-openjdk-6 - JWSGI plugin for uWSGI (OpenJDK 6)
    openjdk-6-jre - OpenJDK Java runtime, using Hotspot JIT
    openjdk-6-jre-headless - OpenJDK Java runtime, using Hotspot JIT (headless)
    openjdk-7-jre - OpenJDK Java runtime, using Hotspot JIT
    openjdk-7-jre-headless - OpenJDK Java runtime, using Hotspot JIT (headless)
    openjdk-7-jre-lib - OpenJDK Java runtime (architecture independent libraries)

### 2.选择版本进行安装 ###

    root@Itble:~# apt-get install openjdk-7-jdk

### 3.设置环境变量 ###
    
    root@Itble:~# vi /etc/profile

添加如下内容：

    #set java environment
    JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-amd64
    JRE_HOME=$JAVA_HOME/jre
    CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    export JAVA_HOME JRE_HOME CLASS_PATH PATH

让修改生效：

    root@Itble:~# source /etc/profile

### 4.验证 ###

    root@Itble:~# java -version
    java version "1.7.0_79"
    OpenJDK Runtime Environment (IcedTea 2.5.5) (7u79-2.5.5-0ubuntu0.12.04.1)
    OpenJDK 64-Bit Server VM (build 24.79-b02, mixed mode)

Ubuntu的apt-get安装方式和CentOS的yum安装方式很类似，这里就算作一种方法。
