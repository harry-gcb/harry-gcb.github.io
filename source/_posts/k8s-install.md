---
layout: title
title: 使用kubeadm工具安装Kubernetes集群
date: 2023-04-25 20:44:46
tags: [k8s]
---

## 安装Docker

虽然Kubernetes 1.24版本之后移除了对Docker的支持，但是本文还是决定使用Docker作为k8s容器的引擎，参考[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

{% tabs docker-install %}

<!-- tab Ubuntu -->

卸载旧的Docker版本

```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
```

更新 apt 包索引并安装使用Docker apt 仓库所需要的包：

```shell
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

添加Docker官方的GPG Key

```shell
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

添加Docker仓库

```shell
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu" $(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

更新apt包索引，安装Docker

```shell
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

验证是否安装成功

```shell
sudo usermod -aG docker $USER # 将用户添加到docker用户组后重新登录
docker version
```

<!-- endtab -->

{% endtabs %}

## 安装kubeadm

{% tabs k8s-install %}

<!-- tab Ubuntu 22.04 -->

更新 apt 包索引并安装使用 Kubernetes apt 仓库所需要的包：

```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

下载公开签名秘钥：
```shell
curl -fsSLo /etc/apt/keyrings/kubernetes-aliyun-keyring.gpg https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg
```

添加 Kubernetes apt 仓库：
```shell
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-aliyun-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：
```shell
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

<!-- endtab -->

<!-- tab Ubuntu 20.04 -->

更新 apt 包索引并安装使用 Kubernetes apt 仓库所需要的包：

```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

添加并信任APT证书：
```shell
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
```

添加 Kubernetes apt 仓库：
```shell
sudo add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main"
```

更新 apt 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：
```shell
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
<!-- endtab -->

{% endtabs %}

## 修改kubeadm的默认配置

查看初始化的默认配置，以及查看镜像列表

```shell
kubeadm config print init-defaults | tee init-config.yml
```

对生成的文件`init-config.yml`进行编辑，例如自定义镜像仓库地址和IP地址

```yaml
localAPIEndpoint:
  advertiseAddress: 192.168.1.102
nodeRegistration:
  criSocket: unix:///var/run/cri-dockerd.sock
  name: master
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
```

## 下载kubernetes的相关镜像

为了加快`kubeadm`创建集群的过程，可以先将所需镜像下载下来。国内无法访问`registry.k8s.io`，可以修改配置文件来加速下载

```shell
kubeadm config images list --config=init-config.yml
kubeadm config images pull --config=init-config.yml
```

## 运行kubeadm init命令安装Master节点

镜像下载结束后，即可初始化Kubernetes集群

```shell
sudo kubeadm init --config=init-config.yml
```

看到`Your Kubernetes control-plane has initialized successfully!`的提示，就说明Kubernetes安装成功。按照安装成功的提示，将配置文件拷贝到用户目录下

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

此时已经可以通过`kubectl`对kubernetes集群进行访问和操作了

```shell
kubectl -n kube-system get configmap # 查看命名空间kube-system中的ConfigMap列表
NAME                                                   DATA   AGE
coredns                                                1      8m
extension-apiserver-authentication                     6      8m3s
kube-apiserver-legacy-service-account-token-tracking   1      8m3s
kube-proxy                                             2      8m
kube-root-ca.crt                                       1      7m2s
kubeadm-config                                         1      8m1s
kubelet-config                                         1      8m1s
```

## 将新的Node加入节点

对于新节点的添加，系统准备和安装Master节点的过程是一致的

## 错误记录

### 未安装cri-dockerd

本文安装的是Kubernetes 1.27的版本，而且将Docker作为容器引擎，需要安装cri-dockerd发生以下错误的原因是没有安装cri-dockerd

```shell
time="2023-04-26T15:17:15Z" level=fatal msg="pulling image: rpc error: code = Unavailable desc = connection error: desc = \"transport: Error while dialing dial unix /var/run/cri-dockerd.sockk: connect: no such file or directory\""
, error: exit status 1
```

**解决方案**

下载cri-dockerd

```shell
git clone https://github.com/Mirantis/cri-dockerd.git
```

编译cri-dockerd

```shell
cd cri-dockerd
mkdir bin
go build -o bin/cri-dockerd
mkdir -p /usr/local/bin
```

安装cri-dockerd

```shell
sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
sudo cp -a packaging/systemd/* /etc/systemd/system
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
```

**扩展**

默认情况下，Kubernetes使用CRI与Pod中运行的容器进行交互。如果你指定运行时，kubeadm 会自动尝试通过扫描已知的端点列表来检测已安装的容器运行时。1.27版本支持的CRI端点如下：

{% note info %}

1.24版本之前，Kubernetes可以直接使用Docker作为容器引擎，而1.24之后Kubernetes移除了对Docker的支持，而Docker本身并没有实现CRI，所以如果Kubernetes仍将Docker作为容器引擎的话，需要安装额外的服务cri-dockerd

{% endnote %}

{% tabs CRI %}

<!-- tab Linux -->

| 运行时                            | Unix域套接字                                 |
| --------------------------------- | -------------------------------------------- |
| containerd                        | `unix:///var/run/containerd/containerd.sock` |
| CRI-O                             | `unix:///var/run/crio/crio.sock`             |
| Docker Engine（使用 cri-dockerd） | `unix:///var/run/cri-dockerd.sock`           |

<!-- endtab -->

<!-- tab Windows -->

| 运行时                            | Windows命名管道路径                      |
| --------------------------------- | ---------------------------------------- |
| containerd                        | `npipe:////./pipe/containerd-containerd` |
| Docker Engine（使用 cri-dockerd） | `npipe:////./pipe/cri-dockerd`           |

<!-- endtab -->

{% endtabs %}

### swap未关闭

`kubeadm init`初始化Kubernetes集群时出现此警告，原因是未关闭swap：

```
[WARNING Swap]: swap is enabled; production deployments should disable swap unless testing the NodeSwap feature gate of the kubelet
```
**解决方案**

临时关闭swap

```shell
swapoff -a # 临时关闭
free -m    # 检查效果
               total        used        free      shared  buff/cache   available
Mem:            3889         376        1362           1        2150        3262
Swap:              0           0           0
swapon -a # 临时打开
```

永久关闭swap

```shell
vim /etc/fstab   # 编辑/etc/fstab，注释掉swap分区记录所在的行
# /swap.img       none    swap    sw      0       0
systemctl reboot # 重启系统
```

**扩展**

swap是linux系统用来临时存储内存过剩数据的空间。尽管swap可以增加系统的可用内存，但在运行Kubernetes时，建议禁用swap。这是因为当内存不足时，swap会将一部分内存存储到磁盘交换文件中，从而导致性能下降并可能导致应用程序崩溃。

此外，在执行大型容器操作时，内存压力可能很高。如果系统启用了swap，那么可能会有一些容器被停止或无法正常工作。为避免这种情况，建议在Kubernetes节点上禁用swap。

有兴趣的朋友可以看下这个[issue](https://github.com/kubernetes/kubernetes/issues/53533)


### bridge-nf-call-iptables不存在

`kubeadm init`初始化Kubernetes集群时出现此错误，原因是`/proc/sys/net/bridge/bridge-nf-call-iptables`文件不存在或值不为1：

```shell
[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
```

**解决方案**

```shell
modprobe br_netfilter # 载入br_netfilter模块
vim /etc/sysctl.conf  # 编辑sysctl.conf，添加以下内容
net.bridge.bridge-nf-call-iptables=1
sudo sysctl -p # 激活配置
```

**扩展**

对于插件开发人员以及时常会构建并部署 Kubernetes 的用户而言， 插件可能也需要特定的配置来支持 kube-proxy。 iptables 代理依赖于 iptables，插件可能需要确保 iptables 能够监控容器的网络通信。 例如，如果插件将容器连接到 Linux 网桥，插件必须将 `net/bridge/bridge-nf-call-iptables` sysctl 参数设置为 `1`，以确保 iptables 代理正常工作。

默认情况下，如果未指定 kubelet 网络插件，则使用 `noop` 插件， 该插件设置 `net/bridge/bridge-nf-call-iptables=1`，以确保简单的配置 （如带网桥的 Docker）与 iptables 代理正常工作。

### ip_forward未设置为1

`kubeadm init`初始化Kubernetes集群时出现此错误，原因是`/proc/sys/net/ipv4/ip_forward`的内容未设置为1，即需要打开IP路由转发功能

```
[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
```

**解决方案**

```shell
vim /etc/sysctl.conf  # 编辑sysctl.conf，添加以下内容
net.ipv4.ip_forward=1
sudo sysctl -p # 激活配置
```

### 无法启动pause容器

`kubeadm init`初始化Kubernetes集群时出现此`CreatePodSandbox`错误，查看日志，发现这个pod使用的是`registry.k8s.io/pause:3.6`的镜像，与之前`kubeadm config images list`查看的镜像版本不符合

```shell
Apr 29 18:13:16 master kubelet[3067]: E0429 18:13:16.559646    3067 pod_workers.go:1281] "Error syncing pod, skipping" err="failed to \"CreatePodSandbox\" for \"kube-controller-manager-master_kube-system(d79d
81b39a8fbfd55e971ba990a6e84f)\" with CreatePodSandboxError: \"Failed to create sandbox for pod \\\"kube-controller-manager-master_kube-system(d79d81b39a8fbfd55e971ba990a6e84f)\\\": rpc error: code = Unknown d
esc = failed to get sandbox image \\\"registry.k8s.io/pause:3.6\\\": failed to pull image \\\"registry.k8s.io/pause:3.6\\\": failed to pull and unpack image \\\"registry.k8s.io/pause:3.6\\\": failed to resolv
e reference \\\"registry.k8s.io/pause:3.6\\\": failed to do request: Head \\\"https://asia-east1-docker.pkg.dev/v2/k8s-artifacts-prod/images/pause/manifests/3.6\\\": dial tcp 108.177.125.82:443: i/o timeout\"
" pod="kube-system/kube-controller-manager-master" podUID=d79d81b39a8fbfd55e971ba990a6e84f
```

**解决方案**

尽快`kubeadm config images list`显示使用`pause:3.9`的镜像，但是在初始化过程中，仍然在使用`pause:3.6`的镜像，可以在初始化命令中直接制定使用的镜像，但似乎与`--config`选项不兼容。可以先通过`--image-repository`将镜像下载下来，后面再初始化时单独使用`--config`选项即可。似乎不那么优雅，建议还是用Docker作为后端引擎使用

```shell
kubeadm init --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```