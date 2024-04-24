[TOC]

# 2. 集群环境搭建

## 2.2 环境搭建

### 2.2.3 安装Docker

```shell
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

```shell
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

```shell
# 在安装kubernetes集群之前，必须要提前准备好集群所需的镜像，所需镜像可以通过一下命令查看

```

