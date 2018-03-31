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
```

* 启动kafka服务
``` sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-server-start /usr/local/etc/kafka/server.properties &
```

* 创建topic
```sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
# Created topic "test".
```

* 查看创建的topic
``` sh
cd /usr/local/Cellar/kafka/1.0.0
./bin/kafka-topics --list --zookeeper localhost:2181

# test
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
./bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning

# message1
# essage2
```

* 删除topic
``` sh
./bin/kafka-topics --delete --zookeeper localhost:2181 --topic test
# 如果 kafka 启动时加载的配置文件中 server.properties 没有配置delete.topic.enable=true，那么此时的删除并不是真正的删除，而是把 topic 标记为：marked for deletion
```
