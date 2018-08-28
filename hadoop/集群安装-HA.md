
### HA设置
```sh
$HADOOP_PREFIX/sbin/hadoop-daemon.sh start journalnode
```

### 手动配置failover
先停止所有的`hadoop`进程, 在`hdfs-site.xml`中配置
```xml
<configuration>
  <property>
    <!-- 设置集群编号-->
    <name>dfs.nameservices</name>
    <value>mycluster</value>
  </property>
  <property>
   <!-- 设置namenode列表-->
    <name>dfs.ha.namenodes.mycluster</name>
    <value>fjr-ofckv-72-238,fjr-ofckv-72-237</value>
  </property>
  <property>
    <!-- 设置nodenode的rpc访问地址 -->
    <name>dfs.namenode.rpc-address.mycluster.fjr-ofckv-72-238</name>
    <value>fjr-ofckv-72-238:8020</value>
  </property>
  <property>
    <name>dfs.namenode.rpc-address.mycluster.fjr-ofckv-72-237</name>
    <value>fjr-ofckv-72-237:8020</value>
  </property>
  <property>
    <!-- 设置namenode的http访问地址 -->
    <name>dfs.namenode.http-address.mycluster.fjr-ofckv-72-238</name>
    <value>fjr-ofckv-72-238:50070</value>
  </property>
  <property>
    <name>dfs.namenode.http-address.mycluster.fjr-ofckv-72-237</name>
    <value>fjr-ofckv-72-237:50070</value>
  </property>
  <property>
    <!-- 设置journalnode集群访问地址 -->
    <name>dfs.namenode.shared.edits.dir</name>
    <value>qjournal://fjr-ofckv-72-238:8485;fjr-ofckv-72-237:8485;fjr-ofckv-72-236:8485/mycluster</value>
  </property>
  <property>
    <!-- 配置dns客户端 -->
    <name>dfs.client.failover.proxy.provider.mycluster</name>
    <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
  </property>
  <property>
    <!-- 配置Kill掉namenode的方式 1-->
    <name>dfs.ha.fencing.methods</name>
    <value>sshfence</value>
  </property>
  <property>
    <!-- 配置Kill掉namenode的方式 2-->
    <name>dfs.ha.fencing.ssh.private-key-files</name>
    <value>/home/hadoop/.ssh/id_rsa</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///data/hadoop/hdfs/nn</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///data/hadoop/hdfs/dn</value>
  </property>
  <property>
    <!-- 配置journalnode数据文件夹位置 -->
    <name>dfs.journalnode.edits.dir</name>
    <value>/data/hadoop/hdfs/jn</value>
  </property>
  <property>
    <!-- 添加自动failover -->
    <name>dfs.ha.automatic-failover.enabled</name>
    <value>true</value>
  </property>
</configuration>
```

在`core-site.xml`中配置
```xml
<configuration>
  <property>
    <!-- 配置集群ID -->
    <name>fs.defaultFS</name>
    <value>hdfs://mycluster</value>
  </property>
  <property>
    <!-- zk列表 -->
    <name>ha.zookeeper.quorum</name>
    <value>fjr-ofckv-72-238:2181,fjr-ofckv-72-237:2181,fjr-ofckv-72-236:2181</value>
  </property>
  <property>
    <name>ha.zookeeper.session-timeout.ms</name>
    <value>30000</value>
  </property>
</configuration>
```
```sh
# 创建文件目录
su - hadoop
mkdir /data/hadoop/hdfs/jn
chown hadoop.hadoop /data/hadoop/hdfs/jn

# 启动
$HADOOP_PREFIX/sbin/hadoop-daemon.sh start journalnode
# 查看JournalNode进程
jps

```
### zk初始化
初始化集群, 在任一台能访问到zk的机器上执行
```sh
/usr/local/hadoop/bin/hdfs zkfc -formatZK
```
执行完成之后使用`$ZOOKEEPER_HOME/bin/zkCli.sh`查看集群的目录情况:多出来一个`hadoop-ha`即可
```sh
[zk: localhost:2181(CONNECTED) 1] ls /
[zookeeper, hadoop-ha]
```
### 使用zkfc控制集群状态
如何控制`hadoop`的`active`和`standby`状态切换, 使用了`hadoop`的`zkfc`进程, 在两个`namenode`之上执行
```sh
/usr/local/hadoop/sbin/hadoop-daemon.sh --script /usr/local/hadoop/hdfs start zkfc
```
使用jps查看进程, 关注`DFSZKFailoverController`

