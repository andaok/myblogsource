---
title: hadoop基本安装
date: 2012-07-02 17:15:54
updated: 2012-07-02 17:15:54
tags: [hadoop,hdfs,大数据,mapreduce]
categories: hadoop
---

> #### 前言

hadoop分三种运行模式 单机,伪分布,全分布,单机和伪分布是用于开发和调试，全分布是真正的hadoop集群，用于生产环境。

本次安装做全分布模式，包括master节点,snn节点和2个slave节点。

> #### 环境

id | ipaddress | hostname
--- | --- | --- 
master | 192.168.0.111 | c02.cloudiya.com
snn    | 192.168.0.107 | snn.hadoop.cloudiya.com
slave one | 192.168.0.103 | slaveone.hadoop.cloudiya.com
slave two | 192.168.0.105 | slavetwo.hadoop.cloudiya.com

> #### 设置各节点/etc/hosts

```
192.168.0.103 slaveone.hadoop.cloudiya.com
192.168.0.105 slavetwo.hadoop.cloudiya.com
192.168.0.111 c02.cloudiya.com
192.168.0.107 snn.hadoop.cloudiya.com
```

> #### 节点间安全通信设置

a. 在各节点建立相同账户hadoop-user,记得设置密码。

b. 生成ssh密钥对 
```
[hadoop-user@c02 ~]$ ssh-keygen -t rsa  (设置passphrase为空)
```

> #### 分布公钥

```
[hadoop-user@c02 ~]$ scp ~/.ssh/id_rsa.pub  hadoop-user@slaveone.hadoop.cloudiya.com:~/master_key
[hadoop-user@c02 ~]$ scp ~/.ssh/id_rsa.pub  hadoop-user@slavetwo.hadoop.cloudiya.com:~/master_key
[hadoop-user@c02 ~]$ scp ~/.ssh/id_rsa.pub  hadoop-user@snn.hadoop.cloudiya.com:~/master_key
[hadoop-user@c02 ~]$ cp  ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys 
```

> #### 设置master的密钥是授权密钥,slavetwo,master,snn需做同样操作

```
[hadoop-user@slaveone ~]$ mkdir ~/.ssh
[hadoop-user@slaveone ~]$ chmod 700 ~/.ssh
[hadoop-user@slaveone ~]$ mv ~/master_key  ~/.ssh/authorized_keys
[hadoop-user@slaveone ~]$ chmod 600 ~/.ssh/authorized_keys 
```

> #### 测试登录

```
[hadoop-user@c02 src]$ ssh 192.168.0.103
[hadoop-user@c02 src]$ ssh 192.168.0.105
[hadoop-user@c02 src]$ ssh 192.168.0.111
[hadoop-user@c02 src]$ ssh 192.168.0.107
```

> #### 在所有节点安装jdk

```
wget http://download.oracle.com/otn-pub/java/jdk/7u3-b04/jdk-7u3-linux-x64.tar.gz
jdk安装于：/usr/local/src/jdk/
```

> #### 在master安装hadoop,在hadoop-user下进行

```
wget http://mirror.bjtu.edu.cn/apache/hadoop/core/hadoop-0.20.2/hadoop-0.20.2.tar.gz
安装hadoop安装于：/usr/local/src/hadoop/
```

> #### 配置hadoop

a. 在hadoop-env.sh添加环境变量

```
export JAVA_HOME=/usr/local/src/jdk
export HADOOP_HOME=/usr/local/src/hadoop
```

b. 修改core-site.xml,指出hdfs文件系统namenode所在。

```
<configuration>
<property>
<name>fs.default.name</name>
<value>hdfs://c02.cloudiya.com:9000</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/home/hadoop/tmp</value>
</property>
</configuration>
```

c. 修改mapred-site.xml,指出jobtracker所在主节点

```
<configuration>
<property>
<name>mapred.job.tracker</name>
<value>c02.cloudiya.com:9001</value>
</property>
</configuration>
```

d. 修改hdfs-site.xml,指定hdfs为文件块保存的副本数.

