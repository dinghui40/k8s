> 本文作者：丁辉

# kubeadm-centos7-containerd-k8s安装

[阿里云yum源加速地址](https://developer.aliyun.com/mirror/) > 点击容器 > 点击Kubernetes

[官网网络插件列表](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/addons/#networking-and-network-policy)

[containerd工具](https://github.com/containerd/nerdctl/releases)

### 准备：

```bash
#环境准备
systemctl stop firewalld
sudo setenforce 0

#禁用selinux and swap
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

#加载内核模块
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

配置底层网络

```
cat > /etc/sysconfig/modules/ipvs.modules <<EOF    
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
```

```
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

```
lsmod | grep netfilter
sysctl -p /etc/sysctl.d/k8s.conf
sysctl --system
```



### 开始部署：

查看版本

```
yum list kubeadm  --showduplicates | sort -r | grep 1.19.0
```

部署个别版本

```
yum install --nogpgcheck  kubeadm-1.18.8-0 kubelet-1.18.8-0 kubectl-1.18.8-0 -y
```

kubelet加入开机自启并启动 > 导出k8s初始化文件

```
systemctl enable kubelet && systemctl start kubelet
kubeadm config print init-defaults > k8sinit.yaml
```

修改文件

```
vim k8sinit.yaml
```

主要修改这几个位置

```
advertiseAddress: $IP
name: $Name
kubernetesVersion: $默认版本
serviceSubnet: $IP
imageRepository: registry.aliyuncs.com/google_containers
```

配置命令补全和简化

```
vim ~/.bashrc
```

```
source <(crictl completion bash)
source <(kubectl completion bash)
source <(kubeadm completion bash)
alias k="kubectl"
```

```
source /root/.bashrc
```

#安装之前请注意需要将 k8sinit.yaml 文件里面设置的 name 名写入到 /etc/hosts 文件里，要不会安装超时

开始安装

```
kubeadm init --config=k8sinit.yaml
```

安装完成配置 kubectl config

> 有可能和生成的不一致，请按照安装完成后提示的操作

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
```

重置 k8s 

> echo y|kubeadm reset

```
kubeadm reset
```

加入集群生成token

```bash
kubeadm token create --ttl 0 --print-join-command
```

添加网络（按需要修改网络配置）这里网络使用的flannel

```
kubectl apply -f https://ghproxy.com/raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

