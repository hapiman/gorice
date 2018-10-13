## `phoenix`主要用于连接`hbase`

### 安装
```sh
# 根据hbase的版本下载
wget http://archive.apache.org/dist/phoenix/apache-phoenix-4.13.1-HBase-1.2/bin/apache-phoenix-4.13.1-HBase-1.2-bin.tar.gz
# 解压
tar -xvzf apache-phoenix-4.13.0-HBase-1.2-bin.tar.gz
mv apache-phoenix-4.14.0-HBase-1.2-bin phoenix
mv phoenix /usr/local/hbase
```

### 复制文件
将`phoenix-*-server.jar`文件复制到每个hbase节点的`/usr/local/hbase/lib`目录下

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

**问题二**

org.apache.phoenix.exception.PhoenixIOException: SYSTEM.CATALOG
Caused by: org.apache.hadoop.hbase.TableNotFoundException: SYSTEM.CATALOG

先关闭hbase（包括HMaster和regionServer）, 执行`./hbase clean --cleanAll`, 然后重启

**问题三**

Caused by: org.apache.hadoop.hbase.ipc.RemoteWithExtrasException(org.apache.hadoop.hbase.DoNotRetryIOException): org.apache.hadoop.hbase.DoNotRetryIOException: Class org.apache.phoenix.coprocessor.MetaDataEndpointImpl cannot be loaded Set `hbase.table.sanity.checks` to false at conf or table descriptor if you want to bypass sanity checks

直接在`hbase-site.xml`中添加
```xml
<property>
  <name>hbase.table.sanity.checks</name>
  <value>false</value>
</property>
```
`hbase.table.sanity.checks=true`主要用于hbase的各种参数检查, 如果设置为`true`, 则会按照下面的逻辑顺序检查
```
1.check max file size，hbase.hregion.max.filesize，最小为2MB
2.check flush size，hbase.hregion.memstore.flush.size，最小为1MB
3.check that coprocessors and other specified plugin classes can be loaded
4.check compression can be loaded
5.check encryption can be loaded
6.Verify compaction policy
7.check that we have at least 1 CF
8.check blockSize
9.check versions
10.check minVersions <= maxVerions
11.check replication scope
12.check data replication factor, it can be 0(default value) when user has not explicitly set the value, in this case we use default replication factor set in the file system.
```
