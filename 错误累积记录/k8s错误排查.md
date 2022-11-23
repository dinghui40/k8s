> 本文作者：丁辉

在我们部署 k8s 的时候，最近会遇到 flannel 一直重启的情况（报错 pod cidr not assigne）

解决方法

```
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
```

```
- --allocate-node-cidrs=true
- --cluster-cidr=10.244.0.0/16
```

```
systemctl restart kubelet
```

k8s-kubeadm单节点搭建导致主节点污点禁止pod运行

```
#查看污点
kubectl describe nodes master | grep -A 3 Taints
#删除
kubectl taint nodes <node> <taints>-
#一次删除多个
kubectl taint nodes --all node-role.kubernetes.io/control-plane- node-role.kubernetes.io/master-
```

#k8s-kubeadm-containerd搭建网络通讯问题

```
vim /etc/kubernetes/manifests/kube-controller-manager.yaml
增加参数：

- --allocate-node-cidrs=true
- --cluster-cidr=10.244.0.0/16

systemctl restart kubelet
```

k8s-ns 删除强制处理

```
1.
kubectl delete ns test --force --grace-period=0

2.
-o json > /tmp/dev.json

kubectl proxy --port=8081

curl -k -H "Content-Type:application/json" -X PUT --data-binary @test.json http://127.0.0.1:8081/api/v1/namespaces/<you delete ns>/finalize
```

K8s "tab" 报错

```
yum install bash-completion -y
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
```

删除Evicted pod

```
#!/bin/bash
for each in $(kubectl get pods|grep Evicted|awk '{print $1}');
do
  kubectl delete pods $each
done
```

```
kubectl get pods | grep Evicted | awk '{print $1}' | xargs kubectl delete pod
```

