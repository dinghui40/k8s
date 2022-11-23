> 本文作者：丁辉

# k8s部署Grafana

## 官网文档地址：

[Grafana官网部署文档](https://grafana.com/docs/grafana/latest/setup-grafana/installation/kubernetes/)

[k8s官网 PersistentVolume 介绍](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

### 介绍：

>  提示如果是初始化k8s环境，没有pv将报错，默认官网部署Grafana不会帮助你创建pv

### 开始部署：

创建本地之久化目录

```
mkdir /mnt/data
chmod 777 /mnt/data
```

编写yaml文件 > 创建pv

```
vi grafana-pv.yaml
```

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

```
kubectl apply -f grafana.yaml
```

> 注意 grafana 官方 yaml 如果没有指定 storageClassName ，需要我们手动添加
>
> 为`grafana.yaml`文件相同位置指定`storageClassName: manual`进行绑定

### 访问

本地访问

```
kubectl port-forward service/grafana 3000:3000
```

访问`localhost:3000`

外网访问

```
vi grafana-svc.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
spec:
  selector:
    app: grafana
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
    nodePort: 30001
  type: NodePort
```

访问`IP:30001`



Grafana默认初始化密码: `admin/admin`

