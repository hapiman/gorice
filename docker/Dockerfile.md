`Dockerfile`配置说明

- FROM

功能为指定基础镜像，并且必须是第一条指令。如果不以任何镜像为基础，那么写法为：FROM scratch。同时意味着接下来所写的指令将作为镜像的第一层开始
```
FROM <image>
FROM <image>:<tag>
FROM <image>:<digest>
```
三种写法，其中`tag`和`digest`是可选项，如果没有选择，那么默认值为latest

- RUN
运行指定的命令, RUN命令有两种格式
```
1. RUN <command>
2. RUN ["executable", "param1", "param2"]
```

两种写法比对：
```
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME
RUN ["/bin/bash", "-c", "echo hello"]
```
注意：多行命令不要写多个RUN，原因是Dockerfile中每一个指令都会建立一层，多少个RUN就构建了多少层镜像，会造成镜像的臃肿、多层，不仅仅增加了构件部署的时间，还容易出错，RUN书写时的换行符是`\`。

- CMD
功能为容器启动时要运行的命令，语法有三种写法
```
1. CMD ["executable","param1","param2"]
2. CMD ["param1","param2"]
3. CMD command param1 param2
```
举例说明两种写法：
```
CMD [ "sh", "-c", "echo $HOME"
CMD [ "echo", "$HOME" ]
```
补充细节：参数需使用双引号，docker解析的是一个JSON array

`RUN`命令会在`image`文件构建阶段执行，执行的结果会打包进入image文件；

`CMD`文件会在容器启动之后执行，一个`Dockerfile`可以包含多个`RUN`命令，但是只能有一个`CMD`命令

- LABEL
功能是为镜像指定标签，一个Dockerfile种可以有多个LABEL，如下：
```
LABEL multi.label1="value1" \
multi.label2="value2" \
other="value3"
```
说明：**LABEL会继承基础镜像种的LABEL，如遇到key相同，则值覆盖**

- MAINTAINER
指定作者：`MAINTAINER <name>`

- EXPOSE
功能为暴漏容器运行时的监听端口给外部，但是EXPOSE并不会使容器访问主机的端口，如果想使得容器与主机的端口有映射关系，必须在容器启动的时候加上 -P参数

- ENV
功能为设置环境变量，语法有两种，两者的区别就是第一种是一次设置一个，第二种是一次设置多个
```
1. ENV <key> <value>
2. ENV <key>=<value>
```

- ADD
一个复制命令，把文件复制到镜像中。
```
1. ADD <src>... <dest>
2. ADD ["<src>",... "<dest>"]
```
<dest>路径的填写可以是容器内的绝对路径，也可以是相对于工作目录的相对路径；<src>可以是一个本地文件或者是一个本地压缩文件，还可以是一个url；如果把<src>写成一个url，那么ADD就类似于wget命令
```
ADD test relativeDir/
ADD test /relativeDir
ADD http://example.com/foobar /
```
尽量不要把<scr>写成一个文件夹，如果<src>是一个文件夹了，复制整个目录的内容,包括文件系统元数据



- COPY
```
1. COPY <src>... <dest>
2. COPY ["<src>",... "<dest>"]
```
**与ADD的区别，COPY的<src>只能是本地文件，其他用法一致**


- ENTRYPOINT

功能是启动时的默认命令
```
1. ENTRYPOINT ["executable", "param1", "param2"]
2. ENTRYPOINT command param1 param2
```

与CMD比较说明：

相同点：只能写一条，如果写了多条，那么只有最后一条生效；容器启动时才运行，运行时机相同。

不同点：

ENTRYPOINT不会被运行的command覆盖，而CMD则会被覆盖，如当执行语句加一个命令`/bin/bash`，比如 docker run -it [image] /bin/bash，`CMD`会被忽略掉，命令`bash`将被执行；

如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD指令不是一个完整的可执行命令，那么CMD指定的内容将会作为ENTRYPOINT的参数；

如果我们在Dockerfile种同时写了ENTRYPOINT和CMD，并且CMD是一个完整的指令，那么它们两个会互相覆盖，谁在最后谁生效。


- VOLUME

`VOLUME 本地文件夹 容器目录`，可实现挂载功能，将本地文件夹或者其他容器种得文件夹挂在到这个容器中
```
VOLUME ["/var/log/"]
VOLUME /var/log
VOLUME /var/log /var/db
```

- USER
设置启动容器的用户，可以是用户名或UID
```
USER daemo
USER UID
```


- WORKDIR

`WORKDIR /path/to/workdir`


- ARG

设置变量命令，ARG命令定义了一个变量，在docker build创建镜像的时候，使用 --build-arg <varname>=<value>来指定参数

如果用户在build镜像时指定了一个参数没有定义在Dockerfile种，那么将有一个Warning提示: `[Warning] One or more build-args [foo] were not consumed.`

我们可以定义一个或多个参数，如下：
```
FROM busybox
ARG user1=someuser
ARG buildno=1
```
