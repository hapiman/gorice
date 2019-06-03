
### docker介绍
Docker将应用程序与该程序的依赖，打包在一个文件里面。

image 文件生成的容器实例，本身也是一个文件，称为容器文件。
一旦容器生成，就会同时存在两个文件： image 文件和容器文件。
关闭容器并不会删除容器文件，只是容器停止运行而已，可以使用`docker rm`来删除。

### docker操作命令

列出所有image文件：`docker images`

下载镜像： `docker image pull [image name]`

查看镜像信息：`docker inspect pengj/nodeappk8s:v1.0.0`，查看镜像的某一部分信息：`docker inspect -f {{".Size"}} pengj/nodeappk8s:v1.0.0`

查看镜像的具体内容：`docker history  pengj/nodeappk8s:v1.0.0`

搜索镜像：`docker search mysql`，另外加上过滤参数`docker search --filter=stars=100 mysql`

删除镜像：`docker rmi [image]`，加上`-f`参数能够强制删除，即便有容器使用，一般**不推荐**

清理镜像：`docker image prune`

构建镜像： `docker image build -t [username]/[repository]:[tag]`

发布镜像：`docker image push [username]/[repository]:[tag]`

给镜像打标签, 使用`tag`命令添加镜像标签，指向同一镜像文件：`docker tag ubuntu:lastest myubuntu:lastest`

运行及进入容器：见👇【docker run和docker exec使用】

查看docker中所有的容器：`docker ps -a`

杀死某个容器：见👇【删除容器的方法汇总】

暂停某个容器：`docker container stop [containID]`

查看某个容器日志：见👇【查看日志docker logs使用】

### 查看日志docker logs使用
`docker logs -f -t --since="2017-05-31" --tail=10 edu_web_1`
`--since`: 此参数指定了输出日志开始日期，即只输出指定日期之后的日志。
`-f`: 查看实时日志
`-t`: 查看日志产生的日期
`-tail=10`: 查看最后的10条日志。
`edu_web_1`: 容器名称(容器名称)


### docker run和docker exec使用
运行镜像，使用`docker run`
`docker run -it -p 8000:8000 -v /src/webapp(本地绝对):/opt/webapp(镜像绝对路径)  ubuntu `

`-i` 打开标准输入接受用户的输入

`-t` 让docker分配一个伪终端(pseudo-tty)，并绑定到容器的标准输入上

`-u` 执行命令的用户

```sh
docker run -p 容器端口:宿主端口
    -v 宿主目录:容器目录
    -d 后台运行
    -e 设置环境变量
    -name 容器名称
    --rm 容器终止之后，自动删除文件
```

进入容器，查看状态，其中`i`,`t`参数说明和`docker run`一致
```
docker exec -it [container_name/container_id] /bin/bash
```

### 删除容器的方法汇总
- 方法一：
```sh
#显示所有的容器，过滤出Exited状态的容器，取出这些容器的ID，
sudo docker ps -a|grep Exited|awk '{print $1}'
#查询所有的容器，过滤出Exited状态的容器，列出容器ID，删除这些容器
sudo docker rm `docker ps -a|grep Exited|awk '{print $1}'`
```

- 方法二：
```sh
#删除所有未运行的容器（已经运行的删除不了，未运行的就一起被删除了）
sudo docker rm $(docker ps -a -q)
```

- 方法三：
```sh
#根据容器的状态，删除Exited状态的容器
sudo docker rm $(sudo docker ps -qf status=exited)
```

- 方法四：
```sh
# 删除处于stop的容器或者状态status=Exited的容器
sudo docker container prune
```
终止运行的容器文件，依然会占据硬盘空间，可以使用`docker rm [containerID]`命令删除。

### docker 如何删除none镜像
删除none的镜像，要先删除镜像中的容器。要删除镜像中的容器，必须先停止容器。
```sh
# 停止容器
docker stop $(docker ps -a | grep "Exited" | awk '{print $1 }')
# 删除容器
docker rm $(docker ps -a | grep "Exited" | awk '{print $1 }')
# 删除镜像
docker rmi $(docker images | grep "none" | awk '{print $3}')
```

### 创建镜像的两种方法

#### 第一、基于已有的镜像创建
主要是基于命令：
```
docker container commit
    -a, --author="": 作者信息；
    -c, --change=[]: 可以在提交的时候执行 Dockerfile 指令，如 CMD、ENTRYPOINT、ENV、EXPOSE、LABEL、ONBUILD、USER、VOLUME、WORIR 等；
    -m, --message="": 提交信息；
    -p, --pause=true: 提交时，暂停容器运行。
```
示例导入过程，需要记住当前容器的编号，假设为`axa0c8cfec3a`：
```sh
docker run -it xx:yy /bin/bash
touch news.txt
exit
```
执行提交命令： `docker container commit -m "Added nes.txt file" -a "hapiman" axa0c8cfec3a news:0.1`
查看当前镜像列表：`docker images`

#### 第二、基于 Dockerfile 来创建
最常见的方式，略过


### docker的导入导出操作

#### 方法一、使用`docker save`和`docker load`

查看当前镜像列表：`docker images`

导出`tar`文件：`docker save f59c7e5b1817 >zwx_ub.tar` 或者 `docker save -o zwx_ub.tar f59c7e5b1817`

加载镜像： `docker load < zwx_ub.tar` 或者 `docker load -i zwx_ub.tar`

查看当前镜像列表：`docker images`

#### 方法二、使用`docker export`和 `docker import`

导入：`cat ubuntu-14.04-x86_64-minimal.tar.gz | docker import - ubuntu:zwx`

导出：`docker export 16f568766019 > ubuntu.tar`，将镜像编号为`16f568766019`的镜像导出

#### 两者区别

`docker save images_name`：将一个镜像导出为文件，再使用docker load命令将文件导入为一个镜像，会保存该镜像的的**所有历史记录**。比docker export命令导出的文件大，很好理解，因为会保存镜像的所有历史记录。
`docker export container_id`：将一个容器导出为文件，再使用docker import命令将容器导入成为一个新的镜像，但是相比docker save命令，容器文件会**丢失所有元数据和历史记录**，仅保存容器当时的状态，相当于虚拟机快照
