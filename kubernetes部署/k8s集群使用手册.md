> 本文作者：丁辉

# 使用手册

> 本文用于记录不常用，但是非常有用的k8s使用内容

## 节点如何加入集群？

查看待加入节点

```
kubectl get csr
```

查看节点信息

```
kubectl describe csr $name
```

批准加入

```
kubectl certificate approve $name
```

查看结果

```
kubectl get csr
#CONDITION=Approved,Issued
```

## 从集群删除 Node

要删除一个节点前，要先清除掉上面的 pod,然后运行下面的命令删除节点

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

## 给 Node 打标签

查看所有节点状态

```bash
kubectl get nodes

NAME         STATUS   ROLES   ----
192.168.1.10   Ready    <none>
192.168.1.20   Ready    <none>
192.168.1.30   Ready    <none>
```

`192.168.1.10` 的 `Master` 打标签

```bash
kubectl label node 192.168.1.10 node-role.kubernetes.io/master='k8s-master1'
node/192.168.1.10 labeled
```

`192.168.1.30` 的 `Node` 打标签

```bash
kubectl label node 192.168.1.30 node-role.kubernetes.io/master='k8s-node1'
node/192.168.1.30 labeled
kubectl label node 192.168.1.30 node-role.kubernetes.io/node='k8s-node1'
node/192.168.1.30 labeled

kubectl get node
NAME         STATUS   ROLES   ----
192.168.1.10   Ready    <none>
192.168.1.20   Ready    <none>
192.168.1.30   Ready    <none>
```

删除掉 `192.168.1.30` 上的 `master` 标签

```bash
kubectl label node 192.168.1.30 node-role.kubernetes.io/master-
node/192.168.1.30 labeled

kubectl get node
NAME         STATUS   ROLES   ----
192.168.1.10   Ready    k8s-master1
192.168.1.20   Ready    <none>
192.168.1.30   Ready    k8s-node1
```


