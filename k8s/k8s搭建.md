### 安装说明

操作系统版本:
```sh
cat /proc/version
# Linux version 3.10.0-862.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) ) #1 SMP Fri Apr 20 16:44:24 UTC 2018
rpm -q centos-release
# centos-release-7-5.1804.el7.centos.x86_64
cat /etc/redhat-release
# CentOS Linux release 7.5.1804 (Core)
```
docker版本:
```sh
docker --version
# Docker version 18.09.6, build 481bc77156
```
kubernetes版本
```sh
kubelet --version
# Kubernetes v1.14.2
```

### 安装步骤

* 1.设置ssh使服务器之间互信

* 2.关闭 SeLinux 和 FireWall [所有机器执行]
```sh
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

* 3.安装 docker [所有机器执行]
```sh
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r
yum -y install docker-ce
docker --version
# Docker version 17.06.2-ce, build cec0b72
systemctl start docker
systemctl status docker
systemctl enable docker
```

* 4.安装镜像 [所有机器执行]
```sh
# 安装镜像
docker pull mirrorgooglecontainers/kube-apiserver:v1.14.2
docker pull mirrorgooglecontainers/kube-controller-manager:v1.14.2
docker pull mirrorgooglecontainers/kube-scheduler:v1.14.2
docker pull mirrorgooglecontainers/kube-proxy:v1.14.2
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd:3.3.10
docker pull coredns/coredns:1.3.1
docker pull registry.cn-shenzhen.aliyuncs.com/cp_m/flannel:v0.10.0-amd64

# 取别名
docker tag mirrorgooglecontainers/kube-apiserver:v1.14.2 k8s.gcr.io/kube-apiserver:v1.14.2
docker tag mirrorgooglecontainers/kube-controller-manager:v1.14.2 k8s.gcr.io/kube-controller-manager:v1.14.2
docker tag mirrorgooglecontainers/kube-scheduler:v1.14.2 k8s.gcr.io/kube-scheduler:v1.14.2
docker tag mirrorgooglecontainers/kube-proxy:v1.14.2 k8s.gcr.io/kube-proxy:v1.14.2
docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker tag coredns/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
docker tag registry.cn-shenzhen.aliyuncs.com/cp_m/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64

# 删除镜像
docker rmi mirrorgooglecontainers/kube-apiserver:v1.14.2
docker rmi mirrorgooglecontainers/kube-controller-manager:v1.14.2
docker rmi mirrorgooglecontainers/kube-scheduler:v1.14.2
docker rmi mirrorgooglecontainers/kube-proxy:v1.14.2
docker rmi mirrorgooglecontainers/pause:3.1
docker rmi mirrorgooglecontainers/etcd:3.3.10
docker rmi coredns/coredns:1.3.1
docker rmi registry.cn-shenzhen.aliyuncs.com/cp_m/flannel:v0.10.0-amd64
```

* 5.安装Master

(1) 初始化
```
kubeadm init --kubernetes-version=v1.14.2 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap
```
`kubernetes-version`: 当前k8s版本
`pod-network-cidr`: 用于指定Pod的网络范围。该参数使用依赖于使用的网络方案，本文将使用经典的flannel网络方案。
`service-cidr`:

如果没有问题, 则会得到输出结果: [输出结果](https://github.com/hapiman/gorice/blob/master/k8s/kubeadm_init.md)

(2) 设置`.kube/config`
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

(3) 保存输出中`kubeadm join`行命令, 在node节点会执行
```
kubeadm join 10.255.73.26:6443 --token xfnfrl.4zlyx5ecu4t7n9ie \
    --discovery-token-ca-cert-hash sha256:c68bbf21a21439f8de92124337b4af04020f3332363e28522339933db813cc4b
