> 本文作者：丁辉

[jumpserver官网地址](https://docs.jumpserver.org/zh/master/install/setup_by_fast/?h=labels)

> 本文记录 helm 部署时遇到的问题，和解决方案(本来很简单不想记录，但是问题实在头疼)

> 根据官网往下走，开始部署 jumpserver
>
> 遇到问题: 
>
> 1.对接 nfs 困难，不自动创建pv
>
> 2.组件启动一直处于准备状态未启动

问题解决:

1.

[解决方案](https://github.com/dinghui40/huawei/blob/master/Nfs/nfs.md)

啊，简单说一下就是去 Kubernetes SIGs 转了一圈，找到个最新版对接 nfs 存储的方案，具体就不说了

[Kubernetes SIGs](https://github.com/kubernetes-sigs)

2.

[解决方案](https://github.com/dinghui40/k8s/blob/master/%E9%94%99%E8%AF%AF%E7%B4%AF%E7%A7%AF%E8%AE%B0%E5%BD%95/%E4%BD%BF%E7%94%A8%E6%97%B6%E9%81%87%E5%88%B0%E9%97%AE%E9%A2%98.md)

可以看这篇文章

###### 部署 service

```
vi jump-svc.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: jump-service
spec:
  selector:
    app.jumpserver.org/name: jms-web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30000
  type: NodePort
```

ok啦