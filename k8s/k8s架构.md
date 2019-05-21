### 架构图
![](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/structure.png)
![](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/structure2.png)


### k8s组件

![master work flow](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/master.png)

1.Kubecfg将特定的请求，比如创建Pod，发送给Kubernetes Client。

2.Kubernetes Client将请求发送给API server。

3.API Server根据请求的类型，比如创建Pod时storage类型是pods，然后依此选择何种REST Storage API对请求作出处理。

4.REST Storage API对的请求作相应的处理。

5.将处理的结果存入高可用键值存储系统Etcd中。

6.在API Server响应Kubecfg的请求后，Scheduler会根据Kubernetes Client获取集群中运行Pod及Minion/Node信息。

7.依据从Kubernetes Client获取的信息，Scheduler将未分发的Pod分发到可用的Minion/Node节点上。

**Master运行三个组件：**
* apiserver[资源操作入口]
作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。
它维护的REST对象将持久化到etcd（一个分布式强一致性的key/value存储）。

* scheduler[集群分发调度器]
集群调度器,负责集群的资源调度，为新建的Pod分配机器。

1.Scheduler收集和分析当前Kubernetes集群中所有Minion/Node节点的资源(内存、CPU)负载情况，然后依此分发新建的Pod到Kubernetes集群中可用的节点。

2.实时监测Kubernetes集群中未分发和已分发的所有运行的Pod。

3.Scheduler也监测Node节点信息，由于会频繁查找Node节点，Scheduler会缓存一份最新的信息在本地。

4.Scheduler在分发Pod到指定的Node节点后，会把Pod相关的信息`Binding`写回API Server

* controller-manager[内部管理控制中心]
负责执行各种控制器，目前有两类：
(1) `endpoint-controller`：定期关联service和Pod(关联信息由endpoint对象维护)，保证service到Pod的映射总是最新的。
(2) `replication-controller`：定期关联replicationController和Pod，保证replicationController定义的复制数量与实际运行Pod的数量总是一致的。

**Node运行两个组件：**
![kubelet结构图](https://raw.githubusercontent.com/hapiman/gorice/master/k8s/node.png)
* kubelet[节点上的Pod管家]

1.负责Node节点上pod的创建、修改、监控、删除等全生命周期的管理

2.定时上报本Node的状态信息给API Server。

3.kubelet是Master API Server和Node之间的桥梁，接收Master API Server分配给它的commands和work，通过kube-apiserver间接与Etcd集群交互，读取配置信息。
* proxy[负载均衡、路由转发]

1.Proxy是为了解决外部网络能够访问跨机器集群中容器提供的应用服务而设计的，运行在每个Node上。
Proxy提供`TCP/UDP sockets`的proxy，每创建一种Service，Proxy主要从etcd获取Services和Endpoints的配置信息（也可以从file获取），
然后根据配置信息在Node上启动一个Proxy的进程并监听相应的服务端口，当外部请求发生时，Proxy会根据`Load Balancer`将请求分发到后端正确的容器处理。

2.Proxy不但解决了同一主宿机相同服务端口冲突的问题，还提供了Service转发服务端口对外提供服务的能力，Proxy后端使用了随机、轮循负载均衡算法。
