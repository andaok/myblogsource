---
title: 通过thrift api框架访问hdfs
date: 2013-09-13 14:25:54
updated: 2013-09-13 14:25:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### 通过thrift api框架访问hdfs

可在任何客户端通过thrift框架管理hdfs上数据.

注: start_thrift_server.sh不一定要在namenode上运行,datanodes上皆运行.

以下操作均在namenode(192.168.0.112)下执行.

> #### 编译相关类 

```
cloudiyadatauser$ cd hadoop  && ant compile
cloudiyadatauser$ cd /opt/cloudiyaDataCluster/hadoop/src/contrib/thriftfs && ant compile && cd -
```

> #### 修改启动文件

```
cloudiyadatauser$ vi /opt/cloudiyaDataCluster/hadoop/src/contrib/thriftfs/scripts/start_thrift_server.sh
增加:
#export the hadoop configuration ,avoiding local filesystem default
CLASSPATH=$CLASSPATH:$TOP/conf

cloudiyadatauser$ chmod +x  /opt/cloudiyaDataCluster/hadoop/src/contrib/thriftfs/scripts/start_thrift_server.sh
```

> #### 启动HadoopThriftServer

```
cloudiyadatauser$ /usr/bin/nohup  /bin/sh  /opt/cloudiyaDataCluster/hadoop/src/contrib/thriftfs/scripts/ start_thrift_server.sh 9999 & 
```

> #### 检查是否启动成功

```
cloudiyadatauser$ /opt/cloudiyaDataCluster/jdk/bin/jps
28963 HadoopThriftServer
7958 Jps
24407 NameNode
24920 JobTracker
```

> #### 客户端设置

```
#mkdir  HDFSClient  && cd HDFSClient
#scp -r  192.168.0.112: /opt/cloudiyaDataCluster/hadoop/src/contrib/thriftfs/gen-py   ./
#scp  192.168.0.112:/opt/cloudiyaDataCluster/hadoop/src/contrib/thriftfs/scripts/hdfs.py  ./
```

操作HDFS数据示例:

```
import sys
sys.path.append("gen-py")
from hdfs import hadoopthrift_cli

host = "192.168.0.112"
port = "9999"

c = hadoopthrift_cli(host,port)
c.connect()
#c.do_ls("/l123YDT")

c.do_exists("/l123YDT")

c.do_put("/tmp/testfile.mp4  /l123YDT/testfile.mp4")

c.do_ls("/l123YDT")
```
