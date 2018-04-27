### MAC上设置代理

缘由： 在golang开发中遇到某些包`被墙`，如`golang.org/x/sys/unix`和`http://golang.org/x/net/html`等，不能直接下载，需要设置代理来解决。

解决方法：**shadowsocks+polipo为终端设置代理**

```sh
# 下载
brew install polipo
# 在 ~/.bash_profile 里加一个 alias：
alias SET_SOCKS5_PROXY='polipo socksParentProxy=127.0.0.1:1086 proxyAddress=0.0.0.0'

# 在 ~/.bash_profile 里加一个 alias, 8123是执行SET_SOCKS5_PROXY得到不知道是否会出现不同
alias SET_HTTP_HTTPS_PROXY='env http_proxy=http://127.0.0.1:8123 https_proxy=http://127.0.0.1:8123'

# 启动代理
SET_SOCKS5_PROXY

# 在每次命令加上SET_HTTP_HTTPS_PROXY
SET_HTTP_HTTPS_PROXY curl https://www.youtube.com
```





