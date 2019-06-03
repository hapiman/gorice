## 安装过程
> [安装文档原文](https://blog.csdn.net/qq_33530388/article/details/72811705)
- 下载镜像
```sh
docker pull registry.cn-hangzhou.aliyuncs.com/kaibb/hadoop
```
- 启动容器
```sh
docker run --name Master -h Master -d registry.cn-hangzhou.aliyuncs.com/kaibb/hadoop
docker run --name Slave1 -h Slave1 -d registry.cn-hangzhou.aliyuncs.com/kaibb/hadoop
docker run --name Slave2 -h Slave2 -d registry.cn-hangzhou.aliyuncs.com/kaibb/hadoop
```
- 配置SSH--每一台机器上执行
```sh
/etc/init.d/ssh start
# 直接换行往下即可
ssh-keygen -t rsa

cd # 回到帐户主目录
cd .ssh # 回到ssh目录
cat id_rsa.pub > authorized_keys
```
实现每一台机器的`authorized_keys`相同, 都有集群中其他服务器的公钥
![](https://github.com/hapiman/gorice/blob/master/docker/imgs/authorize_keys.png?raw=true)
- 配置hosts

使用`ip addr`来获取当前服务器的IP地址
在每一台机器上, 修改`/etc/hosts`, 将主机名和对应的ip地址添加进去, 方便调用`ssh`

如图:
![](https://github.com/hapiman/gorice/blob/master/docker/imgs/hosts.png?raw=true)
每台服务器都配置好之后, 可以在集群中任意节点执行`ssh Slave1`, `ssh Slave2`, `ssh Master`实现登录

- 配置hadoop
`hadoop`的配置文件必须要重新设置才能将docker启动起来.
首先在登录`Master`节点, 修改下面提及的文件

hadoop-env.sh, 修改有关java的环境
```sh
export JAVA_HOME=/opt/tools/jdk1.8.0_77(当前的JAVA_HOME)
```

core-site.xml, 其中`/hadoop/tmp`默认会创建
```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://Master:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/hadoop/tmp</value>
  </property>
</configuration>
```

hdfs-site.xml, 其中`/hadoop/name`和`/hadoop/data`默认会创建
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/hadoop/data</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/hadoop/name</value>
  </property>
</configuration>
```

mapred-site.xml
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

yarn-site.xml
```xml
<configuration>
 <property>
<name>yarn.resourcemanager.address</name>
 <value>Master:8032</value>
</property>
<property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>Master:8030</value> </property> <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>Master:8031</value>
</property>
<property>
<name>yarn.resourcemanager.admin.address</name>
<value>Master:8033</value>
</property>
<property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>Master:8088</value>
</property>
 <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
 <property>
 <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
 <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
</configuration>
```
/opt/tools/hadoop/etc/hadoop/slaves, 删除已有节点并增加当前的`dataNode`节点, 主节点`Master`通过slaves文件的中的配置ssh连接到从节点
```
Slave1
Slave2
```
在从节点同步主节点的配置信息
```sh
scp core-site.xml hadoop-env.sh hdfs-site.xml mapred-site.xml yarn-site.xml slaves Slave1:/opt/tools/hadoop/etc/hadoop/
scp core-site.xml hadoop-env.sh hdfs-site.xml mapred-site.xml yarn-site.xml slaves  Slave2:/opt/tools/hadoop/etc/hadoop/
# 其中Slave1和Slave2会替换成/etc/hosts中配置的IP
```

- 运行hadoop
格式化操作
```sh
hadoop namenode -format
```

启动hadoop, 如果启动过程中提示关于0.0.0.0地址输入yes或no，输入yes即可
```sh
/opt/tools/hadoop/sbin/start-all.sh
```
在Master上使用jps查看相关进程是否启动
```sh
jps
```
在Slave节点上使用jps查看相关进程是否启动
```sh
jps
```
使用命令查看各节点信息
```sh
hadoop dfsadmin -report
```
如果没有遇到异常,结果如图所示
![](https://github.com/hapiman/gorice/blob/master/docker/imgs/report.png?raw=true)


- 示例

创建文件夹 `hadoop fs -mkdir /input`
查看文件夹 `hadoop fs -ls /`
回到hadoop主目录`/opt/tools/hadoop`,上传文件到input文件夹中`hadoop fs -put README.txt /input/`
在`/opt/tools/hadoop/share/hadoop/mapreduce`目录中执行：`hadoop jar hadoop-mapreduce-examples-2.7.2.jar wordcount /input /output`
查看输出文件夹 `hadoop fs -ls /output/`
查看统计结果 `hadoop fs -cat /output1/part-r-00000`, 至于文件名称是否叫做`part-r-00000`不确定

## 注意事项
1.因为使用镜像安装,每一重启之后会回到初始状态,需要经常备份容器; 另外似乎`/etc/hosts`每一次都会随着容器重启而重置

2.如何固定容器的IP, 使容器不因重启而改变IP

3.如果dataNode启动不起来, 很可能是因为`master`和`slaver`的`clusterID`不同导致, 什么时候会导致不同呢?
