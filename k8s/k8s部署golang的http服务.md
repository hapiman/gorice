## k8s部署golang的http服务

### 制作镜像
```sh
cd $GOPATH/src/github.com/hapiman # 这是我的账号, 请更换为自己的github账号
mkdir goappk8s # 项目名称
cd goappk8s
vim main.go # 创建文件,内容在下面
dep init # 初始化管理工具
dep ensure -add github.com/gin-gonic/gin # 安装依赖包
go run main.go # 测试你的程序没有问题
vim Dockerfile # 创建Dockerfile, 内容在下面
vim start.sh # 创建制作镜像命令, 内容在下面
sh start.sh # 构建镜像
docker tag goappk8s:latest pengj/goappk8s:v1.0.0 # 增加一个标准格式的镜像, 请更换为自己的仓库名字
docker run  -p 8080:8080 pengj/goappk8s:v1.0.0 # 测试你的镜像没有问题
# 这里需要在你的docker仓库新建镜像,之后执行下面的操作; 我的地址为: https://hub.docker.com/u/pengj
# 如果先登录再创建,则推送会报仓库不存在
docker login # 登录, 输入账号和密码
docker push pengj/goappk8s:v1.0.0 # 推送镜像
```

`main.go`内容
```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {

	router := gin.Default()

	router.GET("/ping", func(c *gin.Context) {
		c.String(http.StatusOK, "PONG")
	})

	router.Run(":8080")
}
```

`Dockerfile`内容
```
FROM fuyufjh/go-alpine
WORKDIR $GOPATH/src/github.com/hapiman/goappk8s
COPY . $GOPATH/src/github.com/hapiman/goappk8s

EXPOSE 8080
CMD ["./goappk8s"]
```

`start.sh`内容
```sh
CGO_ENABLED=0 GOOS=linux GOARCH=amd64  go build -v -a -installsuffix cgo -o goappk8s .
docker build -t goappk8s .
```

### 部署镜像

#### 前提
如果你没有搭建`k8s`集群,可以参考[k8s搭建](https://github.com/hapiman/gorice/blob/master/k8s/k8s%E6%90%AD%E5%BB%BA.md)

#### 部署
```sh
# 在master机器上创建goapp-k8s.yaml,内容在下面,你需要更换`pengj/goappk8s:v1.0.0`为你自己的镜像仓库或者直接使用(省事)
vim goapp-k8s.yaml
# 部署配置, 如果多次部署会存在重复文件, 可以使用kubectl delete -f goapp-k8s.yaml删除
kubectl apply -f goapp-k8s.yaml
# 执行成功会输出
# deployment "goapp-deploy" created
# service "goapp-svc" created
# ingress "goapp-ingress" created
```
#### 校验
```sh
# 可能会出现0/2, 1/2的情况, 等一会儿就OK
kubectl get deployments -n kube-apps |grep goapp
# goapp-deploy   2/2     2            2           109m
kubectl get svc -n kube-apps |grep goapp
# goapp-svc   NodePort   10.97.144.116   <none>        8080:31000/TCP   109m
kubectl get ingress -n kube-apps |grep goapp
# goapp-ingress   goappk8s.local             80      110m
kubectl get pods -n kube-apps |grep goapp
# goapp-deploy-5c84c9d676-m7zwc   1/1     Running   0          110m
# goapp-deploy-5c84c9d676-v8r6g   1/1     Running   0          110m
```
**两种访问方式:**

1.使用集群之间可见ip访问, 只能够在集群部署的机器上可用: `curl http://10.97.144.116:8080/ping`

2.使用通用的ip访问, 但是需要使用配置文件中`31000`端口才行

详细代码[在这里](https://github.com/hapiman/goappk8s/blob/master/README.md)

`goapp-k8s.yaml`文件中定义了3类资源`Deployment`、`Service`、`Ingress`;

`Deployment`设置了`replicas: 2`，表示会运行两个`POD`，`strategy`的滚动策略为`RollingUpdate`，`resources`区域定义了一个POD的资源限制，通过`livenessProbe`和`readinessProbe`设置了健康检查

`goapp-k8s.yaml`内容
```yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: goapp-deploy
  namespace: kube-apps
  labels:
    k8s-app: goappk8s
spec:
  replicas: 2
  revisionHistoryLimit: 10
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        k8s-app: goappk8s
    spec:
      containers:
      - image: pengj/goappk8s:v1.0.0
        imagePullPolicy: Always
        name: goappk8s
        ports:
        - containerPort: 8080
          protocol: TCP
        resources:
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 50m
            memory: 50Mi
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 10
          timeoutSeconds: 3
        readinessProbe:
          httpGet:
            path: /ping
            port: 8080
          initialDelaySeconds: 10
          timeoutSeconds: 2

---
apiVersion: v1
kind: Service
metadata:
  name: goapp-svc
  namespace: kube-apps
  labels:
    k8s-app: goappk8s
spec:
  type: NodePort
  ports:
    - name: api
      port: 8080
      targetPort: 8080
      nodePort: 31000
  selector:
    k8s-app: goappk8s

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: goapp-ingress
  namespace: kube-apps
spec:
  rules:
  - host: goappk8s.local
    http:
      paths:
      - path: /
        backend:
          serviceName: goapp-svc
          servicePort: api
```

### 参考文档
[手摸手教你写 Kubernetes 的 golang 服务](https://www.qikqiak.com/post/write-kubernets-golang-service-step-by-step/)
