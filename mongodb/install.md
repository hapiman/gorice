## 写在前面的话

Centos安装mongodb， 下载地址`https://www.mongodb.com/download-center#community`，选择linux标签，选择`Linux 64-bit lagacy x64`。
将显示如下地址 [https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.4.tgz](https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.4.tgz)，这也是当前安装的版本。

## 安装步骤

- 下载
```sh
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.4.tgz
```
- 解压到指定目录
```sh
tar -zxvf mongodb-linux-x86_64-3.6.4.tgz
mv mongodb-linux-x86_64-3.6.4.tgz /usr/local/mongodb
```
- 创建配置文件

程序执行用户用户组信息为`www:www`目录
```sh
# 创建日志文件并修改权限
mkdir -p /data/mongodb/logs
cd /data/mongodb/logs
touch /mongodb.log
sudo chmod www:www -R /data/mongodb
```

创建启动配置文件 `vim /usr/local/mongodb/mongodb.conf`
```sh
# mongodb.conf文件内容
logappend=true
logpath=/data/mongodb/logs/mongodb.log # 数据库日志
fork = true
port = 27017
dbpath=/data/mongodb/ # 数据库目录
```

配置环境变量 `vim /etc/profile`
```sh
export PATH=$PATH:/usr/local/mongodb/bin
```

- 设置开机自启动
```sh
chmod a+x /etc/rc.local
mongod --config /usr/local/mongodb/mongodb.conf
```

- 常用启动关闭命令
```sh
# 启动
mongod --config /usr/local/mongodb/mongodb.conf
# 关闭
mongod --shutdown --config /usr/local/mongodb/mongodb.conf
```

## 安装常见问题

1.可能会遇到`fork`相关的问题，提示设置`fork`为false，·然后启动
