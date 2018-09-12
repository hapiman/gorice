
### 配置用户名
```
useradd hadoop && passwd hadoop
# 登录密钥
# 添加hadoop到sudoers中
chmod u+w /etc/sudoers
# 在/etc/sudoers文件尾添加代码
hadoop ALL=NOPASSWD:ALL
# 使用sudo whoami
```

### 生成rsa密钥
```sh
# 一路绿灯就行
cd
ssh-keygen -t rsa
# 在不同的服务器上生成之后,
chmod 700 ~/.ssh && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && chmod 600 .ssh/authorized_keys
# 确保每一台机器上的authorized_keys保持一致, 不通机器之间都可以通过ssh互通
```

### 安装java
使用`hadoop`用户安装`java`

文件地址: [jdk-8u181-linux-x64.tar.gz](http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz?AuthParam=1536580563_bd197c67d1ce939bb53030d979b2c591)

安装过程:
```
tar -xvzf  jdk-8u181-linux-x64.tar.gz
mv jdk1.8.0_181 java
sudo mv java /usr/local
```

设置环境变量
```sh
cd
vim .bashrc

export JAVA_HOME=/usr/local/java
export JRE_HOME=/usr/local/java/jre
export CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

### 单机安装hadoop

1.安装

```sh
tar -zvxf hadoop-2.6.4.tar.gz
mv hadoop-2.6.4 hadoop
chown hadoop.hadoop hadoop
sudo mv hadoop /usr/local/
```
2.在.bashrc中添加环境变量
```sh
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_PREFIX=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_INSTALL=$HADOOP_HOME
```
3.修改**hadoop-env.sh**
```sh
# 设置java home的目录
export JAVA_HOME=/usr/local/java
# 设置日志的目录
# 在命令行中 sudo mkdir -p /data/logs/hadoop
# 在命令行中 sudo chown hadoop.hadoop /data/logs/hadoop
export HADOOP_LOG_DIR=/data/logs/hadoop

```
4.配置**core-site.xml**
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://fjx-ofckv-72-238:8020</value>
  </property>
</configuration>
```
5.配置**hdfs-site.xml**
```xml
<configuion>
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
</configuion>
```

6.创建目录
```sh
sudo mkdir -p /data/hadoop/hdfs/nn
sudo mkdir -p /data/hadoop/hdfs/dn
sudo chown -R hadoop.hadoop /data/hadoop
```
7.格式话namenode
```sh
hdfs namenode  -format
```
8.集群启动&停止
```sh
$HADOOP_PREFIX/sbin/start-dfs.sh
$HADOOP_PREFIX/sbin/stop-dfs.sh
```
### 集群安装
1.在所有机器上停止并删除`current`目录
```sh
rm -rf /data/hadoop/hdfs/nn/current/
rm -rf /data/hadoop/hdfs/dn/current/
# 关于$HADOOP_PREFIX/sbin/stop-dfs.sh说明, 如果是在datanode上执行会停止当前的datanode节点和namenode节点
```
2.启动节点
```sh
# 启动namenode节点, 使用jps查看 1490 Jps \n 1382 NameNode
/usr/local/hadoop/sbin/hadoop-daemon.sh --script hdfs start namenode
/usr/local/hadoop/sbin/hadoop-daemon.sh --script hdfs stop namenode
# 启动datanode节点, 使用jps查看 13281 DataNode \n 15530 Jps
/usr/local/hadoop/sbin/hadoop-daemon.sh --script hdfs start datanode
/usr/local/hadoop/sbin/hadoop-daemon.sh --script hdfs stop datanode
```

### 遇到的问题

1. WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

(1) 主要原因是`glibc`, 通过代码日志可以看到需要安装一个`2.14的版本`, 通过`strings /lib64/libc.so.6 |grep GLIBC_`查看当前版本
(2) 需要在`hadoop-env.sh`最开头增加行`export HADOOP_OPTS="-Djava.library.path=${HADOOP_HOME}/lib/native"`

如果想看到具体的执行日志,可使用`export HADOOP_ROOT_LOGGER=DEBUG,console`
(3)参看`shell/升级glibc`文档

2. 如果单机,`core-site.xml`文档中可以`fs.defaultFS`设置为本机, 但是对于作为数据节点部署集群,需要设置为`namenode`的hostname
