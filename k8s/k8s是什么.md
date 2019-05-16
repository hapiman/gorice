### k8s出现背景
在分布式系统中, 项目部署在大量的机器上构成集群,如何实现服务自动化部署,更新,下线,负载均衡,容错处理等

### Kubernetes 能做些什么？

> Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications. It groups containers that make up an application into logical units for easy management and discovery.

k8s用于自动部署,伸缩,和管理容器化项目,将容器编排成逻辑的单元组,更容易管理和发现

Kubernetes 提供了大规模部署容器的编排与管理能力。

Kubernetes 编排让你能够构建多容器的应用服务，在集群上调度或伸缩这些容器，以及管理监听它们随时间变化的健康状态。

**总结**
通过Kubernetes,你可以快速有效地响应用户需求：
- 优化部署
快速而有预期地部署你的应用, 极速地扩展你的应用,增加项目的实例,能够实现自动布局、自动重启、自动复制、自动伸缩,并实现应用的`状态检查`与`自我修复`。
- 优化资源利用
跨主机编排容器, 更充分地利用硬件资源来最大化地满足企业应用的需求
- 声明式配置
`etcd`声明式的容器管理，保证所部署的应用按照我们部署的方式运作.

### k8s依赖组件

k8s之后能够提供完整而优秀的编排服务,在于它结合了当前许多优质的开源组件,这些组件包括：

仓库：Atomic Registry、Docker Registry 等。

网络：OpenvSwitch 和智能边缘路由等。

监控：heapster、kibana、hawkular 和 elastic。

安全：LDAP、SELinux、 RBAC 与 支持多租户的 OAUTH。

自动化：通过 Ansible 的 playbook 进行集群的安装和生命周期管理。

服务：大量事先创建好的常用应用模板。

### k8s关键组件
![](https://img-blog.csdn.net/20171208223632158)
`Master（主节点）`： 控制 Kubernetes 节点的机器，也是创建作业任务的地方。

`Node（节点）`： 这些机器在 Kubernetes 主节点的控制下执行被分配的任务。在Node上运行的服务进程包括`docker daemon`，`Kubelet` 和 `Kube-Proxy`。

`Pod`： 由一个或多个容器构成的集合，作为一个整体被部署到一个单一节点。同一个 pod 中的容器共享 IP 地址、进程间通讯（IPC）、主机名以及其它资源(如内存)。Pod 将底层容器的网络和存储抽象出来，使得集群内的容器迁移更为便捷。Kubernetes的其它组件帮助你对 pod 进行负载均衡，以保证有合适数量的容器支撑你的工作负载。Pod的生命周期是通过Replication Controller来管理的。在整个过程中，Pod处于4种状态之一：Pending, Running, Succeeded, Failed。

`Labels`: Service通过Label找到Pod组

`Replication controller（复制控制器）`： 控制一个 pod 在集群上运行的实例数量。确保任何时候Kubernetes集群中有指定数量的Pod副本在运行， 如果少于指定数量的Pod副本，Replication Controller会启动新的Pod，反之会杀死多余的以保证数量不变。

`Service（服务）`： 将服务内容与具体的 pod 分离。Kubernetes 服务代理负责自动将服务请求分发到正确的 pod 处，不管 pod 移动到集群中的什么位置，甚至可以被替换掉。

`Kubelet`： 这个守护进程从master或者其他地方获取本节点需要达到什么状态, 运行在各个工作节点上，负责获取容器列表，运行的副本数量, 网络或者存储如何配置,保证被声明的容器已经启动并且正常运行。

`kubectl`： 这是 Kubernetes 的命令行配置工具。

`kube-proxy`: 实现集群网络服务负载均衡

`Volume(存储卷)`: Volume是Pod中能够被多个容器访问的共享目录。

`Namespace(命名空间)`: 通过将系统内部的对象“分配”到不同的Namespace中，形成逻辑上的不同分组，便于在共享使用整个集群的资源同时还能分别管理。

`Annotation(注解)`：与Label类似，但Label定义的是对象的元数据，而Annotation则是用户任意定义的“附加”信息。

### k8s和docker关系

Docker 技术依然执行它原本的任务。当 kubernetes 把 pod 调度到节点上，节点上的 kubelet 会指示 docker 启动特定的容器。接着，kubelet 会通过 docker 持续地收集容器的信息，然后提交到主节点上。Docker 如往常一样拉取容器镜像、启动或停止容器。不同点仅仅在于这是由自动化系统控制而非管理员在每个节点上手动操作的。

### etcd作用

**etcd: Kubernetes’ brain**

Every component in Kubernetes (the API server, the scheduler, the kubelet, the controller manager, whatever) is stateless. All of the state is stored in a key-value store called etcd, and communication between components often happens via etcd.

简单来说就是数据库(etcd)增删改查，用户可以声明各种高级工作对象(Deployment,Service,ReplicaSet,Job等等)，然后写到etcd里。一堆服务都盯着etcd的特定数据类型的变化呢(通过list watch机制，通过chunk get一个资源实现[3])。

比如，

1，用户调`Rest api` 说起个ReplicaSet，然后api server响应请求，把结果写到etcd里

2，`controller manager`一看，我去，etcd里这有个ReplicaSet,那我就得建N个Pod吧，写etcd里。

3, `Scheduler`一看，我去，etcd里有新的没Node的Pod，来活了，跑一下分配算法，这几个Pod就写到XXX node去吧，写etcd里

4，`kubelet`一看，我去，etcd里写着我这个node上要起个Pod啊，来活了，起Pod起Pod，起完了把状态写etcd里（以上写etcd操作都是通过api server提供的接口实现的

### 结合当前技术栈
1.容器化当前的项目 => 难点在于存储和网络
2.实现docker项目的集群化

### 调研中碰到的问题
1.编排怎么理解?
2.k8s框架解析?
