[TOC]

# 2. 集群环境搭建

## 2.2 环境搭建

### 2.2.3 安装Docker

```powershell
# 1.切换镜像源
[root@master ~]# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.rpo -O /etc/yum.repos.d/docker-ce.repo

# 2.查看当前镜像源中支持的docker版本
[root@master ~]# yum list docker-ce --showduplicates

# 3.安装指定版本的docker-ce
# 必须指定 --setopt=obsoletes=0，否则 yum 会自动安装更高版本
[root@master ~]# yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y

# 4.添加配置文件
# Docker在默认情况下使用Cgroup Driver作为cgroupfs，而Kubernetes推荐使用systemd来代替cgroupfs
[root@master ~]# mkdir /etc/docker
[root@master ~]# cat <<EOF > /etc/docker/daemon.json
> {
>     "exec-opts": ["native.cgroupdriver=systemd"],
>     "registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
> }
> EOF

# 5.启动Docker
[root@master ~]# systemctl restart docker
[root@master ~]# systemctl enable docker
[root@master ~]# docker version
```

### 2.2.4 安装Kubernetes组件

```powershell
# Kubernetes镜像源在国外，这里切换成国内镜像
# 编辑 /etc/yun.repos.d/kubernetes.repo，添加如下配置
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http:/mirrors.aliyun.com/kubernetes/yum/doc/rpc-package-key.gpg

# 安装Kubeadm、Kubelet和Kubectl
[root@node2 ~]# yum install --setopt=obsoletes=0 kubeadm=1.17.4-0 kubelet-1.17.4-0 kubectl-1.17.4-0 -y

# 配置Kubelet的cgroup
# 编辑/etc/sysconfig/kubelet，添加下面的配置
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"

# 设置Kubelete开机自启
[root@master ~]# systemctl enable kubelet
```

### 2.2.5 准备集群镜像

```powershell
# 在安装kubernetes集群之前，必须要提前准备好集群所需的镜像，所需镜像可以通过一下命令查看
[root@master ~]# kubeadm config images list

# 下载镜像
# 此镜像在kubernetes仓库中，由于网络原因无法下载，下面提供了一种解决方案
images=(kube-apiserver:v1.17.4 kube-controller-manager:v1.17.4 kube-scheduler:v1.17.4 kube-proxy:v1.17.4 pause:3.1 etcd:3.4.3-0 coredns:1.6.5)

for imageName in ${images[@]}; do  
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName;
	docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName;
	docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName; 
done
```

### 2.2.6 集群初始化

下面开始对集群进行初始化，并将node节点加入到集群中

> 以下操作只需要在`master`节点上执行即可

```powershell
# 创建集群
[root@master ~]# kubeadm init \
	--kubernetes-version=v1.17.4 \
	--pod-network-cidr=10.244.0.0/16 \
    --service-cidr=10.96.0.0/12 \
    --apiserver-advertise-address=192.168.150.10 # IP换成自己master地址
# 创建必要的文件（根据提示信息安装即可）
[root@master ~]# mkdir -p $HOME/.kube
[root@master ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

> 下面的操作只需要在`node`节点上执行即可

```powershell
# 根据提示信息，复制提示信息中的内容到node节点执行即可
[root@node1 ~]# kubeadm join 192.168.150.10:6443 --token h22qcs.wsvonvmnvk0fdawl \
>     --discovery-token-ca-cert-hash sha256:b9cf9fa75225efaf96ecdcce38b288caeae94ddabb364644a989427abfd6a201
```

### 2.2.7 安装网络插件

knbernetes支持多种网络插件，比如flannel、calico、canal等，任选一种即可，本次选择flannel

> 本次操作依旧只在`master`节点执行，插件使用的是DaemonSet的控制器，它会在每个节点上都运行

```powershell
# 获取fannel的配置文件
[root@master ~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 使用配置文件启动fannel
[root@master ~]# kubectl apply -f kube-flannel.yml

# 稍等片刻，再次查看集群节点状态(速度较慢)
[root@master ~]# kubectl get nodes
NAME     STATUS     ROLES    AGE   VERSION
master   Ready      master   43m   v1.17.4
node1    Ready      <none>   38m   v1.17.4
node2    Ready      <none>   39m   v1.17.4
```

至此，Kubernetes的集群环境搭建完成

## 2.3 服务部署

在Kubernetes中部署一个nginx，测试下集群是否正常工作

```powershell
# 部署nginx
[root@master ~]# kubectl create deployment nginx --image=nginx:1.14-alpine

# 暴露端口
[root@master ~]# kubectl expose deployment nginx --port=80 --type=NodePort

# 查看服务状态
[root@master ~]# kubectl get pods,svc
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        23h
service/nginx        NodePort    10.96.53.197   <none>        80:32193/TCP   15s
```

