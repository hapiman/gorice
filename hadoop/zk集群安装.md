### 新建用户
```sh
useradd zookeeper
passwd zookeeper
# 登录密钥
# 添加hadoop到sudoers中
chmod u+w /etc/sudoers
# 在/etc/sudoers文件尾添加代码
zookeeper ALL=NOPASSWD:ALL
# 使用sudo whoami
```
### 下载安装
地址: `http://archive.apache.org/dist/zookeeper/zookeeper-3.4.6/`
```sh
tar -xvzf zookeeper-3.4.6.tar.gz
mv zookeeper-3.4.6 zookeeper
sudo  mv zookeeper /usr/local/
sudo chown -R zookeeper.zookeeper /usr/local/zookeeper/
```
### 修改配置
1.在`/etc/profile`中增加
``` sh
export ZOOKEEPER_HOME=/usr/local/zookeeper
```
2.设置`dataDir`
```sh
cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo.cfg
sudo chown zookeeper.zookeeper zoo.cfg
vim /usr/local/zookeeper/conf/zoo.cfg
# 修改 dataDir = /data/zookeeper
```
3.在`bin/zkEnv.sh`中修改`ZOO_LOG_DIR`
```sh
ZOO_LOG_DIR=/data/logs/zookeeper # 新增
ZOOBINDIR="${ZOOBINDIR:-/usr/bin}"
ZOOKEEPER_PREFIX="${ZOOBINDIR}/.."
```
4.创建文件夹
```sh
sudo mkdir /data/zookeeper
sudo chown zookeeper.zookeeper /data/zookeeper
sudo mkdir /data/logs/zookeeper
sudo chown zookeeper.zookeeper /data/logs/zookeeper
```
5.在每台`dataDir`目录中创建`myid`文件, 写入这台服务器的`Zookeeper id`, 这个id是一个1-255的数字
``` sh
vim /data/zookeeper/myid # 输入数字
```

### 使用zookeeper用户启动
```sh
$ZOOKEEPER_HOME/bin/zkServer.sh start
$ZOOKEEPER_HOME/bin/zkCli.sh
# 使用ls查看情况, [zk: localhost:2181(CONNECTED) 0]
# 查看启动情况
$ZOOKEEPER_HOME/bin/zkServer.sh status
```
### 配置到所有的节点

``` sh
$ZOOKEEPER_HOME/bin/zkServer.sh stop
# 之前需要删除启动日志信息
ls /data/zookeeper |grep -v myid |xargs rm -rf (删除myid之外的文件)
rm -rf /data/logs/zookeeper/*
# 配置$ZOOKEEPER_HOME/conf/zoo.cfg
# fjr-ofckv-72-238 237 238 是你的hostname
server.238=fjr-ofckv-72-238:2888:3888
server.237=fjr-ofckv-72-237:2888:3888
server.236=fjr-ofckv-72-236:2888:3888
```

### 添加到自启动
使用`root`账户在`/etc/init.d`文件夹下创建`zookeeper`文件
```sh
#!/bin/bash
#chkconfig:2345 80 10
#description: zookeeper service
RETVAL=0
export JAVA_HOME="/usr/local/java"
start() {
  su zookeeper -c "/usr/local/zookeeper/bin/zkServer.sh start"
}
stop() {
  su zookeeper -c "/usr/local/zookeeper/bin/zkServer.sh stop"
}
status() {
  su zookeeper -c "/usr/local/zookeeper/bin/zkServer.sh status"
}
case $1 in
start)
  start
;;
stop)
  stop
;;
status)
  status
;;
esac
exit $RETVAL
```
修改为可执行文件`chmod +x zookeeper`

测试 `service zookeeper start`

添加到自启动 `chkconfig zookeeper on`
