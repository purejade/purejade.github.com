---
layout: post
title:  "小白用户Step By Step 安装Kylin全过程"
date:   2016-07-20 14:53:11 +0800
categories: big data
---

### 准备工作

#### 安装Hadoop、Hive、HBase、ZooKeeper、Mysql

<Strong>hadoop-2.7.2 </Strong>

安装和配置教程[hadoop-setup][hadoop-setup] 

启动start-dfs.sh和start-yarn.sh，用jps来查看是否启动了NodeManager/SecondaryNameNode/DataNode/ResourceManager/NameNode 

<Strong> apache-hive-1.2.1 安装</Strong>

官网： https://hive.apache.org/ 

需要注意，元数据默认保存在Derby中，仅支持“单用户”，不同用户之间不共享配置以及表信息；为了支持“多用户”，则需要将metadata保存在数据库中，可以用mysql来保存。

如何配置见[hive-metadata-config][hive-metadata-config]

hive mysql connector url: https://dev.mysql.com/downloads/file/?id=462849 

mysql配置中可能出现的问题 

http://blog.csdn.net/cjfeii/article/details/49363653 

hive表结构：http://lxw1234.com/archives/2015/07/378.html

在单机上执行，内存会爆满，hive无法启动，如果遇到这种情况，将hive 杀掉重新启动；（建议用服务器来运行集群）

<Strong> hbase-1.1.5 安装 </Strong>
http://abloz.com/hbase/book.html#d613e75  

<Strong> zookeeper-3.4.8 </Strong>

#### 设置环境变量

<pre>
# hadoop
export HADOOP_HOME=/Users/purejade/Desktop/big-data/hadoop-2.7.2
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

#hadoop_hive
export HIVE_HOME=/Users/purejade/Desktop/big-data/apache-hive-1.2.1-bin
export HCAT_HOME=$HIVE_HOME/hcatalog
export HIVE_CONF=$HIVE_HOME/conf
export PATH=$PATH:$HIVE_HOME/bin:$HIVE_HOME/sbin

#hbase
export HBASE_HOME=/Users/purejade/Desktop/big-data/hbase-1.1.5
export PATH=$PATH:$HBASE_HOME/bin

#kylin
export KYLIN_HOME=/Users/purejade/Desktop/big-data/apache-kylin-1.5.2.1-bin
export PATH=$PATH:$KYLIN_HOME/bin

export PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"

</pre> 


#### Kylin 安装

安装完毕HBase、Hadoop和Hive之后，下载和Hbase对应的Kylin版本，然后安装并配置。

主要配置HBase、Hadoop、Hbase、Hive以及Kylin的环境变量 

检查环境变量： sh check-env.sh  检查环境是否配置成功

然后:sh kylin.sh start/stop 来启动或者关闭kylin 

中间遇到的问题： 

    Failed to find metadata store by url: kylin_metadata@hbase  

后来发现，主要是hadoop和hbase的问题，虽然它们可以正常启动，但在查询表时会出问题，重启之后就可以了。

登陆http://hostname:7070/kylin 查看图形界面 

进一步了解：

hbase中保存着kylin的metadata数据，保存的位置在kylin.properties中配置，如 kylin.metadata.url、 kylin.storage.url， 如果需要备份和恢复，可以在bin下运行sh metastore.sh backup etc.

mac 上启动遇到的bug及解决方案参考如下链接：

http://stackoverflow.com/questions/36200523/apache-kylin-unable-to-find-hbase-common-lib  

http://www.yubai.me/index.php/IT/kylin.html

如何使用：

  如何配置database、model和cube，可以运行 sh sample.sh，得到例子，然后登陆
  http://<ip>:7070/kylin 进行查看

#### 建议

hadoop集群需要很大的内存，因此建议部署环境至少8G内存，否则在运行时可能出现内存不够用的情况，导致hive、hbase等卡死。

#### 参考资料

Kylin 官网 http://kylin.apache.org/ 

分布式大数据多维分析（OLAP）引擎Apache Kylin安装体验: http://www.itweet.cn/2016/07/03/kylin-install-use/?utm_source=tuicool&utm_medium=referral 

Kylin环境搭建和操作: http://www.cnblogs.com/shengshengwang/p/5477220.html

[hadoop-setup]: http://purejade.github.io/bigdata/hadoop-setup/
[hive-metadata-config]: http://blog.csdn.net/x_i_y_u_e/article/details/46845609 