如果不启动进程, 两个`namenode`都将保存`standby`状态



### 配置journalnode开机启动
切换到`root`账户,进入`/etc/init.d`目录下, 新建文件`hadoop-journalnode`
```sh
#!/bin/bash
#chkconfig:2345  81 09
#description: hadoop-namenode service
RETVAL=0
# Source hadoop enviroment variables
. /home/hadoop/.bashrc
start() {
  su hadoop -c "$HADOOP_PREFIX/sbin/hadoop-daemon.sh start journalnode"
}
stop() {
  su hadoop -c "$HADOOP_PREFIX/sbin/hadoop-daemon.sh stop journalnode"
}
case $1 in
start)
  start
;;
stop)
  stop
;;
esac
exit $RETVAL
```
设置为可执行: `chmod +x /etc/init.d/hadoop-journalnode`
添加到自启动: `chkconfig hadoop-journalnode on`
检测: `service hadoop-journalnode start/stop`

### 配置namenode开机启动
切换到`root`账户,进入`/etc/init.d`目录下, 新建文件`hadoop-namenode`
```sh
#!/bin/bash
#chkconfig:2345 82 08
#description: hadoop-namenode service
RETVAL=0
# Source hadoop enviroment variables
. /home/hadoop/.bashrc
start() {
  su hadoop -c "$HADOOP_PREFIX/sbin/hadoop-daemon.sh --script hdfs start namenode"
}
stop() {
  su hadoop -c "$HADOOP_PREFIX/sbin/hadoop-daemon.sh --script hdfs stop namenode"
}
case $1 in
start)
  start
;;
stop)
  stop
;;
esac
exit $RETVAL
```
设置为可执行: `chmod +x /etc/init.d/hadoop-namenode`
添加到自启动: `chkconfig hadoop-namenode on`
检测: `service hadoop-namenode start`·

### 配置datanode开机启动

切换到`root`账户,进入`/etc/init.d`目录下, 新建文件`hadoop-datanode`
```sh
#!/bin/bash
#chkconfig:2345 84 06
#description: hadoop-datanode service
RETVAL=0
# Source hadoop enviroment variables
. /home/hadoop/.bashrc
start() {
  su hadoop -c "$HADOOP_PREFIX/sbin/hadoop-daemon.sh --script hdfs start datanode"
}
stop() {
  su hadoop -c "$HADOOP_PREFIX/sbin/hadoop-daemon.sh --script hdfs stop datanode"
}
case $1 in
start)
  start
;;
stop)
  stop
;;
esac
exit $RETVAL
```
设置为可执行: `chmod +x /etc/init.d/hadoop-datanode`
添加到自启动: `chkconfig hadoop-datanode on`
检测: `service hadoop-datanode start`


### 遇到的问题

1. 启动namenode的时候发现不能启动, 查看了log日志 ` ERROR org.apache.hadoop.hdfs.server.namenode.EditLogInputStream: Got error reading edit log input stream http://fjr-ofckv-72-236:8480/getJournal?jid=mycluster&segmentTxId=1&storageInfo=-60%3A2130050496%3A0%3ACID-f77b10ed-4f80-4ad6-a9f1-1166fd67bcfc; failing over to edit log http://fjr-ofckv-72-237:8480/getJournal?jid=mycluster&segmentTxId=1&storageInfo=-60%3A2130050496%3A0%3ACID-f77b10ed-4f80-4ad6-a9f1-1166fd67bcfc`

解决方法: 在`namenode`的机器上执行`hdfs namenode -bootstrapStandby`
