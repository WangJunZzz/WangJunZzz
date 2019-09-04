---
title: Kubernetes
date: 2019-09-02 20:48:58
tags:
---

+ 使用阿里云镜像安装Kubernetes
    - 前期准备 安装Docker;[修改docker镜像源](https://www.kancloud.cn/hfpp2012/docker/467098)
``` bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF
```
```bash
# 禁用主机SELinux，让容器可以读取主机文件系统
setenforce 0
# 关闭防火墙
systemctl disable firewalld
systemctl stop firewalld
# 安装kubeadm和相关工具
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
# 设置开机启动 
systemctl enable kubelet 
systemctl start bubelet
# 执行kubeadm config print init-defaults，可以获取默认的初始化配置
kubeadm config print init-defaults > init.default.yaml
# 对生成的文件进行编辑，可以按需生成合适的配置。
# 例如，如需要定制镜像仓库的地址，以及pod的地址网络范围，可以使用如下配置，保存为init-config.yaml
apiVersion: kubernetes,k8s.io/v1beta1
kind: ClusterConfiguration
imageRepository: docker.io/dustise
kubernetesVersion: v1.14.0
networking:
   podSubnet: "192.168.0.0/16"

# 禁用 swap (启用命令：swapon -a)
swapoff -a

#下载kubernetes所需要的相关镜像
kubeadm config images pull --config=init-config.yaml

```

 ![](/images/image-pull.png)
