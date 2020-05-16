## linux安装redis

打开redis官网`https://redis.io/`，下载软件安装包，`http://download.redis.io/releases/redis-4.0.9.tar.gz`

`wget http://download.redis.io/releases/redis-4.0.9.tar.gz`

`tar -xvzf redis-4.0.9.tar.gz`

`mv redis-4.0.9 /usr/local`

设置后台启动，修改`/usr/local/redis/redis.conf`文件，修改`daemonize no`改为`daemonize yes`

编译`cd /usr/local/redis && make`，编译完成之后，如果没有报错，使用`make test`检查依赖项是否完全安装。

> 安装过程中可能会遇到`You need tcl 8.5 or newer in order to run the Redis test`错误

解决方案如下：

```sh 
wget http://downloads.sourceforge.net/tcl/tcl8.6.1-src.tar.gz  
sudo tar xzvf tcl8.6.1-src.tar.gz  -C /usr/local/  
cd  /usr/local/tcl8.6.1/unix/  
sudo ./configure  
sudo make  
sudo make install
```


使用`/usr/local/redis/src/redis-server /usr/local/redis/redis.conf`启动redis数据库

使用`/usr/local/redis/src/redis-cli shutdown` 关闭数据库




## 设置redis开机启动

1.首先要保证`redis.conf`中的`daemonize=yes`

2.直接增加`vi /etc/init.d/redis`文件，改文件可以从`/utils/redis_init_script`拷贝过来进行适当的修改。
下面代码是一个示例，程序安装在/usr/local/redis下面，需要将文件中的`EXEC`,`CLIEXEC`, `CONF`做适当的修改

```sh
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=6379
EXEC=/usr/local/redis/src/redis-server
CLIEXEC=/usr/local/redis/src/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/usr/local/redis/redis.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```
