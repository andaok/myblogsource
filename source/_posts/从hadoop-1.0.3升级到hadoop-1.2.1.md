---
title: 从hadoop-1.0.3升级到hadoop-1.2.1
date: 2013-09-12 09:18:54
updated: 2013-09-12 09:18:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### 从hadoop-1.0.3升级到hadoop-1.2.1

  将生产环境下hadoop-1.0.3升级到hadoop-1.2.1,且数据不丢失.

> #### hadoop安装目录结构

```
[cloudiyadatauser@c01 opt]$ ls -al /opt/cloudiyaDataCluster/
drwxr-xr-x  18 cloudiyadatauser cloudiyadatauser 4096 Sep 12 17:07 hadoop
drwxr-xr-x  15 cloudiyadatauser cloudiyadatauser 4096 Sep 12 10:31 hadoop-1.2.1
drwxr-xr-x   8 cloudiyadatauser cloudiyadatauser 4096 May  3  2012 jdk
drwxrwxr-x   3 cloudiyadatauser cloudiyadatauser 4096 Sep 12 17:09 UpgradeBackup
```

以下所有操作需在namenode下，以hadoop管理员账号进行，我的管理员账号是cloudiyadatauser。

> #### 确保前一次升级已完成

```
 $/opt/cloudiyaDataCluster/hadoop/bin/hadoop dfsadmin -upgradeProgress status
 There are no upgrades in progress
```

> #### 停止hadoop

在升级过程中停止hadoop, 需确保各节点相关进程都已关闭.
```
$/opt/cloudiyaDataCluster/hadoop/bin/stop-all.sh
```

> #### 备份数据

```
$hadoop fs -lsr /  > UpgradeBackup/dfs-v-old-lsr-1.log
$hadoop fsck / > UpgradeBackup/dfs-v-old-fsck-1.log
$hadoop dfsadmin -report > UpgradeBackup/dfs-v-old-report-1.log
$ cp -r ${dfs.name.dir}  UpgradeBackup/
```

> #### 更改目录

```
$mv  hadoop  hadoop-1.0.3
$mv  hadoop-1.2.1/conf   hadoop-1.2.1/conf.old
$cp -r hadoop-1.0.3/conf  hadoop-1.2.1/
$mv hadoop-1.2.1 hadoop
```

各节点做同样操作，简单方法是在namenode配置好后, scp -r 到各个节点。

> #### 启动升级

```
$/opt/cloudiyaDataCluster/hadoop/bin/start-dfs.sh -upgrade
```

> #### 查看升级状态

```
$/opt/cloudiyaDataCluster/hadoop/bin/hadoop dfsadmin -upgradeProgress status
Upgrade for version -41 has been completed.
Upgrade is not finalized.
```

升级完成!

> #### 后记

当hadoop升级完成后,hadoop依旧保留着旧版本的有关信息，以便你可以方便的对hdfs进行降级操作.

```
$/opt/cloudiyaDataCluster/hadoop/bin/start-dfs.sh -rollback
```

升级完成,Hadoop一次只保存一个版本的备份,当新版本运行几天以后还是没有出现什么问题，你就可以使用运行一段时间后,没有问题再执行升级终结操作  

```
bin/hadoop dfsadmin -finalizeUpgrade
```

该命令把旧版本的备份从系统中删掉了。删除以后rollback 命令就失效了。

> #### 参考

```
http://www.cnblogs.com/ggjucheng/archive/2012/04/22/2465649.html
http://blog.pureisle.net/archives/1845.html
http://trac.nchc.org.tw/cloud/wiki/0428Hadoop_Lab4
```