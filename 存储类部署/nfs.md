> 本文作者：丁辉

## ubuntu部署

安装

```
sudo apt install nfs-kernel-server
```

检查

```
sudo systemctl status nfs-server
```

创建共享目录

```
mkdir /nfs
sudo chmod -R 777 /nfs
```

添加配置文件

```
sudo vim /etc/exports
```

```
/nfs 127.0.0.1(rw,sync,no_subtree_check)
```

加载

```
sudo exportfs -arv
```

查看

```
showmount -e 127.0.0.1
```

客户端部署

```
sudo apt install nfs-common
```

## centos部署

安装

```
yum install nfs-utils rpcbind
```

创建共享目录

```
mkdir /nfs
sudo chmod -R 777 /nfs
```

添加配置文件

```
vi /etc/exports
```

```
/nfs 127.0.0.1(rw,sync,insecure,no_subtree_check,no_root_squash)
```

启动

```
systemctl start rpcbind.service
service nfs start
sudo systemctl enable rpcbind
sudo systemctl enable nfs
```

查看

```
showmount  -e localhost
```

客户端部署

```
yum install nfs-utils
```

## k8s对接本地nfs

1.20版本

```
git clone https://github.com/helm/charts.git 
cd charts/ 
```

```
helm install nfs-storageclass stable/nfs-client-provisioner --set nfs.server=127.0.0.1 --set nfs.path=/nfs
```

修改apiserver参数

```
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```
- --feature-gates=RemoveSelfLink=false
```

1.24版本

[官网介绍](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/blob/master/charts/nfs-subdir-external-provisioner/README.md)

```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=127.0.0.1 \
    --set nfs.path=/nfs
```

## nfs优化

```
echo "options sunrpc tcp_slot_table_entries=128" > /etc/modprobe.d/sunrpc.conf
echo "options sunrpc tcp_max_slot_table_entries=128" >> /etc/modprobe.d/sunrpc.conf
sysctl -w sunrpc.tcp_slot_table_entries=128
```

```
reboot
```

