---
title: 设置默认map和reducer数
date: 2013-02-20 16:25:54
updated: 2013-02-20 16:25:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### 设置默认map和reducer数

在提交作业的客户端设置默认map和reducer数。

> #### 缘起

在用mapreducer处理数据的过程中碰到这样一个事，先把上下文说下.

提交作业命令是:
```
/opt/cloudiyaDataCluster/hadoop/bin/hadoop \

jar /opt/cloudiyaDataCluster/hadoop/contrib/streaming/hadoop-streaming-1.0.3.jar  -file /tmp/9527T3n/mrdata/ \

-mapper mapper.py  -reducer reducer.py -input hdfs://192.168.0.112:50081/9527T3n/inputfiles -output \ 

hdfs://192.168.0.112:50081/9527T3n/9527T3n  -numReduceTasks 1 
```

待处理数据目录是:
```
"hdfs://192.168.0.112:50081/9527T3n/inputfiles/"下只有一个video_1文件，该文件副本为4.大小小于1KB,块大小设置是128MB
```

在客户端开始提交作业后,在192.168.0.112:50072 web端和jobtracker日志都可观测到启用了3个map.

```
2013-02-20 12:48:56,864 INFO org.apache.hadoop.mapred.JobInProgress: job_201302191802_0016: nMaps=3 nReduces=1 max=-1
```

![](/images/设置默认map和reducer数1.jpg)

但实际上只需一个map就行了，video_1不到1kB，只能分一个块，

如果硬是启用3个map , 强行对video_1进行切分(如上图) ，则后面的两个map是读不到数据的.

> #### 解决

在mapred-site.xml下有参数mapred.map.tasks,

它指定的是"默认每个job所使用的map数，意思是假设设置dfs块大小为64M，需要排序一个60M的文件，也会开启2个map线程"

在mapreducer所有相关节点设置mapred.map.tasks=1，但不起作用。

尝试在提交作业的客户端修改 mapred.map.tasks=1，成功。


