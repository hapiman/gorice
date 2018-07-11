`kafka`的安装会依赖`zookeeper`

```sh
# 下载镜像
docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka

# 启动
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
docker run -d --name kafka --publish 9092:9092 --link zookeeper --env KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 --env KAFKA_ADVERTISED_HOST_NAME=宿主服务器IP --env KAFKA_ADVERTISED_PORT=9092 wurstmeister/kafka

# 进入
docker exec -it ${CONTAINER ID} /bin/bash
cd /opt/kafka

# 测试

# 创建主题
bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic mykafka
# 启动生产者
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka
# 启动消费者
bin/kafka-console-consumer.sh --zookeeper zookeeper:2181 --topic mykafka --from-beginning
```
