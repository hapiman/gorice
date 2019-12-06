### Mac上安装kalka

* 直接安装`brew install kafka`

1.会安装`zookeeper`相关环境

2.安装目录`/usr/local/Cellar/kafka/1.0.0`,`1.0.0`是当前kafka的版本，不同时间安装可能不一样

3.可能错误

3.1 brew安装的时可能需要`sudo`权限，但是提示`Running Homebrew as root is extremely dangerous and no longer supported.As Homebrew does not drop privileges on installation you would be giving allbuild scripts full access to your system.`可以使用`sudo chown -R $(whoami) /usr/local`将owner短暂的换成当前用户。

3.2 如果出现`Cannot tap homebrew/core`,`invalid syntax in tap`,`Error: Invalid formula`等类似的问题，可以使用`cd /usr/local/Library/Taps/homebrew/homebrew-core && git fetch && git reset --hard origin/master`命令直接下载文件。之后的安装可能会要求安装java的环境`brew cask install caskroom/versions/java8`

* 安装的配置文件的位置
``` sh
`/usr/local/etc/kafka/server.properties` #kafka服务配置
`/usr/local/etc/kafka/zookeeper.properties` #zookeeper环境配置
```

默认情况下，在代码中能够执行发送和接收任务，但是无法在代码中调用成功，需要修改kafka的配置文件`/usr/local/etc/kafka/server.properties`中的`listeners=PLAINTEXT://localhost:9092`行注释去掉，并增加`localhost`字段。


* 启动zookeeper
```sh
  /usr/local/bin/zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties &
  # 或者
  cd /usr/local/Cellar/kafka/1.0.0
  ./bin/zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties &
```

* 启动kafka服务
``` sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-server-start /usr/local/etc/kafka/server.properties &
```

* 创建topic
```sh
# Created topic "test".
# replication-factor 控制消息保存在几个broker(服务器)上，一般情况下等于broker个数；如果未指定，则采用默认数量，在server.properties中可查。
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

* 查看创建的topic
``` sh
# 查看所有的topic
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-topics --list --zookeeper localhost:2181
# 查看指定topic
./bin/kafka-topics --zookeeper localhost:2181 --describe --topic  test
# 关于结果分析分析
# 第一行显示partitions的概况，列出了Topic名字，partition总数，存储这些partition的broker数
# leader 是该partitons所在的所有broker中担任leader的broker id，每个broker都有可能成为leader
# replicas 显示该partiton所有副本所在的broker列表，包括leader，不管该broker是否是存活，不管是否和leader保持了同步。
# isr in-sync replicas的简写，表示存活且副本都已同步的的broker集合，是replicas的子集
# 详细信息参看 https://blog.csdn.net/lkforce/article/details/77776684
```

* 修改topic（如增加分区数，使用--alter）
```sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-topics  --zookeeper localhost:2181 --alter --topic test --partitions 10
# WARNING: If partitions are increased for a topic that has a key, the partition logic or ordering of the messages will be affected
Adding partitions succeeded!
```

* 生产者：发送消息
``` sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-console-producer --broker-list localhost:9092 --topic test

# > message 1
# > message 2
```
* 消费者：接收消息
```sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning

