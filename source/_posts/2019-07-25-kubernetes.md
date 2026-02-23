---
title: Kubernetes
tags:
  - docker
  - k8s
categories: kubernetes
abbrlink: 9acacb00
date: 2019-07-25 10:32:38
---
2019-07-25-K8S
<!-- more -->

# Kubernetes
## kubernetes 简介:
> **Kubernetes**由Google团队发起并维护的基于Docker的开源容器集群管理系统, 目标是管理跨多个主机的容器, 它不仅支持常见的云平台, 而且支持内部数据中心.
> 核心概念是 **Container Pod**--> 一个 Pod（容器集合）由`一组工作于同一物理工作节点的容器构成`, 组容器拥有相同的网络命名空间, IP以及存储配额, 也可以根据实际情况对每一个 Pod 进行端口映射.
> kubernetes 组件构成:

{% asset_img k8s.jpg [kubernetes组件] %}

## kubernetes 组件构成: 
### Master组件:
+ `kube-apiserver`: **Kubernetes API**, **集群统一入口**, **各组件协调者**, 以HTTP API提供接口服务, **所有对象资源的增删查改**和监听操都交给APIServer处理后再提交给**Etcd存储**;
+ `kube-scheduler`: （调度器）负责对资源进行调度, 分配未分发 Pod绑定到可用的Node节点上, 存储到etcd中;
+ `kube-controller-manager`: 负责管理控制器, 一个资源对应一个控制器, CotrolerManager负责管理监控pod运行状态, 根据 etcd中的信息, 调用 node中的kubelet创建Pod; 

### Node组件:
+ `kubelet`: 是 Master在 Node节点上的 Agent, 负责具体的容器生命周期管理, 根据从数据库中获取的信息来管理容器, 并上报 pod运行状态等, 下载secret、获取容器与节点状态, kubelet将每个 Pod装换成一组容器;
+ `kube-proxy`: 在 Node节点实现 Pod网络代理;
+ `docker/rocker/rkt`: 运行容器

### 第三方服务(master && node):
+ `etcd`: 用来保存集群所有状态的 Key/Value`存储系统`, 所有 Kubernetes组件会通过 API Server来跟 Etcd进行沟通从而保存或读取资源状态。如Pod、service等对象信息;

### kubectl: 
> kubectl 是 Kubernetes 自带的客户端, 可以用它来直接操作 Kubernetes

| command        | mean                                                     |
| -------------- | -------------------------------------------------------- |
| get            | 显示一个或多个资源                                       |
| describe       | 显示特定资源的详细信心                                   |
| create         | 通过filename或stdin创建资源                              |
| update         | 通过filename或stdin更新资源                              |
| delete         | 删除资源                                                 |
| namespace      | 设置并查看当前的Kubernetes命名空间                       |
| log            | 在容器中打印容器的日志                                   |
| rolling-update | 执行给定ReplicationController的滚动更新                  |
| resize         | 为Replication Controller设置新大小                       |
| exec           | 在容器中执行命令                                         |
| port-forward   | 将一个或多个本地端口转发到容器                           |
| proxy          | 运行代理到Kubernetes API服务器                           |
| run-container  | 在群集(cluster)上运行特定映像                            |
| stop           | 通过id或filename正常关闭资源                             |
| expose         | 获取replicated application并将其公开为Kubernetes Service |
| lable          | 更新资源标签                                             |
| config         | 更改kuber相关配置文件                                    |
| cluster-info   | 查看集群信息                                             |
| api-version    | 打印可用的API版本                                        |
| version        | 查看客户端和服务端版本信息                               |
| help           | 查看关于某个命令的帮助信息                               |

