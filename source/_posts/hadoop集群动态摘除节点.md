---
title: hadoop集群动态摘除节点
date: 2012-06-15 16:45:54
updated: 2012-06-15 17:49:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### hadoop集群动态摘除节点

- 将待解除节点的网络地址添加到exclude文件中，不更新include文件。

exclude文件只存在于namenode本地
```
hdfs-site.xml
<property>
<name>dfs.hosts.exclude</name>
<value>exclude</value>
</property>
mapred-site.xml
<property>
<name>mapred.hosts.exclude</name>
<value>exclude</value>
</property>
```

- 重启mapreduce集群，以终止在待解除节点上运行的tasktracker.

- hadoop dfsadmin -refreshNodes

- 转到hadoop ui界面，

查看待解除的节点的管理状态是否已经变为"decommission in progress",此时正在将待解除节点中的块复制到其它datanode中.

- 当待解除的datanode节点的状态变为"decommission"时，表示所有块都已经复制完毕,关闭已经解除的节点

- 从include文件中移除这些节点

- hadoop dfsadmin -refreshNodes

- 从slaves文件中移除节点
