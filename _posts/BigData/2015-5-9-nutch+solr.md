---
layout: post
category: Search
title: 《CentOS 6.5+Nutch 1.7+Solr 4.7+IK 2012》笔记
tagline: by Vv
tags: Nutch Solr
---

# 环境 #
***
<br/>

	Linux版本：CentOS 6.5
	JDK版本：JDK 1.7
	Nutch版本：Nutch 1.7
	Solr版本：Solr 4.7
	IK版本：IK-Analyzer 2012

# 目录 #
***
<br/>

	1.安装JDK
	2.安装Solr
	3.为Solr配置IK分词
	4.安装Nutch


# 内容 #
***
<br/>

## 1.安装JDK ##

### 1.1 在/usr/下创建java/目录，下载JDK包并解压 ###

	[root@localhost ~]# mkdir /usr/java 
	[root@localhost ~]# cd /usr/java
	[root@localhost ~]# curl -O http://download.oracle.com/otn-pub/java/jdk/7u75-b13/jdk-7u75-linux-x64.tar.gz
	[root@localhost java]# tar –zxvf jdk-7u75-linux-x64.gz
	
### 1.2 设置环境变量 ###

	[root@localhost java]# vi /etc/profile

添加以下内容：

	#set JDK environment
	JAVA_HOME=/usr/java/jdk1.7.0_75
	JRE_HOME=$JAVA_HOME/jre
	CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
	PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
	export JAVA_HOME JRE_HOMECLASS_PATH PATH
	
使修改生效：

	[root@localhost java]# source /etc/profile 
	
