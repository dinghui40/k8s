> 本文作者：丁辉

# rook方式-部署ceph

[官网地址](https://rook.io/)

> 本文记录对于 rook 安装 ceph 大家可能需要修改的文件或配置

## 拉取rook git包

[官网地址](https://rook.io/) > Documentation > cephalopods > Quickstart （链接根据版本变化，请依次点击查找）

默认进入 `rook/deploy/examples/` 此位置进行操作

## 更改ceph启动所需要的镜像地址

> images.txt 文件记录了 ceph 所需要用到的镜像
>
> ```
> #下列3个镜像需要依靠网速（请科学上网）
> rook/ceph:v*
> quay.io/ceph/ceph:v*
> quay.io/cephcsi/cephcsi:v*
> 
> #下列5个镜像，阿里云实时同步。地址：registry.aliyuncs.com/google_containers/<image>:<tag>
> registry.k8s.io/sig-storage/csi-attacher:v*
> registry.k8s.io/sig-storage/csi-node-driver-registrar:v*
> registry.k8s.io/sig-storage/csi-provisioner:v*
> registry.k8s.io/sig-storage/csi-resizer:v*
> registry.k8s.io/sig-storage/csi-snapshotter:v*
> 
> #下列3个镜像为不常用镜像可以忽略
> #quay.io/csiaddons/k8s-sidecar:v*
> #quay.io/csiaddons/volumereplication-operator:v*
> #registry.k8s.io/sig-storage/nfsplugin:v*
> ```

修改 operator.yaml 文件

```
#解除此段注释，后面填写镜像
ROOK_CSI_CEPH_IMAGE: ""
ROOK_CSI_REGISTRAR_IMAGE: ""
ROOK_CSI_RESIZER_IMAGE: ""
ROOK_CSI_PROVISIONER_IMAGE: ""
ROOK_CSI_SNAPSHOTTER_IMAGE: ""
ROOK_CSI_ATTACHER_IMAGE: ""

#修改 image 字段
image:
```

修改 cluster.yaml 文件

```
修改 image 字段
image:
```

## 更改ceph启动所需要的磁盘位置以及名字

修改 cluster.yaml 文件

```
storage: 
    useAllNodes: false  #关闭使用所有Node
    useAllDevices: false #关闭使用所有设备

#解除nodes：注释填入节点name和指定磁盘
    nodes: 
      - name: "192.168.1.1"
        devices: 
          - name: "sdb"
          - name: "sdc" 
      - name: "192.168.1.2"
        devices: 
          - name: "sdb"
          - name: "sdc" 
```

## 利用污点选取节点

控制 mon osd mgr

修改 cluster.yaml 文件

```
placement:
    mon:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: ceph-mon
              operator: In
              values:
              - enabled
    osd:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: ceph-osd
              operator: In
              values:
              - enabled
    mgr:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: ceph-mgr
              operator: In
              values:
              - enabled
```

给节点打标签

```
kubectl label nodes {kube-node1,kube-node2,kube-node3} ceph-mon=enabled
kubectl label nodes {kube-node1,kube-node2,kube-node3} ceph-osd=enabled
ceph-mgr最多只能运行2个
kubectl label nodes {kube-node1,kube-node2} ceph-mgr=enabled
```

控制 mds

修改 filesystem.yaml 文件

```
placement:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
           - matchExpressions:
             - key: role
               operator: In
               values:
               - mds-node
```

给节点打标签

```
kubectl label nodes {kube-node1,kube-node2,kube-node3} role=mds-node
```

控制

修改 object.yaml 文件

```
nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: rgw-node
            operator: In
            values:
            - enabled
```

给节点打标签

```
kubectl label nodes {kube-node1,kube-node2,kube-node3} rgw-node=enabled
```

## 清理ceph环境

> 在安装集群的时候不知道为什么会遇到这样的一种情况
>
> 当我在第一次 apply  了cluster.yaml 文件之后
>
> 因为某些原因不得不 delete 重新搭建
>
> 此时我的 mgr 资源不出现了，operator 日志也在 osd 资源那里循环
>
> 解决方案

如果您想拆除集群并启动一个新集群，请注意以下需要清理的资源：

- `rook-ceph`namespace：由`operator.yaml`and `cluster.yaml`（集群CRD）创建的Rook算子和集群
- `/var/lib/rook`: 集群中每个主机上的路径，配置由 ceph mons 和 osds 缓存

请注意，如果您更改了`dataDirHostPath`示例 yaml 文件中的默认命名空间或路径，则需要在这些说明中调整这些命名空间和路径。

