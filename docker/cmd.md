## docker常用命令
```sh
docker ps  # 查看正在运行的容器
docket ps -a  # 查看所有的容器
docker run  -p 8080:80 # 镜像名称
docker cp index.html(文件内容)  17add（容器id）:/usr/xx/xxx（容器目录）  # 拷贝文件 docker cp不会持久化保存
# docker 在容器中所做的操作都是暂时的，没有被保存。

保存方法 docker commit -m ‘xxx’ 容器ID 新生成镜像名称

docker stop 容器id （有序结束）

docker kill 容器ID （立马结束）

docker rmi 镜像名称 （删除镜像）

docker rm 容器ID （删除容器）

--restart=always 自动重启容器

docker rm $(docker ps -a -q) # 列出所有停止的容器 并删除

docker run -p 80:80 -d -v $PWD/html:/usr/local/nginx/html(远程路径) nginx（镜像名称）

docker create -v $PWD/data:/var/mydata —name data_container  ubuntu

docker run -it —volume-from data-container ubuntu /bin/bash

docker exec -it nginx /bin/bash # 进入容器交互方式运行容器，使用mount查看挂载情况

docker tag docker/whalesay pj/whalesay # 创建新的tag镜像

docker push pj/whalesay # 上传镜像

docker login 先登录

docker push 账户名／镜像名，帐户名和登录名不一致会denied: requested access to the resource is denied

docker-compose

docker-compose up -d

docker-compose stop

docker-compose rm 删除停掉的容器
```

使用save命令将 镜像保存为文件 `docker save -o  自定义文件名.tar  已存在的镜像名`
使用load命令将镜像文件保存到本地仓库 ` docker load -i 自定义文件名.tar`
可以使用`docker inspect  镜像文件名或者ID `以查看 文件标签内容
如果要重命名镜像文件名 `docker tag [image id] [name]:[版本]`，示例`docker tag b03b74b01d97 docker-redis:0.0.1`


```sh
docker logs -f -t --since="2017-05-31" --tail=10 edu_web_1
# --since : 此参数指定了输出日志开始日期，即只输出指定日期之后的日志。
# -f : 查看实时日志
# -t : 查看日志产生的日期
# -tail=10 : 查看最后的10条日志。
# edu_web_1 : 容器名称(容器名称)
```

```sh
# docker的导入导出操作==针对[镜像]
docker images
docker save f59c7e5b1817 >zwx_ub.tar
docker load -i zwx_ub.tar
docker images

# import 从本地文件系统导入一个镜像==针对[容器]
# docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
cat ubuntu-14.04-x86_64-minimal.tar.gz  |docker import - ubuntu:zwx
# 或
docker import ubuntu-14.04-x86_64-minimal.tar.gz - ubuntu:zwx
docker export 16f568766019 >ubuntu.tar
cat ubuntu.tar | sudo docker import - test/ubuntu:v1.0
# 用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，也可以使用 docker import 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。
```


# 关闭所有正在运行容器
docker ps | awk  '{print $1}' | xargs docker stop

# 删除所有容器应用
docker ps -a | awk  '{print $1}' | xargs docker rm
# 或者
docker rm $(docker ps -a -q)
