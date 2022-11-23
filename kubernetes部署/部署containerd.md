> 本文作者：丁辉

# 二进制安装-x86-部署containerd



## 二进制安装containerd

### 介绍：

> k8s 1.24 狠心拒绝了docker，那我们只能走containerd这条路了
>
> 本段文本记录 containerd 二进制部署过程，所有安装包需要按照下列链接下载（外网文件，请科学上网）
>
> containerd 二进制部署，基本适用于所有Liunx系统（不管是ubuntu或centos或---），都能一键启动

### 文件准备：

[containerd软件包](https://github.com/containerd/containerd/releases)

[containerd-systemd配置文件](https://github.com/containerd/containerd/blob/main/containerd.service)

[runc文件](https://github.com/opencontainers/runc/releases)

[cni插件包](https://github.com/containernetworking/plugins/releases)

[cri包](https://github.com/kubernetes-sigs/cri-tools/releases)

### 开始部署：

解压 containerd 二进制软件包 > 并将 containerd 加入 systemd 管理 > 启动

```
tar Cxzvf /usr/local containerd-*-linux-amd64.tar.gz
mkdir -p /usr/local/lib/systemd/system/
vi /usr/local/lib/systemd/system/containerd.service
```

```
systemctl daemon-reload
systemctl enable --now containerd
systemctl start containerd
```

安装runc工具

```
install -m 755 runc.amd64 /usr/local/sbin/runc
```

解压cni插件包

```
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-*.tgz
```

生成containerd配置文件

```
mkdir /etc/containerd/
containerd config default > /etc/containerd/config.toml
```

配置 systemd cgroup 驱动程序 > 修改沙箱获取镜像地址

```
vi /etc/containerd/config.toml
```

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true

[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"
```

解压CRI压缩包 > 写入containerd 默认 CRI 套接字（为k8s准备）

```
tar Cxzvf /usr/local/bin crictl-*-linux-amd64.tar.gz
vi /etc/crictl.yaml
```

```
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: false
pull-image-on-create: false
```

重启 containerd

```
systemctl restart containerd
```

[官方文档](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

安装 containerd 命令工具

```
tar Czxvf /usr/local/bin/ nerdctl-*-linux-amd64.tar.gz
```

# yum源部署containerd

## centos7部署containerd

### 介绍：

> 二进制安装，文件下载不下来很难受吧，哈哈哈
>
> [官网文档](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)
>
> crictl 工具请下载官方软件包

下载阿里云的yum源  > 安装依赖 > 添加阿里的docker镜像源 > 替换默认yum源

```
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

yum install wget jq psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 git -y

yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

安装

如果 ctr 无法使用请 yum 下载：crictl-tools

```
yum install containerd -y
containerd config default > /etc/containerd/config.toml
systemctl start containerd
systemctl enable containerd
 
# 修改cgroups为systemd
sed -i 's#SystemdCgroup = false#SystemdCgroup = true#' /etc/containerd/config.toml
systemctl daemon-reload
systemctl restart containerd
```

ctr debug报错问题

```
cat <<EOF> /etc/crictl.yaml 
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

## ubuntu部署containerd

### 介绍：

> 我部署containerd的时候遇到了各个版本 yum 源的问题，所以本次记录 ubuntu 源安装步骤

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

containerd config default | sudo tee /etc/containerd/config.toml
```

配置 systemd cgroup 驱动程序 > 修改沙箱获取镜像地址

```
vi /etc/containerd/config.toml
```

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true

[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"
```

启动

```
systemctl start containerd.service
systemctl daemon-reload
systemctl enable containerd.service
```

ctr debug报错问题

```
cat <<EOF> /etc/crictl.yaml 
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