### 1.3 验证 ###

	[root@localhost java# java -version
	java version "1.7.0_75"
	Java(TM) SE Runtime Environment (build 1.7.0_75-b13)
	Java HotSpot(TM) 64-Bit Server VM (build 24.75-b04, mixed mode)
	
## 2.安装Solr ##

### 2.1 在/usr/下创建solr目录，下载Solr安装包并解压 ###

	[root@localhost ~]# mkdir /usr/solr
	[root@localhost ~]# cd /usr/solr
	[root@localhost solr]# curl -O http://archive.apache.org/dist/lucene/solr/4.7.0/solr-4.7.0.tgz
	[root@localhost solr]# tar –zxvfsolr-4.7.0.tgz
	
### 2.2 启动Jetty ###

这里使用Solr自带的Jetty服务器

	[root@localhost solr]# cd solr-4.7.0/example
	[root@localhost example]# java -jar start.jar
	
### 2.3 验证 ###

在浏览器输入：`http://10.192.87.198:8983/solr#/collection1/query`

## 3.为Solr配置IK分词 ##

### 3.1 下载IK-Analyzer-2012 ###

解压之后，将`IKAnalyzer.cfg.xml、IKAnalyzer2012_FF.jar、stopword.dic`三个文件上传到`/usr/solr/solr-4.7.0/example/solr-webapp/webapp/WEB-INF/lib/`目录下

### 3.2 修改`/usr/solr/solr-4.7.0/example/solr/collection1/conf/schema.xml`配置文件 ###

	[root@localhost solr]# cd /usr/solr/solr-4.7.0/example/solr/collection1/conf/
	[root@localhost solr]# vi schema.xml
	
在`<type></types>`中增加如下内容：

	<fieldTypename="text_ik" class="solr.TextField">
	<analyzer type="index"isMaxWordLength="false"class="org.wltea.analyzer.lucene.IKAnalyzer"/>
	 <analyzer type="query"isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
	</fieldType>

### 3.3 验证 ###

重启Solr，打开`http://10.192.87.198:8983/solr/#/collection1/analysis`，测试一下：


分词结果：


## 4.安装Nutch ##

### 4.1 在`/usr/`下创建nutch目录，下载Nutch安装包并解压 ###

	[root@localhost ~]# mkdir /usr/nutch
	[root@localhost ~]# cd /usr/nutch
	[root@localhost nutch]# curl -O http://archive.apache.org/dist/nutch/1.7/apache-nutch-1.7-bin.tar.gz
	[root@localhost nutch]# tar –zxvf apache-nutch-1.7-bin.tar.gz

### 4.2 修改`nutch-site.xml`配置文件 ###

	[root@localhost nutch]# cd apache-nutch-1.7/conf
	[root@localhost conf]# vi nutch-site.xml

在`<configuration>..</configuration>`中添加字段，如下：

	<configuration>
	  <property>
	    <name>http.agent.name</name>
	    <value>Friendly Crawler</value>
	  </property>
	  <property>
	    <name>parser.skip.truncated</name>
	    <value>false</value>
	  </property>
	</configuration>
	
### 4.3 修改`regex-urlfilter.txt`文件，设置过滤规则 ###

	[root@localhost conf]# vi nutch-site.xml

这里是以正则表达式匹配你希望爬取的网站的地址。
如下面例子，用正则表达式来限制爬虫的范围仅限于sohu.com这个域
修改前：

	+.

修改后：

	+^http://([a-z0-9]*\.)*sohu.com

### 4.4 设定所要爬取的网站 ###

	[root@localhost conf]# cd /usr/nutch/apache-nutch-1.7
	[root@localhost apache-nutch-1.7]# mkdir urls
	[root@localhost apache-nutch-1.7]# echo "http://www.sohu.com">urls/seed.txt

### 4.5 执行命令，进行爬取 ###

	[root@localhost apache-nutch-1.7]# bin/nutch crawl urls -dir crawl -depth 2 -topN 5

使用tree查看`/usr/nutch/apache-nutch-1.7/crawl`目录

	[root@localhost apache-nutch-1.7]# tree crawl/
	crawl/
	├── crawldb
	│   ├── current
	│   │   └── part-00000
	│   │       ├── data
	│   │       └── index
	│   └── old
	│       └── part-00000
	│           ├── data
	│           └── index
	├── linkdb
	│   └── current
	│       └── part-00000
	│           ├── data
	│           └── index
	└── segments
	    ├── 20150326234924
	    │   ├── content
	    │   │   └── part-00000
	    │   │      ├── data
	    │   │      └── index
	    │   ├── crawl_fetch
	    │   │   └── part-00000
	    │   │      ├── data
	    │   │      └── index
	    │   ├── crawl_generate
	    │   │   └── part-00000
	    │   ├── crawl_parse
	    │   │   └── part-00000
	    │   ├── parse_data
	    │   │   └── part-00000
	    │   │      ├── data
	    │   │      └── index
	    │   └── parse_text
	    │      └── part-00000
	    │          ├── data
	    │          └── index
	    └── 20150326234933
	        ├── content
	        │   └── part-00000
	        │      ├── data
	        │      └── index
	        ├── crawl_fetch
	        │   └── part-00000
	        │      ├── data
	        │      └── index
	        ├── crawl_generate
	        │   └── part-00000
	        ├── crawl_parse
	        │   └── part-00000
	        ├── parse_data
	        │   └── part-00000
	        │      ├── data
	        │      └── index
	        └── parse_text
	            └── part-00000
	                ├── data
	                └── index

已经爬取到数据。

### 4.6 集成Solr ###

编辑/usr/solr/solr-4.7.0/example/solr/collection1/conf/schema.xml文件，在<field>…</fields>中增加如下字段：

	   <fieldname="host" type="string" stored="false"indexed="true"/>
	   <field name="digest"type="string" stored="true" indexed="false"/>
	   <field name="segment"type="string" stored="true" indexed="false"/>
	   <field name="boost"type="float" stored="true" indexed="false"/>
	   <field name="tstamp"type="date" stored="true" indexed="false"/>
	   <field name="anchor"type="string" stored="true" indexed="true" multiValued="true"/>
	   <fieldname="cache" type="string" stored="true"indexed="false"/>

重启Solr，重新爬取

	[root@localhost apache-nutch-1.7]# bin/nutch crawl urls -dir crawl -depth 2 -topN 5 -solr http://10.192.86.156:8983/solr

### 4.7 查看结果 ###

在浏览器输入`http://10.192.86.156:8983/solr#/collection1/query`，进行查询.

