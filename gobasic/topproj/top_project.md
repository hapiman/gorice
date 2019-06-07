# 我知道的当前（2019年）最火的golang项目
记录平时看到或者接触到的`golang`用的人较多或者出镜率比较高的项目。

如果我漏了，谢谢帮我指出，我尽快补上。

每一个项目都配上了关注度（star的数量）的图片。

整理过程中收获：

1、了解到docker项目已经改名为moby，另外仓库做了迁移，当前为：`https://github.com/moby/moby`

2、了解到`时间序列数据库`，就是每一个数据都时间属性

# 项目列表

## Gim
仓库地址：`https://github.com/gin-gonic/gin`

`Gin`是用Go语言实现的一款web框架。它的特点和`Martini`类似，但是API的性能更好，大概快40倍。如果你对性能要求极高，尝试一下Gin，不会让你失望。
![](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/gin.png)
## beego

仓库地址：`https://github.com/astaxie/beego`
![](https://beego.me/static/img/beego_purple.png)
一个使用 Go 的思维来帮助您构建并开发 Go 应用程序的开源框架。

一个快速开发 Go 应用的 HTTP 框架，他可以用来快速开发 API、Web 及后端服务等各种应用，是一个 RESTful 的框架，主要设计灵感来源于 tornado、sinatra 和 flask 这三个框架，但是结合了 Go 本身的一些特性（interface、struct 嵌入等）而设计的一个框架。

框架特性：`简单化`，`智能化`，`模块化`，`高性能`。
![](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/beego.png)

## Caddy: Every Site on HTTPS
仓库地址：`https://github.com/mholt/caddy`

已经是一款可以用于生产的开源服务器，具有速度快，易使用，生产效率高的特点。当前已经可以在`Windows`, `Mac`, `Linux`, `BSD`, `Solaris`, and `Android`使用。

![](https://user-images.githubusercontent.com/1128849/36338535-05fb646a-136f-11e8-987b-e6901e717d5a.png)

具有如下特点：

1. 使用`Caddyfile`方便配置

2. Auto HTTPS Caddy 使用 Let’s Encrypt 让你的站点全自动变成全站HTTPS，无需任何配置。当然你想使用自己的证书也是可以的。

3. HTTP/2 全自动支持HTTP/2协议，无需任何配置。

4. 主机虚拟化使多个站点工作

5. 可使用插件扩展m

6. 无需依赖即可运行

7. 为了保证安全连接，使用了TLS session ticket key rotation

![](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/caddy.png)

## nsq

仓库地址：`https://github.com/nsqio/nsq`

实时分发的消息平台，用于极大规模的数据处理，处理量级10亿+。

它提升了分布式和去中心化的拓扑结构，没有单点故障，支持容错和高可用性，并保证消息传递的可靠性。

在操作上，NSQ易于配置和部署（所有参数都在命令行上指定，编译后的二进制文件没有运行时依赖项）。为了获得最大的灵活性，它与数据格式无关（消息可以是JSON、MSGPack、协议缓冲区或其他任何格式）。官方的go和python库是现成的（以及许多其他客户机库），如果您有兴趣构建自己的库，这就是一个协议规范。

![](https://camo.githubusercontent.com/5899f86a964cae96e599de9db4449e3294f104b4/687474703a2f2f6e73712e696f2f7374617469632f696d672f6e73715f626c75652e706e67)

## Hugo

仓库地址：`https://github.com/gohugoio/hugo`

一个静态的，可伸缩的静态网页生成器，宣称世界上最快的建站框架，不过这点和`wordpress`怎么比呢。

`Go`语言编写的静态网站生成器，速度快，易用，可配置

`Hugo`获取一个包含内容和模板的目录，并将其呈现为完整的HTML网站。

![](https://raw.githubusercontent.com/gohugoio/hugoDocs/master/static/img/hugo-logo.png)

## gogs

仓库地址：`https://github.com/gogs/gogs`

`Gogs`是一款极易搭建的自助`Git`服务。

该项目旨在打造一个以最简便的方式搭建简单、稳定和可扩展的自助Git服务。

使用Go语言开发使得Gogs能够通过独立的二进制分发，并且支持Go语言支持的 所有平台，包括 Linux、macOS、Windows 以及 ARM 平台。

![](https://github.com/gogs/gogs/raw/master/public/img/gogs-large-resize.png?raw=true)

## frp

仓库地址：`https://github.com/fatedier/frp`

`frp`是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp 协议，为 http 和 https 应用协议提供了额外的能力，且尝试性支持了点对点穿透。

![](https://github.com/fatedier/frp/raw/master/doc/pic/architecture.png)

## proxypool

仓库地址：`https://github.com/henson/proxypool`

采集免费的代理资源为爬虫提供有效的IP代理

设计架构：

**Getter**：代理获取接口，目前有9个免费代理源，每调用一次就会抓取这些网站最新的100个代理放入Channel，可自行添加额外的代理获取接口；

**Channel**：临时存放采集来的代理，通过访问稳定的网站去验证代理的有效性，有效则存入数据库；

**Schedule**：用定时的计划任务去检测数据库中代理IP的可用性，删除不可用的代理。同时也会主动通过Getter去获取最新代理；

**Api**：代理池的访问接口，提供get接口输出JSON，方便爬虫直接使用。

## lantern

仓库地址：`https://github.com/getlantern/lantern`

区别于，`SS`，它是分布式的，点对点的，通过蓝灯，你可以和自由上网的用户共享网络，对方可以自由上网，你也就自由了。

## shadowsocks-go

仓库地址：`https://github.com/shadowsocks/shadowsocks-go`

`SS`的Go语言实现。

## syncthing

仓库地址：`https://github.com/syncthing/syncthing`

`Syncthing`是一个持续不断的文件同步项目，能够在两台或者多台电脑上同步文件，使用了其独有的对等自由块交换协议，速度极快。

下面列出是这个项目的目标计划，重要性依次递减。当然这只是其中一部分目标，如果想查看更多，可以去看目标文档。

**确保数据的安全性**

保护用户的数据是责无旁贷，该项目采取所有的合理的预防措施来避免用户的文件损坏。

**确保数据不被攻击**

不循序任何未经授权方的窃听或修改。

**易于使用**

**自动化**

**能够在大多数通用的电脑上使用**

![](https://github.com/syncthing/syncthing/raw/master/assets/logo-text-128.png)

## kubernetes

仓库地址：`https://github.com/kubernetes/kubernetes`

容器编排工具，实现自动化部署,更新,下线,负载均衡,容错处理等。

三个特点：

**优化部署**：快速而有预期地部署你的应用, 极速地扩展你的应用,增加项目的实例,能够实现自动布局、自动重启、自动复制、自动伸缩,并实现应用的`状态检查`与`自我修复`。

**优化资源利用**：跨主机编排容器, 更充分地利用硬件资源来最大化地满足企业应用的需求

**声明式配置**：`etcd`声明式的容器管理，保证所部署的应用按照我们部署的方式运作.

![](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/k8s-lg.png)

## etcd

仓库地址：`https://github.com/etcd-io/etcd`

![etcd](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/etcd-lg.png)

分布式可靠的键值存储，尤其对分布式系统中极其重要的数据，其特点：

`Simple`: API设计合理，面向用户

`Secure`: 自动的TLS连接，支持客户定制认证

`Fast`: 写入能力大于1w+每秒

`Reliable`: 使用Raft恰当的分发

`etcd`当前频繁的和`Kubernetes`,`locksmith`,`vulcand`, `Doorman`等项目配合使用。

## moby

仓库地址：`https://github.com/moby/moby`

该项目的作用是在容器化生态中组装容器的，以前的大名叫做：`docker`，这个大家都知道。后来经过一段纠结的时刻，改名字了，原因[在这儿](https://www.zhihu.com/question/58805021)

![docker](https://raw.githubusercontent.com/moby/moby/master/docs/static_files/moby-project-logo.png)

Moby是一个以强大的原则为指导的开放式项目，旨在维持模块化和灵活性，对用户经验没有太强的意见。

模块化：该项目包括许多组件，优秀的函数和API共同协作。

可交换：Moby包含足够的组件来构建功能齐全的容器系统，但其模块化架构确保大部分组件可以通过不同的实现进行交换。

可用安全性：Moby提供安全的默认值，而不影响可用性。

以开发人员为中心
## Traefik

仓库地址：`https://github.com/containous/traefik`

`Traefik`是一款开源的反向代理与负载均衡工具。它最大的优点是能够与常见的微服务系统直接整合，可以实现自动化动态配置。
目前支持 Docker、Swarm、Mesos/Marathon、 Mesos、Kubernetes、Consul、Etcd、Zookeeper、BoltDB、Rest API 等等后端模型。

![](https://blog.qikqiak.com/img/posts/traefik-architecture.png)

## influxdb

仓库地址：`https://github.com/influxdata/influxdb`

influxdb是目前比较流行的时间序列数据库。

> 时间序列数据库：最简单的定义就是数据格式里包含Timestamp字段的数据，几乎所有的数据其实都可以打上一个Timestamp字段。时间序列数据的更重要的一个属性是如何去查询它，包括数据的过滤，计算等等。

Influxdb是一个开源的分布式时序、时间和指标数据库，使用go语言编写，无需外部依赖。
它有三大特性：

`时序性（Time Series）`：与时间相关的函数的灵活使用（诸如最大、最小、求和等）；

`度量（Metrics）`：对实时大量数据进行计算；

`事件（Event）`：支持任意的事件数据，换句话说，任意事件的数据我们都可以做操作。

![influxdb](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/influxdb-lg.png)

## prometheus

仓库地址： `https://github.com/prometheus/prometheus`

一个开源的服务监控系统和时间序列数据库。

![](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/prometheus-lg.png)


**首先需要明白：**
`Prometheus`提供的是一整套监控体系, 包括数据的采集，数据存储，报警，甚至是绘图(只不过很烂，官方也推荐使用 grafana)。
而`InfluxDB`只是一个时序数据库。同为时间序列数据库，两者对比：[prometheus和influxdb对比](https://www.jianshu.com/p/d666ee3147d2)

## Grafana

仓库地址：`https://github.com/grafana/grafana`

Grafana是一款开源的，具有功能丰富的度量标准仪表板和图形编辑器，用于Graphite，Elasticsearch，OpenTSDB，Prometheus和InfluxDB等数据显示，定制化高。

![](https://raw.githubusercontent.com/hapiman/gorice/master/gobasic/topproj/images/grafana-lg.png)

## go-ethereum

仓库地址：`https://github.com/ethereum/go-ethereum`

以太坊协议的官方Go语言实现。

## fabric

仓库地址：`https://github.com/hyperledger/fabric`

区块链超级账本`Hyperledger Fabric`实现，用于联盟链开发。

## Drone

仓库地址：`https://github.com/drone/drone`

Drone是一种基于容器技术的持续交付系统。Drone使用简单的YAML配置文件（docker-compose的超集）来定义和执行Docker容器中的Pipelines。

Drone与流行的源代码管理系统无缝集成，包括GitHub，GitHub Enterprise，Bitbucket等。
