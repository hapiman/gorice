安装`1.8`版本

- 下载地址：[jdk-8u171-linux-x64.tar.gz](http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz?AuthParam=1529128754_8f51987d024666063713070a503be0c7)

- 解压及移动
```sh
tar -zxvf jdk-8u171-linux-x64.tar.gz
mv  jdk1.8.0_171 java
mv java /usr/local
```

- 关联环境
```sh
# 编辑/etc/profile
vim /etc/profile

JAVA_HOME=/usr/local/java
JRE_HOME=/usr/local/java/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH

source /etc/profile
```

- 检验
```sh
java -version
```