# message1
# essage2
```

* 删除topic
``` sh
./bin/kafka-topics --delete --zookeeper localhost:2181 --topic test
# 如果 kafka 启动时加载的配置文件中 server.properties 没有配置delete.topic.enable=true，那么此时的删除并不是真正的删除，而是把 topic 标记为：marked for deletion
```

* 查看topic各个分区的消息的信息

能够看到topic消费到的offset，--time -1表示最大位移 --time -2表示最早位移。

```sh
./bin/kafka-run-class  kafka.tools.GetOffsetShell --broker-list  127.0.0.1:9092 --topic test --time -1
# test:8:0
# test:2:0
# test:5:0
# test:4:0
# test:7:0
# test:1:0
# test:9:0
# test:3:0
# test:6:0
# test:0:5
```

* Consumer-Groups管理

查看消费者组类型
```sh
./bin/kafka-consumer-groups --bootstrap-server localhost:9092 --list
```

查看消费者组详情(能够查看消费的进度，分区等数据)
```sh
./bin/kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group g-test
```
结果：
```
Consumer group 'g-test' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
g-test          test            1          955             2563            1608            -               -               -
g-test          test            0          56657           58265           1608            -               -               -
```

重设消费者组位移
```sh
# 最早处
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupname --reset-offsets --all-topics --to-earliest --execute
# 最新处
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupname --reset-offsets --all-topics --to-latest --execute
# 某个位置
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupname --reset-offsets --all-topics --to-offset 2000 --execute
# 调整到某个时间之后得最早位移
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupname --reset-offsets --all-topics --to-datetime 2019-09-15T00:00:00.000
```

删除消费者组 
```sh
./bin/kafka-consumer-groups --zookeeper localhost:2181 --delete --group groupname
```

读取消费者组消息 
```sh
./bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic test 
--from-beginning
--consumer-property group.id=g-test
```

* **三种消费脚本** 
```sh
# 一般使用 
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topicname --from-beginning
# 指定groupid
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topicname --from-beginning
--consumer-property group.id=old-consumer-group
# 指定分区
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topicname --from-beginning
--partition 0
```


* 停止运行
``` sh
./bin/kafka-server-stop
./bin/zookeeper-server-stop
```

### 如何设置多个broker

* 创建配置文件
```sh
cd /usr/local/etc/kafka
cp server.properties config/server-1.properties
cp server.properties config/server-2.properties
```
* 修改文件
在`server-1.properties`文件中
```sh
broker.id=1
listeners=PLAINTEXT://:9093
log.dirs=/usr/local/var/lib/kafka-logs-1
```
在`server-2.properties`文件中
```sh
broker.id=2
listeners=PLAINTEXT://:9094
log.dirs=/usr/local/var/lib/kafka-logs-2
```

* 启动`zookeeper`
```sh
  /usr/local/bin/zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties &
  # 或者
  cd /usr/local/Cellar/kafka/1.0.0
  ./bin/zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties &
```

* 启动kafka服务
``` sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-server-start /usr/local/etc/kafka/server.properties &
./bin/kafka-server-start /usr/local/etc/kafka/server-1.properties &
./bin/kafka-server-start /usr/local/etc/kafka/server-2.properties &
```
* 创建多个topic
```sh
./bin/kafka-topics --zookeeper localhost:2181 --replication-factor 3 --describe --topic  my-replicated-topic
# 查看所有的topic
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-topics --list --zookeeper localhost:2181
```

* 查看分区信息
```sh
./bin/kafka-topics --describe --zookeeper localhost:2181 --topic my-replicated-topic
```

Topic:my-replicated-topic	PartitionCount:1	ReplicationFactor:3	Configs:
Topic: my-replicated-topic	Partition: 0	 Leader: 0	Replicas: 1,2,0	Isr: 0

其中，

"leader" 节点是1.

"replicas" 信息，在节点1,2,0上，不管node死活，只是列出信息而已.

"isr" 工作中的复制节点的集合. 也就是活的节点的集合.

* 生产者：发送消息
``` sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-console-producer --broker-list localhost:9092 --topic my-replicated-topic
```

* 消费者：接收消息
```sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic my-replicated-topic --from-beginning
```

* 直接先杀调`broker1`,也就是`leader`节点
```sh
ps | grep server-1.properties
# 20376 ttys006    0:16.60 /Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home/bin/java
kill -9 20376
#  leader节点发送改变，从日志可以看出； 同时当前的活着的可用节点为2,0
```

Topic:my-replicated-topic	PartitionCount:1	ReplicationFactor:3	Configs:
Topic: my-replicated-topic	Partition: 0	Leader: 2	Replicas: 1,2,0	Isr: 2,0

* 消费者：接收消息2
```sh
# 再次接收数据，数据没有发送改变
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic my-replicated-topic --from-beginning
```

### 遇到的问题
1.连结不能建立的问题
```
[Consumer clientId=consumer-1, groupId=console-consumer-2028] Connection to node -1 (localhost/127.0.0.1:9092) could not be established. Broker may not be available.
```
