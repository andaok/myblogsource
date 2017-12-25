---
title: Namenode Manual Failover(元数据备份采用nfs)
date: 2012-06-20 11:31:54
updated: 2012-06-20 11:31:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### Namenode Manual Failover(元数据备份采用nfs)

通过备份在nfs上的元数据恢复namenode.

> #### 前言

元数据备份通过NFS放在snn（c05.cloudiya.com）下，namenode故障手动切换到snn,snn升级为namenode,
新snn和datanode（c02.cloudiya.com）是一个节点。

> #### 恢复前hadoop结构

node     | ipaddrees     |    hostname
--- | --- | ---     
namenode | 192.168.0.112 | c01.cloudiya.com
snn      | 192.168.0.110 | c05.cloudiya.com
datanode | 192.168.0.111 | c02.cloudiya.com
datanode | 192.168.0.108 | c12.cloudiya.com
datanode | 192.168.0.109 | c03.cloudiya.com

> #### 恢复后hadoop结构

node     |   ipaddrees    |    hostname    
--- | --- | ---            
namenode | 192.168.0.110 | c05.cloudiya.com
datanode and snn | 192.168.0.111 | c02.cloudiya.com
datanode | 192.168.0.108 | c12.cloudiya.com
datanode | 192.168.0.109 | c03.cloudiya.com

> #### 分发新namenode的公钥到其它节点

分发做namenode备份的snn(c05.cloudiya.com)的公钥到其它节点，做到c05.cloudiya.com无密码自动登录其它节点。
在公钥分发好后 ，一定要测试下。

```
ssh c02.cloudiya.com
ssh c12.cloudiya.com
ssh c03.cloudiya.com
ssh c05.cloudiya.com
```

> #### 检查所有节点是否hadoop相关进程都已关闭

检查所有节点是否hadoop相关进程都已关闭,如没有关闭，请关闭之!

> ####  在新namenode节点（c05.cloudiya.com）修改配置
> ##### 修改core-site.xml

```
 fs.default.name  -->  hdfs://c05.cloudiya.com:50081
```

> ##### 修改hdfs-site.xml

```
dfs.name.dir  -->  原snn下NFS目录 /hadoop_meta_data
```

> ##### 修改mapred-site.xml

```
mapred.job.tracker -->  c05.cloudiya.com:50082
```

> ##### 修改masters文件指向新的snn --> c02.cloudiya.com

> ##### 在新namenode节点（c05.cloudiya.com）分发配置文件到其它节点

> ##### 启动hadoop
