---
layout: post
title:  "MAC+IDEA搭建Hadoop开发环境"
date:   2016-06-30 18:47:29 +0800
categories: bigdata
---

### 环境配置 
MAC笔记本 + 8G内存 + Intel i5 + Hadoop2.7.2 + Intellij IDEA
(本来想要使用cloudera hadoop vm来配置的，这个至少4个处理器和8G内存)

### 下载和配置
首先在hadoop官网下载hadoop，这里使用的版本是2.7.2[Link][hadoop]
Hadoop配置分为单节点、伪分布式以及分布式，由于单机，这里主要介绍前两种。

<strong>英文安装</strong>[en-setup] <strong>中文版安装</strong>[cn-setup]

大概分为以下几步：

1. 配置hadoop-env.sh中JAVA_HOME

2. 可以将bin/*和sbin/*加入到环境变量（如~/.bash_profile, source ~/.bash_profile）

3. 如果伪分布，则需要配置core-site.xml 和  hdfs-site.xml 

4. 资源管理器YARN，则要配置yarn-site.xml 和 mapred-site.xml
	
### Hadoop配置文件
1.core-site.xml 主要配置了Hadoop系统中一些通用属性。 如临时目录(hadoop-tmp-dir)、默认路径(fs.defaultFs)、文件流缓冲大小(io.file.buffer-size)、计算校验和（io.bytes.per.checksum）、压缩编码(io.compression.codecs)

2.hdfs-*.xml 主要配置HDFS的属性

存放元数据(fsimage)的目录(dfs.namenode.name.dir)、存放元数据事务处理文件(edits文件)目录(difs.namenode.edits.dir)等 

3.mapred-site.xml 

主要配置Map和Reducer执行时环境等配置,以及YARN辅助配置； 

4.yarn-site.xml 

YARN守护进程的配置文件，主要负责资源管理等
	
	
### Intellij IDEA 开发WordCount统计
	
主要参考[maven-IEAD-HADOOP],先下载MAVEN，然后配置环境变量[maven+mac]
	
在mac上运行遇到的问题：Hadoop java.io.IOException: Mkdirs failed to create /some/path [参考stackoverflow][stackoverflow-1]

Answers：

	zip -d mahout-examples-0.6-cdh4.0.0-job.jar META-INF/LICENSE 
	
###MAVEN 配置

	<repositories>
        <repository>
            <id>apache</id>
            <name>apache</name>
            <url>http://maven.apache.org</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.7.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-core</artifactId>
            <version>1.2.1</version>
        </dependency>
    </dependencies>			
	
### 参考： 

参考深入理解Hadoop（原书第2版）解析 
	
[hadoop]: http://hadoop.apache.org/releases.html
[en-setup]: http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html
[cn-setup]: http://www.powerxing.com/install-hadoop/
[maven-IEAD-HADOOP]: http://chenbiaolong.com/2015/03/26/%E6%90%AD%E5%BB%BAHadoop%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/ 
[stackoverflow-1]: http://stackoverflow.com/questions/10522835/hadoop-java-io-ioexception-mkdirs-failed-to-create-some-path/11379938#11379938
[maven-mac]: http://www.jianshu.com/p/191685a33786
