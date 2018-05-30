## Ubuntu16.04搭建Hyperledger Fabric 1.1开发环境

### 一、先决条件
1.安装docker和docker-compose
参考博客：[http://blog.csdn.net/diligent_lee/article/details/79098302](http://blog.csdn.net/diligent_lee/article/details/79098302)

2.安装node
只需要两条命令：

```sh
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```
在安装node的同时会自动安装npm。安装完毕之后通过以下命令验证安装：
```sh
node -v
npm -v
```
参考博客：[http://blog.csdn.net/w20101310/article/details/73135388](http://blog.csdn.net/w20101310/article/details/73135388)

3.安装go及其环境变量配置
参考 [https://github.com/hapiman/gorice/blob/master/gobasic/install.md](https://github.com/hapiman/gorice/blob/master/gobasic/install.md)

### 二、下载Fabric
1.创建工作目录
```sh
mkdir -p $GOPATH/src/github.com/hyperledger/
```
2.下载Fabric
```sh
cd $GOPATH/src/github.com/hyperledger
git clone https://github.com/hyperledger/fabric.git
```

### 三、运行Fabric实例
输入以下命令切换工作路径：

```sh
cd ./fabric/scripts/
ls
./bootstrap.sh
```
下载时间较长。。。下载时间较长。。。
```sh
===> List out hyperledger docker images
hyperledger/fabric-tools       latest              6a8993b718c8        6 weeks ago         1.33GB
hyperledger/fabric-tools       x86_64-1.0.5        6a8993b718c8        6 weeks ago         1.33GB
hyperledger/fabric-couchdb     latest              9a58db2d2723        6 weeks ago         1.5GB
hyperledger/fabric-couchdb     x86_64-1.0.5        9a58db2d2723        6 weeks ago         1.5GB
hyperledger/fabric-kafka       latest              b8c5172bb83c        6 weeks ago         1.29GB
hyperledger/fabric-kafka       x86_64-1.0.5        b8c5172bb83c        6 weeks ago         1.29GB
hyperledger/fabric-zookeeper   latest              68945f4613fc        6 weeks ago         1.32GB
hyperledger/fabric-zookeeper   x86_64-1.0.5        68945f4613fc        6 weeks ago         1.32GB
hyperledger/fabric-orderer     latest              368c78b6f03b        6 weeks ago         151MB
hyperledger/fabric-orderer     x86_64-1.0.5        368c78b6f03b        6 weeks ago         151MB
hyperledger/fabric-peer        latest              c2ab022f0bdb        6 weeks ago         154MB
hyperledger/fabric-peer        x86_64-1.0.5        c2ab022f0bdb        6 weeks ago         154MB
hyperledger/fabric-javaenv     latest              50890cc3f0cd        6 weeks ago         1.41GB
hyperledger/fabric-javaenv     x86_64-1.0.5        50890cc3f0cd        6 weeks ago         1.41GB
hyperledger/fabric-ccenv       latest              33feadb8f7a6        6 weeks ago         1.28GB
hyperledger/fabric-ccenv       x86_64-1.0.5        33feadb8f7a6        6 weeks ago         1.28GB
hyperledger/fabric-ca          latest              002c9089e464        6 weeks ago         238MB
hyperledger/fabric-ca          x86_64-1.0.5        002c9089e464        6 weeks ago         238MB
```

并且，这个脚本运行完毕之后，我们会发现当前目录下多了一个bin文件夹，这个文件夹中所包含的可执行文件正是我们运行例子所需要的。现在我们需要把这个文件夹复制到我们需要运行的例子的上层目录中，输入命令：
```sh
cp -rf bin/ ./../examples/
```

运行例子：**e2e_cli**
```sh
cd ./../examples/e2e_cli/
./network_setup.sh up
```
正确运行结果：

===================== All GOOD, End-2-End execution completed =====================
```sh
 _____   _   _   ____            _____   ____    _____
| ____| | \ | | |  _ \          | ____| |___ \  | ____|
|  _|   |  \| | | | | |  _____  |  _|     __) | |  _|
| |___  | |\  | | |_| | |_____| | |___   / __/  | |___
|_____| |_| \_| |____/          |_____| |_____| |_____|
```

`ctrl+c` 关闭窗口

```sh
# docker日志
8c090ce9f968        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   24 minutes ago      Up 25 minutes                                                                                   dev-peer1.org2.example.com-mycc-1.0
378dc9e2e34a        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   25 minutes ago      Up 25 minutes                                                                                   dev-peer0.org1.example.com-mycc-1.0
d315253e4124        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   25 minutes ago      Up 26 minutes                                                                                   dev-peer0.org2.example.com-mycc-1.0
bb8bf09e8947        hyperledger/fabric-tools                                                                               "/bin/bash -c './scr…"   26 minutes ago      Up 27 minutes                                                                                   cli
7b2ca6b79f74        hyperledger/fabric-orderer                                                                             "orderer"                26 minutes ago      Up 27 minutes       0.0.0.0:7050->7050/tcp                                                      orderer.example.com
66bb21136868        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   26 minutes ago      Up 27 minutes       9093/tcp, 0.0.0.0:32793->9092/tcp                                           kafka2
45fe10aa4faa        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   26 minutes ago      Up 27 minutes       9093/tcp, 0.0.0.0:32792->9092/tcp                                           kafka0
d4c5d712e892        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   26 minutes ago      Up 27 minutes       9093/tcp, 0.0.0.0:32791->9092/tcp                                           kafka3
a10d2ac8d83f        hyperledger/fabric-kafka                                                                               "/docker-entrypoint.…"   26 minutes ago      Up 27 minutes       9093/tcp, 0.0.0.0:32790->9092/tcp                                           kafka1
46daff66628d        hyperledger/fabric-peer                                                                                "peer node start"        26 minutes ago      Up 27 minutes       0.0.0.0:7051-7053->7051-7053/tcp                                            peer0.org1.example.com
6d0c16ecccda        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   26 minutes ago      Up 27 minutes       0.0.0.0:32789->2181/tcp, 0.0.0.0:32786->2888/tcp, 0.0.0.0:32783->3888/tcp   zookeeper2
3185b6f5e1ce        hyperledger/fabric-peer                                                                                "peer node start"        26 minutes ago      Up 27 minutes       0.0.0.0:10051->7051/tcp, 0.0.0.0:10052->7052/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com
aea879a18233        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   26 minutes ago      Up 27 minutes       0.0.0.0:32788->2181/tcp, 0.0.0.0:32785->2888/tcp, 0.0.0.0:32782->3888/tcp   zookeeper1
71d64005adab        hyperledger/fabric-zookeeper                                                                           "/docker-entrypoint.…"   26 minutes ago      Up 27 minutes       0.0.0.0:32787->2181/tcp, 0.0.0.0:32784->2888/tcp, 0.0.0.0:32781->3888/tcp   zookeeper0
954ab230a77c        hyperledger/fabric-peer                                                                                "peer node start"        26 minutes ago      Up 27 minutes       0.0.0.0:9051->7051/tcp, 0.0.0.0:9052->7052/tcp, 0.0.0.0:9053->7053/tcp      peer0.org2.example.com
e47e9b5b9346        hyperledger/fabric-peer                                                                                "peer node start"        26 minutes ago      Up 27 minutes       0.0.0.0:8051->7051/tcp, 0.0.0.0:8052->7052/tcp, 0.0.0.0:8053->7053/tcp      peer1.org1.example.com

```
`./network_setup.sh down` 关闭实例，同时结束docker中运行的container

### 四、手动测试Fabric网络

启动环境`./network_setup.sh up`

使用官方提供的小例子，`channel`名字为`mychannel`，链码的名字为`mycc`；
在命令行中执行 `docker exec -it cli bash`，此时用户为`root@bb8bf09e8947`，在`/opt/gopath/src/github.com/hyperledger/fabric/peer`目录下
查询a用户余额 `peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'`, 当前余额为：90
```
root@bb8bf09e8947:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
2018-05-30 07:10:52.283 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2018-05-30 07:10:52.283 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2018-05-30 07:10:52.283 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2018-05-30 07:10:52.283 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2018-05-30 07:10:52.283 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 005 java chaincode disabled
2018-05-30 07:10:52.284 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0ABF070A6708031A0C08FC9CB9D80510...6D7963631A0A0A0571756572790A0161
2018-05-30 07:10:52.284 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: BF25D922A81FDD09E9D0A97A56398BC435B643F2D4E9803EFFA6FC47A6E700F9
Query Result: 90
2018-05-30 07:10:52.312 UTC [main] main -> INFO 008 Exiting.....
root@bb8bf09e8947:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -c '{"Args":["invoke","a","b","50"]}'
```

执行转账操作 `peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -c '{"Args":["invoke","a","b","50"]}'`
```sh
root@bb8bf09e8947:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -c '{"Args":["invoke","a","b","50"]}'
2018-05-30 07:11:21.363 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2018-05-30 07:11:21.363 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2018-05-30 07:11:21.375 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2018-05-30 07:11:21.375 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2018-05-30 07:11:21.376 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 005 java chaincode disabled
2018-05-30 07:11:21.377 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0ABF070A6708031A0C08999DB9D80510...696E766F6B650A01610A01620A023530
2018-05-30 07:11:21.378 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: 5859313706C79A40C06FB1FB8D757108EA1E2C14BB09B72AB3667FEA2D8A0FAF
2018-05-30 07:11:21.413 UTC [msp/identity] Sign -> DEBU 008 Sign: plaintext: 0ABF070A6708031A0C08999DB9D80510...54774A86FC8F3118170546CAB2BAF2A9
2018-05-30 07:11:21.413 UTC [msp/identity] Sign -> DEBU 009 Sign: digest: 39C154E0D23EB7B8EACFF822597B984A38178CFE423FE67A31B9BA834A2191A5
2018-05-30 07:11:21.446 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 00a ESCC invoke result: version:1 response:<status:200 message:/"OK" > payload:"\n \3508\202\300\016a\334XN\r\356lO\217-\320\245Y\254!\202=\214\177R\2442\026L\037n\360\022Y\nE\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\003\022-\n\004mycc\022%\n\007\n\001a\022\002\010\004\n\007\n\001b\022\002\010\004\032\007\n\001a\032\00240\032\010\n\001b\032\003260\032\003\010\310\001\"\013\022\004mycc\032\0031.0" endorsement:<endorser:"\n\007Org1MSP\022\252\006-----BEGIN CERTIFICATE-----\nMIICKDCCAc+gAwIBAgIRAKKlj8viViRBo9jvCiPQT5gwCgYIKoZIzj0EAwIwczEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh\nLm9yZzEuZXhhbXBsZS5jb20wHhcNMTgwNTMwMDcwMjU1WhcNMjgwNTI3MDcwMjU1\nWjBqMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN\nU2FuIEZyYW5jaXNjbzENMAsGA1UECxMEcGVlcjEfMB0GA1UEAxMWcGVlcjAub3Jn\nMS5leGFtcGxlLmNvbTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABHLoYXAbLWdH\nik/fvtT5yiqx2QpwtJIjWaiH3YUFxPdLCWBjWCMo/E5futnGI1DtehYr96f8qQpQ\nDKyCBNCFfhijTTBLMA4GA1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1Ud\nIwQkMCKAIByC57MA9p6Nn3fp1JUuXWgMsLXw7PfYKWDgFDbjuK3sMAoGCCqGSM49\nBAMCA0cAMEQCIAtuiOL1MGhFb+SA4c13KY9m5lw/5FmbUVFqC5sYn4VKAiA40xnK\nyRpyqzJ0pOcTy4rYCQHAbn80TVmXAGMhBDrusg==\n-----END CERTIFICATE-----\n" signature:"0E\002!\000\312\324\354\243\272\377\201(f\361L\035/\r\\T\206\250\206\242\271\220\213i\245\217l#\t\\\001\307\002 P,^\243\205\003\274\004\202\323\036B\273S\264\317TwJ\206\374\2171\030\027\005F\312\262\272\362\251" >
2018-05-30 07:11:21.447 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00b Chaincode invoke successful. result: status:200
2018-05-30 07:11:21.448 UTC [main] main -> INFO 00c Exiting.....
```

再次查看用户余额 `peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'`，当前余额为40
```sh
root@bb8bf09e8947:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
2018-05-30 07:13:35.938 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
2018-05-30 07:13:35.938 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
2018-05-30 07:13:35.938 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
2018-05-30 07:13:35.938 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
2018-05-30 07:13:35.939 UTC [chaincodeCmd] getChaincodeSpec -> DEBU 005 java chaincode disabled
2018-05-30 07:13:35.939 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0ABF070A6708031A0C089F9EB9D80510...6D7963631A0A0A0571756572790A0161
2018-05-30 07:13:35.939 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: 1525C65DEA36A36E97ECDE6E4CF35E1F171B18F72083A48C2FCD6BFA963DE19C
Query Result: 40
2018-05-30 07:13:35.962 UTC [main] main -> INFO 008 Exiting.....
```

关闭`fabric`网络

1. 退出命令行`exit`

2. 结束整个网络
```sh
cd ~/go/src/github.com/hyperledger/fabric/examples/e2e_cli
./network_setup.sh down
```

3.退出日志
```sh
e2e_cli git:(release-1.1) ✗ ./network_setup.sh down
setting to default channel 'mychannel'
WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
Stopping cli                    ... done
Stopping orderer.example.com    ... done
Stopping kafka2                 ... done
Stopping kafka0                 ... done
Stopping kafka3                 ... done
Stopping kafka1                 ... done
Stopping peer0.org1.example.com ... done
Stopping zookeeper2             ... done
Stopping peer1.org2.example.com ... done
Stopping zookeeper1             ... done
Stopping zookeeper0             ... done
Stopping peer0.org2.example.com ... done
Stopping peer1.org1.example.com ... done
Removing cli                    ... done
Removing orderer.example.com    ... done
Removing kafka2                 ... done
Removing kafka0                 ... done
Removing kafka3                 ... done
Removing kafka1                 ... done
Removing peer0.org1.example.com ... done
Removing zookeeper2             ... done
Removing peer1.org2.example.com ... done
Removing zookeeper1             ... done
Removing zookeeper0             ... done
Removing peer0.org2.example.com ... done
Removing peer1.org1.example.com ... done
Removing network e2ecli_default
8c090ce9f968
378dc9e2e34a
d315253e4124
Untagged: dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab:latest
Deleted: sha256:15edfdb2d9ff7b57bbfedf74018e4449b0916e0b90d4686fdf6d8c42038b29c3
Deleted: sha256:a89e77fcb4f672fb26ede6ab15d0a2951e401c62443fb6fac7bb9a3032d90251
Deleted: sha256:50f682c346c4e7600b5423f379bcad5ac898fd31461cc2b52274ba7e1eb9dd42
Deleted: sha256:7f24a0a2e9ec445f6448914ea9d5dfacb89c0951c2cf9c84565ecc41fe7119d7
Untagged: dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9:latest
Deleted: sha256:a24db30e3ee464e451abef3f6217248cb572b33a3fc7d8fece55be1f22bfa2b8
Deleted: sha256:29d615f24ab4d2392eae8ff96227310ca7bc587efd71444479ac1a9065ce6e61
Deleted: sha256:47b5237cc479562c3f0af8f837235b540dc8d1e36477605933bfc60906e985ce
Deleted: sha256:02d3633db929577d6daffde750b2b32a47af54e2c4d11a65b098cb662a620e57
Untagged: dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b:latest
Deleted: sha256:cd850d0eb31aed8a2327023009d27f38772572afab7c428bbbe5fc244a889165
Deleted: sha256:eebd0d49d8bc51edd298b1e73bf23618c5ad524c5279afee321dfba05f01ad1f
Deleted: sha256:9ca3c412833a48886cde42ce851c18fc12bf5e19690e37b626ce22bf80d9d0da
Deleted: sha256:b8f31538023781df032abb68eb7b1ccb91e5ed1ac0a1f231522b84ff752eebf2
```
