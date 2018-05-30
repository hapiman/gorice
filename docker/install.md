### docker安装过程说明及问题解决

**docker的发布列表**[https://download.docker.com/linux/centos/7/x86_64/stable/Packages/](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)


1.对于`CentOS`系统，Docker官方提供三种安装方式，这里主要说说前面两种安装方式。

第一种，主要是通过网络实时下载安装，比较简单，但是需要翻墙，公司服务器没有翻墙，所以不能使用，会出现如`Operation timed out after 30001 milliseconds with 0 out of 0 bytes received`类似的错误

第二种，主要是在本地借助梯子使用手动下载，然后scp上传到远程服务器上手动安装，具体步骤如下：

1.下载docker安装包，下载地址为[https://download.docker.com/linux/centos/7/x86_64/stable/Packages/](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)
`wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm`

2.将安装包scp到远程服务器

3.使用`yum localinstall *`安装`rpm`包，*代表当前目录的文件，因为我的docker安装包是放在当前目录的。

特别注意，在安装的时候如果遇到`Processing Dependency: container-selinux >= 2.9 for package: docker-ce-17.12.0.ce-1.el7.centos.x86_64`这种问题，请下载[container-selinux-2.36-1.gitff95335.el7.noarch.rpm](http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.36-1.gitff95335.el7.noarch.rpm)文件并上传到docker文件所在的目录使用`yum localinstall *`安装。

4.安装的过程中可能会弹出一些提示信息，如下，可以通过执行`yum-config-manager --save --setopt=docker-ce-stable.skip_if_unavailable=true`和`yum-config-manager --disable docker-ce-stable`解决类似`Operation timed out after 30001 milliseconds`的问题

```
One of the configured repositories failed (Docker CE Stable - x86_64),
 and yum doesn't have enough cached data to continue. At this point the only
 safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Run the command with the repository temporarily disabled
            yum --disablerepo=docker-ce-stable ...

     4. Disable the repository permanently, so yum won't use it by default. Yum
        will then just ignore the repository until you permanently enable it
        again or use --enablerepo for temporary usage:

            yum-config-manager --disable docker-ce-stable
        or
            subscription-manager repos --disable=docker-ce-stable

     5. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=docker-ce-stable.skip_if_unavailable=true
```

5.启动docker命令`systemctl start docker`，通过`ps -ef | grep docker`查询docker进程状态

### 其他

1.升级`sudo yum -y upgrade /path/to/package.rpm`

2.卸载命令，使用`sudo yum remove docker-ce`和`sudo rm -rf /var/lib/docker`

3.安装之后遇到类似问题`ERRO[0000] failed to dial gRPC: cannot connect to the Docker daemon. Is 'docker daemon' running on this host?: dial unix /var/run/docker.sock: connect: permission denied
context canceled`

主要是因为没有在root帐户下启动，使用下面的命令
```
su root # 先切换到root用户, 再执行以下命令
systemctl enable docker # 开机自动启动docker

systemctl start docker # 启动docker
systemctl restart docker # 重启dokcer
```

4.如果在执行docker的过程中遇到类似`docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/create: dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.`

请执行 `sudo usermod -a -G docker $USER`命令，主要是因为权限的问题


5. 如果在下载镜像时遇到`error pulling image configuration: net/http: TLS handshake timeout`，直接修改`/etc/docker/daemon.json`为国内镜像
```sh
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

6. 连接问题 `Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`
在mac上遇到这个问题，是因为mac上没有启动，直接点击docker图标启动
