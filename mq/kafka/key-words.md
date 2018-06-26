### Topic
将消息分门别类，每一类消息称之为话题(Topic)

逻辑上一个Topic的消息虽然保存于一个或多个broker上，但用户只需指定消息的 Topic 即可生产或消费数据而不必关心数据存于何处



### Partition

Topic物理分组，一个Topic可以分为多个Partition，每个Partition是一个有序的队列。Partition中的每条消息都会被分配一个有序的id。如果没有指定，则会使用server.properties中默认配置数量，也就是`num.partitions`的值。
![](https://raw.githubusercontent.com/hapiman/gorice/master/mq/kafka/imgs/anatomy.png)

虽然增加分区数可以提供kafka集群的吞吐量，但是也会增加不可用及延迟的风险。因为多的分区数，意味着需要打开更多的文件句柄、增加点到点的延时、增加客户端的内存消耗。

分区数也限制了consumer的并行度，即限制了并行consumer消息的线程数不能大于分区数

分区数也限制了producer发送消息是指定的分区。如创建topic时分区设置为1，producer发送消息时通过自定义的分区方法指定分区为2或以上的数都会出错的；这种情况可以通过`alter –partitions`来增加分区数。

### Producer
发布消息的对象即为生产者(Producer)

### Consumer
订阅消息并处理发布某类消息的对象称之为话题消费者（Consumer）

### Broker
已发布的消息保存在一组服务器中，这组服务器就是`kafka`集群。集群中的每一个服务器都是一个代理（Broker）。
消费者可以订阅一个或者多个话题，并从`Broker`拉数据，从而消费这些已发布的消息

一个broker可以容纳多个topic。
一个Broker上可以有一个Topic的多个Partition，每个Partition的Leader随机存在于某一个Broker，这样实现了Topic的读写的负载均衡

### Consumer Group

每个`Consumer`属于一个特定的`Consumer Group`(可为每个 Consumer 指定Group Name，若不指定 Group Name 则属于默认的 Group）。

这是 Kafka 用来实现一个 `Topic 消息的广播`（发给所有的 Consumer ）和`单播`（发给任意一个 Consumer ）的手段。

一个 Topic 可以有多个 Consumer Group。Topic 的消息会复制（不是真的复制，是概念上的）到所有的 Consumer Group，但每个 Consumer Group 只会把消息发给该 Consumer Group 中的一个 Consumer。如果要实现广播，只要每个 Consumer 有一个独立的 Consumer Group 就可以了。如果要实现单播只要所有的 Consumer 在同一个 Consumer Group 。用 Consumer Group 还可以将 Consumer 进行自由的分组而不需要多次发送消息到不同的 Topic 。
