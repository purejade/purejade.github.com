---
layout: post
title:  "Druid系列翻译与实践——Druid 集群搭建"
date:   2016-11-10 10:47:29 +0800
categories: druid
---

# Druid 集群搭建

Druid 是可部署为可扩展的容错群集。在本文档中，我们将安装一个简单的集群，并讨论如何进一步配置以满足您的需求。

集群中建议部署多台Historicals和MiddleManagers节点，它们主要负责数据处理，因此最好服务器能够提供较好的内存和硬件配置。Coordinator和Overlord节点可以混合部署，提供元数据和资源协同的功能，部署2~3台，在Brokers提供查询，也可以提供多台。

## 环境

Coordinator 和 Overlord：负责集群上元数据和资源协调

- 4 vCPUs
- 15 GB RAM
- 80 GB SSD storage

Historicals 和 MiddleManagers：负责数据处理

- 8 vCPUs
- 61 GB RAM
- 160 GB SSD storage

Brokers： 负责接收请求，以及汇总结果

- 8 vCPUs
- 61 GB RAM
- 160 GB SSD storage

java 1.7+

zookeeper

mysql5.7.16

hadoop

## 下载

```markdown
curl -O http://static.druid.io/artifacts/releases/druid-0.9.1.1-bin.tar.gz
tar -xzf druid-0.9.1.1-bin.tar.gz
cd druid-0.9.1.1
```



## 配置

在conf/druid/_common/common.runtime.properties中配置

```markdown
#扩展
druid.extensions.loadList=["mysql-metadata-storage","druid-hdfs-storage"] #mysql和hdfs

#zookeeper
druid.zk.service.host=hostIp  
druid.zk.paths.base=/druid

#mysql
druid.metadata.storage.type=mysql
druid.metadata.storage.connector.connectURI=jdbc:mysql://IP:PORT/druid
druid.metadata.storage.connector.user=root
druid.metadata.storage.connector.password=password

#deep storage
druid.storage.type=hdfs
druid.storage.storageDirectory=/druid/segments

# indexer
druid.indexer.logs.type=hdfs
druid.indexer.logs.directory=/druid/indexing-logs

#service name
druid.selectors.indexing.serviceName=druid/overlord
druid.selectors.coordinator.serviceName=druid/coordinator

#monitoring
druid.monitoring.monitors=["com.metamx.metrics.JvmMonitor"]
druid.emitter=logging
druid.emitter.logging.logLevel=info

注意：如果配置mysql extension,需要在官网下载mysql-metadata-storage，并解压放到extensions中。
如果将需要hadoop extension，则需要将hadoop配置文件core-site.xml/hdfs-site.xml/mapred-site.xml  yarn-site.xml拷贝到conf/druid/_common下,同时不要保持hadoop时间与查询时间一致，建议都采用UTC，则可以修改mapred-site.xml，将
	  <property>
         <name>mapreduce.map.java.opts</name>
          <value>-Xmx1G -Duser.timezone=UTC</value>
        </property>
      <property>
         <name>mapreduce.reduce.java.opts</name>
         <value>-Xmx1G -Duser.timezone=UTC</value>
      </property>
```



## 启动

```markdown
java `cat conf/druid/coordinator/jvm.config | xargs` -cp conf/druid/_common:conf/druid/coordinator:lib/* io.druid.cli.Main server coordinator
java `cat conf/druid/overlord/jvm.config | xargs` -cp conf/druid/_common:conf/druid/overlord:lib/* io.druid.cli.Main server overlord

java `cat conf/druid/historical/jvm.config | xargs` -cp conf/druid/_common:conf/druid/historical:lib/* io.druid.cli.Main server historical
java `cat conf/druid/middleManager/jvm.config | xargs` -cp conf/druid/_common:conf/druid/middleManager:lib/* io.druid.cli.Main server middleManager

如果您正在使用Kafka或通过HTTP进行基于推送的流获取，则还可以在部署MiddleManagers和Historicals的硬件上启动Tranquility Server。对于大规模生产，MiddleManagers和Tranquility Server仍然混部署。如果您使用流处理器运行Tranquility（而不是服务器），则可以将Tranquility与流处理器共部署，而不需要Tranquility Server。
curl -O http://static.druid.io/tranquility/releases/tranquility-distribution-0.8.0.tgz
tar -xzf tranquility-distribution-0.8.0.tgz
cd tranquility-distribution-0.8.0
bin/tranquility <server or kafka> -configFile <path_to_druid_distro>/conf/tranquility/<server or kafka>.json


java `cat conf/druid/broker/jvm.config | xargs` -cp conf/druid/_common:conf/druid/broker:lib/* io.druid.cli.Main server broker
```