### Kubernetes CNI网络最强对比：Flannel、Calico、Canal和Weave
+ **[Kubernetes 网络](https://blog.csdn.net/RancherLabs/article/details/88885539)**  

## 搭建  
+ **[参考文章2](https://www.kubernetes.org.cn/5462.html)**    
### 社区方案: `优点`是搭建简单、参考文档细致; `缺点`是太杂乱, 升级不便 
### 官方推荐1: `Kubeadm` 
+ `优点`: 简单、官方推荐、升级方便、支持高可用; `缺点`: 不易维护 
### 官方推荐2: `Binary`
+ `优点`: 易于维护、灵活、升级方便; `缺点`: 搭建太过复杂, 相关文档太少

## Kubeadm:
### 环境准备:
#### 1.硬件信息:
| 系统类型  | IP             | 节点   | Hostname |
| :-------- | :------------- | :----- | :------- |
| CentOS7.2 | 192.168.137.13 | master | m1       |
| CentOS7.2 | 192.168.137.14 | master | m2       |
| CentOS7.2 | 192.168.137.15 | master | m3       |
| CentOS7.2 | 192.168.137.30 | worker | n1       |
| CentOS7.2 | 192.168.137.32 | worker | n2       |

{% asset_img k8s单master.jpg [单-master] %}  

#### 2.设置主机名`hostname`, 每一台配置`hosts`文件域名解析:  <div id="kubeadm"></div>
```bash
cat <<EOF >>/etc/hosts   # 追加
  192.168.137.13 m1
  192.168.137.13 m2
  192.168.137.13 m3
  192.168.137.30 n1
  192.168.137.32 n2
  EOF
```

#### 3.关闭`防火墙`、`SELinux`及`swap`:
```bash
systemctl stop firewalld
systmectl disable firewalled

iptables -F
iptables -X && iptables -F -t nat
iptables -X -t nat && iptables -P FORWARD ACCEPT

setenforce 0
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab 
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
```

#### 4.配置内核参数，将桥接的IPv4流量传递到iptables的链
```bash
cat <<EOF > /etc/sysctl.d/k8s.conf   # 多行输入
  net.bridge.bridge-nf-call-iptables=1
  net.bridge.bridge-nf-call-ip6tables=1
  net.ipv4.ip_forward=1
  vm.swappiness=0
  vm.overcommit_memory=1
  vm.panic_on_oom=0
  fs.inotify.max_user_watches=89100
  EOF

sysctl --system
sysctl -p /etc/sysctl.d/kubernetws.conf
```

#### 5.配置国内yum源、Docker源及Kubernets源:
```bash
yum install -y wget
mkdir /etc/yum.repos.d/bak && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
wget http://mirrors.cloud.tencent.com/repo/epel-7.repo -O /etc/yum.repos.d/epel.repo 
yum clean all && yum makecache

wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

cat <<EOF > /etc/yum.repo.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
  EOF
```

### 软件安装:
#### 6.所有节点安装docker、kuberadm、kubelet及kubectl
+ kubeadm: 部署集群用的命令
+ kubelet: 在集群中每台机器上都要运行的组件，负责管理pod、容器的生命周期  
+ kubectl: 集群管理工具(可选, 只要在控制集群的节点上安装即可)

```bash
yum update 
yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp

yum remove -y docker* container-selinux
yum install -y docker-ce-18.06.1.ce-3.el7

cat <<EOF > /etc/docker/daemon.json
  {
    "graph": "/docker/data/path",	# df -h 找空间比较大的, 默认 /var/lib/docker
    "exec-opts": ["native.cgroupdrive=systemd"]	// 默认 cgroups
    "registry-mirrors": ["https://registry.docker-cn.com"] 
  }
  EOF

systemctl enable docker && systemctl start docker
docker -version
yum list kubeadm --showduplicates | sort -r	  # 找到要安装的版本号
yum install -y kubelet-1.13.3 kubeadm-1.13.3 kubectl-1.13.3

# 设置kubelet的cgroupdriver (kubelet的cgroupdriver默认为systemd, 如果上面没有设置docker的exec-opts为systemd, 这里就需要将kubelet的设置为cgroupfs)
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf	 
systemctl enable kubelet	# 设置开机启动暂时不启动
```

### 部署配置:
#### 7.下载参考配置文件
+ **[关于镜像下载问题](https://www.jianshu.com/p/74a6673ff78b)**   
+ **[kubeadm-config.yaml配置参考](https://github.com/HikoQiu/kubeadm-install-k8s)**   
+ **[gcr.io_mirror](https://github.com/anjia0532/gcr.io_mirror/tree/master/google-containers)**  

```bash
#!/bin/bash
# 下载镜像
images=(kube-proxy-amd64:v1.13.3 kube-apiserver-amd64:v1.13.3 kube-controller-manager-amd64:v1.13.3 kube-scheduler-amd64:v1.13.3 pause:3.1 etcd-amd64:3.2.18 coredns:1.1.3)
for imageName in ${images[@]}
do
    docker pull anjia0532/google-containers.$imageName
    docker tag anjia0532/google-containers.$imageName k8s.gcr.io/$imageName
    docker rmi anjia0532/google-containers.$imageName
done
./k8s-docker-images.sh
```

#### 8.部署mater节
```
kubeadm init \
  --kubernetes-version=1.13.3 \
  # 单master: api server地址是master本机IP地址
  --apiserver-advertise-address=192.168.137.13 \ 	
  --image-repository registry.aliyuncs.com/google_containers \
  --service-cidr=10.1.0.0/16 \	   # 负载均衡虚拟IP
  --pod-network-cidr=10.244.0.0/16 

kubeadm join :6443 --token kekvgu.nw1n76h84f4camj6 --discovery-token-ca-cert-hash sha256:4ee74205227c78ca62f2d641635afa4d50e6634acfaa8291f28582c7e3b0e30e

ls /etc/kubernetes/pki	 # 证书路径
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
  kubectl get node
  NAME		STATUS		ROLES		AGE		VERSION
  m1		NotReady	master		5m46s	v1.13.3	
```

#### 9.部署POD网络, 部署Flanne 网络插件
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
kubectl get pods -n kube-ststem
```

#### 10.加入Kubernetes Node, 即向集群添加新节点
```
kubeadm join --token <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:code
# 在所有 node节点 上执行如下命令:
kubeadm join 192.168.137.13:6443 --token kekvgu.nw1n76h84f4camj6 --discovery-token-ca-cert-hash sha256:4ee74205227c78ca62f2d641635afa4d50e6634acfaa8291f28582c7e3b0e30e

# master 查看node 状态
  kubectl get node
  NAME		STATUS		ROLES		AGE		VERSION
  m1		Ready		master		15m		v1.13.3	
  n1		Ready		<none>		4s		v1.13.3
  n2		Ready		<none>		88s		v1.13.3	

kubectl get pods -n kube-system
kubectl delete pods kube-flannel-ds-amd64-6jf7t -n kube-system
```

#### 11.创建Pod以验证集群是否正常
```bash
kubectl create deployment nginx --image=nginx
kubectl get pods
  NAME					READY	STATUS				RESTARTS	AGE
  nginx-5c7588df-fhhb9	0/1		ContainerCreating	0			28s
  kubectl expose deployment nginx --port=80 --type=NodePort
	
  kubectl get pods,svc -o wide	# 查看详细信息  
# 测试: 192.168.137.30:80
```

#### 12.部署Dashboard(master节点)
```yml
wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

# vim 手动修改yaml文件Dashboard 的获取地址
spec:
  containers:
    - name: kubernetes-dashboard
    - image: loveone/kubernetes-dashboard-amd64:v1.10.1
	
  spec:
    ports:
      - port: 443
         targetPort: 8443
         # nodePort: 30001	定义外部访问端口
        type: NodePort

docker pull loveone/kubernetes-dashboard-amd64:v1.10.1 # 也可以先下载镜像
kubectl apply -f kubernetes-dashboard.yaml
kubectl create -f kubernetes-dashboard.yaml

# 查看服务状态
kubectl get deployment kubernetes-dashboard -n kube-system
kubectl get pods -n kube-system -o wide
kubectl get services -n kube-system
kubectl get pods,svc -n kube-system
  NAME							TYPE		CLUSTER_IP	EXTERNAL_IP	PORT(S)			AGE
  service/kube-dns				ClusterIP	10.1.0.10	<none>		53/UDP,53/TCP	27m
  service/kubernetes-dashboard	NodePort	10.1.225.37	<none>		443:32569/TCP	53s

nestat -anltp |grep 8443
# 测试: https://192.168.137.13:32569
```

#### 13.登录认证
{% asset_img dashboard.jpg [dashboard认证界面] %}
  
```bash
# 查看访问Dashboard的认证令牌
kubectl create serviceaccount dashboard-admin -n kube-system
kubectl create clusterrolebinding  dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

kubectl get secret -n kube-system
  NAME						TYPE													DATA	AGE
  dashboard-admin-token-w6rtn	kuberbetes.io/service-account-token	default-token-82hvk	3		60s
kubectl describe secrets dashboard-admin-token-w6rtn -n kube-system
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

{% asset_img dashboard-页面.jpg [dashboard页面] %}

## 二进制部署:












