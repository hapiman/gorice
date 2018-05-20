在虚拟机linux上安装好redis后,使用本地机器进行访问redis
- 1:访问出现Could not connect to Redis at (ip地址):6379: Connection refused
最有可能的原因就是没有关闭防火墙,
1) 重启后生效
        开启： chkconfig iptables on
        关闭： chkconfig iptables off
        2) 即时生效，重启后失效
        开启： service iptables start
        关闭： service iptables stop
- 2:启动出现以下异常
Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.

解决:设置redis.conf参数protected-mode no

- 3:启动出现WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
关于vm.overcommit_memory参数的详解
  0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，      并把错误返回给应用进程。
  1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
  2， 表示内核允许分配超过所有物理内存和交换空间总和的内存
  编辑/etc/sysctl.conf ，改vm.overcommit_memory=1，然后sysctl -p 使配置文件生效

- 4:启动出现 The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
先看一下redis.conf中关于tcp-backlog参数的解释
此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度，当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，

默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。该内核参数默认值一般是128，

对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。在/etc/sysctl.conf中添加:net.core.somaxconn = 2048，然后在终端中执行sysctl -p。

解决方法注释中已给出

- 5:访问又出现Could not connect to Redis at (ip地址):6379: Connection refused
redis.conf中关于bind参数的解释
注释掉本机,局域网内的所有计算机都能访问。
band localhost   只能本机访问,局域网内计算机不能访问。
bind  局域网IP    只能局域网内IP的机器访问, 本地localhost都无法访问。
解决办法就是注释掉bind配置

- 6:访问出现Creating Server TCP listening socket *:6379: unable to bind socket
参考这篇文章点击打开链接
1. 将IPv6的网卡进行关闭（具体方法进行百度，这里不进行详细叙述）
2. 添加在redis.conf中添加 bind 0.0.0.0



注:该配置与解决方法只适用于运行环境下,生产环境下未经过测试
