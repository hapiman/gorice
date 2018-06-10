Hyperledger fabric V1.0的架构
如下图所示：

`application`提供各种语言的SDK接口。

`membership`也就是fabric-ca提供成员服务，用来管理身份，提供授权和认证。

`peer`负责模拟交易和记账

- Endorser（背书）用来当peer执行一个交易以后返回yes/no。
- Committer将验证过的区块追加到通道上各个账本的副本。
- Ledger就是账本啦。
- Chaincode用来编写业务逻辑，交易指令用来修改资产，可以理解为 fabric 网络对外提供的一个交互接口（智能合约）。
- Event是fabric提供的一个事件框架，比如链代码事件，拒绝事件，注册事件等，在编写代码的时候可以订阅这些事件来实现自己的业务逻辑。
- o-service用来实现共识。

交易流程
交易过程如下图所示：


Application向一个或多个peer节点发送对交易的背书请求。

Peer的endorse执行背书，但并不将结果提交到本地账本，只是将结果返回给应用。

应用收集所有背书节点的结果后，将结果广播给orderers，orderers执行共识，生成block，通过消息通道批量的将block发布给peer节点，更新lerdger。

可以看一下下面这张图，一样的过程，更加突出了多个peer背书的过程。


在介绍一下在交易过程中扮演重要角色的channel

channel是构建在Fabric网络上的私有区块链，实现了数据的隔离和保密。channel是由特定的peer所共享的，并且交易方必须通过该通道的正确验证才能与账本进行交互。

如下图所示:


可以看到不同颜色的channel隔离了不同的peer之间的通信。

