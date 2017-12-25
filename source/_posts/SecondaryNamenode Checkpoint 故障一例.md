---
title: SecondaryNamenode Checkpoint 故障一例
date: 2012-06-20 09:18:54
updated: 2012-06-20 09:18:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### SecondaryNamenode Checkpoint 故障一例

> #### SecondaryNameNode Side 错误提示

```
2012-06-19 19:56:45,711 ERROR org.apache.hadoop.security.UserGroupInformation: PriviledgedActionException as:hadoop-user cause:java.net.ConnectException: Connection refused
2012-06-19 19:56:45,711 ERROR org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode: Exception in doCheckpoint: 
2012-06-19 19:56:45,712 ERROR org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode: java.net.ConnectException: Connection refused
```

> #### 解决办法

From Google
```
I am not sure what could be the exact issue but when configuring secondary NN to NN, you need to tell your SNN where the actual NN resides. Try adding - dfs.http.address on your secondary namenode machine having value as <NN:port> on hdfs-site.xml Port should be on which your NN url is opening - means your NN web browser http port.
```

你需要告诉snn,nn是在什么地方,在snn下hdfs-site.xml加入

```
<property>
<name>dfs.http.address</name>
<value>namenode ipaddress:50071</value>
</property>
```

上面只是指出了一个方面，还有一个地方, 你需要告诉snn，它自己的ip地址与http端口是多少,在snn下hdfs-site.xml加入

```
<property>
<name>dfs.secondary.http.address</name>
<value>snn ip address:50090</value>
</property>
```

> #### SecondaryNamenode do checkpoint的过程

namenode : 10.2.10.16

namendoe http port : 50071

secondarynamenode : cwapub008.cloudiya.com

secondarynamenode http port : 50090

```
2012-06-20 09:19:41,386 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: Number of transactions: 0 Total time for transactions(ms): 0Number of transactions batched in Syncs: 0 Number of syncs: 0 SyncTimes(ms): 0 
2012-06-20 09:19:41,507 INFO org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode: Downloaded file fsimage size 1234 bytes.
2012-06-20 09:19:41,509 INFO org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode: Downloaded file edits size 4 bytes.
2012-06-20 09:19:41,509 INFO org.apache.hadoop.hdfs.util.GSet: VM type       = 64-bit
2012-06-20 09:19:41,509 INFO org.apache.hadoop.hdfs.util.GSet: 2% max memory = 19.33375 MB
2012-06-20 09:19:41,509 INFO org.apache.hadoop.hdfs.util.GSet: capacity      = 2^21 = 2097152 entries
2012-06-20 09:19:41,509 INFO org.apache.hadoop.hdfs.util.GSet: recommended=2097152, actual=2097152
2012-06-20 09:19:41,513 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: fsOwner=hadoop-user
2012-06-20 09:19:41,514 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: supergroup=supergroup
2012-06-20 09:19:41,514 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: isPermissionEnabled=true
2012-06-20 09:19:41,514 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: dfs.block.invalidate.limit=100
2012-06-20 09:19:41,515 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
2012-06-20 09:19:41,515 INFO org.apache.hadoop.hdfs.server.namenode.NameNode: Caching file names occuring more than 10 times 
2012-06-20 09:19:41,525 INFO org.apache.hadoop.hdfs.server.common.Storage: Number of files = 11
2012-06-20 09:19:41,527 INFO org.apache.hadoop.hdfs.server.common.Storage: Number of files under construction = 0
2012-06-20 09:19:41,527 INFO org.apache.hadoop.hdfs.server.common.Storage: Edits file /hadoopdata/snn/current/edits of size 4 edits # 0 loaded in 0 seconds.
2012-06-20 09:19:41,527 INFO org.apache.hadoop.hdfs.server.namenode.FSNamesystem: Number of transactions: 0 Total time for transactions(ms): 0Number of transactions batched in Syncs: 0 Number of syncs: 0 SyncTimes(ms): 0 
2012-06-20 09:19:41,562 INFO org.apache.hadoop.hdfs.server.common.Storage: Image file of size 1234 saved in 0 seconds.
2012-06-20 09:19:41,739 INFO org.apache.hadoop.hdfs.server.common.Storage: Image file of size 1234 saved in 0 seconds.
2012-06-20 09:19:41,896 INFO org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode: Posted URL 10.2.10.16:50071putimage=1&port=50090&machine=cwapub008.cloudiya.com&token=-32:915239575:0:1340155168000:1340155108788
2012-06-20 09:19:42,056 INFO org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode: Checkpoint done. New Image Size: 1234
```