## 加载数据

### 数据获取方法

Druid支持流式（实时）和基于文件（批量）摄取方法。最常用的配置是：

1. 文件 - 从HDFS，S3，本地文件或任何支持的Hadoop文件系统批量加载数据。如果您的数据集已经在文件中，我们建议使用此方法
2. 流推送： 使用Tranquility将数据流实时地推送到Druid，Tranquility是用于向Druid发送流的客户端库。如果您的数据集源是Kafka，Storm，Spark Streaming或您自己的流系统，我们建议使用此方法
3. 基于流的拉取 - 使用实时节点将数据流从外部数据源直接拉到Druid

## Batch Ingestion

1） 数据准备

```
{"time": "2015-09-01T00:00:00Z", "url": "/foo/bar", "user": "alice", "latencyMs": 32}
```

2）配置文件准备

```
{
  "type" : "index_hadoop",
  "spec" : {
   "ioConfig" : {
      "type" : "hadoop",
      "inputSpec" : {
        "type" : "static",
        "paths" : "/native-format.csv"
      }
    },
    "dataSchema" : {
      "dataSource" : "new_dwd_native",
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "HOUR",
        "queryGranularity" : "NONE",
        "intervals" : ["2016-09-31T00:00:00.000/2016-09-03T00:00:00.000"]
      },
      "parser" : {
        "type" : "string",
        "parseSpec" : {
          "format" : "tsv",
	  "delimiter":"\t",
          "columns" : [col1,col2,timestamp],
          "dimensionsSpec" : {
            "dimensions" : [
              col1,
              col2
            ]
          },
          "timestampSpec" : {
            "column" : "timestamp"
          }
        }
      },
      "metricsSpec" : [
        {
          "name" : "count",
          "type" : "count"
        },
        {
          "name" : "eventid_unique",
          "type" : "hyperUnique",
          "fieldName" : "col1"
        },
        {
          "name" : "user_unique",
          "type" : "hyperUnique",
          "fieldName" : "col2"
        }
      ]
    },
    "tuningConfig" : {
      "type" : "hadoop",
      "partitionsSpec" : {
        "type" : "hashed",
        "targetPartitionSize" : 5000000
      },
      "jobProperties" : {}
    }
  }
}
```

3）提交

```
curl -X 'POST' -H 'Content-Type:application/json' -d @my-index-task.json OVERLORD_IP:8090/druid/indexer/v1/task

返回{“task”："index_hadoop_2qianwan_dwd_native_2016-11-07T07:51:36.092Z"}
```



4) 检查状态 

```
curl OVERLORD_IP:8090/druid/indexer/v1/task/{taskId}/status
```



5) 常见错误

1）数据格式不正确 

```
7477e2d5b08031c4444ca09e50995571-2016-09T01:00:00.000Z
```

2）intervals区间不包含数据，要保证intervals时间戳和数据、hadoop执行都保持同一个时区，最好都是UTC，否则可能存在时间不匹配情况。

```
Error: com.metamx.common.ISE: outputPath[var/druid/hadoop-tmp/new_dwd_native/2016-11-07T083057.006Z/e27a2d3eeb3f472f8e04f2d627f51c26/20160830T170000.000Z_20160830T180000.000Z/partitions.json] must not exist.
```

3）hadoop/reduce 分配内存过小，导致执行失败

```
Current usage: 1.0 GB of 1 GB physical memory used; 2.5 GB of 4 GB virtual memory used. Killing container.
Container killed on request. Exit code is 143
Container exited with a non-zero exit code 143
```



## 可视化数据

Druid是面向高级用户的分析应用程序的理想选择。有许多种可视化开源应用程序来探索Druid中的数据。我们建议尝试Pivot，Caravel或Metabase，开始可视化摄取数据之旅。

Caravel 是 [airbnb](https://github.com/airbnb/superset)开源的可视化工具，利用python编写，支持[**SQLAlchemy**](https://www.google.co.jp/search?q=SQLAlchemy&spell=1&sa=X&ved=0ahUKEwik-L6tl53QAhXIh1QKHd3uD7wQvwUIGigA)连接各类数据库，如Druid或者Kylin等，可视化功能丰富，并且支持权限系统和Dashboard。https://github.com/airbnb/superset

Metabase 是[metabase](https://github.com/metabase)开源的可视化工具，利用[Clojure](http://www.oschina.net/news/50084/clojure-1-6)编写，也支持连接各类数据库，和Caravel功能类似。

[Imply](https://imply.io/)是Druid商业化版本，也提供免费版本试用，里面集成了Druid和Pivot可视化控件



> https://groups.google.com/forum/#!forum/druid-user