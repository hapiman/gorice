### 架构图
Kubernetes集群各模块通信方式。
![](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/structure.png)

Kubernetes集群各模块，主要分为客户端，`Master`节点，`Node`节点，`etcd`集群。
![](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/structure2.png)

Kubernetes关键字

`Master（主节点）`： 控制Kubernetes节点的机器，也是创建作业任务的地方。

`Node（节点）`： 这些机器在Kubernetes主节点的控制下执行被分配的任务。在Node上运行的服务进程包括`docker daemon`，`Kubelet` 和 `Kube-Proxy`。

`Pod`： 由一个或多个容器构成的集合，作为一个整体被部署到一个单一节点。同一个 pod 中的容器共享 IP 地址、进程间通讯（IPC）、主机名以及其它资源(如内存)。Pod 将底层容器的网络和存储抽象出来，使得集群内的容器迁移更为便捷。Kubernetes的其它组件帮助你对 pod 进行负载均衡，以保证有合适数量的容器支撑你的工作负载。Pod的生命周期是通过`Replication Controller`来管理的。在整个过程中，Pod处于4种状态之一：Pending, Running, Succeeded, Failed。

`Labels`: Service通过Label找到Pod组

`Replication controller（复制控制器）`： 控制一个 pod 在集群上运行的实例数量。确保任何时候Kubernetes集群中有指定数量的Pod副本在运行， 如果少于指定数量的Pod副本，Replication Controller会启动新的Pod，反之会杀死多余的以保证数量不变。

`Service（服务）`： 将服务内容与具体的pod分离。Kubernetes服务代理负责自动将服务请求分发到正确的pod处，不管pod移动到集群中的什么位置，甚至可以被替换掉。Service是定义一系列Pod以及访问这些Pod的策略的一层抽象。
![](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/images/srv.png)

`Kubelet`： 这个**守护进程**从master或者其他地方获取本节点需要达到什么状态, 运行在各个工作节点上，负责获取容器列表，运行的副本数量, 网络或者存储如何配置,保证被声明的容器已经启动并且正常运行。

`kubectl`： 这是 Kubernetes 的命令行配置工具。

`kube-proxy`: 实现集群网络服务负载均衡

`Volume(存储卷)`: Volume是Pod中能够被多个容器访问的共享目录。

`Namespace(命名空间)`: 通过将系统内部的对象“分配”到不同的Namespace中，形成逻辑上的不同分组，便于在共享使用整个集群的资源同时还能分别管理。

`Annotation(注解)`：与Label类似，但Label定义的是对象的元数据，而Annotation则是用户任意定义的“附加”信息。

### Kubernetes关键组件

#### Master三个组件：

第一、`apiserver`[资源操作入口]

作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。
它维护的REST对象将持久化到etcd（一个分布式强一致性的key/value存储）。

第二、`scheduler`[集群分发调度器]

集群调度器,负责集群的资源调度，为新建的Pod分配机器。

1.scheduler收集和分析当前Kubernetes集群中所有Node节点的资源(内存、CPU)负载情况，然后依此分发新建的Pod到Kubernetes集群中可用的节点。

2.实时监测Kubernetes集群中未分发和已分发的所有运行的Pod。

3.scheduler也监测Node节点信息，由于会频繁查找Node节点，Scheduler会缓存一份最新的信息在本地。

4.scheduler在分发Pod到指定的Node节点后，会把Pod相关的信息`Binding`写回API Server

第三、`controller-manager`[内部管理控制中心]

负责执行各种控制器，目前有两类：

(1) `endpoint-controller`：定期关联service和Pod(关联信息由endpoint对象维护)，保证service到Pod的映射总是最新的。

(2) `replication-controller`：定期关联replicationController和Pod，保证replicationController定义的复制数量与实际运行Pod的数量总是一致的。

#### Node两个组件：

![kubelet结构图](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/node.png)

第一、`kubelet`[节点上的Pod管家]：

1.负责Node节点上pod的创建、修改、监控、删除等全生命周期的管理

2.定时上报本Node的状态信息给API Server。

3.kubelet是Master和Node之间的桥梁，接收Master分配给它的commands和work，通过kube-apiserver间接与Etcd集群交互，读取配置信息。

第一、`proxy`[负载均衡、路由转发]：

1.Proxy是为了解决外部网络能够访问跨机器集群中容器提供的应用服务而设计的，运行在每个Node上。
Proxy提供`TCP/UDP sockets`的proxy，每创建一种Service，Proxy主要从etcd获取Services和Endpoints的配置信息），然后根据配置信息在Node上启动一个Proxy的进程并监听相应的服务端口，当外部请求发生时，Proxy会根据`Load Balancer`将请求分发到后端正确的容器处理。

2.Proxy不但解决了同一主宿机相同服务端口冲突的问题，还提供了Service转发服务端口对外提供服务的能力，Proxy后端使用了随机、轮循负载均衡算法。

### k8s执行流程

![master work flow](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/master.png)

1.Kubecfg将特定的请求，比如创建Pod，发送给Kubernetes Client。

2.Kubernetes Client将请求发送给API server。

3.API Server根据请求的类型，比如创建Pod时storage类型是pods，然后依此选择何种REST Storage API对请求作出处理。

4.REST Storage API对的请求作相应的处理。

5.将处理的结果存入高可用键值存储系统Etcd中。

6.在API Server响应Kubecfg的请求后，Scheduler会根据Kubernetes Client获取集群中运行Pod及Node信息。

7.依据从Kubernetes Client获取的信息，Scheduler将未分发的Pod分发到可用的Minion/Node节点上。

### 参考文档

- [深入学习Kubernetes(二):基础组件与概念](https://lihaoquan.me/2017/3/8/get-concepts-in-kubernetes.html)
- [k8s是什么](https://github.com/hapiman/gorice/blob/master/k8s/k8s%E6%98%AF%E4%BB%80%E4%B9%88.md)

