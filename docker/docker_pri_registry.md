## 搭建docker私有仓库

### 安装docker
参考文章：[k8s搭建](https://github.com/hapiman/gorice/blob/master/k8s/k8s%E6%90%AD%E5%BB%BA.md)

### 仓库搭建
私有仓库的搭建直接通过官方提供的`registry`镜像提供的功能即可。

1、下载镜像 `docker pull registry`

2、启动容器
```sh
docker run -d \
-v /opt/data/registry:/var/lib/registry \
-p 5000:5000 \
--restart=always \
--name registry \
registry
```
其中，

`/opt/data/registry`是镜像仓库的本地目录；

`/var/lib/registry`是镜像仓库的临时目录，容器重启或者销毁仓库中的镜像都会消失；

`--restart=always`设置`docker`启动的时，已有的容器跟着启动。

### 功能测试
假设镜像仓库所在服务器的IP地址为`IP`，则：

查看仓库中的镜像：
```sh
# 查看所有镜像，{"repositories":["busybox"]}
curl -XGET http://IP:5000/v2/_catalog
# 查看某个镜像详细信息，{"name":"busybox","tags":["latest"]}
curl -XGET http://IP:5000/v2/busybox/tags/list
```

上传镜像：
```sh
# 从公共仓库中下载busybox
docker pull busybox
# 修改标签
docker tag busybox IP:5000/busybox
# 上传
docker push IP:5000/busybox
```

下载镜像：
```sh
docker pull IP:5000/busybox
```
### 遇到的问题
```
The push refers to a repository [IP:5000/busybox]
Get https://IP:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```
`docker`仓库默认使用`https`的访问方式。在没有部署证书情况下，需要在使用`docker`私有仓库的机器上的`daemon.json`文件中配置`insecure-registries`为仓库地址和端口。

```sh
vim /etc/docker/daemon.json
# 配置json文件 { "insecure-registries" : ["IP:5000"] }
systemctl restart docker.service
# 启动docker
```
另外，在网上看到可以直接在仓库所在的机器的配置
```sh
vi /usr/lib/systemd/system/docker.service
# 设置如下，但是没有起效果
# ExecStart=/usr/bin/dockerd --insecure-registry IP:5000
```
