### linux golang环境安装

下载地址[https://golang.org/dl/](https://golang.org/dl/)

安装示例

1. 下载`wget https://dl.google.com/go/go1.10.2.linux-amd64.tar.gz`

2. 解压`tar -zxf go1.10.2.linux-amd64.tar.gz -C /usr/local/`

3. 修改GOPATH和GOHOME
```sh
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/home/golang
```
