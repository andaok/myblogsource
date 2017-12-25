---
title: Namenode Manual Failover
date: 2012-07-6 09:18:54
updated: 2012-07-6 09:18:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### 前言

如果主节点挂掉了，硬盘数据需要时间恢复或者不能恢复了，现在又想立刻恢复HDFS，
这个时候就可以import checkpoint。步骤如下：

- 拿一台和原来机器一样的机器，包括配置和文件，一般来说最快的是拿你节点机器中的一台，立马能用（部分配置要改成NameNode的配置）
- 创建一个空的文件夹，该文件夹就是配置文件中dfs.name.dir所指向的文件夹。
- 拷贝你的secondary NameNode checkpoint出来的文件，到某个文件夹，该文件夹为fs.checkpoint.dir指向的文件夹
- 执行命令bin/hadoop namenode -importCheckpoint

这样NameNode会读取checkpoint文件，保存到dfs.name.dir。但是如果你的dfs.name.dir包含合法的fsimage，是会执行失败的。
因为NameNode会检查fs.checkpoint.dir目录下镜像的一致性，但是不会去改动它。 

以上是Namenode Manual Failover的依据,来自Google. 

以下是我实验的过程！

> #### 实验过程
> 
> ##### 旧结构

node | ipaddress
--- | ---
namenode | 10.2.10.16
snn | 10.2.10.17 
datanode | 10.2.10.18 
datanode | 10.2.10.19 
datanode | 10.2.10.20 

> ##### 新结构

node | ipaddress
--- | ---
namenode and datanode |  10.2.10.18 
snn | 10.2.10.17
datanode | 10.2.10.19 
datanode | 10.2.10.20 

> ##### 过程

> ###### (1)关闭相关进程

一般情况下，在namenode节点down后，其它节点相关进程仍在运行，
在failover前，需要把这些进程先关闭掉hadoop-user$ hadoop-daemon.sh stop datanode|tasktracker|secondarynamenode

> ###### (2)在新namenode 10.2.10.18上生成hadoopuser的公匙

```
hadoop-user$ ssh-keygen -t  rsa
```

> ###### (3)分发新namenode公匙到其它节点

```
hadoop-user$ ssh-copy-id  10.2.10.17
hadoop-user$ ssh-copy-id  10.2.10.18
hadoop-user$ ssh-copy-id  10.2.10.19
hadoop-user$ ssh-copy-id  10.2.10.20
```

> ###### (4)在新namenode 10.2.10.18上做如下动作

- 修改core-site.xml
```
fs.default.name  -->  hdfs://10.2.10.18:50081
```
- 依据dfs.name.dir和fs.checkpoint.dir指定的目录在10.2.10.18上创建相应目录
```
<property>
<name>fs.checkpoint.dir</name>
<value>/hadoopdata/snn</value>
</property>
<property>
<name>dfs.name.dir</name>
<value>/hadoopdata/dfs/name</value>
</property>
```
```
hadoop-user$mkdir -p  /hadoopdata/snn
hadoop-user$mkdir -p  /hadoopdata/dfs/name
```
- 修改mapred-site.xml
```
mapred.job.tracker -> 10.2.10.18:50082
```
- 修改hdfs-site.xml
```
dfs.http.address  ->  10.2.10.18:50071
dfs.secondary.http.address -> 10.2.10.17:50090
```
注：上面两参数对snn do checkpoint很重要。
- master, slaves, include文件在此都不需更改

- 将原snn 10.2.10.17下fs.checkpoint.dir 指定的目录/hadoopdata/snn的所有文件copy到10.2.10.18下的/hadoopdata/snn目录下.
```
hadoop-user$ scp -r hadoop-user@10.2.10.17:/hadoopdata/snn/*   /hadoopdata/snn
```
- 将新namenode 10.2.10.18下conf目录覆盖到其它节点
```
hadoop-user$ scp -r /opt/hadoop/hadoop/conf/*   hadoop-user@10.2.10.17:/opt/hadoop/hadoop/conf/
hadoop-user$ scp -r /opt/hadoop/hadoop/conf/*   hadoop-user@10.2.10.19:/opt/hadoop/hadoop/conf/
hadoop-user$ scp -r /opt/hadoop/hadoop/conf/*   hadoop-user@10.2.10.20:/opt/hadoop/hadoop/conf/
```
- 执行命令
```
hadoop-user$ /opt/hadoop/hadoop/bin/hadoop namenode -importCheckpoint
```
检查在/hadoopdata/dfs/name是否已经有了元数据信息，如有先关闭namenode进程，再start-all.sh启动整个集群