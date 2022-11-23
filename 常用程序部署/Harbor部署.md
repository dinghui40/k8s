> 本文作者：丁辉

[官网地址](https://goharbor.io/docs/2.6.0/install-config/harbor-ha-helm/)

[包地址](https://github.com/goharbor/harbor/releases)

## docker-compose部署

```
tar -zxvf harbor-offline-installer-v*.tgz
cd harbor
cp harbor.yml.tmpl harbor.yml
vi harbor.yml
```

修改 hostname 等参数

部署

```
sh ./install.sh
```

## helm部署

修改 values.yaml

```
$ cd harbor    
$ ls
cert  Chart.yaml  conf  LICENSE  README.md  templates  values.yaml
$ vim  values.yaml
expose:
  type: nodePort         # 使用NodePort的服务访问方式。   
  tls:
    enabled: false    # 关闭tls安全加密认证
...
externalURL: http://192.168.1.1:30000 # 使用nodePort且关闭tls认证，则此处需要修改为http协议和expose.nodePort.ports.http.nodePort指定的端口号，IP即为kubernetes的节点IP地址

# 持久化存储配置部分
persistence:
  enabled: true   # 开启持久化存储
  resourcePolicy: "keep"
  persistentVolumeClaim:        # 定义Harbor各个组件的PVC持久卷部分
    registry:          # registry组件（持久卷）配置部分
      existingClaim: ""
    storageClass: "harbor-storageclass"
      subPath: ""
      accessMode: ReadWriteMany          # 卷的访问模式，需要修改为ReadWriteMany，允许多个组件读写，否则有的组件无法读取其它组件的数据
      size: 5Gi
    chartmuseum:     # chartmuseum组件（持久卷）配置部分
      existingClaim: ""
      storageClass: "harbor-storageclass"
      subPath: ""
      accessMode: ReadWriteMany
      size: 5Gi
    jobservice:    # 异步任务组件（持久卷）配置部分
      existingClaim: ""
      storageClass: "harbor-storageclass"    #修改，同上
      subPath: ""
      accessMode: ReadWriteOnce
      size: 1Gi
    database:        # PostgreSQl数据库组件（持久卷）配置部分
      existingClaim: ""
      storageClass: "harbor-storageclass"
      subPath: ""
      accessMode: ReadWriteMany
      size: 1Gi
    redis:    # Redis缓存组件（持久卷）配置部分
      existingClaim: ""
      storageClass: "harbor-storageclass"
      subPath: ""
      accessMode: ReadWriteMany
      size: 1Gi
    trivy:         # Trity漏洞扫描插件（持久卷）配置部分
      existingClaim: ""
      storageClass: "harbor-storageclass"
      subPath: ""
      accessMode: ReadWriteMany
      size: 5Gi
...
metrics:
  enabled: true  # 是否启用监控组件
  core:
    path: /metrics
    port: 8001
  registry:
    path: /metrics
    port: 8001
  jobservice:
    path: /metrics
    port: 8001
  exporter:
    path: /metrics
    port: 8001
###以下的trace为2.4版本的功能，不需要修改。
```

