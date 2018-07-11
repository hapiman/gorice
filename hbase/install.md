### 安装
```sh
# 下载镜像
docker pull harisekhon/hbase
# 运行容器,默认直接进入容器,可直接体验命令行操作
docker run -ti harisekhon/hbase
# 当启动界面关闭之后,在其他界面直接进入容器,执行相关操作
docker exec -it 50ee02d0f4ee /hbase/bin/hbase shell
```
### 基本命令
[http://www.cnblogs.com/nexiyi/p/hbase_shell.html](http://www.cnblogs.com/nexiyi/p/hbase_shell.html)
