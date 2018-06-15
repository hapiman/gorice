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

* 启动zookeeper
```sh
  /usr/local/bin/zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties &
  # 或者
  cd /usr/local/Cellar/kafka/1.0.0
  ./zookeeper-server-start /usr/local/etc/kafka/zookeeper.properties &
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

* 增加分区数
```sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-topics  --zookeeper localhost:2181 --alter --topic test --partitions 10
# WARNING: If partitions are increased for a topic that has a key, the partition logic or ordering of the messages will be affected
Adding partitions succeeded!
```

* 生产者：发送消息
``` sh
cd /usr/local/Cellar/kafka/1.0.0
.bin/kafka-console-producer --broker-list localhost:9092 --topic test

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

* 查看topic消费到的offset
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

* 停止运行
``` sh
./bin/kafka-server-stop
./bin/zookeeper-server-stop
```
