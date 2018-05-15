### 安装最新版本

点击https://git-scm.com/download/linux 页面中`on kernel.org`
git仓库列表[https://mirrors.edge.kernel.org/pub/software/scm/git/](https://mirrors.edge.kernel.org/pub/software/scm/git/)
安装官方文档

准备条件
- 更新 `sudo yum update`
- 安装预设 `yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel`

1. 删除 `yum remove git`

2. 下载 `wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz`

3. 解压 `tar -zxvf git-2.9.5.tar.gz &&  cd git-2.9.5`

4. 编译 `make prefix=/usr/local all`

5. 再编译 `sudo make prefix=/usr/local install`

好像第4 5步可以将git命令加在全局中，可直接使用git;
如果没有起作用，使用下面命令
```sh
sudo vim /etc/profile
# 在文件中增加export PATH=/usr/local/git/bin:$PATH行，加到全局路径中
source /etc/profile
```
·
