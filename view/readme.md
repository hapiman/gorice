### 使用golang的公司及业务

#### 360消息推送系统

360消息推送系统服务于50+内部产品，万款开发平台App，实时长连接数亿量级，日独数十亿量级，1分钟内可以实现亿量级广播，日下发峰值百亿量级，400台物理机，3000多个实例分布在9个独立集群中，每个集群跨国内外近10个IDC

#### 今日头条内部服务

当前后端服务超过80%的流量是跑在 Go 构建的服务上。微服务数量超过100个，高峰 QPS 超过700万，日处理请求量超过3000亿；

#### 哔哩哔哩

对外统一的API Gateway，对内的运营平台，以及各种数据总线、Proxy、微服务框架、IM等中间件和基础设施等全都是Go开发，是国内对微服务架构一个非常不错的实践

### 京东消息推送

http://www.flysnow.org/2017/09/13/go-for-company.html

#### 猎豹移动消息推送

[goim](https://github.com/Terry-Mao/goim)

#### 滴滴

#### 区块链

[200行代码实现区块链](https://github.com/hapiman/blockchain-tutorial)

[区块链来了，Go工程师成为升值最快的职业](https://mp.weixin.qq.com/s?__biz=MjM5OTcxMzE0MQ==&mid=2653370723&idx=1&sn=e753afa14d8abf9e82521359dfe97975&chksm=bce4d9798b93506fe11b45b0d1213af4ca0b2f365bb458c98068a608e3dcd928d9786751a494&scene=0&key=4808987c5be8f35bba13b51a6f86e5148064ac5c05d3db71c9c92f629a070b1d50e9604ebc2a5605d7e519dadf03a83e02478e6acbf898176d92c9e6044f2292fb761577438a7290ac027d94b420104d&ascene=0&uin=MTA3NjI4MDE4MA%3D%3D&devicetype=iMac+MacBookPro11%2C1+OSX+OSX+10.12.6+build(16G29)&version=12020810&nettype=WIFI&lang=zh_CN&fontScale=100&pass_ticket=STpbAI7BdPuA16AoaMNCon%2BT3uSJOFlOzMyPXm1ZQkocdG9rUPxtwYboN9bEHIh%2F)

[go-ethereum](https://github.com/ethereum/go-ethereum)

[fabric](https://github.com/hyperledger/fabric)

[tendermint](https://github.com/tendermint/tendermint)

#### k8s

[Kubernetes基础篇：主要特性、基本概念与总体架构](http://shiyanjun.cn/archives/1671.html)
> Kubernetes是一个开源的容器编排引擎，它支持自动化部署、大规模可伸缩、应用容器化管理。我们在完成一个应用程序的开发时，需要冗余部署该应用的多个实例，同时需要支持对应用的请求进行负载均衡，在Kubernetes中，我们可以把这个应用的多个实例分别启动一个容器，每个容器里面运行一个应用实例，然后通过内置的负载均衡策略，实现对这一组应用实例的管理、发现、访问，而这些细节都不需要应用开发和运维人员去进行复杂的手工配置和处理。
