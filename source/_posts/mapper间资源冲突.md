---
title: mapper间资源冲突
date: 2013-02-20 16:25:54
updated: 2013-02-20 16:25:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### mapper间资源冲突

由同一作业启用的多个mapper在同一任务节点运行时，各个mapper在该节点使用的资源不能相冲突.

如在mapper.py里有这样一段程序：

```
f = open("/tmp/vid.txt","r")
f.read()
subprocess.call("rm -f /tmp/vid.txt",shell=True)
```

如果同一作业有两个mapper被分配到同一任务节点执行,

那前一个mapper在执行完后，删除了后一个mapper需要的资源"/tmp/vid.txt",那后面这个mapper就执行失败了.

所以对于每个mapper在任务节点使用的资源相互之间隔离开,互不影响.

> #### 示例

在mapper.py有这样一段程序：
```
 subprocess.call("%s fs -put /tmp/vid/240p.part1v.mp4 hdfs://video/vid/",shell=True)
```
某个mapper在执行失败后(但以上程序已执行)，会重新执行该mapper，当新的mapper执行到以上程序时会报错失败.



