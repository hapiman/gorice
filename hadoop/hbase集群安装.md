### 创建用户
```sh
useradd hbase
passwd hbase
```
### 修改`.bashrc`文件
```sh
export JAVA_HOME=/usr/local/java
export PATH=$PATH:$JAVA_HOME/bin
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
```
### 设置`supergroup`用户组
```sh
sudo groupadd supergroup
sudo groupmems -g supergroup -a hbase
```
### 安装hbase
```sh
wget http://archive.apache.org/dist/hbase/1.2.2/hbase-1.2.2-bin.tar.gz
tar -xvzf hbase-1.2.2-bin.tar.gz
mv hbase-1.2.2 hbase
sudo mv hbase /usr/local/
sudo chown -R hbase.hbase /usr/local/hbase
# 创建日志
sudo mkdir /data/logs/hbase
sudo chown -R hbase.hbase /data/logs/hbase
# 创建数据目录
sudo mkdir /data/hbase
sudo chown -R hbase.hbase  /data/hbase
```

### 将hadoop集群的`hdfs-site.xml`和`core-site.xml`放到`hbase/conf`

### 配置hbase-env.sh
```sh
# 设置不使用系统的`zookeer`,使用独立的
export HBASE_MANAGES_ZK=false
# 设置`hadoop`配置文件路径
export HBASE_CLASSPATH=/usr/local/hadoop/etc/hadoop
# 修改Hbase日志输出文件
export HBASE_LOG_DIR=/data/logs/hbase
```

### 配置hbase-site.xml
```xml
<configuration>
  <property>
    <!-- hbase存储根目录 -->
    <name>hbase.rootdir</name>
    <value>hdfs://mycluster/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/data/hbase/zookeeper</value>
  </property>
  <property>
    <!-- 分布式开关 -->
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <!-- zookeeper集群地址 -->
    <name>hbase.zookeeper.quorum</name>
    <value>fjx-ofckv-72-238,fjx-ofckv-72-237,fjx-ofckv-72-236</value>
  </property>
  <property>
    <name>hbase.table.sanity.checks</name>
    <value>false</value>
  </property>
</configuration>
```
**备注**: 将上面的两个文件分别同步到集群中每台机器上

### 手动启动·
```sh
# 启动Master
$HBASE_HOME/bin/hbase-daemon.sh start master
# 使用jps查看是否存在HMaster节点
jps
# 启动RegionServer
$HBASE_HOME/bin/hbase-daemon.sh start regionserver
# 使用jps查看是否存在HRegionServer节点
jps
```
使用ip:16010查看安装情况

### master主节点自启动
在`/etc/init.d`目录下执行`vim hbase-master`
```sh
#!/bin/bash
#chkconfig:2345 98 02
#description: hbase-master service
RETVAL=0
# Source hadoop enviroment variables
. /home/hbase/.bashrc
start() {
  su hbase -c "$HBASE_HOME/bin/hbase-daemon.sh start master"
}
stop() {
  su hbase -c "$HBASE_HOME/bin/hbase-daemon.sh stop master"
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
使用`root`用户, 执行`chmod +x hbase-master`, `chkconfig hbase-master on`
使用`sudo service hbase-master start/stop`来测试

### RegionServer自启动
在`/etc/init.d`目录下执行`vim hbase-regionserver`
```sh
#!/bin/bash
#chkconfig:2345 99 01
#description:hbase-regionserver service
RETVAL=0
# Source hadoop enviroment variables
. /home/hbase/.bashrc
start() {
  su hbase -c "$HBASE_HOME/bin/hbase-daemon.sh start regionserver"
}
stop() {
  su hbase -c "$HBASE_HOME/bin/hbase-daemon.sh stop regionserver"
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
使用`root`用户, 执行`chmod +x hbase-regionserver`, `chkconfig hbase-regionserver on`
使用`sudo service hbase-regionserver start/stop`来测试
