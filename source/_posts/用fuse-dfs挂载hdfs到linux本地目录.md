> #### 用fuse-dfs挂载hdfs到linux本地目录
> ##### 需要的组件

组件 | 版本
---  | ---
os   | redhat 6.0
hadoop | 0.20.203.0
gcc | 4.4.5
jdk | 1.6.0.31
fuse | 2.8.3

其它：automake，autoconf，m4，libtool，pkgconfig，fuse，fuse-devel，fuse-libs。

> ##### 下载安装ant

```
#tar zxvf apache-ant-1.8.2-bin.tar.gz
ant位于/usr/local/src/apache-ant-1.8.2/
```

> ##### 环境变量

```
HADOOP_HOME="/usr/local/src/lanhadoop/hadoopinstall"
JAVA_HOME="/usr/local/src/jdk1.6.0_31"
export HADOOP_HOME JAVA_HOME
export OS_NAME=linux
export OS_ARCH=amd64
export OS_BIT=64
export LD_LIBRARY_PATH=$JAVA_HOME/jre/lib/amd64/server:$HADOOP_HOME/build/c++/Linux-amd64-64/lib:/usr/local/lib:/usr/lib
#export ANT_HOME=/usr/local/ant
export PATH=$PATH:$HADOOP_HOME/bin:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
#export FUSE_HOME=/usr/local/src/fuse-2.8.7
```

> ##### 编译libhdfs

进入hadoop安装路径运行以下命令

```
#/usr/local/src/apache-ant-1.8.2/bin/ant  compile-c++-libhdfs -Dlibhdfs=1 -Dcompile.c++=1
#ln -s c++/Linux-$OS_ARCH-$OS_BIT/lib build/libhdfs
```

这一步是编译libhdfs，因为libhdfs需要交叉编译，直接到src里面编译会报错，所以需要用ant编译。注意OS_ARCH和OS_BIT必须设置，否则会失败。

> ##### 编译fuse-dfs

hadoop 0.20.203.0版本中fuse存在一个bug，需要先修改掉才能继续编译。
(在hadoop 1.0.1下，不需再修改fuse_connect.c，此BUG已修复)

```
打开$HADOOP_HOME/src/contrib/fuse-dfs/src/fuse_connect.c

找到
hdfsFS fs = hdfsConnectAsUser(hostname, port, user, (const char **)groups, numgroups);

修改为
hdfsFS fs = hdfsConnectAsUser(hostname, port, user);

然后运行编译
#/root/apache-ant-1.8.2/bin/ant compile-contrib -Dlibhdfs=1 -Dfusedfs=1
```

如果编译失败，比较可能的原因是找不到libhdfs，请参看第一步的ln -s。

> ##### 环境配置

编辑/etc/fuse.conf，写入以下内容
```
user_allow_other
mount_max=100
```

编辑$HADOOP_HOME/build/contrib/fuse-dfs/fuse_dfs_wrapper.sh
```
if [ "$HADOOP_HOME" = "" ]; then
export HADOOP_HOME=/usr/local/src/lanhadoop/hadoopinstall
fi

export PATH=$HADOOP_HOME/build/contrib/fuse_dfs:$PATH

for f in ls $HADOOP_HOME/lib/*.jar $HADOOP_HOME/*.jar ; do
export  CLASSPATH=$CLASSPATH:$f
done

if [ "$OS_ARCH" = "" ]; then
export OS_ARCH=amd64
fi

if [ "$JAVA_HOME" = "" ]; then
export  JAVA_HOME=/usr/local/src/jdk1.6.0_31
fi

if [ "$LD_LIBRARY_PATH" = "" ]; then
export LD_LIBRARY_PATH=$JAVA_HOME/jre/lib/$OS_ARCH/server:/usr/local/share/hdfs/libhdfs/:/usr/local/lib
fi

fuse_dfs $@
```

> ##### mount

设置文件权限
```
chmod +x /data/soft/hadoop-2.20.1/build/contrib/fuse-dfs/fuse_dfs_wrapper.sh 
chmod +x /data/soft/hadoop-2.20.1/build/contrib/fuse-dfs/fuse_dfs 
ln -s /data/soft/hadoop-2.20.1/build/contrib/fuse-dfs/fuse_dfs_wrapper.sh /usr/local/bin 
ln -s /data/soft/hadoop-2.20.1/build/contrib/fuse-dfs/fuse_dfs /usr/local/bin/ 
mkdir /mnt/dfs
```

手动挂载hdfs文件系统
```
fuse_dfs_wrapper.sh dfs://192.168.1.11:54310 /mnt/dfs
```

开机自动挂载hdfs文件系统
```
vi /etc/fstab 
fuse_dfs_wrapper.sh dfs://192.168.1.11:54310 /mnt/dfs    fuse rw,auto 0 0
```

