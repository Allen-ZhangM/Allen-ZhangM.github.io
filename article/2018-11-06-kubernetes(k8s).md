---
sort: 
---
## 简介
Kubernetes (K8s)是Google在2014年发布的一个开源项目。
据说Google的数据中心里运行着20多亿个容器，而且Google十年多前就开始使用容器技术。
最初，Google开发了一个叫Borg的系统（现在命名为Omega)来调度如此庞大数量的容器和工作负载。在积累了 这么多年的经验后，Google决定重写这个容器管理系统，并将其贡献到开源社区，让全世界都能受益。
这个项目就是Kubernetes。简单地讲，Kubernetes是Google Omega的开源版本。
从2014年第一个版本发布以来，Kubernetes迅速获得开源社区的追捧，包括Red Hat、VMware、Canonical在内 的很多有影响力的公司加入到开发和推广的阵营。目前Kubernetes已经成为发展最快、市场占有率最高的容器编 排引擎产品。
## k8s的基本概念
在部署前，必须先学习Kubernetes的几个重要概念，它们是组成Kubernetes集群的基石。
### Cluster (集群)
Cluster是计算、存储和网络资源的集合，Kubernetes利用这些资源运行各种基于容器的应用。
### Master (控制主节点)
Master是Cluster的大脑，它的主要职责是调度，即决定将应用放在哪里运行。Master运行Linux操作系统，可以 是物理机或者虚拟机。为了实现高可用，可以运行多个Master。调度应用程序、维护应用程序的所需状态、扩展应 用程序和滚动更新都是master的主要工作。
### Node (节点)
Node的职责是运行容器应用。Node由Master管理，Node负责监控并汇报容器的状态，同时根据Master的要求管 理容器的生命周期。

