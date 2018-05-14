请参看文档 [https://docs.docker.com/compose/install/#install-compose](https://docs.docker.com/compose/install/#install-compose)

安装前查看最新安装版本[https://github.com/docker/compose/releases](https://github.com/docker/compose/releases)

主要分为三步

1. 下载

```sh
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

2.权限修改
```sh
sudo chmod +x /usr/local/bin/docker-compose
```

3.查看安装结果

```sh
docker-compose --version
```
