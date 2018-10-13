### 下载

(1) 下载页面
`http://www.rabbitmq.com/install-standalone-mac.html`

(2) 下载地址`https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.7/rabbitmq-server-mac-standalone-3.7.7.tar.xz`

### 安装
```
tar -xvzf rabbitmq-server-mac-standalone-3.7.7.tar.xz
mv rabbitmq_server-3.7.7 rabbitmq
mv rabbitmq /usr/local
```

### 启动
```sh
# 启动之后,使用CMD+C依然不能kill掉MQ,需要执行ps -ef | grep rabbitmq
# 使用http://localhost:15672可访问页面, 默认的用户guest,密码guest
# guest这个默认的用户只能通过http://localhost:15672 来登录，处于安全的考虑，其他的IP无法直接使用这个账号
./sbin/rabbitmq-server start

```

### 问题
启动RabbitMQ后，没法访问Web管理页面,需要执行下面的安装命令才行
```sh
rabbitmqctl start_app
rabbitmq-plugins enable rabbitmq_management
rabbitmqctl stop
```
