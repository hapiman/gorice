### 引用库
iOS推送使用库 [apns2](https://github.com/sideshow/apns2),优化方式[APNS HTTP 2 Push Speed](https://github.com/sideshow/apns2/wiki/APNS-HTTP-2-Push-Speed)

[第三方buford](https://github.com/RobotsAndPencils/buford)，参考文档[基于http2的apns的push发送平台](https://blog.csdn.net/rodgexue/article/details/54290676)

`API`接口框架[gin](https://github.com/gin-gonic/gin)


### 部署方式

docker，实现多实例，参考[使用docker构建golang的线上环境](https://www.cnblogs.com/jackluo/p/7819975.html)

[docker部署go应用](http://bazingafeng.com/2017/09/14/deploying-a-go-application-in-docker/)

### 其它

1.10个实例依次启动，启动10个镜像，挂掉之后，自动重启

2.运维协助只要有`docker`就可以

3.优化，将来可以加入`k8s`

### 疑问

负载均衡，对于多个实例，是如何处理端口的。

pm2是否会持续启动

构建镜像文件大，速度问如何解决,[gin](https://github.com/codegangsta/gin)会检测代码的变动和更改
