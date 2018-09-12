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
如果没有生效, 退出当前用户, 即可生效

### 问题
- 问题1
```
Can't locate ExtUtils/MakeMaker.pm in @INC (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5 .) at Makefile.PL line 3.
BEGIN failed--compilation aborted at Makefile.PL line 3.
make[1]: *** [perl.mak] Error 2
make: *** [perl/perl.mak] Error 2
```
执行安装命令: `yum install perl-ExtUtils-MakeMaker package`

- 问题2
```
GIT_VERSION = 2.9.5
    * new build flags
    CC credential-store.o
In file included from cache.h:4:0,
                 from credential-store.c:1:
git-compat-util.h:280:25: fatal error: openssl/ssl.h: No such file or directory
 #include <openssl/ssl.h>
                         ^
compilation terminated.
make: *** [credential-store.o] Error 1
```