```
<configuration>
<property>
<name>dfs.replication</name>
<value>2</value>
</property>
<property>
<name>dfs.data.dir</name>
<value>/home/hadoop/dfs/data</value>
</property>
<property>
<name>dfs.name.dir</name>
<value>/home/hadoop/dfs/name</value>
</property>
<property>
<name>dfs.datanode.du.reserved</name>
<value>1073741824</value>
</property>
</configuration>
```

e. 在master中添加snn所在

```
snn.hadoop.cloudiya.com
```

f. 在slaves中添加slave节点所在

```
slaveone.hadoop.cloudiya.com
slavetwo.hadoop.cloudiya.com
```

> #### 推送master下hadoop到snn,slaveone,slavetwo

先在snn,slaveone,slavetwo下相应目录建hadoop目录，chown给hadoop-user用户

```
[hadoop-user@c02 ~]$ scp -r /usr/local/src/hadoop/*  slaveone.hadoop.cloudiya.com:/usr/local/src/hadoop
[hadoop-user@c02 ~]$ scp -r /usr/local/src/hadoop/*  slavetwo.hadoop.cloudiya.com:/usr/local/src/hadoop
[hadoop-user@c02 ~]$ scp -r /usr/local/src/hadoop/*  snn.hadoop.cloudiya.com:/usr/local/src/hadoop
```

推送安装文件到各节点.

> #### 在各节点设置hadoop数据相关目录

```
mkdir /home/hadoop && chown hadoop-user:hadoop-user /home/hadoop
```

> #### 格式化hdfs

```
[hadoop-user@c02 ~]$ /usr/local/src/hadoop/bin/hadoop namenode -format
```

> #### 启动hadoop

```
[hadoop-user@c02 ~]$ /usr/local/src/hadoop/bin/start-all.sh 
```

> #### 启动信息

```
[hadoop-user@c02 src]$ /usr/local/src/hadoop/bin/start-all.sh 
starting namenode, logging to /usr/local/src/hadoop/logs/hadoop-hadoop-user-namenode-c02.cloudiya.com.out
slaveone.hadoop.cloudiya.com: starting datanode, logging to /usr/local/src/hadoop/logs/hadoop-hadoop-user-datanode-slaveone.hadoop.cloudiya.com.out
slavetwo.hadoop.cloudiya.com: starting datanode, logging to /usr/local/src/hadoop/logs/hadoop-hadoop-user-datanode-slavetwo.hadoop.cloudiya.com.out
snn.hadoop.cloudiya.com: starting secondarynamenode, logging to /usr/local/src/hadoop/logs/hadoop-hadoop-user-secondarynamenode-snn.hadoop.cloudiya.com.out
starting jobtracker, logging to /usr/local/src/hadoop/logs/hadoop-hadoop-user-jobtracker-c02.cloudiya.com.out
slaveone.hadoop.cloudiya.com: starting tasktracker, logging to /usr/local/src/hadoop/logs/hadoop-hadoop-user-tasktracker-slaveone.hadoop.cloudiya.com.out
slavetwo.hadoop.cloudiya.com: starting tasktracker, logging to /usr/local/src/hadoop/logs/hadoop-hadoop-user-tasktracker-slavetwo.hadoop.cloudiya.com.out
```

> #### 验证hadoop是否启动 

```
master
[hadoop-user@c02 ~]$ /usr/local/src/jdk/bin/jps
2135 Jps
28957 JobTracker
28805 NameNode
   
slaveone
[hadoop-user@slaveone logs]$ /usr/local/src/jdk/bin/jps
2067 DataNode
2262 Jps
2152 TaskTracker

slavetwo
[hadoop-user@slavetwo logs]$ /usr/local/src/jdk/bin/jps
1791 TaskTracker
2334 Jps
1706 DataNode

snn
[hadoop-user@snn logs]$ /usr/local/src/jdk/bin/jps
1735 Jps
1676 SecondaryNameNode
```

> #### web ui

```
hdfs
http://c02.cloudiya.com:50070/
jobtraker
http://c02.cloudiya.com:50030/
```