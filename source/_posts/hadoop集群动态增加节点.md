---
title: hadoop集群动态增加节点
date: 2012-07-19 11:09:54
updated: 2012-07-19 11:09:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### hadoop集群动态增加节点

- 将新节点的网络地址或域名加入include文件中

include文件只存在于namenode本地
```
hdfs-site.xml
<property>
<name>dfs.hosts</name>
<value>/usr/local/src/lanhadoop/hadoopv101/conf/include</value>
</property>
mapred-site.xml
<property>
<name>mapred.hosts</name>
<value>/usr/local/src/lanhadoop/hadoopv101/conf/include</value>
</property>
```

- 在 namenode下，运行以下指令，更新namenode经过审核的一系列datanode集合。

```
$hadoop  dfsadmin -refreshNodes
```

- 在namenode下，更新slaves文件，将新节点域名或IP加入，这样hadoop控制脚本将会将新节点包括在未来的操作之下。

- 在新节点启动datanode

```
$hadoop-daemon.sh start  datanode
```

如果新datanode的IP或域名没有加入include中，datanode将不会启动，在log你会看到如下日志

```
2012-05-08 16:11:28,018 ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: org.apache.hadoop.ipc.RemoteException: org.apache.hadoop.hdfs.server.protocol.DisallowedDatanodeException: 
Datanode denied communication with namenode: slaveone.hadoop.cloudiya.com:50010
```

- 在新节点启动tasktracker

```
$hadoop-daemon.sh  start  tasktracker
```

- 检查新的datanode和tasktracker是否启动成功。

注：
hadoop集群中的各节点都需要能互相ping通的 ， 
如果集群配置的用的是各节点的域名，那需在各节点设置正确的DNS或者在每节点/etc/hosts做正确配置。
如果你的集群配置都是采用的IP地址，那只要能互相ping通就好。

- 集群重新平衡

以免旧块都集中在几台旧datanode上，负载也就集中在那几台datanode了， 重平衡后 ，一些块移动到新的datanode,负载也就均衡了.

```
$/usr/local/src/lanhadoop/hadoopv101/bin/start-balancer.sh  -threshold 5  （默认是10%）
```

> #### hadoop集群均衡器

> ##### 负载均衡阈值

在负载均衡时，有个可选的参数threshold，它代表：
```
datanode的使用率（该节点上已使用的空间与空间容量之间的比率）与集群的使用率（集群中已使用的空间与集群的空间容量之间的比率）非常接近，差距不超过给定的阀值。
```

> ##### 均衡器工作过程

均衡器程序（balancer）程序是一个hadoop守护进程，它将块从忙碌的datanode移动到相对清闲的datanode，同时坚持副本放置策略，将副本分散到不同的机架，以降低数据损坏率，它不断移动块，直到集群达到平衡。

> ##### 复制数据带宽限制

在不同节点间复制复制数据的带宽是有限制的，默认值是1MB/s,可通过hdfs-site.xml进行调节。

```
<property>  　　
        <name>dfs.balance.bandwidthPerSec</name>  　　
        <value>1048576</value>  　　
        <description>  　　　　
        Specifies the maximum amount of bandwidth that each datanode  　　　　
        can utilize for the balancing purpose in term of  　　　　
        the number of bytes per second.  　　
        </description>  
</property>
```

