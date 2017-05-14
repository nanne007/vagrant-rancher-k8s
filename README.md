##  Bootstraping Kubernetes with Rancher

涉及到的工具：Vagrant on Virtualbox, Rancher, Kubernetes, NFS Storage Provisioner

机器配置： MacBook Pro 2015(8G Memory, 128G Disk)

步骤：

- Rancher Server 的部署。
- 使用 Vagrant 启动三个虚拟机节点。
- 用三个节点构建 K8s 集群。
- 基于 NFS 为 K8s 集群提供 Storage 的 Dynamic Provisioning。

### Rancher Server 的部署

[用 Docker Compose 将 Rancher Server 部署在本机。](rancher-server/README.md)

访问 http://127.0.0.1:18080 可看到 Rancher UI 。

### Vagrant 启动三个虚拟机

 执行: `vagrant up`。

Vagrant 会在虚拟机中安装最新的 Docker Engine，参见 Vagrantfile。

VirtualBox 会在本机创建一个 `192.168.32.1` 的网卡，可利用 ifconfig 查看。
三个虚拟机的ip地址依次是： 192.168.32.1[1-3]

接下来中的部署会使用这些 IP 来互相访问。

### 用三个节点构建 K8s 集群

首先在 Rancher 里面构建一个 K8s 环境，然后通过 Add Host 将三个虚拟机加入到该集群。

在安装 K8s 服务的时候，需要访问 Google 的Docker Registry，但是国内有GFW限制。
为了绕过这个，可以在 Docker 配置(`etc/default/docker`)里添加 http_proxy。
这里我在本机启动了一个 [privoxy+shadowsocks] 的Docker Container(`bluebu/shadowsocks-privoxy`)，
监听在 8118 端口。

``` shell
export http_proxy="http://192.168.32.1:8118"
```

然后重启 Docker Server。


搞定之后，就可以等待 K8s 启动完毕。

### NFS Storage Provisioner ###

要想能够顺利使用 K8s，Storage 是一个绕不开的话题。

有状态服务的部署一般都涉及到 Storage，比如 ZooKeeper, Kafka, Hbase 等等。
K8s 提供了 [Dynamic Provisioning 的功能](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)，
也内置了不少 Provisioner，包括 GCE, AWS, Azure, Glusterfs, Cinder, Ceph RBD。

这里，我使用了 NFS，这是一个外部实现。具体参见：https://github.com/kubernetes-incubator/external-storage

部署很简单：

``` shell
cd nfs/deploy/kubernetes
kubectl apply -f statefulset.yaml
kubectl apply -f class.yaml
kubectl apply -f claim.yaml
```


