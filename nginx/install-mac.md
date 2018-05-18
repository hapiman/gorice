## mac 安装linux

删除 `brew uninstall nginx`

安装 `brew install nginx`

查看安装信息 `sudo brew info nginx`
1.说明nginx的安装目录 `/usr/local/Cellar/nginx/1.13.12`
2.说明nginx当前的配置文件目录 `/usr/local/etc/nginx/nginx.conf`
3.说明nginx会直接读取的配置文件夹 `/usr/local/etc/nginx/servers/`
4.暴露启动命令 `brew services start nginx`

使用`which`查看`nginx`命令目录为`/usr/local/bin/nginx`

启动`nginx`服务命令(后台启动)，`brew services start nginx`；
可以直接通过`nginx -c nginx.conf`命令直接启动

关闭`nginx`服务命令，`brew services stop nginx`或者`nginx -s stop`

重启`nginx`服务命令，`brew services restart nginx`或者``nginx -s reload`

文件检测`nginx -t`


## 收获

通过`brew`安装的程序，log文件默认放在`/usr/local/var/log`, 如nginx的安装文件的日志目录为`/usr/local/var/log/nginx`
另外系统相关的日志，可能会放在`/var/log`下面，一般软件的目录都存在这两个目录下面。



