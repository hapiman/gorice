## `phoenix`主要用于连接`hbase`

### 安装
```sh
# 根据hbase的版本下载
wget http://archive.apache.org/dist/phoenix/apache-phoenix-4.13.1-HBase-1.2/bin/apache-phoenix-4.13.1-HBase-1.2-bin.tar.gz
# 解压
tar -xvzf apache-phoenix-4.14.0-HBase-1.2-bin.tar.gz
mv apache-phoenix-4.14.0-HBase-1.2-bin phoenix
mv phoenix /usr/local/hbase
```
### 启动
```sh
cp /usr/local/hbase/conf/hbase-site.xml /usr/local/hbase/phoenix/
/usr/local/hbase/phoenix/bin/sqlline.py xx,yy,zz:2181
```

### 相关问题

**问题一**
Error: ERROR 103 (08004): Unable to establish connection. (state=08004,code=103)
java.sql.SQLException: ERROR 103 (08004): Unable to establish connection.
Caused by: java.io.IOException: java.lang.reflect.InvocationTargetException
Caused by: java.lang.reflect.InvocationTargetException
Caused by: java.lang.ExceptionInInitializerError
Caused by: java.lang.IllegalArgumentException: java.net.UnknownHostException: mycluster
Caused by: java.net.UnknownHostException: mycluster

`mycluster`是集群的名字, 如果集群有问题,就会导致`phoenix`不能识别`mycluster`, 从而连不上`hbase`;
解决方案就是将`hadoop`的`core-site.xml`,`hdfs-site.xml`文件复制到`hbase`的`/usr/local/hbase/conf`目录即可.
