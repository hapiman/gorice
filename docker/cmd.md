## docker常用命令
```sh
docker ps  # 查看正在运行的容器
docket ps -a  # 查看已经删除的容器
docker run  -p 8080:80 # 镜像名称
docker cp index.html(文件内容)  17add（容器id）:/usr/xx/xxx（容器目录）  # 拷贝文件 docker cp不会持久化保存
# docker 在容器中所做的操作都是暂时的，没有被保存。

保存方法 docker commit -m ‘xxx’ 容器ID 新生成镜像名称

docker stop 容器id （有序结束）

docker kill 容器ID （立马结束）

docker rmi 镜像名称 （删除镜像）

docker rm 容器ID （删除容器）

--restart=always 自动重启容器

sudo docker rm 'docker ps -a -q'列出所有停止的容器 并删除

docker run -p 80:80 -d -v $PWD/html:/usr/local/nginx/html(远程路径) nginx（镜像名称）

docker create -v $PWD/data:/var/mydata —name data_container  ubuntu

docker run -it —volume-from data-container ubuntu /bin/bash

docker exec -it nginx /bin/bash 进入容器交互方式运行容器，使用mount查看挂载情况

docker tag docker/whalesay pj/whalesay 创建新的tag镜像

docker push pj/whalesay 上传镜像

docker login 先登录

docker push 账户名／镜像名，帐户名和登录名不一致会denied: requested access to the resource is denied

docker-compose

docker-compose up -d

docker-compose stop

docker-compose rm 删除停掉的容器
```

