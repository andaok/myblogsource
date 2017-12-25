---
title: hadoop运行一段时间后,无法stop-all.sh停止
date: 2012-07-20 09:28:54
updated: 2012-07-20 09:28:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### hadoop运行一段时间后,无法stop-all.sh停止

在hadoop集群运行一段时间后 ，无法通过stop-all.sh关闭hadoop.

> #### From Google

```
今天发现一个问题，当hadoop集群运行一段时间以后，无法停止服务。执行stop-all的时候提示 no tasktracker to stop ，no  datanode to stop。
而当我把所有节点手动kill掉以后，执行start-all和stop-all均没有问题。在邮件群组里问 了一下，最后结论如下：

stop-all.sh会调用stop-mapred.sh和 stop-dfs.sh去停止jobtracker, tasktrackers; namenode, datanodes。

Jobtracker和namenode的停止是在本地通过调用hadoop-daemon完成的，而tasktracker,和datanode 的停止是通过调用hadoop-daemons来完成的。

Hadoop-daemon实质上是ssh到每一个slave去执行一个当地的hadoop- daemon命令，比如：hadoop-daemon stop datanoade。
Hadoop-daemon  stop command会通过kill -0 `cat command.pid` 来测试进程是否存在，如果这个测试中有错误产生，就会报”no command to stop ”

可能原因： pid 文件丢了，导致 hadoop-daemon.sh stop XXX 时找不到进程号。
解决办法：默认 pid 文件放在 /tmp 目录下，不太安全。可以在 conf/hadoop-env.sh 里设置 HADOOP_PID_DIR 环 境变量改变 pid文件的存放目录.
```

> #### 实际解决

在发现hadoop集群无法关闭后，可以在原放pid文件目录下手动新建各进程的pid文件,如:

```
-rw-rw-r-- 1 hadoop-user hadoop-user 6 May  5 13:12 hadoop-hadoop-user-jobtracker.pid
-rw-rw-r-- 1 hadoop-user hadoop-user 6 May  5 13:12 hadoop-hadoop-user-namenode.pid
-rw-rw-r-- 1 hadoop-user hadoop-user 6 May  5 13:12 hadoop-hadoop-user-secondarynamenode.pid
```

再通过jps程序查到现在正在运行的各进程的进程号.

```
13916 SecondaryNameNode
13739 NameNode
19243 Jps
14031 JobTracker
```

将进程号写入新建的pid文件，在集群各节点依次照做，最后stop-all.sh ,看是不是可以关闭了。
