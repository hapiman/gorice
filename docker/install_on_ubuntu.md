 ### 在unbuntu上手动安装docker环境

官方安装文档 [https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

手动安装节点[https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-from-a-package](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-from-a-package)

安装软件下载页面[https://download.docker.com/linux/ubuntu/dists/](https://download.docker.com/linux/ubuntu/dists/)
直接进入`pool/stable/`目录下面选择相应的架构，一般都是`amd64的`，所以直接进入`pool/stable/amd64`选择安装包

下载完成之后，使用`sudo dpkg -i 软件包位置`安装即可

```sh
dpkg: dependency problems prevent configuration of docker-ce:
 docker-ce depends on libltdl7 (>= 2.4.2); however:
  Package libltdl7 is not installed.
 docker-ce depends on libsystemd-journal0 (>= 201); however:
  Package libsystemd-journal0 is not installed.

dpkg: error processing package docker-ce (--install):
 dependency problems - leaving unconfigured
Processing triggers for ureadahead (0.100.0-16) ...
ureadahead will be reprofiled on next reboot
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Errors were encountered while processing:
 docker-ce
```
安装`apt-get install -y libltdl7 libsystemd-journal0`


### 当安装libltdl7报错如下：
```sh
Depends: libseccomp2 (>= 2.3.0) but 2.2.3-3ubuntu3 is to be installed
Recommends: aufs-tools but it is not going to be installed
```
执行 `apt-get install -y libltdl7 libseccomp2`

使用 `sudo usermod -aG docker whoami`修改执行权限