```

(4) 配置 kubect
```
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile
echo $KUBECONFIG
```

(5) 安装Pod网络
安装 Pod网络是 Pod之间进行通信的必要条件，k8s支持众多网络方案，这里我们依然选用经典的`flannel`方案

a.在任意位置新建文件 `kube-flannel.yaml`, 文件内容: [文件内容](https://github.com/hapiman/gorice/blob/master/k8s/kube-flannel.yaml)

b.首先设置系统参数 `sysctl net.bridge.bridge-nf-call-iptables=1`

c.使用 `kube-flannel.yaml`文件, `kubectl apply -f kube-flannel.yaml`

d.检查pod网络是否正常 `kubectl get pods --all-namespaces -o wide`, 如果READY都为1/1, 则正常
```
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE     IP             NODE              NOMINATED NODE   READINESS GATES
kube-system   coredns-fb8b8dccf-2hwr4                   1/1     Running   0          7h44m   10.244.0.3     pjr-ofckv-73-26   <none>           <none>
kube-system   coredns-fb8b8dccf-nwqt9                   1/1     Running   0          7h44m   10.244.0.2     pjr-ofckv-73-26   <none>           <none>
```
e.查看节点状态 `kubectl get nodes`
```
pjr-ofckv-73-26   Ready    master   7h47m   v1.14.2
```

* 6 添加Node节点
(1)执行`master`节点init的时候输出的`kubeadm join`,即
```
kubeadm join 10.255.73.26:6443 --token xfnfrl.4zlyx5ecu4t7n9ie \
    --discovery-token-ca-cert-hash sha256:c68bbf21a21439f8de92124337b4af04020f3332363e28522339933db813cc4b
```
如果在部署master节点的时候没有保存, 则可以通过`kubeadm token list`找回, ip即为master节点所在机器的ip, 端口为`6443`(不确定)

(2) 校验节点状态 `kubectl get nodes`

所有的节点皆为`Ready`状态表示集群正常

```
pjr-ofckv-73-24   Ready    <none>   47m     v1.14.2
pjr-ofckv-73-25   Ready    <none>   7h34m   v1.14.2
pjr-ofckv-73-26   Ready    master   7h55m   v1.14.2
```

(3) 查看所有Pod状态 `kubectl get pods --all-namespaces -o wide`

所有的的组件的READY皆为`1/1`

```
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE     IP             NODE              NOMINATED NODE   READINESS GATES
kube-system   coredns-fb8b8dccf-2hwr4                   1/1     Running   0          7h55m   10.244.0.3     pjr-ofckv-73-26   <none>           <none>
kube-system   coredns-fb8b8dccf-nwqt9                   1/1     Running   0          7h55m   10.244.0.2     pjr-ofckv-73-26   <none>           <none>
kube-system   etcd-pjr-ofckv-73-26                      1/1     Running   0          7h54m   10.255.73.26   pjr-ofckv-73-26   <none>           <none>
kube-system   kube-apiserver-pjr-ofckv-73-26            1/1     Running   0          7h54m   10.255.73.26   pjr-ofckv-73-26   <none>           <none>
kube-system   kube-controller-manager-pjr-ofckv-73-26   1/1     Running   0          7h54m   10.255.73.26   pjr-ofckv-73-26   <none>           <none>
kube-system   kube-flannel-ds-amd64-9qhcl               1/1     Running   0          48m     10.255.73.24   pjr-ofckv-73-24   <none>           <none>
kube-system   kube-flannel-ds-amd64-xmrzz               1/1     Running   0          7h51m   10.255.73.26   pjr-ofckv-73-26   <none>           <none>
kube-system   kube-flannel-ds-amd64-zqdzp               1/1     Running   0          7h34m   10.255.73.25   pjr-ofckv-73-25   <none>           <none>
kube-system   kube-proxy-kgcxj                          1/1     Running   0          7h34m   10.255.73.25   pjr-ofckv-73-25   <none>           <none>
kube-system   kube-proxy-rpn4z                          1/1     Running   0          7h55m   10.255.73.26   pjr-ofckv-73-26   <none>           <none>
kube-system   kube-proxy-tm8df                          1/1     Running   0          48m     10.255.73.24   pjr-ofckv-73-24   <none>           <none>
kube-system   kube-scheduler-pjr-ofckv-73-26            1/1     Running   0          7h54m   10.255.73.26   pjr-ofckv-73-26   <none>           <none>
```

### 安装问题
* The connection to the server localhost:8080 was refused - did you specify the right host or port?
在node节点安装的时候会碰到该问题, 原因是node节点所在服务器缺少文件`/etc/kubernetes/admin.conf`,
解决方法:
1.需要将master节点上的`/etc/kubernetes/admin.conf`文件复制到node节点`/etc/kubernetes/admin.conf`中
2.设置变量
```
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```

* node节点如果一直都处于`not ready`状态
我在第一次安装的时候, 在node节点没有下载镜像, 在执行`kubeadm join`加入集群时, 确实能够加入集群,但是一直处于`UnReady`状态,通过`tail /var/log/messages`参看错误日志,才知道是因为镜像没有安装, 因为安装的时候程序会自动去`k8s.gcr.io`节点下拉镜像,不幸的是,没有`梯子`
