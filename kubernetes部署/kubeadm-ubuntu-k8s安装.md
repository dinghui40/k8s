> cat /etc/os-release

本次部署为云服务器containerd部署

```
cat >> /etc/sysctl.conf<<'EOF'
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness = 0
EOF
```

关闭swap和防火墙#云主机此步可省略

```
swapoff -a
sysctl -w vm.swappiness=0
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

systemctl disable --now firewalld
systemctl disable --now dnsmasq
systemctl disable --now NetworkManager
setenforce 0
sed -i   's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
确认状态
getenforce
显示：Disabled
```

按照版本选取

### Ubuntu 22.04

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable"
```

### Ubuntu 21.10

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu impish stable"
```

### Ubuntu 21.04

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu hirsute stable"
```

### Ubuntu 20.10 

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu groovy stable"
```

### Ubuntu 20.04

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

### Ubuntu 19.10 

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu eoan stable"
```

### Ubuntu 19.04

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu disco stable"
```

### Ubuntu 18.10

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu cosmic test"
```

### Ubuntu 18.04

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```

### Ubuntu 17.10

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu artful stable"
```

### Ubuntu 16.04

```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable"
```

安装containerd

```
apt update

apt install containerd.io
 
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

apt install ipset ipvsadm -y
mkdir /etc/sysconfig/modules -p
vim /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack

chmod +755 /etc/sysconfig/modules/ipvs.modules 
bash /etc/sysconfig/modules/ipvs.modules
lsmod | grep -e ip_vs -e nf_conntrack
containerd config default | sudo tee /etc/containerd/config.toml

vi /etc/containerd/config.toml
修改
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"
SystemdCgroup = true

systemctl daemon-reload
systemctl enable containerd.service

systemctl status containerd.service

modprobe br_netfilter
sysctl -p

apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt install bash-completion

vim .bashrc
source <(crictl completion bash)
source <(kubectl completion bash)
source <(kubeadm completion bash)
source /root/.bashrc

vim /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false

crictl config runtime-endpoint unix:///run/containerd/containerd.sock

crictl image ls
#
#vim /etc/containerd/config.toml

修改runtime_type

runtime_type = "io.containerd.runtime.v1.linux"
#
systemctl enable kubelet
kubeadm config print init-defaults > init.yaml
vim init.yaml 

<
advertiseAddress: 
name: 
kubernetesVersion:
serviceSubnet:
imageRepository: registry.aliyuncs.com/google_containers
>

kubeadm init --config=init.yaml
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf

curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O
vim calico.yaml
改ip
kubectl apply -f calico.yaml 


https://blog.csdn.net/weixin_41831919/article/details/108349510

kubectl describe nodes master | grep -A 3 Taints
kubectl taint nodes

vim /etc/kubernetes/manifests/kube-controller-manager.yaml
增加参数：

--allocate-node-cidrs=true
--cluster-cidr=10.244.0.0/16

systemctl restart kubelet

-o json > /tmp/dev.json
kubectl proxy
curl -k -H “Content-Type: application/json” -X PUT --data-binary @dev.json http://127.0.0.1:8001/api/v1/namespaces/dev/finalize
https://www.likecs.com/show-911662.html
```

```
https://www.cnblogs.com/hahaha111122222/p/15802880.html
https://blog.csdn.net/tongzidane/article/details/124621975
https://github.com/kubernetes/kubernetes/issues/70202
```

```
apt-get update
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet kubeadm kubectl
```

