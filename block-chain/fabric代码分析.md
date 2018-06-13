### 关于依赖镜像的说明

`fabric-ccenv`用于构建链码，当前包括`github.com/hyperledger/fabric/core/chaincode/shim ("shim")`包
`fabric-baseimage`, `fabric-zookeeper`, `fabric-kafka`, `fabric-couchdb`, `fabric-baseos`作为基础镜像，当前的版本皆为`x86_64-0.4.6`


`release/darwin-amd64/bin` 或者 `release/linux-amd64/bin`下面的可执行文件：
`configtxgen`, `configtxlator`, `cryptogen`, `orderer`, `peer`, `get-docker-images.sh`

`PKI` 公钥基础设施


`bccsp`全称`Blockchain crypto provider`选择哪一种加密算法使用

`bcssp`加密服务代码，用来提供区块链相关的算法标准和他们的实现，BCCSP通过Membership Service Provider（成员服务提供者）给相关核心功能和客户端SDK提供加密算法相关的服务。
`common`全局公用代码
`core`核心功能代码，如共识模块，背书模块等
`events`事件代码，用户生产和消费信息
`gossip`实现gossip协议，是一种可达到去中心化，有一定容错能力且可达到最终一致的传播算法
`msp` 会员服务代码目录 (Member Service Provider)
`order` 用于消息的订阅与分发处理
`protos` 包括各种协议和消息的protobuf
`gotools` golang开发相关工具安装
`bddtests` 测试模块，含有大量`bdd`测试用例
`devenv` 配置开发环境

### 研究方向和模块

前提：熟悉`makefile`,`shell`命令，`docker-composer`的`yaml`配置文件

1. 搭建`kafka`和`zookeeper`集群

2. [`fabric CA`](https://hyperledgercn.github.io/hyperledgerDocs/ca-setup_zh/)(MSP Membership Service Provider)的运行原理, 将颁发与校验证书，以及用户认证背后的所有密码学机制与协议都抽象了出来。一个MSP可以自己定义身份，以及身份的管理（身份验证）与认证（生成与验证签名）规则；MSP抽象提供：`具体的身份格式`,`用户证书验证`, `用户证书撤销`, `签名生成和验证`。`Fabric-CA`用于生成证书和密钥，以真正的初始化MSP，`Fabric-CA`是用于身份管理的MSP接口的默认实现。 即，MSP只是一个接口，Fabric-CA是MSP接口的一种实现。

3. 加密算法服务`bcssp`

4. 去中心化网络协议`gossip`

5. `peer`及`order`逻辑

6. `chaincode`
