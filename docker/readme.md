### docker介绍

> Docker将应用程序与该程序的依赖，打包在一个文件里面。
> image 文件生成的容器实例，本身也是一个文件，称为容器文件。也就是说，一旦容器生成，就会同时存在两个文件： image 文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。

* docker下载包https://hub.docker.com/
* [https://www.docker-cn.com/registry-mirror](linux修改下载仓库)


### docker操作命令

* 下载镜像 `docker image pull hello-world`
* 列出所有image文件`docker image ls`
* 下载某个`image`文件 ` docker image pull [image name]`
* 发布镜像
`docker image build -t [username]/[repository]:[tag]`
`docker image push [username]/[repository]:[tag]`
* 运行某个image文件`docker container run [image name]`，`docker container run`命令具有自动抓取`image`文件的功能
* 手动停止某一个命令`docker container kill [containID]`
* 查看docker中正在运行着的程序`docker ps -a`
* 列出本机正在运行的容器 `docker container ls`
* 列出本机所有容器，包括终止运行的容器 `docker container ls --all`
* 使用`docker container start`和`docker container stop`启动和停止已有containerid的镜像
* `docker container logs`命令用来查看 docker 容器的输出，
即容器里面 Shell 的标准输出。如果docker run命令运行容器的时候，没有使用-it参数，就要用这个命令查看输出。
* 终止运行的容器文件，依然会占据硬盘空间，可以使用`docker container rm [containerID]`命令删除。
* 列出所有的`container`, `docker container ls --all`
* 创建`image`,`docker image build -t koa-demo .`,-t表示image名称，后面有个`.`，表示当前目录
* 容器终止后自动删除容器文件，使用`-rm`参数，`docker container run --rm -p 8000:3000 -it koa-demo /bin/bash`
* 关于容器的`CMD`命令
如何区分`CMD`和`RUN`命令， `RUN`命令会在`image`文件构建阶段执行，执行的结果会打包进入image文件；`CMD`文件会在容器启动之后执行。一个`Dockerfile`可以包含多个`RUN`命令，但是只能有一个`CMD`命令，指定了`CMD`命令之后，`docker container run`命令就不能够福建命令额，否者会覆盖`CMD`命令。


