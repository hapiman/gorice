## k8s常见的命令

配置文件：`~/.kube/config`，使用`kubectl config`查看或者修改配置，是会修改文件的。

配置文件可以指定`Cluster`，`Namespace`，`User`的配置，并设置`Context`


简版配置文件

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/shanyue/.minikube/ca.crt
    server: https://192.168.99.100:8443
  name: dev
contexts:
- context:
    cluster: dev
    namespace: Business
    user: shanyue
  name: dev
current-context: dev
kind: Config
preferences: {}
users:
- name: shanyue
  user:
    client-certificate: /Users/shanyue/.minikube/client.crt
    client-key: /Users/shanyue/.minikube/client.key
```

```sh
# 查看配置
kubectl config view

# 查看集群列表
kubectl config get-clusters

# 查看 Context 列表
kubectl config get-contexts

# 设置当前 Context
kubectl config use-context dev

```

`kubectl create` 代表根据文件创建资源，可以是 Deployment，也可以是 Pod。

`kubectl run` 代表根据镜像创建资源。
一般在 CI 中作 deploy 时会使用`kubectl apply`命令，根据配置文件更新资源

```sh
kubectl create -f app.yaml
kubectl run --image=k8s.gcr.io/echoserver:1.10 --port=8080

# 替换资源
kubectl replace -f 文件名
# 删除资源
kubectl delete -f 文件名
# 获取service信息
kubectl get svc -o wide
# 查看具体pod详情，在排错时尤为实用
kubectl describe pod POD-NAME  -n NAMESPACE
# 查看service详情
kubectl describe svc SERVICE-NAME  -n NAMESPACE

```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
  namespace: dev
  labels:
    name: app
spec:
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
  selector:
    name: app

---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: app
  namespace: dev
  labels:
    name: app
spec:
  template:
    metadata:
      labels:
        name: app
    spec:
      containers:
      - name: app
        image: node
        imagePullPolicy: Always
        env:
        - name: PORT
          value: "8080"
        ports:
        - containerPort: 8080
```

访问资源
```sh
# 获取当前 Context 下所有的 Deployment
kubectl get deployments

# 获取当前 Context 下所有的 Pod
kubectl get pods

# 获取当前 Cluster 下所有的 Pod
kubectl get pods --all-namespaces

# 获取特定 Pod 的状态
kubectl describe pod $app

# 进入某个 Pod 容器里边的命令行
kubectl exec -it $app bash

# 查看某个 Pod 的日志
kubectl logs $app

# 查看当前组件
kubectl get componentstatus

# 查看当前的namespaces
kubectl get deployment --all-namespaces
```

### 基本概念
`Service`是定义一系列Pod已经访问这些Pod的策略的一层抽象。`Service`通过Label找到`Pod组`。
因为`Service`是抽象的，在图表中看不到其存在。‘

Label：

k8s中的容器api对象都是通过Label进行 ,label的实质是一系列的k/v键值对
label是`replication controller`和`service`运行的基础
二者通过label进行判别node上运行的pod。

Replication Controller(`RC`)：

Recplication Controller用来管理Pod的副本，保证集群中存在置顶数量的pod副本，集群中副本的数量大于置顶数量，则会杀掉置顶数量之外的多余容器数量，反之则会启动少于置顶数量个数的容器，保证数量不变，**RC**是实现`弹性伸缩`、`动态扩容`和`滚动升级`的核心。

`ReplicaSet`：

升级版本的`Replication Controller`，负责控制集群中运行的`pod`的数量。虽然`ReplicaSet`可以被单独使用，但是目前它多被`Deployment`用于进行`Pod`的创建、更新与删除。

`Deployment`：

Kubernetes提供了一种更加简单的更新RC和Pod的机制，叫做`Deployment`。
通过在`Deployment`中描述你所期望的集群状态，`Deployment Controller`会将现在的集群状态在一个可控的速度下逐步更新成你所期望的集群状态。
为了保证pod的数量和健康，90%的功能与`Replication Controller`完全一样，新一代的`Replication Controller`，功能更加完善。

`Context`：

`kubeconfig`文件可以包含`context`元素，每个`context`都是一个由（`集群、命名空间、用户`）描述的三元组。您可以使用 `kubectl config use-context`去设置当前的`context`。命令行工具`kubectl`与当前`context`中指定的集群和命名空间进行通信，并且使用当前`context`中包含的用户凭证。

`Job`：

从程序的运行形态上来区分，我们可以将Pod分为两类：`长时运行服务`（jboss、mysql等）和`一次性任务`（数据计算、测试）。
RC创建的Pod都是长时运行的服务，而Job创建的Pod都是一次性任务。
在Job的定义中，restartPolicy（重启策略）只能是Never和OnFailure。
Job可以控制一次性任务的Pod的完成次数（Job-->spec-->completions）和并发执行数（Job-->spec-->parallelism），当Pod成功执行指定次数后，即认为Job执行完毕。




### 疑问
如何查看当前集群之下有哪些`namespace`?


### 参考文档，发文章前核对知识点是否都以掌握
https://juejin.im/post/5b53255de51d451949092cba

https://juejin.im/post/5c98b1785188252d665f57be