Node是Kubernetes集群中的工作机器，可以是物理机或虛拟机。每个工作节点都有一个kubelet,它是管理节 点并与Kubernetes Master节点进行通信的代理。节点上还应具有处理容器操作的容器运行时，例如Docker。 —个Kubernetes工作集群至少有三个节点。Master管理集群，而Node (节点）用于托管正在运行的应用程 序。 当你在Kubernetes上部署应用程序时，你可以告诉master启动应用程序容器。Master调度容器在集群的节点上 运行。节点使用Master公开的Kubernetes API与Master通信。用户也可以直接使用Kubernetes的API与集 群交互。
![k8s-cluster](https://raw.githubusercontent.com/Allen-ZhangM/Allen-ZhangM.github.io/master/img/posts/k8s-cluster.png)
### Pod (资源对象）
Pod是Kubernetes的最小工作单元。每个Pod包含一个或多个容器。Pod中的容器会作为一个整体被Master调度到
一个Node上运行。

Kubernetes引入Pod主要基于下面两个目的：
- 可管理性。有些容器天生就是需要紧密联系，一起工作。Pod 提供了比容器更高层次的抽象，将它们封装到一个部署单元中。Kubernetes以Pod为最小单位进行调度、扩展、共 享资源、管理生命周期。
- 通信和资源共享。Pod中的所有容器使用同一个网络namespace,即相同的IP地址 和Port空间。它们可以直接用localhost通信。同样的，这些容器可以共享存储，当Kubernetes挂载volume到 Pod,本质上是将volume挂载到Pod中的每一个容器。

**Pods有两种使用方式：**
- 运行单一容器

one-container-per-Pod是Kubernetes最常见的模型，这种情况下，只是将单个容器简单封装成Pod。即便是只有 —个容器，Kubernetes管理的也是Pod而不是直接管理容器。
- 运行多个容器

问题在于：哪些容器应该放到一个Pod中？答案是：这些容器联系必须非常紧密，而且需要直接共享资源。
举个例子，如图所示，这个Pod包含两个容器：
—个是File Puller(文件拉取器)，一个是Web Server。

![k8s-cluster](https://raw.githubusercontent.com/Allen-ZhangM/Allen-ZhangM.github.io/master/img/posts/k8s-pod.png)

File Puller会定期从外部的Content Manager中拉取最新的文件，将其存放在共享的volume中。Web Server从
volume读取文件，响应Consumer的请求。

这两个容器是紧密协作的，它们一起为Consumer提供最新的数据；同时它们也通过volume共享数据，所以放到
一个Pod是合适的。
### Controller（控制器）
Kubernetes通常不会直接创建Pod，而是通过Controller来管理Pod的。Controller中定义了Pod的部署特性，比如
有几个副本、在什么样的Node上运行等。为了满足不同的业务场景，Kubernetes提供了多种Controller，包括
Deployment、ReplicaSet、DaemonSet、StatefuleSet、Job等，我们逐一讨论。 
- Deployment是最常用的Controller，比如通过创建Deployment来部署应用的。Deployment可以管理Pod的多个副本，并确保Pod按照期望的状态运行。
- ReplicaSet实现了Pod的多副本管理。使用Deployment时会自动创建ReplicaSet，也就是说Deployment是通过ReplicaSet来管理Pod的多个副本的，我们通常不需要直接使用ReplicaSet。 
- DaemonSet用于每个Node最多只运行一个Pod副本的场景。正如其名称所揭示的，DaemonSet通常用于运行daemon。
- StatefuleSet能够保证Pod的每个副本在整个生命周期中名称是不变的，而其他Controller不提供这个功能。当某个Pod发生故障需要删除并重新启动时，Pod的名称会发生变化，同时StatefuleSet会保证副本按照固定的顺序启动、更新或者删除。
- Job用于运行结束就删除的应用，而其他Controller中的Pod通常是长期持续运行。

### Service（服务）
Deployment可以部署多个副本，每个Pod都有自己的IP，外界如何访问这些副本呢？ 通过Pod的IP吗？ 要知道
Pod很可能会被频繁地销毁和重启，它们的IP会发生变化，用IP来访问不太现实。
 
答案是Service。 Kubernetes Service定义了外界访问一组特定Pod的方式。Service有自己的IP和端口，Service为
Pod提供了负载均衡。

Kubernetes运行容器（Pod）与访问容器（Pod）这两项任务分别由Controller和Service执行。
### Namespace（命名空间）
Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。
如果有多个用户或项目组使用同一个Kubernetes Cluster，如何将他们创建的Controller、Pod等资源分开呢？ 答
案就是Namespace。

Namespace可以将一个物理的Cluster逻辑上划分成多个虚拟Cluster，每个Cluster就是一个Namespace。不同
Namespace里的资源是完全隔离的。

Kubernetes默认创建了两个Namespace：

- default：创建资源时如果不指定，将被放到这个Namespace中。
- kube-system：Kubernetes自己创建的系统资源将放到这个Namespace中。

## k8s的安装部署
### k8s的搭建方式
#### 方式一
minikube  
官方给出的单机版搭建方式。  
难易程度：简单  
适用于：初学者  
使用场景：学习  
对于网络环境有要求  
[minikube](https://github.com/kubernetes/minikube)
#### 方式二
kubeadm  
官方给出的集群版搭建方式  
难易程度：偏难  
适用于：工作与学习  
使用场景：工作  
对于网络环境有要求  
[kubeadm](https://github.com/kubernetes/kubeadm)
#### 方式三
纯手动二进制  
难易程度：疯狂  
适用于：工作  
使用场景：工作  
对于网络环境有要求  
[google首席布道师](https://github.com/kelseyhightower/kubernetes-the-hard-way)  
[中文完整流程](https://www.qikqiak.com/post/manual-install-high-available-kubernetes-cluster/)

### 安装前的准备
系统版本：ubuntu18.04 LTS  
机器数量：3台虚拟机  
配置最低需求：1核 2G 20G硬盘  
部署划分：（同一网段下三台主机）  
k8s-master001: 　192.168.110.116  
k8s-node001: 　　192.168.110.117  
k8s-node002: 　　192.168.110.118  
### 三台机器共同基本操作
#### 安装基本工具
```
#由于权限原因全程使用root用户进行操作
$ sudo -i
#安装相关软件
$ apt-get install apt-transport-https ca-certificates curl software-properties-common lrzsz -y
```
#### 禁用SWAP
```
$ swapoff -a
$ sed -i '/ swap / s/^/#/' /etc/fstab
```
#### 更新软件
```
#检查更新
$ apt-get update
#更新已安装软件
$ apt-get upgrade
```
#### 防火墙的
```
#1804查看防火墙状态
$ sudo ufw status
#关闭防火墙
$ sudo ufw disable
#开启防火墙
$ sudo ufw enable
```
#### 主机名的查看与设置
```
#查看主机名
$hostname
#设置主机名 (修改主机名需要两个文件)
$ sudo vim /etc/hostname
$ sudo vim /etc/hosts
#重启
$ reboot
```
#### Docker的安装  
```
#docker相关软件
$ sudo apt-get install apt-transport-https ca-certificates curl software-propertiescommon lrzsz -y
#离线安装docker
$ tar xzvf docker_v18.03.1_ce.tar.gz
$ cd docker_v18.03.1_ce && ./install.sh
#配置加速器
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s
http://f1361db2.m.daocloud.io
#由于全程使用root账户所以就不需要进行docker权限的设置了
```
### master主机操作
#### 搭建镜像仓库
```
#获取仓库镜像
$ docker pull registry
#启动仓库容器
$ docker run --restart=always --name=registry -d -p 5000:5000 registry
```
#### 进行仓库配置
```
#编辑docker配置文件
sudo vim /etc/default/docker
#最末尾添加 当前仓库主机ip
DOCKER_OPTS="--insecure-registry 192.168.110.116:5000"
#创建服务依赖文件
$ sudo mkdir -p /etc/systemd/system/docker.service.d
$ sudo vim /etc/systemd/system/docker.service.d/Using_Environment_File.conf
#内容如下：
[Service]
EnvironmentFile=-/etc/default/docker
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// $DOCKER_OPTS
#重载服务配置文件
$ systemctl daemon-reload
#重启docker
$ systemctl restart docker
```
#### 安装Kubeadm等程序
[下载002.001.k8s.deb.v1.11.1.tar.gz](https://pan.baidu.com/s/1nmC94Uh-lIl0slLFeA1-qw)
```
#依赖
apt-get install ethtool ebtables -y
#将002.001.k8s.deb.v1.11.1.tar.gz解压
$ tar xzvf 002.002.k8s.node.v1.11.1.tar.gz
#进入文件夹
$ cd k8s.deb.v1.11.1
#启动安装脚本
$ ./install.sh
#脚本内容
dpkg -i apt-transport-https_1.6.3_all.deb
#k8s网络接口插件
dpkg -i kubernetes-cni_0.6.0-00_amd64.deb
dpkg -i cri-tools_1.11.0-00_amd64.deb
dpkg -i socat_1.7.3.2-2ubuntu2_amd64.deb
#kubelet：负责 Pod 的创建、启动、监控、重启、销毁等工作，同时与 Master 节点协作，实现集群管理的基本功能。
dpkg -i kubelet_1.11.1-00_amd64.deb
#命令行工具
dpkg -i kubectl_1.11.1-00_amd64.deb
#kubeadm官方给出的安装方法
dpkg -i kubeadm_1.11.1-00_amd64.deb
```
#### 导入k8s安装所需镜像
```
#解压压缩包
$ tar xzvf 002.002.k8s.master.v1.11.1.tar.gz
#进入文件进行安装
$ cd k8s.master.v1.11.1
#运行脚本进行镜像的导入
$ ./loadall.sh
```
#### 安装k8s
```
#解压压缩包
$tar xzvf 003.kubeadm_init.tar.gz
#进入文件
cd kubeadm_init
#进行初始化安装
$ ./kubeadm_init.sh #注意修改脚本中初始化的网络地址
```
**执行后内容**
```
Your Kubernetes master has initialized successfully!
To start using your cluster, you need to run the following as a regular user:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/
You can now join any number of machines by running the following on each node
as root:
#其他的都不是太重要主要是这一句是用来让其他node节点加入集群的操作所以要保存
kubeadm join 192.168.110.158:6443 --token ttw6dz.d2v8nqtyx5kipxlu --discoverytoken-ca-cert-hash
sha256:df233ca36396ba0ecc2b727ca8b4dd6074091c27f7644ae054d671d5b0d27b79
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
configmap/calico-config created
service/calico-typha created
deployment.apps/calico-typha created
daemonset.extensions/calico-node created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
serviceaccount/calico-node created
```
**如果失败**
```
#如果创建失败或者是集群加入失败可以使用如下命令重新开始
kubeadm reset重新设置
```
### Node节点操作
#### 设置加速器文件
```
#执行加速器命令
$ curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
#修改加速器文件
sudo vim /etc/docker/daemon.json
#内容如下 的ip为默认使用的仓库
{"registry-mirrors": ["http://f1361db2.m.daocloud.io"],"insecure-registries": ["192.168.110.116:5000"]}
```
#### 安装Kubeadm等程序
```
#依赖
apt-get install ethtool ebtables -y
#将002.001.k8s.deb.v1.11.1.tar.gz解压
$ tar xzvf 002.002.k8s.node.v1.11.1.tar.gz
#进入文件夹
$ cd k8s.deb.v1.11.1
#启动安装脚本
$ ./install.sh
```
#### 安装node节点相关程序
```
#解压
tar xzvf 004.kubernetes-dashboard.tar.gz
#进入
cd kubernetes-dashboard
#执行脚本
./install.sh
```
#### 使用命令加入集群
```
#这个命令是在创建master的时候或得到的
$ kubeadm join 192.168.110.116:6443 --token clgahe.stw4d93ihds0k5d4 --discoverytoken-ca-cert-hash sha256:2ccc882ca893f79749fe7ba8e81a9ed87772cfff4dda6f88db68b6858e00081b
#如果不成功请重新来过
```
### 在Master上安装Dashboard
web ui 图形化界面
```
#解压
$ tar xzvf 004.kubernetes-dashboard.tar.gz
#进入
$ cd kubernetes-dashboard
#执行
$ ./install.sh
```
## k8s的基本命令
### 环境变量设置
```
#默认情况下，我们没有办法直接使用tab方式补全kubernetes的命令
#获取相关环境配置
kubectl completion bash
#加载这些配置
source <(kubectl completion bash)
#注意: "<(" 两个符号之间没有空格放到当前用户的环境文件中
echo "source <(kubectl completion bash)" >> ~/.bashrc
#测试效果
kubectl dedelete describe
```
### 查看全部信息
```
kubectl get service,pods,deployment,nodes,namespaces -o wide
```
### docker run
**使用docker**
```
$ docker run -d --name nginx-app -p 80:80 nginx
a9ec34d9878748d2f33dc20cb25c714ff21da8d40558b45bfaec9955859075d0
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
a9ec34d98787 nginx "nginx -g 'daemon of 2 seconds ago Up 2 seconds 0.0.0.0:80->80/tcp, 443/tcp nginx-app
```
**使用kubectl：**
```
# start the pod running nginx
$ kubectl run --image=nginx nginx-app --port=80 --env="DOMAIN=cluster"
```
#### docker ps
**使用docker：**
```
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
a9ec34d98787 nginx "nginx -g 'daemon of About an hour ago Up About an hour 0.0.0.0:80->80/tcp, 443/tcp nginx-app
```
**与kubectl：**
```
$ kubectl get po
NAME READY STATUS RESTARTS AGE
nginx-app-5jyvm 1/1 Running 0 1h
```
### docker exec
如何在容器中执行命令？参考[kubectl_exec](http://kubernetes.kansea.com/docs/user-guide/kubectl/kubectl_exec/)
**使用docker：**
```
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
a9ec34d98787 nginx "nginx -g 'daemon of 8 minutes ago Up 8 minutes 0.0.0.0:80->80/tcp, 443/tcp nginx-app
$ docker exec a9ec34d98787 cat /etc/hostname
a9ec34d98787
```
**使用kubectl：**
```
$ kubectl get po
NAME READY STATUS RESTARTS AGE
nginx-app-5jyvm 1/1 Running 0 10m
$ kubectl exec nginx-app-5jyvm -- cat /etc/hostname
nginx-app-5jyvm
```
**交互式命令**  
**使用docker：**
```
$ docker exec -ti a9ec34d98787 /bin/sh
# exit
```
**使用kubectl：**
```
$ kubectl exec -ti nginx-app-5jyvm -- /bin/sh
# exit
```
### docker logs
**使用docker：**
```
$ docker logs -f a9e
192.168.9.1 - - [14/Jul/2015:01:04:02 +0000] "GET / HTTP/1.1" 200 612 "-"
"curl/7.35.0" "-"
192.168.9.1 - - [14/Jul/2015:01:04:03 +0000] "GET / HTTP/1.1" 200 612 "-"
"curl/7.35.0" "-"
```
**使用kubectl：**
```
$ kubectl logs -f nginx-app-zibvs
10.240.63.110 - - [14/Jul/2015:01:09:01 +0000] "GET / HTTP/1.1" 200 612 "-"
"curl/7.26.0" "-"
10.240.63.110 - - [14/Jul/2015:01:09:02 +0000] "GET / HTTP/1.1" 200 612 "-"
"curl/7.26.0" "-"
```
### docker stop 和 docker rm
**使用docker：**
```
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED
STATUS PORTS NAMES
a9ec34d98787 nginx "nginx -g 'daemon of 22 hours ago Up
22 hours 0.0.0.0:80->80/tcp, 443/tcp nginx-app
$ docker stop a9ec34d98787
a9ec34d98787
$ docker rm a9ec34d98787
a9ec34d98787
```
**使用kubectl：**
```
$ kubectl get deployment nginx-app
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
nginx-app 1 1 1 1 2m
$ kubectl get po -l run=nginx-app
NAME READY STATUS RESTARTS AGE
nginx-app-2883164633-aklf7 1/1 Running 0 2m
$ kubectl delete deployment nginx-app
deployment "nginx-app" deleted
$ kubectl get po -l run=nginx-app
# Return nothing
```
**注意，不要直接删除pod，使用kubectl请删除拥有该pod的Deployment。如果直接删除pod，则Deployment将会重新创建该pod。**
### docker version
**使用docker：**
```
$ docker version
Client version: 1.7.0
Client API version: 1.19
Go version (client): go1.4.2
Git commit (client): 0baf609
OS/Arch (client): linux/amd64
Server version: 1.7.0
Server API version: 1.19
Go version (server): go1.4.2
Git commit (server): 0baf609
OS/Arch (server): linux/amd64
```
**使用kubectl：**
```
$ kubectl version
Client Version: version.Info{Major:"0", Minor:"20.1", GitVersion:"v0.20.1",GitCommit:"", GitTreeState:"not a git tree"}
Server Version: version.Info{Major:"0", Minor:"21+", GitVersion:"v0.21.1-411-g32699e873ae1ca-dirty", GitCommit:"32699e873ae1caa01812e41de7eab28df4358ee4",GitTreeState:"dirty"}
```
### docker info
**使用docker：**
```
$ docker info
```
**使用kubectl：**
```
$ kubectl cluster-info
```
### docker inspect
**使用docker：**
```
$ docker inspect nginx
```
**使用kubectl：**
```
$ kubectl describe deployment.extensions/nginx-app
```